# 跨端同步记录

格式：`日期 | 变更内容 | Web | Android | 鸿蒙`

---

## 2026-03-11：ergouPM 初始化

从 Android 版提取共享定义，建立协调中心。

| 变更 | Web | Android | 鸿蒙 |
|------|-----|---------|------|
| API 接口规范整理 (`api/endpoints.md`) | ✅ 来源 | ✅ 已对接 | ⬜ 首期对接记账 |
| 系统 Prompt 提取 (`llm/system-prompt.md`) | N/A | ✅ 来源 | ⬜ 待实现 |
| 工具定义提取 (`llm/tools.json`) | N/A | ✅ 来源 | ⬜ 首期实现记账+通用 |
| Web 工程迁移至 `ergouWeb/` | ✅ 已迁移 | - | - |
