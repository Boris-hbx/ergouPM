## Why

二狗的灵魂状态（性格参数 + 关系阶段）当前仅存于 Android 本地，Web 端和鸿蒙端无法共享，导致同一用户在不同设备上的二狗"人格"割裂。ADR-002 已决定将灵魂状态迁移至后端统一管理，本变更实施其 Phase 1：Web 后端部分。

## What Changes

- 新增 `soul_states` 表，存储每个用户的灵魂状态（7 个参数 + 关系阶段 + 交互计数）
- 新增 `soul_evolution_log` 表，记录每次参数变化的审计日志
- 新增 `GET /api/soul-state` 端点，返回当前用户的灵魂状态
- 新增 `PUT /api/soul-state` 端点，支持部分字段更新（管理员重置或未来用户自定义）
- 用户注册时自动初始化默认灵魂状态
- 将 Prompt 构建逻辑从 Android 客户端迁移至后端（Rust 重写 `buildPersonality()` / `buildSpeakingStyle()` / `buildBehavior()` / `buildToneExamples()`）
- `/api/chat` 对话完成后，后端异步触发灵魂演化（±0.05 上限，每日最多 10 次）
- 关系阶段按 `total_interactions` 阈值自动升级（10→相识，50→熟悉，150→亲近，500→至交）

## Capabilities

### New Capabilities

- `soul-state-storage`: 灵魂状态的持久化存储、CRUD API、用户注册时自动初始化
- `soul-evolution`: 对话后自动演化灵魂参数 + 关系阶段升级 + 演化日志记录
- `soul-prompt-builder`: 根据灵魂状态动态构建系统 Prompt（从 Android 端迁移至后端 Rust 实现）

### Modified Capabilities

（无已有 spec 需修改）

## Impact

- **ergouWeb 后端**：`db.rs` 新增 2 张表；新增 `soul_state` 模块（handler + service）；`chat` 模块需集成演化触发；`prompt` 模块需新建
- **API 层**：新增 2 个端点 `GET/PUT /api/soul-state`，已在 `api/endpoints.md` 预定义
- **LLM 集成**：后端需读取 `llm/system-prompt.md` 的静态部分 + `soul_states` 的动态部分，拼装完整 system prompt
- **客户端**：本变更不涉及客户端改动（Phase 2/3 另行处理）
- **数据库迁移**：需要 migration 脚本，现有用户无灵魂状态记录，首次 GET 时应自动创建默认值
