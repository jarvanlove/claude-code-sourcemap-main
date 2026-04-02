# 业务域专用 Tool 模板库

> 返回入口：[[记忆库/语义记忆/claude-code-sourcemap-main/README|README]]
>
> 关联文档：
> - [[Agent Runtime 术语表]]
> - [[业务需求到 Agent Runtime 的转译手册]]
> - [[Agent 设计模板与范式]]
> - [[业务场景 Agent 蓝图库]]
> - [[Prompt / Policy / Tool 三层边界说明]]

## 文档定位

这份文档的目标，是为不同业务域提供一组可直接复用的 `Tool Spec 模板`。  
它不是要替代具体实现，而是让 AI 在接到某类业务需求时，能快速生成更稳的工具层设计，而不是每次临时发明接口。

---

## 1. 使用原则

### 1.1 Tool 模板的作用

Tool 模板主要帮助你解决三件事：

1. 定义输入输出 schema
2. 定义 side effects 与权限边界
3. 定义错误模型与并发安全性

### 1.2 不要把模板当成最终接口

模板是起点，不是终点。  
正式落地时还需要结合：

- 你的业务系统
- 组织策略
- 数据模型
- 审计要求

---

## 2. 通用 Tool Spec 模板

```yaml
tool:
  name: <string>
  purpose: <一句话说明用途>
  domain: <coding|research|sales|support|ops|content|knowledge>
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
      - <副作用说明>

  concurrency:
    is_concurrency_safe: <true|false>

  permission:
    approval_required: <true|false>
    policy_tags:
      - <filesystem|network|crm|payment|prod|messaging>

  observability:
    log_inputs: <full|partial|redacted>
    log_outputs: <full|partial|redacted>
    emit_audit_event: <true|false>

  errors:
    structured_errors:
      - code: <string>
        meaning: <string>
```

---

## 3. Coding Domain Tools

### 3.1 文件读取工具

```yaml
tool:
  name: file_read
  purpose: 读取指定文件内容
  domain: coding
  category: read
  input_schema:
    type: object
    properties:
      path: { type: string }
      start_line: { type: integer }
      end_line: { type: integer }
    required: [path]
  output_schema:
    type: object
    properties:
      content: { type: string }
      path: { type: string }
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [filesystem]
```

### 3.2 文件修改工具

```yaml
tool:
  name: file_edit
  purpose: 修改文件内容
  domain: coding
  category: write
  input_schema:
    type: object
    properties:
      path: { type: string }
      patch: { type: string }
    required: [path, patch]
  output_schema:
    type: object
    properties:
      updated: { type: boolean }
      path: { type: string }
  side_effects:
    has_side_effects: true
    description:
      - 修改工作区文件
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [filesystem]
```

### 3.3 Shell 执行工具

```yaml
tool:
  name: shell_exec
  purpose: 执行 shell 命令
  domain: coding
  category: write
  input_schema:
    type: object
    properties:
      command: { type: string }
      cwd: { type: string }
      timeout_ms: { type: integer }
    required: [command]
  output_schema:
    type: object
    properties:
      stdout: { type: string }
      stderr: { type: string }
      exit_code: { type: integer }
  side_effects:
    has_side_effects: true
    description:
      - 可能修改文件
      - 可能调用外部程序
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [exec, filesystem]
```

---

## 4. Research Domain Tools

### 4.1 Web Search 工具

```yaml
tool:
  name: web_search
  purpose: 根据查询词检索网页结果
  domain: research
  category: search
  input_schema:
    type: object
    properties:
      query: { type: string }
      domains: { type: array, items: { type: string } }
      recency_days: { type: integer }
    required: [query]
  output_schema:
    type: object
    properties:
      results:
        type: array
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [network]
```

### 4.2 页面抓取工具

```yaml
tool:
  name: web_fetch
  purpose: 获取指定网页正文内容
  domain: research
  category: read
  input_schema:
    type: object
    properties:
      url: { type: string }
    required: [url]
  output_schema:
    type: object
    properties:
      title: { type: string }
      content: { type: string }
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [network]
```

---

## 5. Sales Domain Tools

### 5.1 CRM 读取工具

```yaml
tool:
  name: crm_read
  purpose: 查询客户、线索、活动记录
  domain: sales
  category: read
  input_schema:
    type: object
    properties:
      entity_type: { type: string }
      entity_id: { type: string }
      filters: { type: object }
    required: [entity_type]
  output_schema:
    type: object
    properties:
      records: { type: array }
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [crm]
```

### 5.2 CRM 更新工具

```yaml
tool:
  name: crm_update
  purpose: 更新客户或线索状态
  domain: sales
  category: write
  input_schema:
    type: object
    properties:
      entity_type: { type: string }
      entity_id: { type: string }
      updates: { type: object }
    required: [entity_type, entity_id, updates]
  output_schema:
    type: object
    properties:
      success: { type: boolean }
  side_effects:
    has_side_effects: true
    description:
      - 修改 CRM 数据
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [crm]
```

### 5.3 外发消息工具

```yaml
tool:
  name: outbound_message_send
  purpose: 发送邮件或站内消息给客户
  domain: sales
  category: write
  input_schema:
    type: object
    properties:
      channel: { type: string }
      recipient_id: { type: string }
      content: { type: string }
    required: [channel, recipient_id, content]
  output_schema:
    type: object
    properties:
      message_id: { type: string }
      status: { type: string }
  side_effects:
    has_side_effects: true
    description:
      - 对外发送消息
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [messaging, external]
```

---

## 6. Support Domain Tools

### 6.1 工单查询工具

```yaml
tool:
  name: ticket_read
  purpose: 查询工单详情和历史
  domain: support
  category: read
  input_schema:
    type: object
    properties:
      ticket_id: { type: string }
    required: [ticket_id]
  output_schema:
    type: object
    properties:
      ticket: { type: object }
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [support]
```

### 6.2 升级人工工具

```yaml
tool:
  name: escalation_create
  purpose: 创建人工升级任务
  domain: support
  category: meta
  input_schema:
    type: object
    properties:
      ticket_id: { type: string }
      reason: { type: string }
      priority: { type: string }
    required: [ticket_id, reason]
  output_schema:
    type: object
    properties:
      escalation_id: { type: string }
  side_effects:
    has_side_effects: true
    description:
      - 创建人工处理流程
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: false
    policy_tags: [support, workflow]
```

---

## 7. Ops / DevOps Domain Tools

### 7.1 日志检索工具

```yaml
tool:
  name: log_search
  purpose: 查询日志
  domain: ops
  category: search
  input_schema:
    type: object
    properties:
      service: { type: string }
      query: { type: string }
      time_range: { type: object }
    required: [service, query]
  output_schema:
    type: object
    properties:
      hits: { type: array }
  side_effects:
    has_side_effects: false
  concurrency:
    is_concurrency_safe: true
  permission:
    approval_required: false
    policy_tags: [infra, logs]
```

### 7.2 生产环境执行工具

```yaml
tool:
  name: prod_exec
  purpose: 对生产基础设施执行操作
  domain: ops
  category: write
  input_schema:
    type: object
    properties:
      command: { type: string }
      target: { type: string }
    required: [command, target]
  output_schema:
    type: object
    properties:
      status: { type: string }
      output: { type: string }
  side_effects:
    has_side_effects: true
    description:
      - 修改生产环境状态
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [prod, exec, infra]
```

---

## 8. Content Domain Tools

### 8.1 草稿发布工具

```yaml
tool:
  name: content_publish
  purpose: 将草稿发布到目标渠道
  domain: content
  category: write
  input_schema:
    type: object
    properties:
      destination: { type: string }
      content_id: { type: string }
    required: [destination, content_id]
  output_schema:
    type: object
    properties:
      publication_id: { type: string }
  side_effects:
    has_side_effects: true
    description:
      - 对外发布内容
  concurrency:
    is_concurrency_safe: false
  permission:
    approval_required: true
    policy_tags: [publishing, external]
```

---

## 9. Tool 评审速查表

以后 AI 设计一个 Tool 时，至少检查：

- 是否有明确输入 schema
- 是否有明确输出 schema
- 是否描述 side effects
- 是否声明并发安全
- 是否声明 permission
- 是否有结构化错误

---

## 10. 给 AI 的使用模板

```text
请基于附带的业务域专用 Tool 模板库、术语表、需求转译手册和 Agent 模板文档，针对以下业务需求输出一组 Tool Specs。

要求：
1. 先判断属于哪个业务域
2. 优先复用相近模板
3. 输出最终工具时，必须包含：
- name
- purpose
- category
- input_schema
- output_schema
- side_effects
- concurrency
- permission
- errors
4. 如果某能力不应该是 Tool，而应该是 Task / Agent / Skill，也必须指出
```

---

## 11. 一句话版本

> 这份模板库的作用，是让 AI 在设计工具层时不再临场发挥，而是沿着业务域已有范式快速产出稳定的 Tool Specs。
