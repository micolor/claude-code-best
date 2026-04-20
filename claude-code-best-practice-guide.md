# Claude Code Best Practices 最佳实践指南

> 基于官方文档 (https://code.claude.com/docs/en/best-practices) 整理
> 
> 最后更新：2026 年 4 月 19 日

---

## 目录

- [核心约束：上下文窗口](#核心约束上下文窗口)
- [提供验证方式](#提供验证方式)
- [先探索、再计划、后编码](#先探索再计划后编码)
- [提供具体上下文](#提供具体上下文)
- [配置环境](#配置环境)
- [有效沟通](#有效沟通)
- [管理会话](#管理会话)
- [自动化与扩展](#自动化与扩展)
- [避免常见失败模式](#避免常见失败模式)
- [发展直觉](#发展直觉)

---

## 核心约束：上下文窗口

**关键约束**：Claude 的上下文窗口会快速填满，填满后性能下降。

上下文窗口包含：
- 所有对话消息
- Claude 读取的每个文件
- 每个命令输出

**影响**：一个调试会话或代码库探索可能消耗数万 tokens。当上下文接近满时，Claude 可能"忘记"早期指令或犯错。

**管理策略**：
- 用 [custom status line](/en/statusline) 持续追踪上下文使用
- 参见 [Reduce token usage](/en/costs#reduce-token-usage) 了解降低策略

---

## 提供验证方式

> **最高杠杆原则**：包含测试、截图或预期输出，让 Claude 能自我验证。

Claude 能自我验证时表现显著更好：运行测试、对比截图、验证输出。

### 验证策略对比

| 策略 | 低效提示 | 高效提示 |
|------|----------|----------|
| **提供验证标准** | "implement email validation" | "write validateEmail. test: user@example.com=true, invalid=false. run tests" |
| **视觉验证 UI** | "make dashboard better" | "[paste screenshot] implement design. take screenshot, compare differences" |
| **解决根因** | "build failing" | "build fails with [paste error]. fix and verify. address root cause" |

**验证方式**：
- 测试套件
- Linter
- Bash 命令检查输出
- [Claude in Chrome extension](/en/chrome) 验证 UI

> **提示**：如果无法验证，就不要发布。

---

## 先探索、再计划、后编码

> **核心原则**：分离研究和规划与实现，避免解决错误问题。

直接编码可能产生解决错误问题的代码。使用 [Plan Mode](/en/common-workflows#use-plan-mode-for-safe-code-analysis) 分离探索与执行。

### 四阶段工作流

| 阶段 | 模式 | 示例 |
|------|------|------|
| **Explore** | Plan Mode | "read /src/auth and understand how we handle sessions" |
| **Plan** | Plan Mode | "I want to add Google OAuth. What files need to change? Create a plan." |
| **Implement** | Normal Mode | "implement the OAuth flow from your plan. write tests" |
| **Commit** | Normal Mode | "commit with a descriptive message and open a PR" |

**何时跳过计划**：
- 范围清晰、修复简单（拼写错误、添加日志、重命名变量）
- 可用一句话描述 diff 时，跳过计划

**何时需要计划**：
- 不确定方法
- 修改多个文件
- 不熟悉被修改的代码

> **提示**：Plan Mode 增加开销，小任务直接执行。

---

## 提供具体上下文

> **原则**：指令越精确，需要的修正越少。

Claude 能推断意图，但无法读心。引用具体文件、说明约束、指向示例模式。

### 提示策略

| 策略 | 低效提示 | 高效提示 |
|------|----------|----------|
| **界定范围** | "add tests for foo.py" | "write a test for foo.py covering edge case where user is logged out. avoid mocks." |
| **指向来源** | "why does ExecutionFactory have weird api?" | "look through ExecutionFactory's git history and summarize how its api came to be" |
| **引用模式** | "add a calendar widget" | "look at HotDogWidget.php pattern. follow it to implement calendar widget" |
| **描述症状** | "fix the login bug" | "users report login fails after timeout. check auth flow in src/auth/, write failing test, fix it" |

> 模糊提示可用于探索，如 "what would you improve in this file?" 可发现未想到的问题。

### 提供丰富内容

| 方式 | 说明 |
|------|------|
| **`@` 引用文件** | 直接读取文件，不用描述位置 |
| **粘贴图片** | 复制/粘贴或拖放截图 |
| **提供 URL** | 文档和 API 参考，用 `/permissions` 允许域名 |
| **Pipe 数据** | `cat error.log | claude` |
| **让 Claude 自取** | 用 Bash、MCP 工具或读取文件 |

---

## 配置环境

几个设置步骤能让 Claude Code 在所有会话中更有效。

### 编写有效的 CLAUDE.md

> 运行 `/init` 生成初始文件，然后持续优化。

CLAUDE.md 是 Claude 每次对话开始时读取的特殊文件。包含 Bash 命令、代码风格、工作流规则。

**包含 vs 排除**：

| ✅ 包含 | ❌ 排除 |
|--------|--------|
| Claude 无法猜测的 Bash 命令 | Claude 可从代码推断的内容 |
| 与默认不同的代码风格规则 | Claude 已知的标准语言约定 |
| 测试指令和偏好运行器 | 详细 API 文档（链接文档） |
| 仓库礼仪（分支命名、PR 规范） | 频繁变化的信息 |
| 项目特定架构决策 | 长解释或教程 |
| 开发环境 quirks（必需 env vars） | 文件逐个描述 |
| 常见陷阱或非明显行为 | 自明实践如"写干净代码" |

**关键原则**：
- 保持简洁，每行问："删除这行会导致 Claude 犯错吗？"
- 膨胀的 CLAUDE.md 让 Claude 忽略实际指令
- 用 `@path/to/import` 导入其他文件
- 用 "IMPORTANT" 或 "YOU MUST" 强调关键规则

**放置位置**：
- `~/.claude/CLAUDE.md` — 全局
- `./CLAUDE.md` — 项目根（提交 git）
- `./CLAUDE.local.md` — 个人项目设置（添加到 .gitignore）
- 父目录 — monorepo 支持
- 子目录 — 按需加载

### 配置权限

减少权限提示的三种方式：

| 方式 | 说明 |
|------|------|
| **Auto mode** | 分类器模型审查命令，仅阻止高风险操作 |
| **Permission allowlists** | 允许已知安全的特定工具 |
| **Sandboxing** | OS 级隔离，限制文件系统和网络访问 |

参见 [permission modes](/en/permission-modes)、[permission rules](/en/permissions)、[sandboxing](/en/sandboxing)。

### 使用 CLI 工具

CLI 工具是与外部服务交互最节省上下文的方式。

| 工具 | 用途 |
|------|------|
| `gh` | GitHub 操作 |
| `aws` | AWS 服务 |
| `gcloud` | Google Cloud |
| `sentry-cli` | Sentry |

> Claude 也善于学习新 CLI 工具：`Use 'foo-cli-tool --help' to learn, then use it to solve A, B, C.`

### 连接 MCP 服务器

运行 `claude mcp add` 连接 Notion、Figma、数据库等外部工具。

参见 [MCP servers](/en/mcp)。

### 设置 Hooks

Hooks 在特定点自动运行脚本，确保操作每次执行。

| 特点 | 说明 |
|------|------|
| **确定性** | 保证操作发生（不同于 CLAUDE.md 的 advisory） |
| **自动化** | 文件保存后自动测试、安全拦截等 |

Claude 可以帮你写 hooks：*"Write a hook that runs eslint after every file edit"*

参见 [Hooks](/en/hooks-guide)。

### 创建 Skills

Skills 扩展 Claude 的领域知识。创建 `.claude/skills/xxx/SKILL.md`：

```markdown
---
name: api-conventions
description: REST API design conventions
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
```

Skills 也可定义可调用的工作流：

```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
1. Use `gh issue view` to get issue details
2. Implement changes
3. Write and run tests
4. Create PR
```

用 `disable-model-invocation: true` 禁止自动调用。

### 创建 Subagents

定义专门助手处理隔离任务，在单独上下文中运行：

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
Review for: injection vulnerabilities, auth flaws, secrets, insecure handling.
```

调用方式：*"Use a subagent to review this code for security issues."*

参见 [Subagents](/en/sub-agents)。

### 安装 Plugins

运行 `/plugin` 浏览市场。插件打包 skills、hooks、subagents、MCP。

参见 [Plugins](/en/plugins)、[Extend Claude Code](/en/features-overview)。

---

## 有效沟通

### 询问代码库问题

向 Claude 提问资深工程师级别的问题：
- 日志如何工作？
- 如何创建新 API endpoint？
- 这行代码的作用？
- 处理了哪些边缘情况？
- 为什么调用 `foo()` 而不是 `bar()`？

> 这是有效的入职工作流，减少其他工程师负担。

### 让 Claude 采访你

大型功能前，让 Claude 先采访：

```
I want to build [brief description]. Interview me using AskUserQuestion.
Ask about implementation, UI/UX, edge cases, tradeoffs.
Don't ask obvious questions.
Keep interviewing until covered, then write spec to SPEC.md.
```

完成后在新会话中执行，有干净上下文和书面规格。

---

## 管理会话

### 早期频繁修正

> 修正越快，结果越好。

| 操作 | 效果 |
|------|------|
| `Esc` | 中途停止 Claude，保留上下文可重定向 |
| `Esc + Esc` 或 `/rewind` | 打开回滚菜单，恢复之前状态 |
| `"Undo that"` | 让 Claude 撤销更改 |
| `/clear` | 重置上下文 |

**关键**：同一问题纠正超过两次 → `/clear` 新会话 + 更好提示。

### 积极管理上下文

| 命令 | 用途 |
|------|------|
| `/clear` | 任务间重置上下文 |
| `/compact <instructions>` | 指定压缩重点 |
| `Esc + Esc` → Summarize from here | 压缩部分对话 |
| `/btw` | 快速问题不进入历史 |

> 可在 CLAUDE.md 中配置压缩行为：*"When compacting, preserve modified files list"*

### 使用 Subagents 调查

委托探索，保持主对话干净：

```
Use subagents to investigate how our auth system handles token refresh.
```

Subagents 在单独上下文运行，报告摘要，不消耗主对话。

### 使用检查点回滚

每个 Claude 操作创建检查点。可恢复：
- 对话
- 代码
- 或两者

> 检查点仅追踪 Claude 的更改，不是 Git 的替代品。

参见 [Checkpointing](/en/checkpointing)。

### 恢复对话

```bash
claude --continue    # 恢复最近对话
claude --resume      # 选择历史对话
```

用 `/rename` 给会话命名便于查找。把会话当作分支：不同工作流有独立上下文。

---

## 自动化与扩展

### 非交互模式

```bash
# 单次查询
claude -p "Explain what this project does"

# 结构化输出
claude -p "List all API endpoints" --output-format json

# 流式输出
claude -p "Analyze this log file" --output-format stream-json
```

适用于 CI、pre-commit hooks、自动化工作流。

### 多会话并行

| 方式 | 说明 |
|------|------|
| Desktop app | 可视管理多本地会话，每个有独立 worktree |
| Web 版 | 云端隔离 VM |
| Agent teams | 自动协调多会话 |

**Writer/Reviewer 模式**：
- Session A 写代码
- Session B 审查（新上下文无偏见）

### 批量处理文件

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

用 `--allowedTools` 限制权限。

### Auto mode 自动运行

```bash
claude --permission-mode auto -p "fix all lint errors"
```

分类器审查命令，阻止高风险操作。

---

## 避免常见失败模式

| 模式 | 问题 | 解决方案 |
|------|------|----------|
| **Kitchen sink session** | 一个会话处理多个无关任务 | `/clear` 切换任务 |
| **反复纠正** | 同一问题纠正超过两次 | `/clear` + 更好提示开始 |
| **过度 CLAUDE.md** | 文件太长导致 Claude 忽略部分规则 | 精简，删 Claude 已正确执行的指令 |
| **Trust-then-verify gap** | 实现看起来正确但不处理边缘情况 | 始终提供验证 |
| **无限探索** | 无范围调查读取大量文件 | 限制范围或用 subagents |

---

## 发展直觉

这些模式是起点，非固定规则。

有时应该：
- 让上下文积累（复杂问题历史有价值）
- 跳过计划（探索性任务）
- 用模糊提示（看 Claude 如何解读）

注意：
- 什么有效时，记住你做了什么（提示结构、上下文、模式）
- Claude 困难时，分析原因（上下文噪音？提示模糊？任务太大？）

随时间发展直觉：知道何时具体、何时开放，何时计划、何时探索。

---

## 相关资源

- [How Claude Code works](/en/how-claude-code-works) — 代理循环、工具、上下文管理
- [Extend Claude Code](/en/features-overview) — skills、hooks、MCP、subagents、plugins
- [Common workflows](/en/common-workflows) — 调试、测试、PR 等工作流
- [CLAUDE.md](/en/memory) — 项目约定和持久上下文