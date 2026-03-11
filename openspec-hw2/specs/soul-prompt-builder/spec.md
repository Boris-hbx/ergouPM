## ADDED Requirements

### Requirement: 从数据库加载灵魂状态用于 Prompt 构建
系统 SHALL 在处理 `POST /api/chat` 时从 `soul_states` 表读取当前用户的灵魂状态，用于构建系统 Prompt。

#### Scenario: 正常加载
- **WHEN** 用户发起对话
- **THEN** 系统从数据库读取该用户的灵魂状态，传入 Prompt 构建流程

#### Scenario: 灵魂状态不存在时懒创建
- **WHEN** 用户发起对话且无灵魂状态记录
- **THEN** 系统以默认值创建灵魂状态记录，并使用默认值构建 Prompt

### Requirement: 静态人设模板加载
系统 SHALL 加载 `llm/system-prompt.md` 中定义的静态人设内容作为 Prompt 基础。

静态部分包含：
- 身份（"你是二狗，一个私人AI助手"）
- 基础性格（冷静克制、专业精准、务实导向等）
- 引经据典规则
- 说话方式规则
- 记忆指令格式
- 安全边界

#### Scenario: 静态内容完整包含
- **WHEN** 系统构建 Prompt
- **THEN** 最终 Prompt 包含身份、基础性格、引经据典、说话方式、记忆指令、安全边界六个部分

### Requirement: 动态性格段落生成
系统 SHALL 根据 `warmth_level`、`trust_level`、`verbosity_level`、`proactivity_level` 生成动态性格描述。

#### Scenario: 低温度低信任（stranger 阶段典型）
- **WHEN** `warmth_level` ≤ 0.3 且 `trust_level` ≤ 0.2
- **THEN** 动态性格段落强调克制、保持距离、不过分热情

#### Scenario: 高温度高信任（close/intimate 阶段典型）
- **WHEN** `warmth_level` ≥ 0.7 且 `trust_level` ≥ 0.7
- **THEN** 动态性格段落体现关心、主动问候、适度表达情感

### Requirement: 动态说话风格生成
系统 SHALL 根据 `classical_ratio` 生成文白比例的说话风格指令。

#### Scenario: 高文言比例
- **WHEN** `classical_ratio` ≥ 0.8
- **THEN** 说话风格指令要求以文言为主，白话为辅，技术内容除外

#### Scenario: 低文言比例
- **WHEN** `classical_ratio` ≤ 0.7
- **THEN** 说话风格指令要求文白混用，降低古文频率

### Requirement: 动态行为段落生成
系统 SHALL 根据 `relationship_stage` 生成对应关系阶段的行为指令。

#### Scenario: stranger 阶段行为
- **WHEN** `relationship_stage` 为 "stranger"
- **THEN** 行为指令强调正式、保持边界、不主动关心私事

#### Scenario: familiar 阶段行为
- **WHEN** `relationship_stage` 为 "familiar"
- **THEN** 行为指令允许适度主动、可以提及过往对话、偶尔关心

#### Scenario: intimate 阶段行为
- **WHEN** `relationship_stage` 为 "intimate"
- **THEN** 行为指令体现高度信任、主动关心、语气亲近自然

### Requirement: 语气示例生成
系统 SHALL 根据当前关系阶段生成 2-3 个典型语气示例，帮助 LLM 理解期望的表达方式。

#### Scenario: stranger 阶段示例
- **WHEN** `relationship_stage` 为 "stranger"
- **THEN** 示例体现正式、简洁、保持距离的语气

#### Scenario: close 阶段示例
- **WHEN** `relationship_stage` 为 "close"
- **THEN** 示例体现熟悉、温和、适度关心的语气

### Requirement: Prompt 拼装顺序
系统 SHALL 按以下顺序拼装最终系统 Prompt：
1. 静态身份
2. 静态基础性格 + 动态性格段落
3. 引经据典规则
4. 静态说话方式 + 动态说话风格
5. 动态行为段落
6. 语气示例
7. 记忆指令
8. 安全边界

#### Scenario: 拼装完整性
- **WHEN** 系统完成 Prompt 拼装
- **THEN** 最终 Prompt 包含上述全部 8 个部分，顺序正确

#### Scenario: Prompt 作为 system message 传递
- **WHEN** 系统调用 LLM API
- **THEN** 拼装好的 Prompt 作为对话的 system message（role: system）传递
