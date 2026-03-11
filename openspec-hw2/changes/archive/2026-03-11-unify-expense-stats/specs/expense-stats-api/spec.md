# Spec: expense-stats-api

统一记账统计接口，合并原 `/summary` + `/analytics` 的能力。

## 接口定义

### `GET /api/expenses/stats`

**认证**：需要登录（Session Cookie）

**查询参数**：

| 参数 | 类型 | 必填 | 默认 | 说明 |
|------|------|------|------|------|
| period | string | 否 | "month" | 统计周期：`day` / `week` / `month` |
| date | string | 否 | 今天 | 基准日期，格式 `YYYY-MM-DD` |

**日期范围计算**：

| period | from | to |
|--------|------|-----|
| day | date 当天 | date 当天 |
| week | date 所在周一 | date 所在周日 |
| month | date 所在月 1 日 | date 所在月末 |

### 响应结构

```json
{
  "success": true,
  "stats": {
    "period": "month",
    "from": "2026-03-01",
    "to": "2026-03-31",
    "total_amount": 4500.00,
    "entry_count": 42,
    "tag_totals": [
      { "tag": "超市", "amount": 800.00, "count": 12 },
      { "tag": "餐饮", "amount": 650.00, "count": 8 }
    ],
    "category_totals": [
      {
        "category": "食品杂货",
        "amount": 1200.00,
        "count": 15,
        "percentage": 26.7
      }
    ],
    "daily": [
      { "date": "2026-03-01", "amount": 150.00 },
      { "date": "2026-03-02", "amount": 0.00 }
    ],
    "comparison": {
      "prev_total": 3800.00,
      "change_percent": 18.4
    }
  }
}
```

## 字段规格

### tag_totals
- 按金额降序排列
- 每条：原始标签名、该标签总金额、该标签条目数
- **实现**：UC1-S4 步骤，直接按 tags JSON 数组聚合

### category_totals
- 按金额降序排列
- 9 个固定分类：食品杂货、餐饮、交通、购物、住房、娱乐、医疗、教育、其他
- percentage = (该分类金额 / total_amount) × 100，保留 1 位小数
- **分类映射规则**：每条记账取第一个非"其他"标签，通过 `tag_to_category()` 映射
- **实现**：复用现有 analytics 逻辑

### daily
- 包含日期范围内的每一天（含无数据的天，amount 填 0）
- 按日期升序排列
- **实现**：复用现有 analytics 逻辑

### comparison
- prev_total：上一个同等周期的总金额（本月→上月，本周→上周，今天→昨天）
- change_percent = ((total_amount - prev_total) / prev_total) × 100
- 若 prev_total = 0 且 total_amount > 0，change_percent = 100
- 若 prev_total = 0 且 total_amount = 0，change_percent = 0

## 错误处理

| 情况 | 响应 |
|------|------|
| 未登录 | 401 `{ "error": "unauthorized" }` |
| period 参数非法 | 400 `{ "error": "invalid period, must be day/week/month" }` |
| date 格式非法 | 400 `{ "error": "invalid date format, expected YYYY-MM-DD" }` |

## 废弃策略

- `GET /api/expenses/summary` — 标记 deprecated，行为不变，响应中添加 `"deprecated": true`
- `GET /api/expenses/analytics` — 标记 deprecated，行为不变，响应中添加 `"deprecated": true`
- 旧接口在所有端迁移完成前保持可用

## 与用例的映射

| 规格 | 用例 |
|------|------|
| 接口定义、查询参数 | UC1-S1 |
| 日期范围计算 | UC1-S2 |
| tag_totals | UC1-S4 |
| category_totals | UC1-S4 |
| daily | UC1-S4 |
| comparison | UC1-S4 |
| 响应结构 | UC1-S5 |
| 无数据处理 | UC1-E3a |
| 标签无法映射 | UC1-E4a |
| 废弃策略 | UC2 |
