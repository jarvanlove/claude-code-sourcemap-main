# 给 AI 的标准总提示词

> 返回入口：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]
>
> 关联文档：
> - [[Agent Runtime 术语表]]
> - [[业务需求到 Agent Runtime 的转译手册]]
> - [[Agent 设计模板与范式]]
> - [[Claude Code 架构全景与时序分析]]
> - [[Agent Runtime 实施路线图]]
> - [[业务场景 Agent 蓝图库]]
> - [[Agent 评审清单]]

## 文档定位

这份文档的目标，是提供一套可长期复用的高质量总提示词模板。  
以后你只要把这份文档和配套知识库一起喂给 AI，再附上你的业务需求，AI 就应该能稳定输出：

- 需求拆解
- runtime 对象映射
- agent / tool / task / permission / resume 设计
- implementation plan

这份文档不是理论说明，而是 `可直接复制使用的提示词库`。

---

## 1. 使用原则

### 1.1 先喂知识库，再喂需求

推荐顺序：

1. [[Agent Runtime 术语表]]
2. [[业务需求到 Agent Runtime 的转译手册]]
3. [[Agent 设计模板与范式]]
4. [[Claude Code 架构全景与时序分析]]
5. 当前文档
6. 你的业务需求

### 1.2 明确 AI 当前扮演的角色

不要让 AI 以“泛助手”角色工作。  
明确让它扮演：

- `Agent Runtime Architect`
- `Multi-Agent System Designer`
- `Coding Agent Platform Designer`

### 1.3 约束 AI 必须使用文档中的术语

必须强制 AI 使用：

- QueryEngine
- Query Loop
- Tool
- Task
- Agent
- Skill
- Plugin
- Permission Context
- Resume

否则 AI 会重新发明术语，导致输出不可复用。

---

## 2. 标准总提示词：需求转 Runtime 设计

```text
你现在不是普通助手，而是 Agent Runtime Architect。

我会给你一组基础文档，它们定义了我要求你使用的设计语言、决策规则和模板：

1. Agent Runtime 术语表
2. 业务需求到 Agent Runtime 的转译手册
3. Agent 设计模板与范式
4. Claude Code 架构全景与时序分析

你的任务不是泛泛而谈，也不是只写 prompt，而是基于这些文档，把我的业务需求转译成一个可实现的 agent runtime 设计。

你必须严格遵守以下要求：

1. 使用文档中的术语：
- QueryEngine
- Query Loop
- Tool
- Task
- Agent
- Skill
- Plugin
- Permission Context
- Resume

2. 不允许把 Agent 直接等同于 Prompt。
3. 不允许只给“做一个 agent”的空泛答案。
4. 必须先做需求拆解，再做 runtime 映射，再做实现设计。
5. 必须显式说明哪些部分借鉴 Claude Code，哪些部分需要按我们的业务改造。

请按以下结构输出：

1. 需求拆解
- 将业务需求拆成任务单元
- 标注每个任务单元的输入、输出、风险、时长、是否需要人工审批

2. Runtime 对象映射
- 哪些应实现为 Tool
- 哪些应实现为 Task
- 哪些应实现为 Agent
- 哪些应实现为 Skill
- 哪些应实现为 Plugin

3. Agent 设计
- 主控 agent 是否存在
- 需要哪些 worker / verifier / monitor
- 每个 agent 的 purpose、input、output、tool boundary、permission mode、execution mode、resume policy

4. Tool 设计
- 列出核心 tools
- 定义其输入输出、side effects、并发安全性、权限要求

5. Task 设计
- 列出核心 tasks
- 定义状态机、进度、输出、通知、恢复机制

6. Permission / Policy 设计
- 默认允许什么
- 需要审批什么
- 自动模式下必须剥离什么
- 组织策略如何施加

7. State / Resume 设计
- 哪些状态在会话中维护
- 哪些持久化
- 如何 resume

8. Frontend Shell 设计
- 用户从哪里触发
- 如何看到状态
- 如何继续、停止、审批

9. 实现计划
- 按 MVP / Phase 2 / Phase 3 输出
- 明确模块依赖顺序

最后请额外输出：
- 关键风险
- 架构取舍
- 最小可行版本建议
```

---

## 3. 标准总提示词：直接生成 Agent Specs

```text
你现在作为 Agent Spec Designer 工作。

请基于以下基础文档：
1. Agent Runtime 术语表
2. 业务需求到 Agent Runtime 的转译手册
3. Agent 设计模板与范式

针对我提供的业务需求，输出一组完整 Agent Specs。

要求：
1. 不要泛泛介绍，要直接输出 specs。
2. 每个 agent 必须明确：
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

3. 如果你认为不该用 agent，而应该用 tool / task / skill / plugin，也必须明确指出并说明理由。
4. 如果需要多 agent，请标明主控 agent 与 worker / verifier / monitor 之间的关系。
```

---

## 4. 标准总提示词：直接生成实现计划

```text
你现在作为 Agent Runtime Implementation Planner 工作。

请基于以下文档：
1. Agent Runtime 术语表
2. 业务需求到 Agent Runtime 的转译手册
3. Agent 设计模板与范式
4. Claude Code 架构全景与时序分析
5. Agent Runtime 实施路线图

你的任务是针对我的业务需求，输出一个可执行 implementation plan。

要求：
- 必须分层描述：
  - Frontend Shell
  - Conversation Orchestrator
  - QueryEngine
  - Query Loop
  - Tool Registry
  - Task Runtime
  - Permission / Policy
  - Resume / Memory
- 每一层都要说明：
  - 为什么需要
  - 做什么
  - 与其他层如何连接
  - MVP 怎么做
  - 后续怎么增强
- 最后输出：
  - 依赖顺序
  - 风险点
  - 测试与验证策略
```

---

## 5. 标准总提示词：让 AI 先提炼需求再设计

适合需求尚不清晰时使用。

```text
你现在作为 Agent Product Architect 工作。

我会给你以下基础文档：
1. Agent Runtime 术语表
2. 业务需求到 Agent Runtime 的转译手册
3. Agent 设计模板与范式

在你开始设计 runtime 之前，请先做需求澄清，但不要泛泛追问。

你必须：
1. 先总结你目前对业务目标的理解
2. 列出你认为还缺失的关键设计信息
3. 只提出最关键的 5 个以内问题
4. 每个问题都必须说明：这个信息会影响 Tool / Task / Agent / Permission / Resume 中的哪一项设计

当我回答后，你再按 runtime 设计结构输出方案。
```

---

## 6. 高质量输出判定规则

以后判断 AI 输出是否合格，可以用这组规则：

### 合格输出必须满足

1. 先拆需求，再做设计
2. 明确 runtime 对象映射
3. 明确 agent 边界
4. 明确 permission 边界
5. 明确 resume 语义
6. 明确实现顺序

### 不合格输出的典型特征

- 只讲“做一个智能 agent”
- 只有 prompt 没有 runtime
- 没有 tool / task 边界
- 没有审批和权限设计
- 没有恢复机制
- 没有分阶段实施计划

---

## 7. 最推荐的使用模式

### 模式一：研究阶段

输入：

- [[Claude Code 架构全景与时序分析]]
- [[Agent Runtime 术语表]]

目标：

- 理解 runtime 思维

### 模式二：设计阶段

输入：

- [[Agent Runtime 术语表]]
- [[业务需求到 Agent Runtime 的转译手册]]
- [[Agent 设计模板与范式]]
- 当前文档

目标：

- 产出完整系统设计

### 模式三：实施阶段

输入：

- 所有文档
- 你的具体业务需求

目标：

- 输出实现路线图与模块级计划

---

## 8. 一句话版本

> 这份文档是把知识库“压缩成可直接投喂 AI 的操作入口”，让 AI 不再只会空谈 agent，而是能按统一 runtime 语言输出可实现的系统设计。
