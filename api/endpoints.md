# 二狗 API 接口规范

> 后端：ergouWeb (Rust Axum)
> Base URL: 由部署环境决定
> 认证：Session Cookie (登录后自动携带)

---

## 认证 `/api/auth`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /register | 注册 |
| POST | /login | 登录 |
| POST | /logout | 登出 |
| POST | /guest | 游客登录 |
| GET | /me | 获取当前用户信息 |
| POST | /change-password | 修改密码 |
| PUT | /avatar | 更新头像 |

## 待办 `/api/todos`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | / | 列表 |
| POST | / | 创建 |
| GET | /counts | 统计数量 |
| PUT | /batch | 批量更新 |
| GET | /{id} | 详情 |
| PUT | /{id} | 更新 |
| DELETE | /{id} | 删除 |
| POST | /{id}/restore | 恢复 |
| DELETE | /{id}/permanent | 永久删除 |

## 例行 `/api/routines`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | / | 列表 |
| POST | / | 创建 |
| DELETE | /{id} | 删除 |
| POST | /{id}/toggle | 切换完成状态 |

## 例行回顾 `/api/reviews`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | / | 列表 |
| POST | / | 创建 |
| PUT | /{id} | 更新 |
| DELETE | /{id} | 删除 |
| POST | /{id}/complete | 标记完成 |
| POST | /{id}/uncomplete | 取消完成 |

## 记账 `/api/expenses`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | / | 列表（支持月份筛选） |
| POST | / | 创建记账条目 |
| GET | /stats | ✅ **统一统计**（合并 summary + analytics，推荐使用） |
| GET | /summary | ~~月度汇总~~ ⚠️ deprecated，请用 /stats |
| GET | /analytics | ~~分析数据~~ ⚠️ deprecated，请用 /stats |
| GET | /tags | 标签列表 |
| GET | /rates | 汇率 |
| POST | /parse-preview | 解析预览（小票识别） |
| GET | /{id} | 详情 |
| PUT | /{id} | 更新 |
| DELETE | /{id} | 删除 |
| POST | /{id}/photos | 上传照片 |
| POST | /{id}/parse | 解析小票 |
| DELETE | /photos/{photo_id} | 删除照片 |

### `GET /api/expenses/stats` 统一统计接口

合并原 `/summary`（标签明细）和 `/analytics`（分类汇总 + 每日趋势）的能力，新增环比对比。

**查询参数**：

| 参数 | 类型 | 必填 | 默认 | 说明 |
|------|------|------|------|------|
| period | string | 否 | "month" | 统计周期：`day` / `week` / `month` |
| date | string | 否 | 今天 | 基准日期，格式 `YYYY-MM-DD` |

**响应**：

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
      { "tag": "超市", "amount": 800.00, "count": 12 }
    ],
    "category_totals": [
      { "category": "食品杂货", "amount": 1200.00, "count": 15, "percentage": 26.7 }
    ],
    "daily": [
      { "date": "2026-03-01", "amount": 150.00 }
    ],
    "comparison": {
      "prev_total": 3800.00,
      "change_percent": 18.4
    }
  }
}
```

**分类映射**（后端 `tag_to_category()` 维护）：

| 分类 | 包含标签 |
|------|---------|
| 食品杂货 | 超市、杂货、肉类、蔬菜、水果、海鲜、零食、饮料、奶制品、调料、面包、生鲜、食材 |
| 餐饮 | 餐饮、外卖、餐厅、咖啡、奶茶、早餐、午餐、晚餐、火锅、快餐、甜点、酒吧 |
| 交通 | 交通、加油、停车、公交、地铁、打车、出租车、高铁、机票、油费、租车 |
| 购物 | 购物、衣服、鞋子、电子、数码、家居、家电、日用品、化妆品 |
| 住房 | 住房、房租、水电、网费、物业、维修、家具、电话费 |
| 娱乐 | 娱乐、电影、游戏、旅游、景点、KTV、运动、健身 |
| 医疗 | 医疗、药品、看病、体检、牙科、保健 |
| 教育 | 教育、书籍、课程、培训、文具 |
| 其他 | 无法匹配以上分类的标签 |

> 详细规格见 `openspec-hw2/changes/unify-expense-stats/specs/expense-stats-api/spec.md`

## 差旅 `/api/trips`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | / | 列表 |
| POST | / | 创建 |
| GET | /{id} | 详情 |
| PUT | /{id} | 更新 |
| DELETE | /{id} | 删除 |
| POST | /{id}/items | 添加费用项 |
| PUT | /items/{item_id} | 更新费用项 |
| DELETE | /items/{item_id} | 删除费用项 |
| POST | /items/{item_id}/photos | 上传费用项照片 |
| DELETE | /photos/{photo_id} | 删除照片 |
| POST | /{id}/collaborators | 添加协作者 |
| DELETE | /{id}/collaborators/{uid} | 移除协作者 |
| GET | /{id}/export/xlsx | 导出 Excel |
| GET | /{id}/export/photos | 导出照片 |
| GET | /{id}/export/bundle | 导出打包 |
| POST | /analyze | 分析费用项 |

## 对话 `/api/chat` & `/api/conversations`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /chat/ | 发送对话（二狗） |
| GET | /chat/usage | 使用量统计 |
| GET | /conversations/ | 对话列表 |
| GET | /conversations/{id}/messages | 对话消息 |
| DELETE | /conversations/{id} | 删除对话 |
| POST | /conversations/{id}/rename | 重命名 |

## 灵魂状态 `/api/soul-state` ✅ 已实现

| 方法 | 路径 | 说明 | 状态 |
|------|------|------|------|
| GET | / | 获取当前用户灵魂状态（无记录时自动创建默认值） | ✅ |
| PUT | / | 手动调整灵魂状态（部分更新，范围校验） | ✅ |

> 灵魂状态在每次 `/api/chat` 对话后由后端异步演化（±0.05 上限，每日最多 10 次）。
> 注册时自动初始化默认灵魂状态。Prompt 构建已集成灵魂状态动态段落。
> 详见 [ADR-002](../docs/decisions/002-soul-state-sync.md)。

## 名言 `/api/quotes`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /random | 随机名言 |

## 英语 `/api/english`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /scenarios | 场景列表 |
| POST | /scenarios | 创建场景 |
| ... | ... | (待补充完整) |
