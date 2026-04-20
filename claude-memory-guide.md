# Claude Code 记忆系统指南

## 一、两种记忆方式

Claude Code 通过两种机制在会话间保持持久化上下文：

| 特性 | CLAUDE.md 文件 | 自动记忆 (Auto Memory) |
|------|---------------|------------------------|
| **谁编写** | 你 | Claude |
| **内容** | 指令和规则 | 学习和模式 |
| **作用域** | 项目/用户/组织 | 每个工作树 |
| **加载** | 每次会话完整加载 | 每次会话（前 200 行/25KB） |
| **用途** | 编码标准、工作流、项目架构 | 构建命令、调试见解、偏好设置 |

---

## 二、CLAUDE.md 文件

### 文件位置与作用域

| 作用域 | 位置 | 共享对象 | 用途 |
|--------|------|----------|------|
| **Managed** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 全组织 | IT 部署的组织级指令 |
| **Project** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队 | 项目级指令，通过版本控制共享 |
| **User** | `~/.claude/CLAUDE.md` | 个人 | 个人全局偏好 |
| **Local** | `./CLAUDE.local.md` | 个人 | 项目个人偏好（应加入 `.gitignore`） |

### 加载顺序

Claude 从当前工作目录向上遍历目录树，加载所有发现的 CLAUDE.md 文件：

```
当前目录：foo/bar/
加载：foo/bar/CLAUDE.md → foo/CLAUDE.md → ~/.claude/CLAUDE.md
```

同一目录内，`CLAUDE.local.md` 在 `CLAUDE.md` 之后加载，优先级更高。

### 编写有效指令

**大小**：每个文件控制在 200 行以内，过长会降低遵循度。

**结构**：使用 Markdown 标题和列表组织指令。

**具体性**：
```markdown
✅ 使用 2 空格缩进
❌ 代码格式化

✅ 提交前运行 `npm test`
❌ 测试你的更改

✅ API 处理器放在 `src/api/handlers/`
❌ 保持文件整洁
```

**一致性**：避免规则冲突，定期审查 CLAUDE.md 文件。

### 引用其他文件

使用 `@path/to/file` 语法引用其他文件：

```markdown
查看 @README 了解项目概述，@package.json 了解可用命令。

# 额外指令
- Git 工作流参考 @docs/git-instructions.md
- 个人偏好 @~/.claude/my-project-instructions.md
```

### 整合 AGENTS.md

如果项目已有 AGENTS.md，可在 CLAUDE.md 中引用：

```markdown
@AGENTS.md

## Claude Code 特定指令
- 在 `src/billing/` 下使用 plan mode
```

### 使用 `.claude/rules/` 组织规则

大型项目可将指令拆分为多个规则文件：

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # 主项目指令
│   └── rules/
│       ├── code-style.md   # 代码风格
│       ├── testing.md      # 测试规范
│       └── security.md     # 安全要求
```

### 路径特定规则

使用 YAML frontmatter 将规则限定到特定文件：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 开发规则

- 所有 API 端点必须包含输入验证
- 使用标准错误响应格式
- 包含 OpenAPI 文档注释
```

**支持的 Glob 模式**：

| 模式 | 匹配 |
|------|------|
| `**/*.ts` | 所有 TypeScript 文件 |
| `src/**/*` | src/ 目录下所有文件 |
| `*.md` | 根目录 Markdown 文件 |
| `src/**/*.{ts,tsx}` | src/ 下 TS 和 TSX 文件 |

### 大型团队管理

#### 排除特定 CLAUDE.md 文件

在大型 monorepo 中，排除不相关团队的 CLAUDE.md：

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

添加到 `.claude/settings.local.json` 保持本地配置。

---

## 三、自动记忆 (Auto Memory)

### 启用/禁用

自动记忆默认启用。

**通过命令**：
```bash
/memory  # 使用切换开关
```

**通过设置** (`~/.claude/settings.json`)：
```json
{
  "autoMemoryEnabled": false
}
```

**通过环境变量**：
```bash
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 claude
```

### 存储位置

每个项目的自动记忆存储在：
```
~/.claude/projects/<project>/memory/
```

`<project>` 路径基于 git 仓库，同一仓库的所有工作树共享一个自动记忆目录。

**自定义存储位置**：
```json
{
  "autoMemoryDirectory": "~/my-custom-memory-dir"
}
```

### 文件结构

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 索引文件，每次会话加载前 200 行/25KB
├── debugging.md       # 调试模式笔记
├── api-conventions.md # API 设计决策
└── ...                # Claude 创建的其他主题文件
```

### 工作机制

1. **会话开始**：加载 `MEMORY.md` 前 200 行或 25KB
2. **会话期间**：Claude 根据需要读写记忆文件
3. **主题文件**：不自动加载，Claude 按需读取

---

## 四、/memory 命令

使用 `/memory` 命令管理记忆文件：

- 查看当前会话加载的所有 CLAUDE.md 和规则文件
- 切换自动记忆开关
- 打开自动记忆文件夹
- 编辑记忆文件

**添加记忆**：
```text
记住：总是使用 pnpm，不是 npm
```
Claude 会将其保存到自动记忆。

**添加到 CLAUDE.md**：
```text
把这个添加到 CLAUDE.md
```

---

## 五、调试记忆问题

### Claude 不遵循 CLAUDE.md

1. 运行 `/memory` 验证文件是否被加载
2. 检查文件位置是否正确
3. 使指令更具体
4. 查找冲突的指令

**调试钩子**：使用 `InstructionsLoaded` 钩子记录哪些指令文件被加载：
```json
{
  "hooks": {
    "InstructionsLoaded": [
      {
        "type": "command",
        "command": "echo 'Loaded: $file'"
      }
    ]
  }
}
```

### CLAUDE.md 文件过大

- 移到 200 行以内
- 使用 `@path` 引用拆分内容
- 使用 `.claude/rules/` 分散指令

### `/compact` 后指令丢失

- 项目根目录 CLAUDE.md 在 compact 后会重新读取
- 子目录 CLAUDE.md 不会自动重新注入
- 会话中给出的指令需要添加到 CLAUDE.md 才能持久化

---

## 六、最佳实践

### 何时写入 CLAUDE.md

- Claude 重复犯同样的错误
- 代码审查发现 Claude 应该知道的项目规范
- 你在聊天中输入上次会话也说过的纠正
- 新团队成员需要相同的上下文

### 何时使用自动记忆

- Claude 发现的构建命令
- 调试过程中获得的见解
- 架构决策记录
- 个人工作流偏好

### 文件组织建议

```
project/
├── CLAUDE.md                 # 核心项目指令（<200 行）
├── CLAUDE.local.md           # 个人本地偏好
├── .claude/
│   ├── CLAUDE.md             # 或放这里（与根目录等价）
│   └── rules/
│       ├── code-style.md     # 代码风格
│       ├── testing.md        # 测试规范
│       └── security.md       # 安全要求
└── ~/.claude/
    ├── CLAUDE.md             # 个人全局偏好
    └── rules/
        └── preferences.md    # 个人编码偏好
```

---

## 七、核心命令速查

| 命令 | 说明 |
|------|------|
| `/init` | 自动生成 CLAUDE.md |
| `/memory` | 管理记忆文件 |
| `/reload-plugins` | 重载插件和规则 |
