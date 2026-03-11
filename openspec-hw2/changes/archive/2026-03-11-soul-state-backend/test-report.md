## Test Report: soul-state-backend

Generated: 2026-03-11

### Use Case Coverage Summary

| Use Case | Happy Path | Extensions | Overall |
|----------|-----------|------------|---------|
| UC1 获取灵魂状态 | ✅ 3/3 | ✅ 2/2 | 100% |
| UC2 手动调整灵魂状态 | ✅ 5/5 | ✅ 1/1 | 100% |
| UC3 对话后演化灵魂状态 | ⚠️ 3/5 | ❌ 0/2 | 43% |
| UC4 构建 Prompt | ✅ 3/3 | — | 100% |
| UC5 注册时初始化 | ✅ 1/1 | — | 100% |

**Overall: 18/22 paths/steps covered (82%)**

### Test Run Results

```
test result: ok. 54 passed; 0 failed; 0 ignored; 0 measured
  - 43 existing tests: all pass
  - 11 new soul-state tests: all pass
```

Soul-state specific tests:
- ✅ `test_get_soul_state_success`
- ✅ `test_get_soul_state_lazy_create`
- ✅ `test_soul_state_unauthenticated_401`
- ✅ `test_put_soul_state_partial_update`
- ✅ `test_put_soul_state_range_validation`
- ✅ `test_put_soul_state_logs_changes`
- ✅ `test_soul_evolution_interaction_count`
- ✅ `test_soul_evolution_relationship_upgrade`
- ✅ `test_soul_state_field_clamping`
- ✅ `test_soul_prompt_building`
- ✅ `test_register_creates_soul_state`

### Covered Requirements

- ✅ **UC1-S1**: 客户端请求灵魂状态 (`tests/api_tests.rs::test_get_soul_state_success`)
- ✅ **UC1-S2**: 系统查找灵魂状态记录 (`tests/api_tests.rs::test_get_soul_state_success`)
- ✅ **UC1-S3**: 系统返回完整灵魂状态 (`tests/api_tests.rs::test_get_soul_state_success`)
- ✅ **UC1-E2a**: 无记录时懒创建 (`tests/api_tests.rs::test_get_soul_state_lazy_create`)
- ✅ **UC1-E-unauth**: 未认证访问返回 401 (`tests/api_tests.rs::test_soul_state_unauthenticated_401`)
- ✅ **UC2-S1**: 提交待更新字段 (`tests/api_tests.rs::test_put_soul_state_partial_update`)
- ✅ **UC2-S2**: 校验字段范围 (`tests/api_tests.rs::test_put_soul_state_range_validation`)
- ✅ **UC2-S3**: 更新指定字段 (`tests/api_tests.rs::test_put_soul_state_partial_update`)
- ✅ **UC2-S4**: 变更写入演化日志 (`tests/api_tests.rs::test_put_soul_state_logs_changes`)
- ✅ **UC2-S5**: 返回完整状态 (`tests/api_tests.rs::test_put_soul_state_partial_update`)
- ✅ **UC2-E2a**: 字段值超出范围返回 400 (`tests/api_tests.rs::test_put_soul_state_range_validation`)
- ✅ **UC3-S2**: total_interactions 递增 (`tests/api_tests.rs::test_soul_evolution_interaction_count`)
- ✅ **UC3-S5**: 参数钳制 (`tests/api_tests.rs::test_soul_state_field_clamping`)
- ✅ **UC3-S6**: 关系阶段自动升级 (`tests/api_tests.rs::test_soul_evolution_relationship_upgrade`)
- ✅ **UC4-S1**: 从 DB 加载灵魂状态 (`tests/api_tests.rs::test_soul_prompt_building`)
- ✅ **UC4-S3**: 动态性格段落 (`tests/api_tests.rs::test_soul_prompt_building`)
- ✅ **UC4-S5**: 关系阶段行为段落 (`tests/api_tests.rs::test_soul_prompt_building`)
- ✅ **UC5-S2**: 注册后创建默认状态 (`tests/api_tests.rs::test_register_creates_soul_state`)

### Uncovered Requirements

- ❌ **UC3-S3**: 每日演化次数限制 — 需要 mock LLM API 或注入 10+ 条 auto 日志后验证跳过
- ❌ **UC3-S4**: LLM 分析对话内容 — 需要 mock LLM API 调用来验证 prompt 和 JSON 解析
- ❌ **UC3-E3a**: 达到每日上限跳过 LLM — 同 UC3-S3
- ❌ **UC3-E4a**: LLM 失败不阻塞 — 需要 mock LLM API 返回错误来验证优雅降级
