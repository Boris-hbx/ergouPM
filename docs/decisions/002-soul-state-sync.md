# ADR-002: 灵魂状态从本地存储迁移至后端同步

## 状态
已采纳 (2026-03-11)

## 背景
二狗的"灵魂状态"（Soul State）定义了 AI 助手的人设参数——温度、信任度、文白比例、主动性等。这些参数影响系统 Prompt 的生成，决定二狗的性格表现。

当前实现（Android 版）：
- 存储：Room 数据库 `soul_state` 表，单行记录（`SoulStateEntity`）
- 演化：`SoulEvolver` 每次对话后通过 LLM 分析调整参数（±0.05 上限，每日最多 10 次）
- 关系升级：基于 `totalInteractions` 自动升阶（10→相识，50→熟悉，150→亲近，500→至交）
- Prompt 构建：`ErgouPrompt.buildSystemPrompt()` 在客户端根据灵魂状态动态生成

**问题**：灵魂状态仅存于 Android 本地，Web 端和鸿蒙端无法共享。同一用户在不同设备上的二狗"人格"割裂。

## 决策

将灵魂状态迁移至后端（ergouWeb），由后端统一存储、演化和构建 Prompt。

### 字段定义

| 字段 | 类型 | 默认值 | 范围 | 说明 |
|------|------|--------|------|------|
| classical_ratio | f32 | 0.9 | 0.6–1.0 | 文白比例 |
| warmth_level | f32 | 0.3 | 0.0–1.0 | 温度（情感关怀程度） |
| verbosity_level | f32 | 0.3 | 0.0–1.0 | 详细程度 |
| proactivity_level | f32 | 0.2 | 0.0–0.8 | 主动性 |
| trust_level | f32 | 0.1 | 0.0–1.0 | 信任度 |
| relationship_stage | String | "stranger" | 五阶段 | stranger→acquaintance→familiar→close→intimate |
| total_interactions | i32 | 0 | ≥0 | 累计对话次数 |

### 架构设计

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Android │  │   Web   │  │ 鸿蒙    │
│ 客户端  │  │ 客户端  │  │ 客户端  │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     │  GET /api/soul-state    │      ← 读取（展示、本地缓存）
     │  POST /api/chat         │      ← 对话（触发演化）
     │            │            │
     └────────────┼────────────┘
                  │
          ┌───────▼───────┐
          │  ergouWeb 后端 │
          │               │
          │  1. 存储灵魂状态│
          │  2. 构建 Prompt │
          │  3. 对话后演化  │
          └───────────────┘
```

### 核心设计决策

#### Q1: 每次对话后谁来更新灵魂状态？

**决策：后端算。**

理由：
- 后端已拥有 `/api/chat` 端点，对话流经后端，天然可在响应后触发演化
- 避免客户端各自实现演化逻辑（当前 Android 的 `SoulEvolver` 需迁移至后端）
- 防止客户端伪造演化结果
- 演化逻辑变更只需更新后端，无需发版

实现：
- `POST /api/chat` 处理对话后，后端异步调用演化逻辑
- 复用 Android 版 `SoulEvolver` 的 Prompt 和调整规则
- 每次调整上限 ±0.05，每日最多 10 次自动调整
- 关系阶段按 totalInteractions 阈值自动升级

#### Q2: 离线时怎么办？

**决策：本地缓存 + 上线拉取，后端为唯一真相源。**

方案：
1. 客户端启动时 `GET /api/soul-state`，缓存到本地
2. 离线时使用本地缓存的灵魂状态（只读）
3. 离线期间无法对话（对话依赖后端），故灵魂状态不会变化
4. 重新上线后重新拉取最新状态

不需要冲突合并——因为只有后端能写入灵魂状态，客户端只读。

#### Q3: 新用户默认状态？

**决策：后端在用户注册时自动创建默认灵魂状态。**

默认值沿用 Android 版：
```
classical_ratio: 0.9   # 高文言比例
warmth_level: 0.3      # 保持距离
verbosity_level: 0.3   # 言简意赅
proactivity_level: 0.2 # 被动响应
trust_level: 0.1       # 初始低信任
relationship_stage: "stranger"
total_interactions: 0
```

### API 设计

#### GET /api/soul-state

获取当前用户的灵魂状态。

- 认证：需要有效 Session
- 响应 200：
```json
{
  "success": true,
  "soul_state": {
    "classical_ratio": 0.9,
    "warmth_level": 0.3,
    "verbosity_level": 0.3,
    "proactivity_level": 0.2,
    "trust_level": 0.1,
    "relationship_stage": "stranger",
    "total_interactions": 0,
    "updated_at": "2026-03-11T10:00:00Z"
  }
}
```

#### PUT /api/soul-state

手动调整灵魂状态（用于管理员重置或未来的用户自定义功能）。

- 认证：需要有效 Session（仅限本人）
- 请求体（部分更新，只传需要改的字段）：
```json
{
  "classical_ratio": 0.75,
  "warmth_level": 0.5
}
```
- 响应 200：返回更新后的完整状态（格式同 GET）
- 约束：值必须在合法范围内，否则返回 400

### 数据库变更

后端 `db.rs` 新增表：

```sql
CREATE TABLE IF NOT EXISTS soul_states (
    user_id TEXT PRIMARY KEY REFERENCES users(id),
    classical_ratio REAL NOT NULL DEFAULT 0.9,
    warmth_level REAL NOT NULL DEFAULT 0.3,
    verbosity_level REAL NOT NULL DEFAULT 0.3,
    proactivity_level REAL NOT NULL DEFAULT 0.2,
    trust_level REAL NOT NULL DEFAULT 0.1,
    relationship_stage TEXT NOT NULL DEFAULT 'stranger',
    total_interactions INTEGER NOT NULL DEFAULT 0,
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS soul_evolution_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id TEXT NOT NULL REFERENCES users(id),
    parameter TEXT NOT NULL,
    old_value REAL NOT NULL,
    new_value REAL NOT NULL,
    trigger_type TEXT NOT NULL DEFAULT 'auto',  -- auto / manual
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### Prompt 构建迁移

当前 Android 的 `ErgouPrompt.buildSystemPrompt()` 逻辑需迁移至后端：
- 后端在处理 `/api/chat` 时，从 `soul_states` 读取状态
- 用 Rust 重写 `buildPersonality()` / `buildSpeakingStyle()` / `buildBehavior()` / `buildToneExamples()` 等方法
- 参考 `llm/system-prompt.md` 中的共享 Prompt 模板

### 迁移计划

#### Phase 1：后端实现（ergouWeb）
1. 新增 `soul_states` 和 `soul_evolution_log` 表
2. 用户注册/首次访问时初始化默认灵魂状态
3. 实现 `GET /api/soul-state` 和 `PUT /api/soul-state`
4. 将 Prompt 构建逻辑迁移至后端（Rust 重写）
5. 在 `/api/chat` 流程中集成灵魂演化

#### Phase 2：Android 迁移（ergou）
1. 启动时从后端拉取灵魂状态，缓存本地
2. 移除本地 `SoulEvolver` 演化逻辑（后端已接管）
3. `SoulScreen` UI 改为展示后端数据
4. 保留 `SoulStateEntity` 作为本地缓存模型

#### Phase 3：鸿蒙端（ergouh）
1. 直接对接后端 API，无需实现本地演化
2. 按需缓存灵魂状态用于离线展示

#### 数据迁移（Android 老用户）
- Android 端检测到后端灵魂状态为默认值且本地有非默认数据时
- 一次性通过 `PUT /api/soul-state` 上报本地状态
- 上报成功后标记迁移完成，后续不再上报

## 理由
- 后端统一管理灵魂状态，彻底解决跨端一致性问题
- 演化逻辑集中在后端，迭代无需客户端发版
- 离线场景简单（只读缓存），无冲突合并复杂度
- 鸿蒙端可直接使用，无需重复实现演化逻辑

## 后果
- 后端需新增 2 个端点 + 2 张表 + Prompt 构建逻辑
- Android 需重构：移除本地演化，改为读取后端状态
- 对话功能依赖网络（已是现状，因 `/api/chat` 本就在后端）
- 灵魂演化日志集中存储，便于后续分析和调优
