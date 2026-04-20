# Claude Code 插件创建指南

## 一、何时使用插件

| 方式 | 技能名称 | 适用场景 |
|------|----------|----------|
| **独立配置** (`.claude/`) | `/hello` | 个人工作流、项目特定定制、快速实验 |
| **插件** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | 团队共享、社区分发、版本管理、跨项目复用 |

**建议**：先用 `.claude/` 快速迭代，准备好分享时再转为插件。

---

## 二、快速开始

### 1. 创建插件目录
```bash
mkdir my-first-plugin
mkdir my-first-plugin/.claude-plugin
```

### 2. 创建插件清单 (plugin.json)
```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

### 3. 添加技能
```bash
mkdir -p my-first-plugin/skills/hello
```

```markdown
---
description: Greet the user with a friendly message
---

Greet the user warmly and ask how you can help them today.
```

### 4. 测试插件
```bash
claude --plugin-dir ./my-first-plugin
/my-first-plugin:hello
```

### 5. 添加技能参数
```markdown
---
description: Greet the user with a personalized message
---

# Hello Skill

Greet the user named "$ARGUMENTS" warmly and ask how you can help them today.
```

```bash
/reload-plugins
/my-first-plugin:hello Alex
```

---

## 三、插件结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件清单（唯一在此目录的文件）
├── skills/                   # 技能 (<name>/SKILL.md)
├── commands/                 # 命令（旧格式，新插件用 skills/）
├── agents/                   # 自定义代理
├── hooks/
│   └── hooks.json           # 事件钩子
├── .mcp.json                 # MCP 服务器配置
├── .lsp.json                 # LSP 服务器配置
├── monitors/
│   └── monitors.json        # 后台监控
├── bin/                      # 可执行文件（添加到 PATH）
└── settings.json             # 默认设置
```

<Warning>
**常见错误**：不要把 `skills/`、`agents/`、`hooks/` 放进 `.claude-plugin/` 里面！它们应该在插件根目录。
</Warning>

---

## 四、高级功能

### 1. 添加技能 (Skills)

```yaml
---
description: Reviews code for best practices. Use when reviewing code or PRs.
---

When reviewing code, check for:
1. Code organization and structure
2. Error handling
3. Security concerns
4. Test coverage
```

### 2. 添加 LSP 服务器

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

### 3. 添加后台监控

```json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

### 4. 默认设置

```json
{
  "agent": "security-reviewer"
}
```

---

## 五、迁移现有配置

```bash
# 创建插件结构
mkdir -p my-plugin/.claude-plugin

# 复制现有配置
cp -r .claude/commands my-plugin/
cp -r .claude/skills my-plugin/
cp -r .claude/agents my-plugin/

# 迁移 hooks
mkdir my-plugin/hooks
# 复制 hooks.json 格式与 settings.json 中 hooks 对象相同
```

### 迁移对比

| 独立配置 (`.claude/`) | 插件 |
|----------------------|------|
| 单项目可用 | 可通过市场共享 |
| `.claude/commands/` | `plugin-name/commands/` |
| `settings.json` 中的 hooks | `hooks/hooks.json` |
| 手动复制分享 | `/plugin install` 安装 |

---

## 六、调试与测试

```bash
# 本地测试插件
claude --plugin-dir ./my-plugin

# 修改后重载（无需重启）
/reload-plugins

# 加载多个插件
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two

# 测试组件
# /plugin-name:skill-name  - 测试技能
# /agents                  - 查看代理
```

### 调试步骤
1. 检查目录结构（在插件根目录，不在 `.claude-plugin/` 内）
2. 单独测试每个组件
3. 使用 `/reload-plugins` 重新加载

---

## 七、发布插件

### 发布前准备
1. 添加 `README.md` 说明文档
2. 使用语义化版本号
3. 团队测试

### 提交到官方市场
- Claude.ai: [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit)
- Console: [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)

---

## 八、核心命令速查

| 命令 | 说明 |
|------|------|
| `claude --plugin-dir <路径>` | 本地测试插件 |
| `/reload-plugins` | 重载插件配置 |
| `/plugin-name:skill` | 执行插件技能 |
| `/agents` | 查看可用代理 |
| `/plugin install <市场>` | 安装市场插件 |
