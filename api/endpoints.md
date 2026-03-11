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
| GET | /summary | 月度汇总 |
| GET | /analytics | 分析数据 |
| GET | /tags | 标签列表 |
| GET | /rates | 汇率 |
| POST | /parse-preview | 解析预览（小票识别） |
| GET | /{id} | 详情 |
| PUT | /{id} | 更新 |
| DELETE | /{id} | 删除 |
| POST | /{id}/photos | 上传照片 |
| POST | /{id}/parse | 解析小票 |
| DELETE | /photos/{photo_id} | 删除照片 |

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
| POST | /chat/ | 发送对话（Web 端阿宝） |
| GET | /chat/usage | 使用量统计 |
| GET | /conversations/ | 对话列表 |
| GET | /conversations/{id}/messages | 对话消息 |
| DELETE | /conversations/{id} | 删除对话 |
| POST | /conversations/{id}/rename | 重命名 |

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
