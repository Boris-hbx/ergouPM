# Design: 统一记账统计接口

## 架构决策

### 合并而非替换
新接口 `/api/expenses/stats` 在后端复用现有的查询和聚合逻辑（来自 `get_summary` 和 `get_analytics`），将两者的返回合并为一个响应。旧接口保留但标记 deprecated。

### 分类映射由后端统一维护
`tag_to_category()` 只在 Web 后端维护，客户端不需要硬编码分类逻辑。category_totals 由后端直接返回，客户端只负责渲染。

### 环比由后端计算
comparison 字段由后端查询上一周期数据并计算，避免客户端多发一次请求。

---

## 各端改动方案

### Web 后端（ergouWeb）— 二狗W

**改动范围**：`server/src/routes/expenses.rs` + `server/src/models/expense.rs`

1. **新增 handler**：`get_stats()`
   - 接收 `ExpenseStatsQuery { period, date }` 参数
   - 复用现有 `get_summary` 的 tag 聚合逻辑 → `tag_totals`
   - 复用现有 `get_analytics` 的分类映射 + 每日聚合逻辑 → `category_totals` + `daily`
   - 新增：查询上一周期 total_amount，计算 change_percent → `comparison`
   - 返回统一 `StatsResponse`

2. **新增 model**：`ExpenseStatsResponse`
   ```
   StatsResponse { period, from, to, total_amount, entry_count,
                   tag_totals[], category_totals[], daily[], comparison }
   ```

3. **注册路由**：`.route("/stats", get(get_stats))`（在 main.rs 和 lib.rs）

4. **旧接口标记 deprecated**：在 `get_summary` 和 `get_analytics` 的响应中加 `"deprecated": true`

**不改**：现有查询逻辑、tag_to_category() 映射表、数据库 schema

### Android（ergou）— 二狗A

**改动范围**：API 层 + ViewModel + UI

1. **NextApiService**：新增 `getExpenseStats(period, date)` 方法
2. **NextModels**：新增 `NextExpenseStats` 数据类（含 tagTotals, categoryTotals, daily, comparison）
3. **ExpenseViewModel**：
   - 改调 `getExpenseStats()` 替代 `getExpenseSummary()`
   - 新增 `categoryTotals` 和 `daily` 状态字段
   - 新增 `comparison` 状态字段（环比）
4. **ExpenseScreen**：
   - 新增分类饼图（Compose Canvas）
   - 新增每日趋势柱状图（Compose Canvas）
   - 新增环比指标显示
5. **LLM 工具**：`ExpenseSummaryTool` 改调新接口，丰富返回内容

### 鸿蒙（ergouh）— 二狗H

**改动范围**：Phase 3 记账功能直接对接新接口

1. **NextApiClient**：实现 `getExpenseStats(period, date)` 方法
2. **ExpenseModels**：定义 `ExpenseStats` 接口（对齐 spec 响应结构）
3. **ExpenseViewModel**：调用新接口，管理统计状态
4. **ExpensePage**：
   - 月度总额 + 环比
   - 标签筛选
   - 分类饼图 + 每日趋势（ArkUI Canvas）
5. **记账工具**：`ExpenseSummaryTool` 调新接口

### PM（ergouPM）— 二狗PM

1. 更新 `api/endpoints.md`：新增 `/stats` 接口定义，标注旧接口 deprecated
2. 通过任务令协调三端实施顺序

---

## 实施顺序

```
Step 1: PM 更新 API 规范（本仓库）
Step 2: 二狗W 实现后端接口 → 部署
Step 3: 二狗A 适配新接口 + 新增图表 UI（与 Step 4 可并行）
Step 4: 二狗H Phase 3 直接用新接口（与 Step 3 可并行）
```

二狗A 和 二狗H 可以并行开发，只要 Step 2 后端部署完成即可。
