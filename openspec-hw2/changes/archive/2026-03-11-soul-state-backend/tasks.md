## 1. 数据库与模型

- [x] 1.1 在 `db.rs::create_tables()` 中添加 `soul_states` 表定义（7 个参数字段 + relationship_stage + total_interactions + updated_at）
- [x] 1.2 在 `db.rs::create_tables()` 中添加 `soul_evolution_log` 表定义（id, user_id, parameter, old_value, new_value, trigger_type, created_at）
- [x] 1.3 在 `db.rs::run_migrations()` 中添加迁移逻辑（检测表是否存在，不存在则创建）
- [x] 1.4 创建 `models/soul_state.rs`：定义 `SoulState` 结构体、`UpdateSoulStateRequest`、`SoulStateResponse`，以及 `row_to_soul_state` 映射函数
- [x] 1.5 在 `models/mod.rs` 中注册 soul_state 模块

## 2. 灵魂状态 CRUD API

- [x] 2.1 创建 `routes/soul_state.rs`：实现 `get_soul_state` handler（含懒创建逻辑）
- [x] 2.2 实现 `update_soul_state` handler（部分更新 + 范围校验 + 演化日志记录 trigger_type="manual"）
- [x] 2.3 提取辅助函数 `ensure_soul_state(db, user_id) -> SoulState`：查询或懒创建默认灵魂状态
- [x] 2.4 在 `main.rs::build_app()` 中注册路由 `GET /api/soul-state` 和 `PUT /api/soul-state`
- [x] 2.5 在 `routes/mod.rs` 中注册 soul_state 模块

## 3. 注册时初始化

- [x] 3.1 在 `auth.rs` 注册逻辑末尾，INSERT 默认灵魂状态记录（失败仅记日志，不阻塞注册）

## 4. 灵魂演化服务

- [x] 4.1 创建 `services/soul_evolution.rs`：实现 `evolve_after_chat(state, user_id, messages)` 主函数
- [x] 4.2 实现 `total_interactions` 递增逻辑
- [x] 4.3 实现每日演化次数检查（查询 soul_evolution_log 当日 auto 记录数）
- [x] 4.4 实现 LLM 演化分析调用：构建精简 prompt，调用 Claude API，解析 JSON 响应
- [x] 4.5 实现参数调整应用：±0.05 钳制 + 字段范围钳制 + 写入 soul_evolution_log
- [x] 4.6 实现关系阶段自动升级：按阈值（10/50/150/500）检查并升级 relationship_stage
- [x] 4.7 在 `services/mod.rs` 中注册 soul_evolution 模块

## 5. 对话后触发演化

- [x] 5.1 在 `routes/chat.rs::chat_handler()` 返回响应前，用 `tokio::spawn` 异步调用 `soul_evolution::evolve_after_chat()`

## 6. Prompt 构建集成

- [x] 6.1 在 `services/context.rs` 中新增 `build_soul_personality(state: &SoulState) -> String`：根据 warmth/trust/verbosity/proactivity 生成动态性格段落
- [x] 6.2 新增 `build_soul_speaking_style(state: &SoulState) -> String`：根据 classical_ratio 生成文白比例指令
- [x] 6.3 新增 `build_soul_behavior(state: &SoulState) -> String`：根据 relationship_stage 生成行为段落
- [x] 6.4 新增 `build_soul_tone_examples(state: &SoulState) -> String`：根据关系阶段生成 2-3 个语气示例
- [x] 6.5 修改 `build_system_prompt_with_page()`：在基础人设之后插入灵魂状态动态段落（按 spec 定义的 8 段拼装顺序）

## 7. 验证与收尾

- [x] 7.1 编译通过，无 warning
- [ ] 7.2 手动测试 GET /api/soul-state（含懒创建场景）
- [ ] 7.3 手动测试 PUT /api/soul-state（部分更新 + 范围校验 + 日志）
- [ ] 7.4 手动测试对话后演化（发一轮对话后检查 soul_states 和 soul_evolution_log）
- [x] 7.5 更新 `api/endpoints.md` 确认灵魂状态端点已标注为已实现
