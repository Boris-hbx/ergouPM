## ADDED Requirements

### Requirement: 对话后自动递增交互计数
系统 SHALL 在每次 `POST /api/chat` 成功响应后将该用户的 `total_interactions` 加 1。

#### Scenario: 对话完成后计数递增
- **WHEN** `POST /api/chat` 成功完成一轮对话
- **THEN** 该用户的 `total_interactions` 加 1

### Requirement: 对话后自动演化灵魂参数
系统 SHALL 在对话完成后通过 LLM 分析对话内容，自动调整灵魂状态参数。

#### Scenario: 正常演化
- **WHEN** 对话完成且当日演化次数 < 10
- **THEN** 系统调用 LLM 分析对话内容，根据分析结果调整 `classical_ratio`、`warmth_level`、`verbosity_level`、`proactivity_level`、`trust_level` 中的一个或多个参数

#### Scenario: LLM 分析失败不阻塞
- **WHEN** LLM 分析调用失败或超时
- **THEN** 灵魂状态保持不变，系统记录错误日志，`total_interactions` 仍然递增

### Requirement: 参数调整上限
系统 SHALL 限制每次自动演化中每个参数的调整幅度不超过 ±0.05。

#### Scenario: 调整幅度钳制
- **WHEN** LLM 建议将 `warmth_level` 从 0.3 调整至 0.5（差值 0.2）
- **THEN** 系统将实际调整钳制为 0.35（即 +0.05）

#### Scenario: 调整后值钳制在合法范围
- **WHEN** 参数调整后值超出字段合法范围（如 `trust_level` 从 0.98 加 0.05 = 1.03）
- **THEN** 系统将值钳制为范围上限（1.0）

### Requirement: 每日演化次数限制
系统 SHALL 限制每个用户每日自动演化（LLM 参数分析）最多 10 次。

#### Scenario: 达到每日上限
- **WHEN** 对话完成且当日演化次数已达 10 次
- **THEN** 系统跳过 LLM 分析和参数调整，仅递增 `total_interactions` 并检查关系阈值

#### Scenario: 未达上限正常演化
- **WHEN** 对话完成且当日演化次数为 5
- **THEN** 系统正常执行 LLM 分析和参数调整，演化计数变为 6

### Requirement: 关系阶段自动升级
系统 SHALL 在 `total_interactions` 越过阈值时自动升级 `relationship_stage`。

阈值定义：
| 交互次数 | 关系阶段 |
|----------|----------|
| 0 | stranger |
| 10 | acquaintance |
| 50 | familiar |
| 150 | close |
| 500 | intimate |

#### Scenario: 从 stranger 升级到 acquaintance
- **WHEN** `total_interactions` 从 9 变为 10
- **THEN** `relationship_stage` 从 "stranger" 升级为 "acquaintance"

#### Scenario: 从 familiar 升级到 close
- **WHEN** `total_interactions` 从 149 变为 150
- **THEN** `relationship_stage` 从 "familiar" 升级为 "close"

#### Scenario: 未越过阈值不升级
- **WHEN** `total_interactions` 从 11 变为 12
- **THEN** `relationship_stage` 保持 "acquaintance" 不变

### Requirement: 演化变更记录日志
系统 SHALL 将每次自动演化的参数变更写入 `soul_evolution_log`，`trigger_type` 为 "auto"。

#### Scenario: 自动演化写日志
- **WHEN** 自动演化调整了 `warmth_level`（0.3→0.35）和 `trust_level`（0.1→0.15）
- **THEN** 系统写入 2 条演化日志，每条包含 parameter、old_value、new_value，trigger_type 为 "auto"

#### Scenario: 关系阶段升级写日志
- **WHEN** `relationship_stage` 从 "stranger" 升级为 "acquaintance"
- **THEN** 系统写入 1 条演化日志，parameter 为 "relationship_stage"，记录变更

### Requirement: 异步执行不阻塞对话响应
系统 SHALL 异步执行灵魂演化，不阻塞 `POST /api/chat` 的响应返回。

#### Scenario: 对话响应不等待演化
- **WHEN** 对话响应流式传输完毕
- **THEN** 系统在后台异步触发演化逻辑，用户已收到完整对话响应
