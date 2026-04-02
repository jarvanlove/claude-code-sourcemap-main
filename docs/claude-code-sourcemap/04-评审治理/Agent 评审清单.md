# Agent 评审清单

> 返回入口：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]
>
> 关联文档：
> - [[Agent Runtime 术语表]]
> - [[业务需求到 Agent Runtime 的转译手册]]
> - [[Agent 设计模板与范式]]
> - [[业务场景 Agent 蓝图库]]
> - [[给 AI 的标准总提示词]]
> - [[业务域专用 Tool 模板库]]
> - [[Prompt ／ Policy ／ Tool 三层边界说明]]

## 文档定位

这份文档的目标，是给你一套稳定的 `review checklist`，用于判断 AI 设计出来的 agent 系统是否合格。  
它适用于：

- 评审 AI 给出的架构方案
- 评审 AI 产出的 Agent Specs
- 评审实现前方案质量

---

## 1. 使用方式

对每个方案，从以下 8 个维度逐项评审：

1. 需求理解
2. Runtime 对象映射
3. Agent 边界
4. Tool 设计
5. Task / 生命周期
6. Permission / Policy
7. Resume / State
8. 实施可行性

建议评分方式：

- `通过`
- `有风险`
- `不通过`

---

## 2. 需求理解清单

### 必查项

- 是否明确了用户目标？
- 是否明确了输入输出？
- 是否明确了成功标准？
- 是否识别了高风险动作？
- 是否识别了长任务？
- 是否识别了人工审批点？

### 一票否决项

- 方案没有先拆需求，直接设计 agent
- 没有明确输入输出

---

## 3. Runtime 对象映射清单

### 必查项

- 是否区分了 Tool / Task / Agent / Skill / Plugin？
- 是否解释了为什么某能力要建模成 Tool 而不是 Agent？
- 是否解释了为什么某流程需要 Task Runtime？
- 是否避免把所有东西都塞给一个大 Agent？

### 一票否决项

- 把所有能力都说成“一个 agent 处理”
- 完全没有 Task 概念

---

## 4. Agent 边界清单

### 必查项

- 每个 agent 的 purpose 是否清晰？
- 输入输出是否清晰？
- 工具边界是否清晰？
- 是否区分主控、研究、执行、验证角色？
- 是否说明 foreground/background/remote？
- 是否有 failure handling？
- 是否有 verification policy？

### 高风险信号

- 一个 agent 同时负责研究、执行、验证
- 一个 worker 被赋予无限工具
- 没有 verifier，但任务却高风险

### 一票否决项

- 只有 prompt，没有 agent spec

---

## 5. Tool 设计清单

### 必查项

- 工具是否有明确输入输出 schema？
- 是否标明副作用？
- 是否标明并发安全性？
- 是否标明权限要求？
- 错误是否结构化？

### 高风险信号

- shell / exec 没有任何额外约束
- 写操作工具没有审批策略

### 一票否决项

- 没有 tool contract，只有“会调用外部接口”

---

## 6. Task / 生命周期清单

### 必查项

- 长任务是否建模成 Task？
- 是否定义 task state？
- 是否定义 progress？
- 是否定义 notification？
- 是否定义 output storage？
- 是否定义 cancel / stop 语义？

### 高风险信号

- 后台任务没有 task id
- 任务完成后没有结果回流路径

### 一票否决项

- 声称支持后台 agent，但没有 task runtime 设计

---

## 7. Permission / Policy 清单

### 必查项

- 是否定义默认允许与禁止？
- 是否定义审批点？
- 是否定义 auto mode stripping？
- 是否区分 user / project / org policy？

### 高风险信号

- 任意代码执行默认放开
- agent 可无限 spawn agent
- 网络与外部 side effect 无约束

### 一票否决项

- 高风险系统没有 permission design

---

## 8. Resume / State 清单

### 必查项

- 是否定义 transcript？
- 是否定义哪些状态要持久化？
- 是否定义 resume 语义？
- 是否说明 task metadata 如何恢复？

### 高风险信号

- 只说“保存聊天记录”就当 resume
- 后台任务无恢复元数据

### 一票否决项

- 长流程系统完全没有恢复设计

---

## 9. 实施可行性清单

### 必查项

- 是否分阶段实施？
- 是否有 MVP？
- 是否说明依赖顺序？
- 是否识别主要风险？
- 是否给出验证方式？

### 高风险信号

- 一开始就追求完整功能
- 没有区分 MVP 和后续阶段

### 一票否决项

- 没有实施顺序，只是概念设计

---

## 10. 评分模板

你以后可以直接让 AI 按这个模板评审另一个 AI 的设计。

```text
请基于附带的 Agent 评审清单，对以下 agent runtime 设计做评审。

按以下结构输出：

1. 总体结论
- 通过 / 有风险 / 不通过

2. 分项评审
- 需求理解
- Runtime 对象映射
- Agent 边界
- Tool 设计
- Task / 生命周期
- Permission / Policy
- Resume / State
- 实施可行性

3. 关键问题
- 列出最严重的 5 个问题

4. 修正建议
- 给出最小修正路径
```

---

## 11. 最常见的不合格方案画像

### 画像一：大一统 Agent

表现：

- 所有事情一个 agent 做
- 没有 task
- 没有 verifier

### 画像二：Prompt 伪装成架构

表现：

- 只有 prompt 设计
- 没有 runtime 对象
- 没有权限和恢复

### 画像三：功能很全但不可实现

表现：

- 一次性堆太多能力
- 没有 MVP
- 没有依赖顺序

### 画像四：有 agent，但没有治理

表现：

- 能 spawn workers
- 但无 permission / audit / resume / verification

---

## 12. 一句话版本

> 这份清单的作用，是把“AI 说得像回事”与“方案真的可落地”区分开来。
