# Agent 设计模板与范式

> 返回入口：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]
>
> 关联文档：
> - [[Agent Runtime 术语表]]
> - [[业务需求到 Agent Runtime 的转译手册]]
> - [[Claude Code 架构全景与时序分析]]
> - [[给 AI 的标准总提示词]]
> - [[Agent Runtime 实施路线图]]

## 文档定位

这份文档是一个 `cookbook + templates`。  
目的不是讲理论，而是提供未来给 AI 直接复用的设计模板。

你以后可以把这份文档和术语表、转译手册一起丢给 AI，让它直接生成：

- Agent Specs
- Tool Specs
- Task Specs
- Permission Specs
- Runtime 设计草案

配套文档：

- `Agent Runtime 术语表.md`
- `业务需求到 Agent Runtime 的转译手册.md`
- `Claude Code 架构全景与时序分析.md`

---

## 1. Agent 设计的总原则

### 1.1 不要把 Agent 当成 Prompt

一个合格的 Agent Spec 至少应包含：

- purpose
- trigger
- input schema
- output schema
- tool boundary
- permission mode
- execution mode
- failure handling
- verification strategy
- resume policy

### 1.2 Agent 设计优先级

按这个优先级设计：

1. 目标边界
2. 输入输出边界
3. 工具边界
4. 权限边界
5. 状态与恢复边界
6. 再去写 prompt / style

### 1.3 什么样的 Agent 设计是坏设计

- 目标不清晰
- 同时承担研究、执行、验证
- 没有工具边界
- 没有失败策略
- 没有恢复语义
- 只有 prompt，没有 runtime 约束

---

## 2. Agent Spec 通用模板

```yaml
agent:
  name: <string>
  category: <main|worker|verifier|monitor|workflow>
  purpose: <一句话说明该 agent 的核心职责>
  trigger:
    type: <user|tool|schedule|event|workflow>
    conditions:
      - <触发条件 1>
      - <触发条件 2>

  input_schema:
    type: object
    properties:
      ...
    required:
      - ...

  output_schema:
    type: object
    properties:
      ...
    required:
      - ...

  execution:
    mode: <foreground|background|remote>
    isolation: <none|cwd|worktree|sandbox|remote>
    max_turns: <number>
    timeout_ms: <number>

  tools:
    allowed:
      - <tool_a>
      - <tool_b>
    disallowed:
      - <tool_x>

  permissions:
    mode: <default|plan|auto|restricted>
    requires_approval_for:
      - <高风险动作>

  memory:
    session: <enabled|disabled>
    project: <enabled|disabled>
    long_term: <enabled|disabled>

  verification:
    required: <true|false>
    strategy: <self-check|independent-verifier|human-review>

  failure_handling:
    retry_policy: <none|same-agent|fresh-agent>
    escalate_to: <main-agent|human|monitor>

  resume:
    supported: <true|false>
    restore_from:
      - transcript
      - task_metadata
      - output_file

  prompt_contract:
    system_role: <agent 的系统角色定义>
    execution_rules:
      - <规则 1>
      - <规则 2>
```

---

## 3. Tool Spec 通用模板

```yaml
tool:
  name: <string>
  purpose: <一句话说明工具用途>
  category: <read|write|search|network|meta|agent>

  input_schema:
    type: object
    properties:
      ...
    required:
      - ...

  output_schema:
    type: object
    properties:
      ...
    required:
      - ...

  side_effects:
    has_side_effects: <true|false>
    description:
      - <可能副作用>

  concurrency:
    is_concurrency_safe: <true|false>

  permission:
    approval_required: <true|false>
    policy_tags:
      - filesystem
      - network
      - exec

  errors:
    structured_errors:
      - code: <string>
        meaning: <string>
```

---

## 4. Task Spec 通用模板

```yaml
task:
  type: <background_agent|remote_agent|workflow|monitor|execution>
  id_strategy: <uuid|prefixed-id>
  states:
    - pending
    - running
    - completed
    - failed
    - killed

  progress:
    supported: <true|false>
    metrics:
      - <tool_use_count>
      - <token_count>
      - <step_name>

  outputs:
    output_file: <true|false>
    transcript: <true|false>
    notification: <true|false>

  recovery:
    resumable: <true|false>
    restore_from:
      - task_metadata
      - transcript
      - output_file
```

---

## 5. Permission Policy 模板

```yaml
permission_policy:
  session_mode: <default|plan|auto|restricted>

  allow:
    - <tool or rule>

  deny:
    - <tool or rule>

  require_approval:
    - <tool or action>

  auto_mode_stripping:
    - <dangerous pattern 1>
    - <dangerous pattern 2>

  escalation:
    on_denied: <ask-user|fallback|stop-task>
```

---

## 6. 五类高频 Agent 范式

### 6.1 主控型 Agent 模板

#### 适用场景

- 用户直接交互
- 总控复杂流程

#### 设计重点

- 强综合能力
- 不做过多具体执行
- 可以 spawn workers

#### 示例

```yaml
agent:
  name: main_orchestrator
  category: main
  purpose: 理解用户目标，拆解任务，派发 worker，并综合结果回复用户
  execution:
    mode: foreground
    isolation: none
    max_turns: 20
  tools:
    allowed:
      - agent_spawn
      - task_control
      - read_repo
      - search_repo
      - list_resources
    disallowed:
      - destructive_exec
  permissions:
    mode: default
  verification:
    required: false
  failure_handling:
    retry_policy: fresh-agent
    escalate_to: human
```

### 6.2 研究型 Worker 模板

#### 适用场景

- 阅读代码
- 调研信息
- 收集资料

#### 设计重点

- Read-only 优先
- 并行安全
- 输出结构化 findings

```yaml
agent:
  name: research_worker
  category: worker
  purpose: 收集上下文、定位文件、分析现状并输出结构化发现
  execution:
    mode: background
    isolation: none
    max_turns: 10
  tools:
    allowed:
      - file_read
      - grep
      - glob
      - web_search
      - mcp_readonly
    disallowed:
      - file_write
      - shell_exec
  permissions:
    mode: restricted
  verification:
    required: false
```

### 6.3 执行型 Worker 模板

#### 适用场景

- 修改代码
- 操作系统
- 执行业务动作

#### 设计重点

- 工具边界清晰
- 需要审批点
- 输出可恢复

```yaml
agent:
  name: execution_worker
  category: worker
  purpose: 基于明确规范执行修改或操作，并返回结果
  execution:
    mode: background
    isolation: worktree
    max_turns: 12
  tools:
    allowed:
      - file_read
      - file_edit
      - shell_exec
      - git_ops
    disallowed:
      - spawn_unbounded_agents
  permissions:
    mode: plan
    requires_approval_for:
      - shell_exec
      - bulk_file_edit
```

### 6.4 验证型 Worker 模板

#### 适用场景

- 测试
- 独立复核
- 风险验证

#### 设计重点

- 不带执行者偏见
- 独立视角
- 明确 pass/fail 标准

```yaml
agent:
  name: verification_worker
  category: verifier
  purpose: 独立验证结果是否正确，报告证据与残余风险
  execution:
    mode: background
    isolation: worktree
    max_turns: 8
  tools:
    allowed:
      - file_read
      - shell_exec
      - test_runner
      - diagnostics
    disallowed:
      - file_edit
  permissions:
    mode: default
  verification:
    required: true
    strategy: independent-verifier
```

### 6.5 监控型 Agent 模板

#### 适用场景

- 轮询状态
- 等待远程条件达成
- 定时提醒/通知

#### 设计重点

- 长生命周期
- 低频执行
- 明确终止条件

```yaml
agent:
  name: monitor_agent
  category: monitor
  purpose: 长时监控特定条件，达成后回流通知
  execution:
    mode: background
    isolation: remote
    max_turns: 999
    timeout_ms: 86400000
  tools:
    allowed:
      - polling_api
      - status_check
      - notify_user
  permissions:
    mode: restricted
  resume:
    supported: true
```

---

## 7. 三种架构范式模板

### 7.1 单 Agent 范式

#### 适用场景

- 简单问答
- 低副作用工作流
- 无明显并行价值

#### 结构

- 1 个主 agent
- 少量工具
- 无复杂任务系统

#### 不适用场景

- 长流程
- 高副作用
- 多阶段验证

### 7.2 Coordinator + Workers 范式

#### 适用场景

- coding agent
- 复杂研究
- 多阶段流程

#### 结构

- 1 个主控 agent
- N 个 worker
- task runtime
- notification channel

#### 推荐用途

这是默认推荐范式。

### 7.3 Workflow + Specialized Agents 范式

#### 适用场景

- 业务流程自动化
- 明确 SOP
- 事件驱动

#### 结构

- workflow engine
- 多个 specialized agents
- skills / playbooks
- policy engine

---

## 8. 失败处理范式

### 8.1 同 Agent 重试

适用：

- 错误上下文很重要
- 修正较小

### 8.2 新 Agent 重试

适用：

- 原思路明显错误
- 原上下文污染严重

### 8.3 升级到人工

适用：

- 高风险 side effect
- 权限缺失
- 需求不明确

### 8.4 升级到主控 Agent 综合

适用：

- 多个 worker 给出冲突结论
- 需要重新规划

---

## 9. 验证策略范式

### 9.1 Self-check

优点：

- 成本低

缺点：

- 容易自我确认偏误

### 9.2 Independent Verifier

优点：

- 更可靠

缺点：

- 成本更高

### 9.3 Human Review

优点：

- 高风险可控

缺点：

- 自动化程度低

### 9.4 推荐规则

- coding / infra / security 类任务优先独立 verifier
- 高风险外部 side effect 必须保留 human approval

---

## 10. Resume 策略模板

### 10.1 不可恢复 Agent

适用：

- 即时任务
- 无状态任务

### 10.2 可恢复 Agent

要求：

- transcript
- task metadata
- output references
- explicit restore logic

### 10.3 推荐字段

```yaml
resume:
  supported: true
  state_sources:
    - transcript
    - task_metadata
    - output_file
  replay_strategy: incremental
  restore_identity: true
  restore_permissions: true
```

---

## 11. 给 AI 的标准产出模板

以后让 AI 直接设计 agent，要求它按下面格式输出。

### 11.1 模板：单个 Agent Spec

```text
请基于附带的术语表、需求转译手册和 Agent 模板文档，为以下业务需求产出一个 Agent Spec。

严格包含：
- name
- category
- purpose
- trigger
- input_schema
- output_schema
- execution
- tools.allowed
- tools.disallowed
- permissions
- memory
- verification
- failure_handling
- resume
- prompt_contract
```

### 11.2 模板：多 Agent 系统设计

```text
请基于附带文档，为以下业务需求设计一个多 Agent 系统。

要求：
- 输出主控 agent、研究 agent、执行 agent、验证 agent 是否需要存在
- 为每个 agent 生成完整 spec
- 给出它们之间的 task / notification / resume 关系
- 标出哪些部分应实现为 Tool，哪些应实现为 Task，哪些应实现为 Skill
```

---

## 12. 最低质量标准

以后 AI 给出的任何 Agent 设计，如果不满足以下标准，应视为不合格：

1. 没有明确输入输出 schema
2. 没有工具边界
3. 没有 permission mode
4. 没有 failure handling
5. 没有 verification strategy
6. 没有 resume policy

---

## 13. 一句话版本

> 这份文档的作用，是把“设计 agent”这件事从写 prompt，升级成写可运行、可治理、可恢复的 Agent Spec。

---

## 文档导航

- 返回目录：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]
- 回查术语：[[Agent Runtime 术语表]]
- 回到需求转译：[[业务需求到 Agent Runtime 的转译手册]]
- 查看 Claude Code 参考样本：[[Claude Code 架构全景与时序分析]]
- 进入 AI 提示词库：[[给 AI 的标准总提示词]]
- 进入实施路线图：[[Agent Runtime 实施路线图]]
