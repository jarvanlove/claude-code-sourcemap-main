# claude-code-sourcemap-main 内容地图 MOC

> 主入口：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]

## 说明

这是一张面向 Obsidian 阅读的 `Map of Content`。  
如果你不想按目录看文件，而想按“理解 -> 设计 -> 实施 -> 评审”的认知顺序阅读，就从这里进入。

---

## 一、理解层

先建立对 Claude Code 与 Agent Runtime 的整体认识：

- [[Claude Code 架构全景与时序分析]]
- [[Agent Runtime 术语表]]
- [[Prompt ／ Policy ／ Tool 三层边界说明]]

这一层解决的问题：

- Claude Code 是什么
- Agent Runtime 的核心对象有哪些
- Prompt、Policy、Tool 三层如何分离

---

## 二、设计层

再把业务需求转译成系统设计：

- [[业务需求到 Agent Runtime 的转译手册]]
- [[Agent 设计模板与范式]]
- [[业务场景 Agent 蓝图库]]
- [[业务域专用 Tool 模板库]]

这一层解决的问题：

- 需求如何转成 Tool / Task / Agent / Skill / Plugin
- Agent 应如何写 spec
- 不同行业场景下有哪些成熟蓝图
- 工具层如何快速模板化

---

## 三、实施层

确定设计后，再进入实现与落地：

- [[Agent Runtime 实施路线图]]
- [[给 AI 的标准总提示词]]
- [[业务需求输入模板]]

这一层解决的问题：

- 从 MVP 到 Phase 3 怎么实施
- 如何直接给 AI 高质量上下文
- 业务需求应该按什么格式整理后喂给 AI

---

## 四、评审层

在设计或实现前后，用这些文档做质量控制：

- [[Agent 评审清单]]
- [[Prompt ／ Policy ／ Tool 三层边界说明]]

这一层解决的问题：

- AI 给出的方案是否真能落地
- 是否把 prompt、权限和工具边界混淆了

---

## 五、推荐阅读路线

### 路线 A：第一次完整阅读

1. [[Claude Code 架构全景与时序分析]]
2. [[Agent Runtime 术语表]]
3. [[Prompt ／ Policy ／ Tool 三层边界说明]]
4. [[业务需求到 Agent Runtime 的转译手册]]
5. [[Agent 设计模板与范式]]
6. [[业务场景 Agent 蓝图库]]
7. [[业务域专用 Tool 模板库]]
8. [[Agent Runtime 实施路线图]]
9. [[给 AI 的标准总提示词]]
10. [[Agent 评审清单]]
11. [[业务需求输入模板]]

### 路线 B：直接拿来做产品

1. [[Agent Runtime 术语表]]
2. [[业务需求输入模板]]
3. [[业务需求到 Agent Runtime 的转译手册]]
4. [[Agent 设计模板与范式]]
5. [[业务场景 Agent 蓝图库]]
6. [[业务域专用 Tool 模板库]]
7. [[给 AI 的标准总提示词]]
8. [[Agent Runtime 实施路线图]]
9. [[Agent 评审清单]]

---

## 六、快速入口

- 想理解 Claude Code：[[Claude Code 架构全景与时序分析]]
- 想统一术语：[[Agent Runtime 术语表]]
- 想开始设计：[[业务需求到 Agent Runtime 的转译手册]]
- 想直接写 Agent Spec：[[Agent 设计模板与范式]]
- 想套场景蓝图：[[业务场景 Agent 蓝图库]]
- 想补工具层模板：[[业务域专用 Tool 模板库]]
- 想直接喂给 AI：[[给 AI 的标准总提示词]]
- 想做实施计划：[[Agent Runtime 实施路线图]]
- 想审方案：[[Agent 评审清单]]
- 想开始填你的需求：[[业务需求输入模板]]
