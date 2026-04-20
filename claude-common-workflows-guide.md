# Claude Code Common Workflows 常用工作流指南

> 基于官方文档 (https://code.claude.com/docs/en/common-workflows) 的实战学习笔记

---

## 目录

- [理解新代码库](#理解新代码库)
- [高效修复 Bug](#高效修复-bug)
- [重构代码](#重构代码)
- [使用专业 Subagents](#使用专业-subagents)
- [Plan 模式安全分析](#plan-模式安全分析)
- [测试工作流](#测试工作流)
- [创建 Pull Requests](#创建-pull-requests)
- [处理文档](#处理文档)
- [使用图片](#使用图片)
- [@ 引用文件和目录](#引用文件和目录)
- [扩展思维模式](#扩展思维模式)
- [恢复之前的对话](#恢复之前的对话)
- [Git Worktrees 并行工作](#git-worktrees-并行工作)
- [连接 MCP 工具](#连接-mcp-工具)
- [桌面通知](#桌面通知)
- [作为 Unix 工具使用](#作为-unix-工具使用)
- [定时任务](#定时任务)

---

## 理解新代码库

### 快速概览

```bash
# 1. 进入项目根目录
cd /path/to/project

# 2. 启动 Claude Code
claude

# 3. 询问高级概览
give me an overview of this codebase

# 4. 深入了解特定组件
explain the main architecture patterns used here
what are the key data models?
how is authentication handled?
```

**技巧**：
- 从广泛问题开始，然后缩小到特定领域
- 询问项目使用的编码约定和模式
- 请求项目特定术语的词表

### 查找相关代码

```bash
# 查找特定功能的代码
find the files that handle user authentication

# 获取组件如何交互的上下文
how do these authentication files work together?

# 理解执行流程
trace the login process from front-end to database
```

**技巧**：
- 具体说明你在找什么
- 使用项目中的领域语言
- 安装代码智能插件以获得精确的"跳转到定义"和"查找引用"功能

---

## 高效修复 Bug

```bash
# 1. 分享错误信息
I'm seeing an error when I run npm test

# 2. 请求修复建议
suggest a few ways to fix the @ts-ignore in user.ts

# 3. 应用修复
update user.ts to add the null check you suggested
```

**技巧**：
- 告诉 Claude 复现问题的命令并获取堆栈跟踪
- 提及任何复现步骤
- 让 Claude 知道错误是间歇性的还是一致的

---

## 重构代码

```bash
# 1. 识别需要重构的旧代码
find deprecated API usage in our codebase

# 2. 获取重构建议
suggest how to refactor utils.js to use modern JavaScript features

# 3. 安全应用变更
refactor utils.js to use ES2024 features while maintaining the same behavior

# 4. 验证重构
run tests for the refactored code
```

**技巧**：
- 让 Claude 解释现代方法的好处
- 请求在需要时保持向后兼容性
- 以小的、可测试的增量进行重构

---

## 使用专业 Subagents

```bash
# 1. 查看可用的 subagents
/agents

# 2. 自动使用 subagents
review my recent code changes for security issues
run all tests and fix any failures

# 3. 显式请求特定 subagent
use the code-reviewer subagent to check the auth module
have the debugger subagent investigate why users can't log in

# 4. 创建自定义 subagent
/agents
# 然后选择 "Create New subagent" 并按照提示定义
```

**创建自定义 Subagent 时定义**：
- 唯一标识符（如 `code-reviewer`、`api-designer`）
- Claude 何时应该使用此 agent
- 它可以访问哪些工具
- 描述 agent 角色和行为的系统提示

**技巧**：
- 在 `.claude/agents/` 中创建项目特定的 subagents 供团队共享
- 使用描述性的 `description` 字段启用自动委托
- 限制工具访问到每个 subagent 实际需要的

---

## Plan 模式安全分析

**Plan 模式** 指示 Claude 通过只读操作分析代码库来创建计划，最适合：
- 探索代码库
- 规划复杂变更
- 安全审查代码

### 何时使用 Plan 模式

| 场景 | 说明 |
|------|------|
| 多步骤实现 | 功能需要编辑许多文件 |
| 代码探索 | 想在更改前彻底研究代码库 |
| 交互式开发 | 想与 Claude 迭代方向 |

### 如何使用 Plan 模式

**在会话期间切换到 Plan 模式**

使用 **Shift+Tab** 循环切换权限模式：
1. Normal Mode → Shift+Tab → Auto-Accept Mode (`⏵⏵ accept edits on`)
2. Auto-Accept Mode → Shift+Tab → Plan Mode (`⏸ plan mode on`)

**在 Plan 模式中启动新会话**

```bash
claude --permission-mode plan
```

**在 Plan 模式中运行"headless"查询**

```bash
claude --permission-mode plan -p "Analyze the authentication system and suggest improvements"
```

### 示例：规划复杂重构

```bash
claude --permission-mode plan
```

```text
I need to refactor our authentication system to use OAuth2. Create a detailed migration plan.
```

Claude 分析当前实现并创建综合计划。通过后续对话优化：

```text
What about backward compatibility?
How should we handle database migration?
```

**技巧**：按 `Ctrl+G` 在默认文本编辑器中打开计划，在 Claude 继续之前直接编辑。

### 配置 Plan 模式为默认

```json
// .claude/settings.json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

---

## 测试工作流

```bash
# 1. 识别未测试的代码
find functions in NotificationsService.swift that are not covered by tests

# 2. 生成测试脚手架
add tests for the notification service

# 3. 添加有意义的测试用例
add test cases for edge conditions in the notification service

# 4. 运行并验证测试
run the new tests and fix any failures
```

Claude 可以生成遵循项目现有模式和约定的测试。请求测试时，具体说明你想验证的行为。

对于全面覆盖，让 Claude 识别你可能错过的边界情况。

---

## 创建 Pull Requests

```bash
# 1. 总结你的变更
summarize the changes I've made to the authentication module

# 2. 生成 PR
create a pr

# 3. 审查和优化
enhance the PR description with more context about the security improvements
```

**技巧**：
- 使用 `gh pr create` 创建 PR 时，会话会自动链接到该 PR
- 可以用 `claude --from-pr <number>` 恢复
- 提交前审查 Claude 生成的 PR
- 让 Claude 强调潜在风险或注意事项

---

## 处理文档

```bash
# 1. 识别未记录的代码
find functions without proper JSDoc comments in the auth module

# 2. 生成文档
add JSDoc comments to the undocumented functions in auth.js

# 3. 审查和增强
improve the generated documentation with more context and examples

# 4. 验证文档
check if the documentation follows our project standards
```

**技巧**：
- 指定想要的文档样式（JSDoc、docstrings 等）
- 请求文档中的示例
- 请求公共 API、接口和复杂逻辑的文档

---

## 使用图片

### 添加图片到对话

**方法**：
1. 拖拽图片到 Claude Code 窗口
2. 复制图片并粘贴到 CLI (`Ctrl+V`，Mac 不要用 `Cmd+V`)
3. 提供图片路径：`"Analyze this image: /path/to/your/image.png"`

### 分析图片

```text
What does this image show?
Describe the UI elements in this screenshot
Are there any problematic elements in this diagram?
```

### 使用图片作为上下文

```text
Here's a screenshot of the error. What's causing it?
This is our current database schema. How should we modify it for the new feature?
```

### 从视觉内容获取代码建议

```text
Generate CSS to match this design mockup
What HTML structure would recreate this component?
```

**技巧**：
- 当文本描述不清楚时使用图片
- 包含错误、UI 设计或图表的截图以获得更好的上下文
- 可以在对话中使用多张图片
- 当 Claude 引用图片（如 `[Image #1]`）时，`Cmd+Click`（Mac）或`Ctrl+Click`（Windows/Linux）在默认查看器中打开

---

## @ 引用文件和目录

### @ 引用语法

```bash
# 引用单个文件
Explain the logic in @src/utils/auth.js

# 引用目录
What's the structure of @src/components?

# 引用 MCP 资源
Show me the data from @github:repos/owner/repo/issues
```

**技巧**：
- 文件路径可以是相对路径或绝对路径
- @ 文件引用会将 `CLAUDE.md` 添加到上下文中
- 目录引用显示文件列表，而非内容
- 可以在单个消息中引用多个文件

---

## 扩展思维模式

**扩展思维** 默认启用，让 Claude 在响应之前逐步推理复杂问题。

### 配置思维模式

| 范围 | 如何配置 | 详情 |
|------|----------|------|
| **努力程度** | `/effort`、`/model` 或 `CLAUDE_CODE_EFFORT_LEVEL` | 控制思维深度 |
| **`ultrathink` 关键词** | 在提示中包含 "ultrathink" | 指示模型在该回合更多推理 |
| **切换快捷键** | `Option+T` (macOS) 或 `Alt+T` (Windows/Linux) | 为当前会话切换思维模式 |
| **全局默认** | `/config` 切换思维模式 | 保存在 `~/.claude/settings.json` 的 `alwaysThinkingEnabled` |
| **限制 Token 预算** | `MAX_THINKING_TOKENS` 环境变量 | 例如：`export MAX_THINKING_TOKENS=10000` |

### 查看思维过程

按 `Ctrl+O` 切换详细模式，查看内部推理（灰色斜体文本）。

### 工作原理

- **扩展思维** 控制 Claude 在响应之前执行的内部推理量
- 更多思维提供更多空间探索解决方案、分析边界情况、自我纠正错误
- 支持努力的模型使用**自适应推理**：根据选择的努力程度动态分配思维 token

**注意**：
- 短语如 "think"、"think hard" 被解释为普通提示指令，不分配思维 token
- 即使思维摘要被编辑，你也会被收取所有思维 token 的费用
- 在交互模式中，思维默认显示为折叠的存根
- 设置 `showThinkingSummaries: true` 显示完整摘要

---

## 恢复之前的对话

### 命令行恢复

```bash
# 继续当前目录中最近的对话
claude --continue

# 打开对话选择器或按名称恢复
claude --resume

# 恢复链接到特定 PR 的会话
claude --from-pr 123

# 按名称恢复
claude --resume auth-refactor
```

### 会话中恢复

```bash
# 在活跃会话中切换到不同对话
/resume

# 按名称恢复
/resume auth-refactor
```

### 命名会话

```bash
# 启动时命名
claude -n auth-refactor

# 会话期间重命名
/rename auth-refactor

# 从选择器重命名
/resume → 选择会话 → Ctrl+R
```

### 会话选择器快捷键

| 快捷键 | 操作 |
|--------|------|
| `↑` / `↓` | 在会话间导航 |
| `→` / `←` | 展开/折叠分组会话 |
| `Enter` | 选择并恢复高亮会话 |
| `Space` | 预览会话内容 |
| `Ctrl+R` | 重命名高亮会话 |
| `/` 或可打印字符 | 进入搜索模式并过滤会话 |
| `Ctrl+A` | 显示此机器上所有项目的会话 |
| `Ctrl+W` | 显示当前仓库所有 worktree 的会话 |
| `Ctrl+B` | 过滤到当前 git 分支的会话 |
| `Esc` | 退出选择器或搜索模式 |

**技巧**：
- **尽早命名会话**：使用 `/rename` 在开始不同任务时
- `--continue` 快速访问当前目录中最近的对话
- `--resume session-name` 当你知道需要哪个会话时
- `--resume`（无参数）当需要浏览和选择时

---

## Git Worktrees 并行工作

### 使用 Worktrees

```bash
# 在名为 "feature-auth" 的 worktree 中启动 Claude
claude --worktree feature-auth

# 自动命名（推荐）
claude --worktree
# 生成如 "bright-running-fox" 的名称

# 启动另一个会话到不同的 worktree
claude --worktree bugfix-123

# 在 worktree 会话中执行任务
claude --worktree feature-auth -p "实现登录功能"
```

### Worktree 特点

| 属性 | 说明 |
|------|------|
| **位置** | `<repo>/.claude/worktrees/<name>` |
| **分支** | 名为 `worktree-<name>` |
| **基础** | 从 `origin/HEAD` 指向的默认远程分支 |
| **隔离** | 每个 worktree 有独立的文件系统和 git 状态 |

### Subagent Worktrees

Subagents 也可以使用 worktree 隔离：

```yaml
# 在自定义 subagent 的 frontmatter 中配置
isolation: worktree
```

或要求 Claude：
```text
use worktrees for your agents
```

### Worktree 清理

| 情况 | 行为 |
|------|------|
| 无变更 | 自动移除 worktree 及其分支 |
| 有变更或提交 | 提示你保留或移除 |

**手动清理**：
```bash
# 查看现有 worktree
git worktree list

# 移除 worktree
git worktree remove ../project-feature-a
```

### 复制 Git 忽略的文件到 Worktrees

创建 `.worktreeinclude` 文件列出要复制的文件：

```text
.env
.env.local
config/secrets.json
```

只有被 git 忽略的文件才会被复制，跟踪的文件永远不会重复。

### 手动管理 Worktrees

```bash
# 使用新分支创建 worktree
git worktree add ../project-feature-a -b feature-a

# 使用现有分支创建
git worktree add ../project-bugfix bugfix-123

# 在 worktree 中启动 Claude
cd ../project-feature-a && claude

# 清理时
git worktree remove ../project-feature-a
```

---

## 连接 MCP 工具

MCP (Model Context Protocol) 允许 Claude Code 连接到外部工具和数据源。

### 添加 MCP 服务器

**方式 1：远程 HTTP 服务器**（推荐）

```bash
# 基本语法
claude mcp add --transport http <name> <url>

# 示例：连接 Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 带认证头
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

**方式 2：远程 SSE 服务器**（已弃用，优先使用 HTTP）

```bash
# 基本语法
claude mcp add --transport sse <name> <url>

# 示例：连接 Asana
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

**方式 3：本地 stdio 服务器**

```bash
# 基本语法
claude mcp add [options] <name> -- <command> [args...]

# 示例：添加 Airtable
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

**重要**：所有选项 (`--transport`、`--env`、`--scope`、`--header`) 必须在服务器名称**之前**。`--` 分隔服务器名称和命令。

### 管理 MCP 服务器

```bash
# 列出所有配置的服务器
claude mcp list

# 获取特定服务器详情
claude mcp get github

# 移除服务器
claude mcp remove github

# 在 Claude Code 内检查状态
/mcp
```

### OAuth 认证

<Steps>
  <Step title="添加需要认证的服务器">
    ```bash
    claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
    ```
  </Step>

  <Step title="在 Claude Code 内认证">
    ```text
    /mcp
    ```
    然后在浏览器中完成登录流程。
  </Step>
</Steps>

### 配置作用域

| 作用域 | 加载位置 | 是否共享 | 存储位置 |
|--------|----------|----------|----------|
| `local`（默认） | 当前项目 | ❌ | `~/.claude.json` |
| `project` | 当前项目 | ✅ | `.mcp.json` |
| `user` | 所有项目 | ❌ | `~/.claude.json` |

```bash
# 本地作用域（默认）
claude mcp add --transport http stripe https://mcp.stripe.com

# 项目作用域（团队共享）
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# 用户作用域（跨项目）
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 引用 MCP 资源

```bash
# 输入 @ 查看可用资源
@

# 引用特定资源
Can you analyze @github:issue://123 and suggest a fix?
Please review @docs:file://api/authentication

# 引用多个资源
Compare @postgres:schema://users with @docs:file://database/user-model
```

### MCP 输出限制

- **默认限制**：25,000 tokens
- **警告阈值**：10,000 tokens
- **自定义限制**：`MAX_MCP_OUTPUT_TOKENS=50000`

```bash
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

---

## 桌面通知

当 Claude 完成或需要输入时，设置桌面通知。

### 配置 Notification Hook

在 `~/.claude/settings.json` 中添加：

**macOS**:
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**Linux**:
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude Code needs your attention'"
          }
        ]
      }
    ]
  }
}
```

**Windows**:
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')\""
          }
        ]
      }
    ]
  }
}
```

### 窄化 Matcher

| Matcher | 触发时机 |
|---------|----------|
| `permission_prompt` | Claude 需要你批准工具使用 |
| `idle_prompt` | Claude 完成并等待下一个提示 |
| `auth_success` | 身份验证完成 |
| `elicitation_dialog` | Claude 问你问题 |

---

## 作为 Unix 工具使用

### 添加到验证流程

```json
// package.json
{
  "scripts": {
    "lint:claude": "claude -p 'you are a linter. report any issues related to typos. report filename and line number.'"
  }
}
```

### 管道输入/输出

```bash
# 管道数据到 Claude
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

### 控制输出格式

```bash
# 文本格式（默认）
cat data.txt | claude -p 'summarize this data' --output-format text > summary.txt

# JSON 格式
cat code.py | claude -p 'analyze this code for bugs' --output-format json > analysis.json

# 流式 JSON 格式
cat log.txt | claude -p 'parse this log file for errors' --output-format stream-json
```

---

## 定时任务

### 调度选项

| 选项 | 运行位置 | 最适合 |
|------|----------|--------|
| [Routines](/en/routines) | Anthropic 管理的基础设施 | 即使计算机关机也运行的任务 |
| [Desktop scheduled tasks](/en/desktop-scheduled-tasks) | 你的机器（桌面应用） | 需要直接访问本地文件的任务 |
| [GitHub Actions](/en/github-actions) | CI 管道 | 与仓库事件绑定的任务 |
| [`/loop`](/en/scheduled-tasks) | 当前 CLI 会话 | 会话打开时的快速轮询 |

### 示例

```bash
# 每 5 分钟检查一次
/loop 5m check if the deploy finished

# 自主维护检查
/loop
```

**技巧**：编写定时任务提示时，明确说明成功标准和如何处理结果。

---

## 询问 Claude 关于其能力

Claude 可以回答关于其自身功能和限制的问题：

```text
can Claude Code create pull requests?
how does Claude Code handle permissions?
what skills are available?
how do I use MCP with Claude Code?
how do I configure Claude Code for Amazon Bedrock?
what are the limitations of Claude Code?
```

**技巧**：
- Claude 始终可以访问最新的 Claude Code 文档
- 问具体问题获取详细答案
- 运行 `/powerup` 获取带有动画演示的互动课程

---

## 下一步

| 资源 | 说明 |
|------|------|
| [Best practices](/en/best-practices) | 充分利用 Claude Code 的模式 |
| [How Claude Code works](/en/how-claude-code-works) | 理解 agent 循环和上下文管理 |
| [Extend Claude Code](/en/features-overview) | 添加 skills、hooks、MCP、subagents 和插件 |

---

*本文档基于 https://code.claude.com/docs/en/common-workflows 整理*
