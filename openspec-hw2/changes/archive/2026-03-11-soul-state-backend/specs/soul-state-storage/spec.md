## ADDED Requirements

### Requirement: 灵魂状态数据模型
系统 SHALL 维护 `soul_states` 表，每个用户一条记录，包含以下字段：

| 字段 | 类型 | 默认值 | 范围 |
|------|------|--------|------|
| user_id | TEXT (PK, FK→users) | — | — |
| classical_ratio | REAL | 0.9 | 0.6–1.0 |
| warmth_level | REAL | 0.3 | 0.0–1.0 |
| verbosity_level | REAL | 0.3 | 0.0–1.0 |
| proactivity_level | REAL | 0.2 | 0.0–0.8 |
| trust_level | REAL | 0.1 | 0.0–1.0 |
| relationship_stage | TEXT | "stranger" | stranger/acquaintance/familiar/close/intimate |
| total_interactions | INTEGER | 0 | ≥0 |
| updated_at | TEXT | datetime('now') | ISO 8601 |

#### Scenario: 表结构完整性
- **WHEN** 数据库初始化
- **THEN** `soul_states` 表存在，`user_id` 为主键且外键关联 `users(id)`，所有字段具有上述默认值

### Requirement: 注册时自动初始化灵魂状态
系统 SHALL 在用户注册成功后为其创建一条默认灵魂状态记录。

#### Scenario: 注册自动创建
- **WHEN** 用户通过 `POST /api/auth/register` 注册成功
- **THEN** 系统在 `soul_states` 表中插入一条该用户的默认值记录

#### Scenario: 初始化失败不阻塞注册
- **WHEN** 灵魂状态插入失败（如约束冲突）
- **THEN** 注册仍然成功，系统记录错误日志，灵魂状态将在首次 GET 时懒创建

### Requirement: 懒创建灵魂状态
系统 SHALL 在查询灵魂状态时，若记录不存在则自动创建默认值记录。

#### Scenario: 首次 GET 触发懒创建
- **WHEN** 已认证用户发送 `GET /api/soul-state` 且无灵魂状态记录
- **THEN** 系统创建默认灵魂状态记录并返回

### Requirement: 获取灵魂状态 API
系统 SHALL 提供 `GET /api/soul-state` 端点，返回当前认证用户的完整灵魂状态。

#### Scenario: 成功获取
- **WHEN** 已认证用户发送 `GET /api/soul-state`
- **THEN** 系统返回 200，body 包含 `success: true` 及 `soul_state` 对象（含全部字段和 `updated_at`）

#### Scenario: 未认证访问
- **WHEN** 未认证请求发送 `GET /api/soul-state`
- **THEN** 系统返回 401

### Requirement: 手动调整灵魂状态 API
系统 SHALL 提供 `PUT /api/soul-state` 端点，支持部分字段更新。

#### Scenario: 成功部分更新
- **WHEN** 已认证用户发送 `PUT /api/soul-state`，body 为 `{"warmth_level": 0.5, "trust_level": 0.3}`
- **THEN** 系统仅更新 `warmth_level` 和 `trust_level`，其余字段不变，返回 200 及更新后的完整状态

#### Scenario: 字段值超出范围
- **WHEN** 请求包含超出合法范围的字段值（如 `warmth_level: 1.5`）
- **THEN** 系统返回 400，body 列出无效字段及合法范围

#### Scenario: 未知字段被忽略
- **WHEN** 请求包含未知字段名（如 `foo: 123`）
- **THEN** 系统忽略未知字段，仅处理有效字段

#### Scenario: 手动调整记录演化日志
- **WHEN** `PUT /api/soul-state` 成功更新字段
- **THEN** 系统为每个变更的字段写入一条演化日志，`trigger_type` 为 "manual"

### Requirement: 演化日志数据模型
系统 SHALL 维护 `soul_evolution_log` 表，记录每次灵魂状态变更。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER (PK, 自增) | — |
| user_id | TEXT (FK→users) | — |
| parameter | TEXT | 变更的字段名 |
| old_value | REAL | 变更前的值 |
| new_value | REAL | 变更后的值 |
| trigger_type | TEXT | "auto" 或 "manual" |
| created_at | TEXT | ISO 8601 时间戳 |

#### Scenario: 日志表结构完整性
- **WHEN** 数据库初始化
- **THEN** `soul_evolution_log` 表存在，`id` 自增主键，`user_id` 外键关联 `users(id)`
