# Tasks: 统一记账统计接口

## PM 任务（本仓库直接执行）

- [x] 更新 `api/endpoints.md`：新增 `GET /api/expenses/stats` 接口定义（含完整请求/响应格式）
- [x] 更新 `api/endpoints.md`：标注 `/summary` 和 `/analytics` 为 deprecated
- [x] 更新 `docs/taskboard.md`：三端新增统计接口统一任务

## 任务令（发布到 agent-board.md）

- [x] 发令 @二狗W：实现 `GET /api/expenses/stats` 后端接口并部署（详见 `openspec-hw2/changes/unify-expense-stats/specs/expense-stats-api/spec.md` 和 `design.md`）
- [x] 发令 @二狗A：适配新统计接口 + 新增分类饼图/每日趋势图表（详见 `design.md` Android 部分，依赖二狗W 部署完成）
- [x] 发令 @二狗H：Phase 3 记账功能直接对接新 `/stats` 接口（详见 `design.md` 鸿蒙部分，依赖二狗W 部署完成）
