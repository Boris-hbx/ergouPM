# Spec-Test Mapping: soul-state-backend
Generated: 2026-03-11

## Use Case ID Mapping

| ID | Use Case |
|----|----------|
| UC1 | 获取灵魂状态 |
| UC2 | 手动调整灵魂状态 |
| UC3 | 对话后演化灵魂状态 |
| UC4 | 基于灵魂状态构建系统 Prompt |
| UC5 | 注册时初始化灵魂状态 |

## Requirement Traceability Matrix

| ID | Requirement | Type | Test Type | Test Case | Status |
|----|-------------|------|-----------|-----------|--------|
| UC1 | 获取灵魂状态 Full Flow | Flow | Integration | `tests/api_tests.rs::test_get_soul_state_success` | ✅ |
| UC1-S1 | 客户端请求灵魂状态 | Step | Integration | `tests/api_tests.rs::test_get_soul_state_success` | ✅ |
| UC1-S2 | 系统查找灵魂状态记录 | Step | Integration | `tests/api_tests.rs::test_get_soul_state_success` | ✅ |
| UC1-S3 | 系统返回完整灵魂状态 | Step | Integration | `tests/api_tests.rs::test_get_soul_state_success` | ✅ |
| UC1-E2a | 无记录时懒创建 | Extension | Integration | `tests/api_tests.rs::test_get_soul_state_lazy_create` | ✅ |
| UC1-E-unauth | 未认证访问 | Extension | Integration | `tests/api_tests.rs::test_soul_state_unauthenticated_401` | ✅ |
| UC2 | 手动调整灵魂状态 Full Flow | Flow | Integration | `tests/api_tests.rs::test_put_soul_state_partial_update` | ✅ |
| UC2-S1 | 提交待更新字段 | Step | Integration | `tests/api_tests.rs::test_put_soul_state_partial_update` | ✅ |
| UC2-S2 | 校验字段范围 | Step | Integration | `tests/api_tests.rs::test_put_soul_state_range_validation` | ✅ |
| UC2-S3 | 更新指定字段 | Step | Integration | `tests/api_tests.rs::test_put_soul_state_partial_update` | ✅ |
| UC2-S4 | 变更写入演化日志 | Step | Integration | `tests/api_tests.rs::test_put_soul_state_logs_changes` | ✅ |
| UC2-S5 | 返回完整状态 | Step | Integration | `tests/api_tests.rs::test_put_soul_state_partial_update` | ✅ |
| UC2-E2a | 字段值超出范围返回 400 | Extension | Integration | `tests/api_tests.rs::test_put_soul_state_range_validation` | ✅ |
| UC3 | 对话后演化 Full Flow | Flow | Unit | `tests/api_tests.rs::test_soul_evolution_interaction_count` | ✅ |
| UC3-S2 | total_interactions 递增 | Step | Unit | `tests/api_tests.rs::test_soul_evolution_interaction_count` | ✅ |
| UC3-S6 | 关系阶段自动升级 | Step | Unit | `tests/api_tests.rs::test_soul_evolution_relationship_upgrade` | ✅ |
| UC3-S3 | 每日演化次数限制 | Step | Unit | | ❌ |
| UC3-S4 | LLM 分析 | Step | Unit | | ❌ |
| UC3-S5 | 参数钳制 | Step | Unit | `tests/api_tests.rs::test_soul_state_field_clamping` | ✅ |
| UC4 | 构建 Prompt Full Flow | Flow | Unit | `tests/api_tests.rs::test_soul_prompt_building` | ✅ |
| UC4-S1 | 从 DB 加载灵魂状态 | Step | Unit | `tests/api_tests.rs::test_soul_prompt_building` | ✅ |
| UC4-S3 | 动态性格段落 | Step | Unit | `tests/api_tests.rs::test_soul_prompt_building` | ✅ |
| UC4-S5 | 关系阶段行为段落 | Step | Unit | `tests/api_tests.rs::test_soul_prompt_building` | ✅ |
| UC5 | 注册时初始化 Full Flow | Flow | Integration | `tests/api_tests.rs::test_register_creates_soul_state` | ✅ |
| UC5-S2 | 注册后创建默认状态 | Step | Integration | `tests/api_tests.rs::test_register_creates_soul_state` | ✅ |

## Use Case Details: 获取灵魂状态 (ID: UC1)

### Main Scenario
- **UC1-S1**: 客户端请求灵魂状态
  - `tests/api_tests.rs::test_get_soul_state_success` GET /api/soul-state returns 200 (Integration)
- **UC1-S2**: 系统查找灵魂状态记录
  - `tests/api_tests.rs::test_get_soul_state_success` (Integration)
- **UC1-S3**: 系统返回完整灵魂状态及 updated_at
  - `tests/api_tests.rs::test_get_soul_state_success` verifies all fields present (Integration)

### Extensions
- **UC1-E2a**: 无灵魂状态记录时懒创建默认值
  - `tests/api_tests.rs::test_get_soul_state_lazy_create` GET without prior INSERT returns defaults (Integration)
- **UC1-E-unauth**: 未认证访问返回 401
  - `tests/api_tests.rs::test_soul_state_unauthenticated_401` (Integration)

### Full Flow Tests
- `UC1` — "获取灵魂状态" -> `tests/api_tests.rs::test_get_soul_state_success` (Integration)

## Use Case Details: 手动调整灵魂状态 (ID: UC2)

### Main Scenario
- **UC2-S1**: 提交待更新的字段
  - `tests/api_tests.rs::test_put_soul_state_partial_update` PUT with partial fields (Integration)
- **UC2-S2**: 校验字段范围
  - `tests/api_tests.rs::test_put_soul_state_range_validation` (Integration)
- **UC2-S3**: 更新指定字段，其余不变
  - `tests/api_tests.rs::test_put_soul_state_partial_update` verifies unchanged fields (Integration)
- **UC2-S4**: 变更写入演化日志 (trigger_type=manual)
  - `tests/api_tests.rs::test_put_soul_state_logs_changes` checks evolution_log rows (Integration)
- **UC2-S5**: 返回更新后完整状态
  - `tests/api_tests.rs::test_put_soul_state_partial_update` verifies response (Integration)

### Extensions
- **UC2-E2a**: 字段值超出范围返回 400
  - `tests/api_tests.rs::test_put_soul_state_range_validation` (Integration)

### Full Flow Tests
- `UC2` — "手动调整灵魂状态" -> `tests/api_tests.rs::test_put_soul_state_partial_update` (Integration)

## Use Case Details: 对话后演化灵魂状态 (ID: UC3)

### Main Scenario
- **UC3-S2**: total_interactions 递增
  - `tests/api_tests.rs::test_soul_evolution_interaction_count` (Unit)
- **UC3-S3**: 每日演化次数检查 -> ❌ 未覆盖（需 mock LLM 调用）
- **UC3-S4**: LLM 分析对话内容 -> ❌ 未覆盖（需 mock LLM 调用）
- **UC3-S5**: 参数钳制（±0.05 + 字段范围）
  - `tests/api_tests.rs::test_soul_state_field_clamping` (Unit)
- **UC3-S6**: 关系阶段自动升级
  - `tests/api_tests.rs::test_soul_evolution_relationship_upgrade` (Unit)

### Extensions
- **UC3-E3a**: 达到每日上限跳过 LLM -> ❌ 未覆盖
- **UC3-E4a**: LLM 失败不阻塞 -> ❌ 未覆盖

### Full Flow Tests
- `UC3` — "对话后演化" -> `tests/api_tests.rs::test_soul_evolution_interaction_count` (Unit, partial)

## Use Case Details: 构建 Prompt (ID: UC4)

### Main Scenario
- **UC4-S1**: 从 DB 加载灵魂状态
  - `tests/api_tests.rs::test_soul_prompt_building` (Unit)
- **UC4-S3**: 动态性格段落
  - `tests/api_tests.rs::test_soul_prompt_building` checks personality keywords (Unit)
- **UC4-S5**: 关系阶段行为
  - `tests/api_tests.rs::test_soul_prompt_building` checks stage-specific text (Unit)

### Full Flow Tests
- `UC4` — "构建 Prompt" -> `tests/api_tests.rs::test_soul_prompt_building` (Unit)

## Use Case Details: 注册时初始化 (ID: UC5)

### Main Scenario
- **UC5-S2**: 注册后创建默认灵魂状态
  - `tests/api_tests.rs::test_register_creates_soul_state` (Integration)

### Full Flow Tests
- `UC5` — "注册时初始化" -> `tests/api_tests.rs::test_register_creates_soul_state` (Integration)
