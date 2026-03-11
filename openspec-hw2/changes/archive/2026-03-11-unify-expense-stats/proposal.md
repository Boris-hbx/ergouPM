## Why

记账统计功能当前存在两个职责重叠的接口（`/summary` 和 `/analytics`），返回结构不同但语义相似。Android 端只调了 `/summary`，缺失分类饼图和每日趋势；鸿蒙端即将进入 Phase 3 记账开发，如果不统一，会再多一套不一致的实现。现在是统一三端统计能力的最佳时机。

## What Changes

- 新增统一统计接口 `GET /api/expenses/stats`，一次返回标签明细、分类汇总、每日趋势、环比数据
- 后端分类映射（`tag_to_category()`）作为标准能力，输出给所有客户端
- **BREAKING**：旧接口 `/summary` 和 `/analytics` 标记为 deprecated（保留向后兼容，但不再作为首选）
- 更新 `api/endpoints.md` 规范，新增统一接口定义
- 各端客户端适配新接口

## Capabilities

### New Capabilities
- `expense-stats-api`: 统一的记账统计接口定义，合并原 summary + analytics 的能力，新增环比对比

### Modified Capabilities
（无已有 spec 需修改）

## Impact

- **Web 后端（ergouWeb）**：新增 `/api/expenses/stats` 路由，复用现有查询逻辑，合并 summary + analytics 返回
- **Android（ergou）**：ExpenseViewModel 改调新接口，新增分类饼图/每日趋势 UI
- **鸿蒙（ergouh）**：Phase 3 记账开发直接对接新接口，不走旧 API
- **PM（ergouPM）**：更新 `api/endpoints.md`，通过任务令协调三端实施
- **LLM 工具**：`expense_summary` 工具可增强，调新接口获取更丰富的统计数据
