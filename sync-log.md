# 跨端同步记录

格式：`日期 | 变更内容 | Web | Android | 鸿蒙`

---

## 2026-03-11：鸿蒙 Phase 1 基础框架搭建

二狗H 完成首期基础框架，代码已推送 [GitHub](https://github.com/Boris-hbx/ergouHarmony)。

| 变更 | Web | Android | 鸿蒙 |
|------|-----|---------|------|
| Logger、Constants、PreferencesUtil、PromptSanitizer | - | - | ✅ 已完成 |
| 数据模型（Chat/LLM/Expense/Tool Models） | - | - | ✅ 已完成 |
| DatabaseHelper（sessions/messages/memories 三表） | - | - | ✅ 已完成 |
| SessionDao、MessageDao、MemoryDao（完整 CRUD） | - | - | ✅ 已完成 |
| EntryAbility 启动初始化 | - | - | ✅ 已完成 |

---

## 2026-03-11：灵魂状态上云（ADR-002）

灵魂状态从 Android 本地存储迁移至后端，实现跨端人设一致。

| 变更 | Web | Android | 鸿蒙 |
|------|-----|---------|------|
| 后端新增 `soul_states` / `soul_evolution_log` 表 | ✅ 已实现 | - | - |
| 后端实现 `GET/PUT /api/soul-state` | ✅ 已实现 | - | - |
| 后端集成灵魂演化（对话后自动调整） | ✅ 已实现 | - | - |
| Prompt 构建逻辑迁移至后端 | ✅ 已实现 | - | - |
| 客户端改为读取后端灵魂状态 | ✅ 已适配 | ✅ 已生效 | ⬜ 直接对接 |
| Android 老用户数据一次性上报 | - | ✅ 已生效 | - |

---

## 2026-03-11：ergouPM 初始化

从 Android 版提取共享定义，建立协调中心。

| 变更 | Web | Android | 鸿蒙 |
|------|-----|---------|------|
| API 接口规范整理 (`api/endpoints.md`) | ✅ 来源 | ✅ 已对接 | ⬜ 首期对接记账 |
| 系统 Prompt 提取 (`llm/system-prompt.md`) | N/A | ✅ 来源 | ⬜ 待实现 |
| 工具定义提取 (`llm/tools.json`) | N/A | ✅ 来源 | ⬜ 首期实现记账+通用 |
| Web 工程迁移至 `ergouWeb/` | ✅ 已迁移 | - | - |
