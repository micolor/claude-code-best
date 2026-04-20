# Claude Code Skills 官方完整指南

> 基于官方文档 (https://code.claude.com/docs/en/skills) 的完整参考与学习笔记

---

## 目录

- [什么是 Skills](#什么是-skills)
- [快速创建](#快速创建)
- [存储位置](#存储位置)
- [配置 Skills](#配置-skills)
- [调用控制](#调用控制)
- [变量替换](#变量替换)
- [高级用法](#高级用法)
- [限制 Skill 访问](#限制-skill-访问)
- [Skill 生命周期](#skill-生命周期)
- [故障排除](#故障排除)
- [Bundled Skills](#bundled-skills)
- [示例 Skills](#示例-skills)
- [最佳实践](#最佳实践)

---

## 什么是 Skills

**Skills** 扩展 Claude 的能力，通过创建 `SKILL.md` 文件实现。Claude 在相关时使用 skills，或你可以直接用 `/skill-name` 调用。

### 何时创建 Skill

| 场景 | 说明 |
|------|------|
| 重复粘贴相同内容 | 每次都粘贴同样的 playbook、checklist、多步骤程序 |
| CLAUDE.md 过于复杂 | 当某部分已经成长为**程序**而非**事实** |
| 长参考材料 | Skill 内容仅在需要时加载，平时几乎不消耗 token |

### Skill vs CLAUDE.md

| 特性 | CLAUDE.md | Skill |
|------|-----------|-------|
| 加载时机 | 每次会话都加载 | 仅在使用时加载 |
| 内容类型 | 项目事实和约定 | 程序、工作流、参考材料 |
| Token 消耗 | 持续占用 | 按需加载，成本极低 |
| 调用方式 | 自动应用 | 自动或 `/skill-name` 调用 |

### 核心优势

1. **按需加载** - 长参考材料几乎不消耗 token，直到你需要它
2. **自动触发** - Claude 可根据描述自动加载相关 skill
3. **手动调用** - 直接用 `/skill-name` 显式调用
4. **支持文件** - 可包含模板、示例、脚本等额外文件
5. **灵活配置** - 控制谁可以调用、在哪里运行、使用什么工具

---

## 快速创建

### 3 步创建第一个 Skill

这个示例创建一个 skill，教 Claude 使用视觉图表和类比解释代码。

```bash
# 1. 创建目录
mkdir -p ~/.claude/skills/explain-code

# 2. 编写 SKILL.md
cat > ~/.claude/skills/explain-code/SKILL.md << 'EOF'
---
name: explain-code
description: 使用视觉图表和类比解释代码。当解释代码如何工作、教授代码库或用户询问"这是如何工作的？"时使用。
---

解释代码时，始终包括：

1. **从类比开始**：将代码与日常生活中的事物进行比较
2. **绘制图表**：使用 ASCII 艺术展示流程、结构或关系
3. **逐步讲解**：一步一步解释代码在做什么
4. **强调陷阱**：常见错误或误解是什么？

保持对话式的解释。对于复杂的概念，使用多个类比。
EOF

# 3. 测试
# 方式 1：让 Claude 自动调用
# 问："这段代码是如何工作的？"

# 方式 2：直接调用
# /explain-code src/auth/login.ts
```

### 测试 Skill

**自动调用** - 询问匹配描述的问题：
```text
How does this code work?
```

**手动调用** - 直接使用 skill 名称：
```text
/explain-code src/auth/login.ts
```

---

## 存储位置

### 4 个层级

| 位置 | 路径 | 适用范围 |
|------|------|----------|
| 企业级 | Managed settings | 组织内所有用户 |
| 个人级 | `~/.claude/skills/` | 所有项目 |
| 项目级 | `.claude/skills/` | 当前项目 |
| 插件 | `<plugin>/skills/` | 启用插件的地方 |

**优先级**：企业级 > 个人级 > 项目级

插件技能使用 `plugin-name:skill-name` 命名空间，因此不会与其他层级冲突。

### 实时变更检测

- 编辑现有 skill → **立即生效**，无需重启
- 创建新的顶级 skills 目录 → **需要重启** 才能被监视

### 嵌套目录发现

在 monorepo 中，Claude 会自动从嵌套的 `.claude/skills/` 目录发现 skills。例如，编辑 `packages/frontend/` 中的文件时，Claude 也会查找 `packages/frontend/.claude/skills/` 中的 skills。

### Skills 目录结构

```text
my-skill/
├── SKILL.md           # 必需 - 主指令和导航
├── template.md        # 供 Claude 填写的模板
├── examples/
│   └── sample.md      # 显示预期格式的输出示例
└── scripts/
    └── validate.sh    # Claude 可执行的脚本
```

**支持文件**：
- `template.md` - Claude 填写的模板
- `examples.md` - 预期输出格式示例
- `scripts/` - Claude 可执行的实用脚本
- 详细的参考文档

在 `SKILL.md` 中引用支持文件：
```markdown
## 额外资源
- 完整的 API 细节，见 [reference.md](reference.md)
- 使用示例，见 [examples.md](examples.md)
```

**建议**：保持 `SKILL.md` 在 500 行以内，将详细的参考材料移到单独的文件。

---

## 配置 Skills

### 两种内容类型

#### 参考内容 (Reference Content)

添加 Claude 应用于当前工作的知识。约定、模式、风格指南、领域知识。

```yaml
---
name: api-conventions
description: 此代码库的 API 设计模式
---

编写 API 端点时：
- 使用 RESTful 命名约定
- 返回一致的错误格式
- 包含请求验证
```

**特点**：内联运行，Claude 可以在对话上下文中使用。

#### 任务内容 (Task Content)

给 Claude 提供特定操作的逐步指令，如部署、提交或代码生成。

```yaml
---
name: deploy
description: 部署应用到生产环境
context: fork
disable-model-invocation: true
---

部署应用：
1. 运行测试套件
2. 构建应用
3. 推送到部署目标
```

**特点**：通常希望直接用 `/skill-name` 调用，而非让 Claude 自动触发。

---

### Frontmatter 配置

```yaml
---
name: my-skill                    # Skill 名称（成为 /slash-command）
description: 做什么，何时使用      # Claude 用它决定何时加载
when_to_use: 额外上下文           # 追加到 description
argument-hint: [issue-number]     # 自动完成时的参数提示
disable-model-invocation: true    # 禁止 Claude 自动调用
user-invocable: false             # 从 / 菜单隐藏
allowed-tools: Read Grep          # 预批准的工具
model: sonnet                     # 使用的模型
effort: high                      # 努力程度
context: fork                     # 在子 agent 中运行
agent: Explore                    # 指定子 agent 类型
hooks:                            # 生命周期钩子
paths: ["**/*.py"]                # 文件模式匹配
memory: user                      # 持久内存范围
shell: bash                       # shell 类型
---
```

### 完整字段列表

| 字段 | 必需性 | 描述 |
|------|--------|------|
| `name` | 否 | Skill 名称（省略则用目录名）。小写字母、数字和连字符（最多 64 字符） |
| `description` | 推荐 | **Claude 用它决定何时应用**。省略则用 markdown 第一段。与 `when_to_use` 合计上限 1,536 字符 |
| `when_to_use` | 否 | 额外上下文，如触发短语或示例请求。追加到 `description` |
| `argument-hint` | 否 | 自动完成时显示的参数提示，如 `[issue-number]` 或 `[filename] [format]` |
| `disable-model-invocation` | 否 | `true` 禁止 Claude 自动调用。用于想手动触发的工作流。默认 `false` |
| `user-invocable` | 否 | `false` 从 `/` 菜单隐藏。用于用户不应直接调用的背景知识。默认 `true` |
| `allowed-tools` | 否 | Skill 激活时无需批准可使用的工具 |
| `model` | 否 | 使用的模型 |
| `effort` | 否 | 努力程度：`low`、`medium`、`high`、`xhigh`、`max` |
| `context` | 否 | `fork` 在子 agent 上下文中运行 |
| `agent` | 否 | 当 `context: fork` 时使用的子 agent 类型 |
| `hooks` | 否 | 绑定到此 skill 生命周期的钩子 |
| `paths` | 否 | 限制 skill 激活的全局模式 |
| `memory` | 否 | 持久内存范围：`user`、`project`、`local` |
| `shell` | 否 | shell 类型：`bash`（默认）或 `powershell` |

---

## 调用控制

### 三种模式

| Frontmatter | 你可以调用 | Claude 可以调用 | 描述加载 |
|-------------|------------|-----------------|----------|
| 默认 | ✅ | ✅ | 总是 |
| `disable-model-invocation: true` | ✅ | ❌ | 不加载 |
| `user-invocable: false` | ❌ | ✅ | 总是 |

### 使用场景

**仅手动调用**（有副作用的工作流）：
```yaml
---
name: deploy
description: 部署到生产环境
disable-model-invocation: true
---

部署 $ARGUMENTS 到生产环境：
1. 运行测试套件
2. 构建应用
3. 推送到部署目标
4. 验证部署成功
```

**仅 Claude 调用**（背景知识）：
```yaml
---
name: legacy-system-context
description: 解释旧系统如何工作
user-invocable: false
---

你是旧系统专家。当相关时解释旧系统如何工作。
```

---

## 变量替换

### 可用变量

| 变量 | 说明 |
|------|------|
| `$ARGUMENTS` | 调用 skill 时传递的所有参数 |
| `$ARGUMENTS[N]` | 通过 0 基索引访问特定参数，如 `$ARGUMENTS[0]` |
| `$N` | `$ARGUMENTS[N]` 的简写，如 `$0`、`$1` |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID |
| `${CLAUDE_SKILL_DIR}` | 包含 skill 的 `SKILL.md` 文件的目录 |

### 示例

**使用所有参数**：
```yaml
---
name: fix-issue
description: 修复 GitHub issue
disable-model-invocation: true
---

修复 GitHub issue $ARGUMENTS，遵循我们的编码标准。
```

使用：`/fix-issue 123` → "修复 GitHub issue 123..."

**使用索引参数**：
```yaml
---
name: migrate-component
description: 将组件从一个框架迁移到另一个
---

迁移 $0 组件从 $1 到 $2。
保留所有现有行为和测试。
```

使用：`/migrate-component SearchBar React Vue`
- `$0` → `SearchBar`
- `$1` → `React`
- `$2` → `Vue`

**多词参数**使用 shell 风格引号：
```
/my-skill "hello world" second
```
- `$0` → `hello world`
- `$1` → `second`

---

## 高级用法

### 1. 注入动态上下文

使用 `` !`<command>` `` 在 skill 内容发送给 Claude 之前执行 shell 命令。

```yaml
---
name: pr-summary
description: 总结 PR 变更
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request 上下文
- PR diff: !`gh pr diff`
- PR 评论：!`gh pr view --comments`
- 变更的文件：!`gh pr diff --name-only`

## 任务
总结这个 pull request...
```

**执行流程**：
1. 每个 `` !`<command>` `` 立即执行（Claude 看不到命令本身）
2. 输出替换 skill 内容中的占位符
3. Claude 接收完全渲染的提示和实际数据

**多行命令**使用 ` ```! ` 代码块：

````markdown
## 环境
```!
node --version
npm --version
git status --short
```
````

**禁用此功能**：在 [settings](/en/settings) 中设置 `"disableSkillShellExecution": true`。

**提示**：要在 skill 中启用**扩展思维**，在 skill 内容中的任何地方包含单词 "ultrathink"。

---

### 2. 在子 Agent 中运行

添加 `context: fork` 到 frontmatter，当你希望 skill 在隔离环境中运行时。

```yaml
---
name: deep-research
description: 深入研究主题
context: fork
agent: Explore
---

研究 $ARGUMENTS 彻底：
1. 使用 Glob 和 Grep 查找相关文件
2. 阅读和分析代码
3. 总结发现并附带具体的文件引用
```

**执行流程**：
1. 创建新的隔离上下文
2. 子 agent 接收 skill 内容作为提示
3. `agent` 字段决定执行环境（模型、工具和权限）
4. 结果被汇总并返回到主对话

### Skill 与 Subagent 的两种方式

| 方式 | 系统提示 | 任务 | 也加载 |
|------|----------|------|--------|
| Skill with `context: fork` | Agent 类型决定 | SKILL.md 内容 | CLAUDE.md |
| Subagent with `skills` 字段 | Subagent 的 markdown 主体 | Claude 的委托消息 | 预加载的 skills + CLAUDE.md |

---

### 3. 预批准工具

```yaml
---
name: commit
description: 提交当前变更
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

`allowed-tools` 授予 Skill 激活时无需批准使用这些工具。

---

### 4. 添加支持文件

```text
my-skill/
├── SKILL.md           # 必需 - 概述和导航
├── reference.md       # 详细 API 文档（需要时加载）
├── examples.md        # 使用示例（需要时加载）
└── scripts/
    └── helper.py      # 实用脚本（执行，不加载）
```

在 `SKILL.md` 中引用：
```markdown
## 额外资源
- API 细节见 [reference.md](reference.md)
- 示例见 [examples.md](examples.md)
```

---

### 5. 生成视觉输出

Skill 可以捆绑和运行任何语言的脚本，生成视觉输出：

```yaml
---
name: codebase-visualizer
description: 生成代码库交互式树形可视化。当探索新仓库、理解项目结构或识别大文件时使用。
allowed-tools: Bash(python *)
---

# 代码库可视化器

生成交互式 HTML 树形视图，展示项目的文件结构，带有可折叠的目录。

## 用法

从项目根目录运行可视化脚本：
```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```

这会创建 `codebase-map.html` 并在默认浏览器中打开。

## 可视化显示

- **可折叠目录**：点击文件夹展开/折叠
- **文件大小**：显示在每个文件旁边
- **颜色**：不同文件类型使用不同颜色
- **目录总计**：显示每个文件夹的聚合大小
```

脚本生成 HTML 文件包含：
- **摘要侧边栏** - 文件数、目录数、总大小、文件类型数
- **条形图** - 按文件类型分解（前 8 个按大小）
- **可折叠树** - 可展开和折叠目录

此模式适用于任何视觉输出：依赖图、测试覆盖率报告、API 文档、数据库架构可视化。

---

## 限制 Skill 访问

### 三种控制方式

**1. 禁用所有 skills** - 在 `/permissions` 中拒绝 Skill 工具：
```text
Skill
```

**2. 允许/拒绝特定 skills** - 使用 [permission rules](/en/permissions)：
```text
# 只允许特定 skills
Skill(commit)
Skill(review-pr *)

# 拒绝特定 skills
Skill(deploy *)
```

语法：`Skill(name)` 精确匹配，`Skill(name *)` 前缀匹配。

**3. 隐藏单个 skills** - 在 frontmatter 中添加 `disable-model-invocation: true`：
```yaml
disable-model-invocation: true  # 从 Claude 上下文中移除
```

---

## Skill 生命周期

### 调用过程

1. 渲染的 `SKILL.md` 作为单条消息进入对话
2. 内容保留到会话结束
3. Claude Code **不会**在后续回合重新读取 skill 文件

**提示**：将应贯穿任务的指导写为持久指令，而非一次性步骤。

### 自动压缩

当对话被总结以释放上下文时：
1. 重新附加每个 skill 的最新调用
2. 保留前 5,000 tokens
3. 共享 25,000 tokens 预算
4. 从最近调用的 skill 开始填充

**如果 skill 似乎停止影响行为**：
- 内容通常仍然存在，模型选择了其他工具或方法
- 加强 `description` 和指令，让模型继续优先使用
- 或使用 [hooks](/en/hooks) 确定性地强制执行行为
- 如果 skill 很大或在它之后调用了其他 skill，在压缩后重新调用它以恢复完整内容

---

## 故障排除

### Skill 未触发

1. 检查 description 是否包含用户自然会说出的关键词
2. 验证 skill 是否出现在 `What skills are available?`
3. 重新表述请求以更匹配 description
4. 直接用 `/skill-name` 调用（如果是 user-invocable）

### Skill 触发过于频繁

1. 让 description 更具体
2. 添加 `disable-model-invocation: true` 如果只想手动调用

### 描述被截断

Skill 描述被加载到上下文中，以便 Claude 知道有什么可用。

| 项目 | 值 |
|------|------|
| 预算 | 动态为上下文窗口的 1% |
| 回退值 | 8,000 字符 |
| 每个条目上限 | 1,536 字符 |

**提高限制**：设置 `SLASH_COMMAND_TOOL_CHAR_BUDGET` 环境变量。

**优化方法**：前置关键用例，精简 `description` 和 `when_to_use` 文字。

---

## Bundled Skills

Claude Code 包含 5 个内置的 Bundled Skills：

| Skill | 作用 |
|-------|------|
| `/simplify` | 审查最近变更的文件，修复代码复用、质量和效率问题 |
| `/batch` | 并行编排代码库的大规模变更 |
| `/debug` | 启用调试日志并排查问题 |
| `/loop` | 重复运行提示，用于定期检查或自主维护 |
| `/claude-api` | 加载 Claude API 参考材料 |

**特点**：
- 基于提示的执行（交给 Claude 编排）
- Claude 可在相关时自动调用
- 可以在 `/skills` 中查看和管理

---

## 示例 Skills

### 代码审查员

```yaml
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
- 输入验证实现
- 良好的测试覆盖率
- 性能考虑已解决

按优先级组织反馈：
- 严重问题（必须修复）
- 警告（应该修复）
- 建议（考虑改进）

包含如何修复问题的具体示例。
```

### 调试器

```yaml
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

### 数据科学家

```yaml
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

### PR 摘要

```yaml
---
name: pr-summary
description: 使用 GitHub CLI 总结 PR 变更
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## PR 上下文
- Diff: !`gh pr diff`
- 评论：!`gh pr view --comments`
- 变更文件：!`gh pr diff --name-only`

## 任务
总结这个 pull request 的变更，突出关键修改和潜在问题。
```

---

## 最佳实践

| 原则 | 说明 |
|------|------|
| **专注** | 每个 skill 应擅长一个特定任务 |
| **详细** | 编写详细的 description，Claude 用它来决定何时委托 |
| **限制工具** | 仅授予必要的权限以确保安全和专注 |
| **支持文件** | 保持 SKILL.md 简洁，详细材料移到单独文件 |
| **版本控制** | 提交 `.claude/skills/` 到 git 与团队共享 |

### 命令速查

```bash
# 创建 Skill
mkdir -p ~/.claude/skills/<name>

# 列出所有 skills
/skills

# 调用 skill
/skill-name args

# 检查 skill 是否可用
/skills
```

---

## 分享 Skills

### 项目 Skills

```bash
# 提交到版本控制
git add .claude/skills/
git commit -m "Add team skills"
```

### 插件 Skills

在 [plugin](/en/plugins) 中创建 `skills/` 目录。

### 企业级 Skills

通过 [managed settings](/en/settings#settings-files) 部署到整个组织。

---

## 相关资源

- **[Subagents](/en/sub-agents)** - 委托任务到专业 agent
- **[Plugins](/en/plugins)** - 打包和分发 skills
- **[Hooks](/en/hooks)** - 自动化工作流
- **[Memory](/en/memory)** - 管理 CLAUDE.md
- **[Commands](/en/commands)** - 内置命令参考
- **[Permissions](/en/permissions)** - 控制工具和 skill 访问

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-04-18 | 1.0.0 | 合并精炼版与完整版 |

---

*本文档基于 https://code.claude.com/docs/en/skills 整理*
