## Context

ergouWeb 后端采用 Rust Axum + SQLite（rusqlite）架构，当前系统 Prompt 由 `services/context.rs::build_system_prompt_with_page()` 构建，包含静态人设 + 任务/记忆/人际上下文，但不含灵魂状态动态部分。

Android 端的灵魂演化（`SoulEvolver`）和 Prompt 动态构建（`ErgouPrompt.buildSystemPrompt()`）逻辑需迁移至后端，使三端共享统一的二狗性格。

现有代码模式：
- 共享状态通过 `AppState { db: Arc<Mutex<Connection>>, ... }`
- Handler 签名：`async fn handler(State(state): State<AppState>, UserId(user_id): UserId, ...) -> Result<Json<T>, StatusCode>`
- 数据库表定义在 `db.rs::create_tables()`，迁移在 `run_migrations()`
- 路由注册在 `main.rs::build_app()` 用 `.nest()` 分组
- Model 定义在 `models/` 目录，routes 在 `routes/`，services 在 `services/`

## Goals / Non-Goals

**Goals:**
- 后端存储和管理灵魂状态，提供 GET/PUT API
- 对话完成后异步演化灵魂参数
- 后端构建包含灵魂状态的完整系统 Prompt
- 完全融入现有代码模式，不引入新的架构概念

**Non-Goals:**
- 客户端改造（Android/鸿蒙适配为后续 Phase）
- 管理后台的灵魂状态管理界面
- 灵魂状态的实时推送（WebSocket）
- 演化算法的优化或重新设计（先 1:1 迁移 Android 逻辑）

## Decisions

### D1: 模块组织 — 新增 soul_state 模块

**决策：** 按现有模式新增 `models/soul_state.rs` + `routes/soul_state.rs`，演化逻辑放在 `services/soul_evolution.rs`，Prompt 构建逻辑扩展 `services/context.rs`。

**理由：** 与现有 todo/reminder/expense 等模块完全一致的分层。演化是服务层逻辑，Prompt 构建已在 context.rs 中，扩展比新建更自然。

**替代方案：**
- 创建独立 `services/soul_prompt.rs` — 但 Prompt 构建逻辑已集中在 context.rs，拆分会增加耦合点。改为在 context.rs 中新增函数即可。

### D2: 数据库变更 — 在 create_tables() 和 run_migrations() 中添加

**决策：** `soul_states` 和 `soul_evolution_log` 两张表在 `db.rs::create_tables()` 中定义（CREATE IF NOT EXISTS），新用户通过 `create_tables` 直接建表，老数据库通过 `run_migrations()` 补建。

**理由：** 这是 ergouWeb 的标准做法（参见 todos、conversations 等表），无独立迁移文件机制。

### D3: 演化触发 — chat handler 完成后 tokio::spawn 异步执行

**决策：** 在 `routes/chat.rs::chat_handler()` 返回响应之后，用 `tokio::spawn` 启动异步任务执行演化逻辑。

```rust
// chat_handler 末尾，response 已构建好
let state_clone = state.clone();
let uid = user_id.clone();
tokio::spawn(async move {
    if let Err(e) = soul_evolution::evolve_after_chat(&state_clone, &uid, &messages).await {
        tracing::error!("灵魂演化失败: {e}");
    }
});
// 返回 response，不等待演化
```

**理由：**
- 不阻塞用户对话响应
- tokio::spawn 是项目中已有的异步模式（用于定时任务、备份等）
- 演化失败仅记日志，不影响对话功能

**替代方案：**
- 在 Axum middleware 中拦截 — 过于隐式，演化依赖对话内容，handler 内触发更直观
- 用消息队列 — 过度工程，SQLite 单进程场景不需要

### D4: 每日演化计数 — 查询 soul_evolution_log 当日记录数

**决策：** 不新增计数字段，直接用 `SELECT COUNT(*) FROM soul_evolution_log WHERE user_id = ? AND trigger_type = 'auto' AND created_at >= date('now')` 判断当日演化次数。

**理由：** 避免维护冗余计数字段。`soul_evolution_log` 表数据量可控（每用户每日最多 10 次 × 数个参数），查询无性能问题。

### D5: LLM 演化分析 — 复用现有 LLM 客户端

**决策：** 演化时复用 `services/llm.rs` 的 Claude API 客户端，发送一个精简 prompt 让 LLM 分析对话情感倾向，输出 JSON 格式的参数调整建议。

演化 Prompt 模板（参考 Android SoulEvolver）：
```
分析以下对话，判断用户与二狗的互动是否应调整以下性格参数。
仅返回需要调整的参数及方向（+/-），幅度由系统控制。
参数：classical_ratio, warmth_level, verbosity_level, proactivity_level, trust_level
对话内容：{recent_messages}
返回 JSON：{"adjustments": {"warmth_level": "+", "trust_level": "+"}}
若无需调整返回：{"adjustments": {}}
```

**理由：** 复用已有基础设施，不引入新的 LLM 调用路径。

### D6: Prompt 构建扩展 — 在 context.rs 中新增灵魂状态段落

**决策：** 在 `build_system_prompt_with_page()` 中，基础人设段落之后插入灵魂状态动态段落。新增辅助函数：

- `build_soul_personality(state: &SoulState) -> String` — 动态性格描述
- `build_soul_speaking_style(state: &SoulState) -> String` — 文白比例指令
- `build_soul_behavior(state: &SoulState) -> String` — 关系阶段行为
- `build_soul_tone_examples(state: &SoulState) -> String` — 语气示例

**理由：** context.rs 已负责 Prompt 拼装，在此扩展符合单一职责。函数拆分保持可读性。

### D7: 懒创建策略 — GET 时自动创建 + 注册时主动创建

**决策：** 双重保障。`POST /api/auth/register` 成功后主动 INSERT 默认状态；`GET /api/soul-state` 和 Prompt 构建时发现记录不存在则 INSERT OR IGNORE 创建默认值。

**理由：** 注册时创建覆盖正常流程；懒创建兜底老用户（数据库已有 users 但无 soul_states 记录）和边界情况。

## Risks / Trade-offs

**[演化 LLM 调用成本]** → 每次对话额外一次 LLM 调用（虽精简）。缓解：每日限 10 次；使用最小 prompt；可考虑后续改为规则引擎替代 LLM 分析。

**[SQLite 并发写入]** → 演化异步写入 + 正常请求并发可能竞争 Mutex。缓解：演化操作轻量（单行 UPDATE + 数行 INSERT），锁持有时间极短；与现有并发模式一致。

**[演化分析 LLM 输出不稳定]** → LLM 返回格式可能不符合预期 JSON。缓解：解析失败时跳过本次演化，仅记日志；±0.05 上限本身就是安全护栏。

**[Prompt 长度增长]** → 灵魂状态动态段落增加 token 消耗。缓解：动态段落控制在 200-400 字以内，相对于整体 context（记忆、人际、任务）影响很小。
