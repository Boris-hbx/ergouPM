## Test Plan: soul-state-backend

Generated: 2026-03-11
Source: test-report.md

### Summary

| ID | UC Step | Reason | Tool |
|----|---------|--------|------|
| TP-1 | UC3-S3 | EXTERNAL_API | Mock HTTP server or manual test |
| TP-2 | UC3-S4 | EXTERNAL_API | Mock HTTP server or manual test |
| TP-3 | UC3-E3a | EXTERNAL_API | Mock HTTP server or manual test |
| TP-4 | UC3-E4a | EXTERNAL_API | Mock HTTP server or manual test |

---

## TP-1: UC3-S3 — 每日演化次数限制

**Blocking reason**: EXTERNAL_API — 演化分析调用 Claude API（Anthropic），测试环境无 API key，无法触发 LLM 分析路径

**Preconditions**
- 服务器已启动，用户已登录
- 用户已有灵魂状态记录
- `soul_evolution_log` 中已有该用户当日 10 条 `trigger_type='auto'` 记录

**Test Steps**
1. 在数据库中为目标用户插入 10 条当日 auto 演化日志
2. 通过 `POST /api/chat` 发送一条对话消息
3. 等待对话完成（约 5 秒让异步演化执行）
4. 查询 `soul_states` 表中该用户的 `total_interactions` 值
5. 查询 `soul_evolution_log` 表中该用户当日 auto 记录数

**Expected Result**
- `total_interactions` 增加了 1（对话计数仍然递增）
- `soul_evolution_log` 当日 auto 记录数仍为 10（未新增 LLM 分析结果）
- 灵魂参数（warmth_level 等）保持不变

**Failure indicators**
- `soul_evolution_log` 当日 auto 记录数超过 10
- 灵魂参数发生变化

**Automation path**
引入 `wiremock` crate 或类似 mock HTTP server，在测试中拦截 Anthropic API 调用。可在 `soul_evolution.rs` 中将 API URL 提取为可配置项，测试时指向 mock server。

---

## TP-2: UC3-S4 — LLM 分析对话内容

**Blocking reason**: EXTERNAL_API — 需要真实 Claude API 调用来验证 prompt 构建和 JSON 响应解析

**Preconditions**
- 服务器已启动，用户已登录
- 环境变量 `ANTHROPIC_API_KEY` 已设置（使用真实或测试 key）
- 用户有灵魂状态记录且当日演化次数 < 10

**Test Steps**
1. 通过 `POST /api/chat` 发送一条带情感倾向的消息（如"谢谢你，二狗，你帮了我大忙"）
2. 等待 10 秒让异步演化完成
3. 查询 `soul_evolution_log` 表中该用户最新的 auto 记录
4. 查询 `soul_states` 表中该用户当前参数值

**Expected Result**
- `soul_evolution_log` 中出现新的 auto 记录
- 参数变化幅度不超过 ±0.05
- 变化方向合理（感谢消息可能增加 warmth_level 或 trust_level）

**Failure indicators**
- 无新的演化日志（LLM 调用可能失败）
- 参数变化超过 ±0.05
- 参数超出字段合法范围

**Automation path**
使用 wiremock 返回预设的 `{"adjustments": {"warmth_level": "+", "trust_level": "+"}}` JSON 响应，验证后续参数更新逻辑。

---

## TP-3: UC3-E3a — 达到每日上限跳过 LLM

**Blocking reason**: EXTERNAL_API — 与 TP-1 相同，需要验证达到上限后不再发出 LLM 请求

**Preconditions**
- 同 TP-1

**Test Steps**
1. 同 TP-1 步骤 1-5
2. 额外检查：对话期间是否有 HTTP 请求发往 `api.anthropic.com`（通过 mock server 或网络日志）

**Expected Result**
- 未向 LLM API 发送请求
- `total_interactions` 仍然递增
- 关系阶段检查仍然执行

**Failure indicators**
- 检测到向 LLM API 的请求

**Automation path**
同 TP-1，使用 mock HTTP server 记录请求数，断言为 0。

---

## TP-4: UC3-E4a — LLM 失败不阻塞

**Blocking reason**: EXTERNAL_API — 需要模拟 LLM API 返回错误来验证优雅降级

**Preconditions**
- 服务器已启动，用户已登录
- `ANTHROPIC_API_KEY` 设置为无效值（如 "invalid-key"）
- 用户有灵魂状态记录

**Test Steps**
1. 记录当前灵魂状态参数值
2. 通过 `POST /api/chat` 发送一条消息
3. 验证对话正常返回（不因演化失败而报错）
4. 等待 5 秒
5. 查询 `soul_states` 表中 `total_interactions`
6. 对比演化前后的灵魂参数

**Expected Result**
- 对话正常返回 200，包含 AI 回复
- `total_interactions` 增加了 1
- 灵魂参数保持不变（因 LLM 调用失败被跳过）
- 服务器日志中有 `[soul-evolution] LLM analysis failed` 错误信息

**Failure indicators**
- 对话返回失败
- `total_interactions` 未递增
- 服务器崩溃或无响应

**Automation path**
在测试中设置 `ANTHROPIC_API_KEY` 为空或无效值，直接调用 `evolve_after_chat()` 并断言不 panic，`total_interactions` 仍递增。

---

## How to Run These Tests

**EXTERNAL_API tests**: 需要以下任一方式：
1. **Mock server 方式**（推荐）：引入 `wiremock` crate，将 API URL 提取为可配置项，测试时指向 mock server
2. **真实 API 方式**：设置 `ANTHROPIC_API_KEY` 环境变量，运行集成测试（消耗 token）
3. **手动测试方式**：启动服务器，用 curl 或前端执行上述步骤，手动检查数据库
