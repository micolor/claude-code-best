# Claude Code Subagents 官方完整指南

> 基于官方文档 (https://code.claude.com/docs/en/sub-agents) 的完整参考

---

## 目录

- [什么是 Subagents](#什么是-subagents)
- [内置 Subagents](#内置-subagents)
- [创建第一个 Subagent](#创建第一个-subagent)
- [配置 Subagents](#配置-subagents)
- [使用 Subagents](#使用-subagents)
- [示例 Subagents](#示例-subagents)
- [最佳实践](#最佳实践)

---

## 什么是 Subagents

**Subagents** 是专门的 AI 助手，处理特定类型的任务。

### 何时使用 Subagent

当某个侧任务会向主对话淹没大量搜索结果、日志或文件内容，而你不会再引用这些内容时，使用 subagent：

- Subagent 在它自己的上下文中完成工作
- 只返回摘要到主对话

### 核心优势

| 优势 | 说明 |
|------|------|
| **保留上下文** | 将探索和实现隔离在独立上下文中 |
| **强制约束** | 限制 subagent 可使用的工具 |
| **复用配置** | 跨项目复用 subagent 配置 |
| **专业化行为** | 针对特定领域的专注系统提示 |
| **控制成本** | 将任务路由到更快、更便宜的模型（如 Haiku） |

### 工作原理

每个 subagent 运行在：
- 独立的上下文窗口
- 自定义系统提示
- 特定工具访问
- 独立权限

当 Claude 遇到匹配 subagent 描述的任务时，会委托给该 subagent，subagent 独立工作并返回结果。

<Note>
如果需要多个 agent 并行工作并相互通信，见 [Agent Teams](/en/agent-teams)。Subagents 在单个会话内工作；Agent Teams 跨独立会话协调。
</Note>

---

## 内置 Subagents

Claude Code 包含几个内置 subagents，Claude 会在适当时自动使用。

### Explore

快速、只读的 agent，优化用于搜索和分析代码库。

| 属性 | 值 |
|------|------|
| **模型** | Haiku（快速、低延迟） |
| **工具** | 只读工具（拒绝 Write 和 Edit） |
| **用途** | 文件发现、代码搜索、代码库探索 |

**使用场景**：Claude 需要搜索或理解代码库而不做修改时。

**彻底程度**：`quick`（ targeted 查找）、`medium`（平衡探索）、`very thorough`（全面分析）。

---

### Plan

在 [Plan 模式](/en/common-workflows#use-plan-mode-for-safe-code-analysis) 中使用的研究 agent，在呈现计划之前收集上下文。

| 属性 | 值 |
|------|------|
| **模型** | 继承自主对话 |
| **工具** | 只读工具 |
| **用途** | 用于计划的代码库研究 |

**使用场景**：在 Plan 模式中，Claude 需要理解代码库时。

---

### General-purpose

用于需要探索和行动的复杂多步骤任务的强大 agent。

| 属性 | 值 |
|------|------|
| **模型** | 继承自主对话 |
| **工具** | 所有工具 |
| **用途** | 复杂研究、多步骤操作、代码修改 |

**使用场景**：任务需要探索和修改、复杂推理或多步依赖步骤时。

---

### Other

其他辅助 agent：

| Agent | 模型 | 使用场景 |
|-------|------|----------|
| `statusline-setup` | Sonnet | 运行 `/statusline` 配置状态行时 |
| `Claude Code Guide` | Haiku | 询问 Claude Code 功能问题时 |

---

## 创建第一个 Subagent

Subagents 是使用 YAML frontmatter 定义的 Markdown 文件。可以手动创建或使用 `/agents` 命令。

### 使用 `/agents` 命令创建

**步骤 1：打开 subagents 界面**

```text
/agents
```

**步骤 2：选择位置**

- 切换到 **Library** 标签
- 选择 **Create new agent**
- 选择 **Personal**（保存到 `~/.claude/agents/`，所有项目可用）

**步骤 3：使用 Claude 生成**

选择 **Generate with Claude**，描述 subagent：

```text
A code improvement agent that scans files and suggests improvements
for readability, performance, and best practices.
```

**步骤 4：选择工具**

对于只读审查，只选择 **Read-only tools**。

**步骤 5：选择模型**

选择 **Sonnet**，平衡能力和速度。

**步骤 6：选择颜色**

挑选背景色，便于在 UI 中识别。

**步骤 7：配置内存**

选择 **User scope** 给予持久内存目录 `~/.claude/agent-memory/`。

**步骤 8：保存并测试**

```text
Use the code-improver agent to suggest improvements in this project
```

---

## 配置 Subagents

### Subagent 作用域

| 位置 | 作用域 | 优先级 | 创建方式 |
|------|--------|--------|----------|
| Managed settings | 组织范围 | 1（最高） | 通过 [managed settings](/en/settings) 部署 |
| `--agents` CLI 标志 | 当前会话 | 2 | 启动时传递 JSON |
| `.claude/agents/` | 当前项目 | 3 | 交互式或手动 |
| `~/.claude/agents/` | 所有项目 | 4 | 交互式或手动 |
| Plugin 的 `agents/` | 启用插件的地方 | 5（最低） | 随插件安装 |

**优先级规则**：高优先级位置的配置覆盖同名的低优先级配置。

### 写入 Subagent 文件

```markdown
---
name: code-reviewer
description: 审查代码质量和最佳实践
tools: Read, Glob, Grep
model: sonnet
---

你是代码审查员。当被调用时，分析代码并提供关于质量、安全和最佳实践的具体、可操作的反馈。
```

**注意**：Subagents 在会话启动时加载。如果手动添加文件，需要重启会话或使用 `/agents` 立即加载。

---

### Frontmatter 字段参考

| 字段 | 必需 | 描述 |
|------|------|------|
| `name` | 是 | 唯一标识符，使用小写字母和连字符 |
| `description` | 是 | Claude 何时应该委托给此 subagent |
| `tools` | 否 | subagent 可使用的工具。省略则继承所有工具 |
| `disallowedTools` | 否 | 要拒绝的工具，从继承或指定列表中移除 |
| `model` | 否 | 使用的模型：`sonnet`、`opus`、`haiku`、完整模型 ID 或 `inherit` |
| `permissionMode` | 否 | 权限模式：`default`、`acceptEdits`、`auto`、`dontAsk`、`bypassPermissions`、`plan` |
| `maxTurns` | 否 | subagent 停止前的最大 agentic 轮数 |
| `skills` | 否 | 启动时加载到 subagent 上下文的 skills |
| `mcpServers` | 否 | subagent 可用的 MCP 服务器 |
| `hooks` | 否 | 绑定到此 subagent 生命周期的钩子 |
| `memory` | 否 | 持久内存范围：`user`、`project`、`local` |
| `background` | 否 | 设为 `true` 始终作为后台任务运行 |
| `effort` | 否 | subagent 激活时的努力程度 |
| `isolation` | 否 | 设为 `worktree` 在临时 git worktree 中运行 |
| `color` | 否 | subagent 的显示颜色：`red`、`blue`、`green` 等 |
| `initialPrompt` | 否 | 当此 agent 作为主会话 agent 运行时自动提交为第一个用户回合 |

---

### 选择模型

`model` 字段控制 subagent 使用的 AI 模型：

| 选项 | 说明 |
|------|------|
| `sonnet` | Sonnet 模型别名 |
| `opus` | Opus 模型别名 |
| `haiku` | Haiku 模型别名 |
| `claude-opus-4-7` | 完整模型 ID |
| `inherit` | 使用与主对话相同的模型（默认） |

**解析顺序**：
1. `CLAUDE_CODE_SUBAGENT_MODEL` 环境变量（如果设置）
2. 每次调用的 `model` 参数
3. subagent 定义的 `model` frontmatter
4. 主对话的模型

---

### 控制 Subagent 能力

#### 可用工具

Subagents 可以使用 Claude Code 的任何内部工具。默认继承主对话的所有工具。

**使用 `tools` 限制（允许列表）**：

```yaml
---
name: safe-researcher
description: 能力受限的研究 agent
tools: Read, Grep, Glob, Bash
---
```

**使用 `disallowedTools` 限制（拒绝列表）**：

```yaml
---
name: no-writes
description: 继承所有工具除了文件写入
disallowedTools: Write, Edit
---
```

如果两者都设置，先应用 `disallowedTools`，然后 `tools` 从剩余池中解析。

#### 限制可派生的 Subagents

当 agent 作为主线程运行时，可以限制它能派生的 subagent 类型：

```yaml
---
name: coordinator
description: 跨专业 agent 协调工作
tools: Agent(worker, researcher), Read, Bash
---
```

只允许派生 `worker` 和 `researcher` subagents。

#### 范围化 MCP 服务器

```yaml
---
name: browser-tester
description: 使用 Playwright 在真实浏览器中测试
mcpServers:
  # 内联定义：仅作用于此 subagent
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  # 引用名称：重用已配置的服务器
  - github
---
```

---

### 权限模式

| 模式 | 行为 |
|------|------|
| `default` | 标准权限检查带提示 |
| `acceptEdits` | 自动接受工作目录中路径的文件编辑 |
| `auto` | Auto 模式：后台分类器审查命令 |
| `dontAsk` | 自动拒绝权限提示 |
| `bypassPermissions` | 跳过权限提示 |
| `plan` | Plan 模式（只读探索） |

<Warning>
谨慎使用 `bypassPermissions`。它跳过权限提示，允许 subagent 执行未经批准的操作。
</Warning>

---

### 预加载 Skills

```yaml
---
name: api-developer
description: 遵循团队约定实现 API 端点
skills:
  - api-conventions
  - error-handling-patterns
---
```

每个 skill 的完整内容注入到 subagent 的上下文中，不只是可供调用。

---

### 启用持久内存

```yaml
---
name: code-reviewer
description: 审查代码质量和最佳实践
memory: user
---
```

| 范围 | 位置 | 使用场景 |
|------|------|----------|
| `user` | `~/.claude/agent-memory/<name-of-agent>/` | subagent 应跨所有项目记住学习 |
| `project` | `.claude/agent-memory/<name-of-agent>/` | subagent 知识是项目特定的，可通过版本控制共享 |
| `local` | `.claude/agent-memory-local/<name-of-agent>/` | subagent 知识是项目特定的，但不应检入版本控制 |

---

### 条件规则与 Hooks

```yaml
---
name: db-reader
description: 执行只读数据库查询
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
```

**验证脚本**：

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# 阻止 SQL 写入操作（不区分大小写）
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\b' > /dev/null; then
  echo "Blocked: Only SELECT queries are allowed" >&2
  exit 2
fi

exit 0
```

---

### 禁用特定 Subagents

在 settings.json 中添加：

```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

或使用 CLI 标志：

```bash
claude --disallowedTools "Agent(Explore)"
```

---

## 使用 Subagents

### 理解自动委托

Claude 基于以下因素自动委托任务：
- 请求中的任务描述
- subagent 配置中的 `description` 字段
- 当前上下文

在 subagent 的 `description` 中包含 "use proactively" 等短语可鼓励主动委托。

---

### 显式调用 Subagents

#### 模式 1：自然语言

```text
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

#### 模式 2：@-mention

```text
@"code-reviewer (agent)" look at the auth changes
```

这会确保特定 subagent 运行，而不是让 Claude 选择。

#### 模式 3：会话范围

```bash
claude --agent code-reviewer
```

这会使用 subagent 的系统提示、工具限制和模型替换默认的 Claude Code 系统提示。

在项目级别设置默认：

```json
{
  "agent": "code-reviewer"
}
```

---

### 前台 vs 后台运行

| 类型 | 行为 |
|------|------|
| **前台** | 阻塞主对话直到完成，传递权限提示和澄清问题 |
| **后台** | 并发运行，继续工作，预先提示所需权限 |

**后台运行**：
- Claude 决定何时后台运行 subagent
- 可以要求 "run this in the background"
- 按 **Ctrl+B** 后台运行任务

**禁用后台任务**：

```bash
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1
```

---

### 常见模式

#### 隔离高容量操作

```text
Use a subagent to run the test suite and report only the failing tests
```

将详细输出保留在 subagent 的上下文中，只返回相关摘要。

#### 并行研究

```text
Research the authentication, database, and API modules in parallel using separate subagents
```

每个 subagent 独立探索其领域，然后 Claude 综合发现。

#### 链式 Subagents

```text
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

每个 subagent 完成任务并返回结果，Claude 将相关上下文传递给下一个 subagent。

---

### 恢复 Subagents

```text
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full context]
```

恢复的 subagents 保留完整的对话历史，包括所有先前的工具调用、结果和推理。

**查找 Agent ID**：
- 询问 Claude 获取 agent ID
- 或在记录文件中查找：`~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`

---

## 示例 Subagents

### 代码审查员

```markdown
---
name: code-reviewer
description: 专业代码审查专家。在编写或修改代码后立即主动审查代码质量、安全性和可维护性。
tools: Read, Grep, Glob, Bash
model: inherit
---

你是确保高质量代码和安全的高级代码审查员。

被调用时：
1. 运行 git diff 查看最近变更
2. 关注修改的文件
3. 立即开始审查

审查清单：
- 代码清晰可读
- 函数和变量命名良好
- 无重复代码
- 适当的错误处理
- 无暴露的秘密或 API 密钥
- 实现输入验证
- 良好的测试覆盖率
- 解决性能考虑

按优先级组织反馈：
- 严重问题（必须修复）
- 警告（应该修复）
- 建议（考虑改进）

包含如何修复问题的具体示例。
```

---

### 调试器

```markdown
---
name: debugger
description: 调试专家，用于错误、测试失败和意外行为。遇到任何问题时主动使用。
tools: Read, Edit, Bash, Grep, Glob
---

你是专注于根因分析的调试专家。

被调用时：
1. 捕获错误消息和堆栈跟踪
2. 识别复现步骤
3. 隔离失败位置
4. 实施最小修复
5. 验证解决方案有效

调试流程：
- 分析错误消息和日志
- 检查最近的代码变更
- 形成和测试假设
- 添加战略调试日志
- 检查变量状态

对于每个问题，提供：
- 根因解释
- 支持诊断的证据
- 具体代码修复
- 测试方法
- 预防建议

专注于修复根本问题，而非症状。
```

---

### 数据科学家

```markdown
---
name: data-scientist
description: 数据分析专家，用于 SQL 查询、BigQuery 操作和数据洞察。主动用于数据分析任务和查询。
tools: Bash, Read, Write
model: sonnet
---

你是专注于 SQL 和 BigQuery 分析的数据科学家。

被调用时：
1. 理解数据分析需求
2. 编写高效 SQL 查询
3. 适当时使用 BigQuery 命令行工具 (bq)
4. 分析和总结结果
5. 清晰呈现发现

关键实践：
- 编写优化的 SQL 查询，带适当过滤器
- 使用适当的聚合和连接
- 包含解释复杂逻辑的注释
- 格式化结果便于阅读
- 提供数据驱动的建议

对于每次分析：
- 解释查询方法
- 记录任何假设
- 突出关键发现
- 基于数据建议下一步

始终确保查询高效且成本效益。
```

---

### 数据库查询验证器

```markdown
---
name: db-reader
description: 执行只读数据库查询。用于分析数据或生成报告时。
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

你是只有只读访问权限的数据库分析师。执行 SELECT 查询来回答关于数据的问题。

被要求分析数据时：
1. 识别哪些表包含相关数据
2. 编写高效的 SELECT 查询，带适当过滤器
3. 清晰呈现结果和上下文

你不能修改数据。如果要求 INSERT、UPDATE、DELETE 或修改 schema，解释你只有只读权限。
```

**验证脚本**：

```bash
#!/bin/bash
# 阻止 SQL 写入操作，允许 SELECT 查询

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

# 阻止写入操作（不区分大小写）
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2
fi

exit 0
```

---

## 最佳实践

### 设计 Subagents

| 原则 | 说明 |
|------|------|
| **专注** | 每个 subagent 应擅长一个特定任务 |
| **详细** | 编写详细的描述，Claude 用它来决定何时委托 |
| **限制工具** | 仅授予必要的权限以确保安全和专注 |
| **检入版本控制** | 与团队共享项目 subagents |

### 何时使用 Subagent

**使用 Subagent** 当：
- 任务产生你不需要在主上下文中的详细输出
- 你想强制特定的工具限制或权限
- 工作是自包含的，可以返回摘要

**使用主对话** 当：
- 任务需要频繁的来回或迭代优化
- 多个阶段共享重要上下文
- 你进行快速、有针对性的更改
- 延迟很重要

---

## 相关资源

- **[Distribute subagents with plugins](/en/plugins)** - 跨团队或项目共享 subagents
- **[Run Claude Code programmatically](/en/headless)** - 使用 Agent SDK 进行 CI/CD 和自动化
- **[Use MCP servers](/en/mcp)** - 给 subagents 访问外部工具和数据的能力
- **[Agent Teams](/en/agent-teams)** - 跨独立会话协调多个 agent

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-04-18 | 1.0.0 | 基于官方文档整理 |

---

*本文档基于官方文档 https://code.claude.com/docs/en/sub-agents 整理。*
