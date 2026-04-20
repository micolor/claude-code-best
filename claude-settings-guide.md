# Claude Code 配置指南

## 一、配置作用域

Claude Code 使用作用域系统决定配置的应用范围。

| 作用域 | 位置 | 共享对象 | 用途 |
|--------|------|----------|------|
| **Managed** | 服务器/MDM/系统级 | 全组织 | IT 强制策略 |
| **User** | `~/.claude/settings.json` | 个人全局 | 个人偏好 |
| **Project** | `.claude/settings.json` | 团队 | 团队共享配置 |
| **Local** | `.claude/settings.local.json` | 个人 | 项目个人覆盖 |

### 优先级（从高到低）

1. **Managed** - 不可覆盖
2. **命令行参数** - 临时会话覆盖
3. **Local** - 个人项目覆盖
4. **Project** - 团队共享
5. **User** - 个人全局

> **注意**：数组类型设置（如 `permissions.allow`、`sandbox.filesystem.allowWrite`）会在多个作用域间**合并**而非替换。

---

## 二、核心配置项

### 权限设置

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### 环境变量

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "MY_API_KEY": "your-key"
  }
}
```

### 模型配置

```json
{
  "model": "claude-sonnet-4-6",
  "availableModels": ["sonnet", "haiku"],
  "effortLevel": "xhigh",
  "alwaysThinkingEnabled": true
}
```

---

## 三、沙箱配置

沙箱隔离 Bash 命令与文件系统和网络。

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker *"],
    "filesystem": {
      "allowWrite": ["/tmp/build", "~/.kube"],
      "denyWrite": ["/etc", "/usr/local/bin"],
      "denyRead": ["~/.aws/credentials"]
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org", "registry.yarnpkg.com"],
      "deniedDomains": ["uploads.github.com"],
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

### 沙箱路径前缀

| 前缀 | 含义 | 示例 |
|------|------|------|
| `/` | 绝对路径 | `/tmp/build` |
| `~/` | 相对家目录 | `~/.kube` → `$HOME/.kube` |
| `./` 或无前缀 | 相对项目根目录 | `./output` → `<project>/output` |

---

## 四、Git 归属设置

```json
{
  "attribution": {
    "commit": "🤖 Generated with Claude Code\n\nCo-Authored-By: Claude",
    "pr": "🤖 Generated with Claude Code"
  }
}
```

隐藏归属信息：
```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

---

## 五、子代理配置

**用户子代理**: `~/.claude/agents/`  
**项目子代理**: `.claude/agents/`

创建子代理文件（`.md` + YAML frontmatter）：

```yaml
---
description: 代码审查专家。审查代码时自动调用。
tools:
  allow: ["Read", "Grep", "Glob"]
  deny: ["Edit", "Write"]
model: claude-sonnet-4-6
---

你是一名资深代码审查员。审查代码时检查：

1. 代码组织和结构
2. 错误处理
3. 安全隐患
4. 测试覆盖率
5. 性能问题

提供具体的改进建议。
```

---

## 六、插件配置

```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@personal": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "security-plugins": {
      "source": {
        "source": "git",
        "url": "https://git.example.com/security/plugins.git"
      }
    }
  }
}
```

### 企业市场限制（仅 Managed）

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    }
  ]
}
```

完全禁止添加市场：
```json
{
  "strictKnownMarketplaces": []
}
```

---

## 七、钩子配置

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint"
          }
        ]
      }
    ],
    "PreCommand": [
      {
        "matcher": "git push",
        "hooks": [
          {
            "type": "http",
            "url": "https://hooks.example.com/pre-push",
            "method": "POST",
            "headers": {
              "Authorization": "Bearer ${MY_TOKEN}"
            },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  },
  "allowedHttpHookUrls": ["https://hooks.example.com/*", "http://localhost:*"],
  "httpHookAllowedEnvVars": ["MY_TOKEN", "HOOK_SECRET"]
}
```

### 仅允许 Managed 钩子

```json
{
  "allowManagedHooksOnly": true
}
```

---

## 八、全局配置（`~/.claude.json`）

这些设置存储在 `~/.claude.json`，不在 `settings.json` 中。

```json
{
  "editorMode": "vim",
  "autoScrollEnabled": true,
  "showTurnDuration": false,
  "spinnerTipsEnabled": true,
  "tui": "fullscreen",
  "voiceEnabled": true,
  "fastModePerSessionOptIn": true,
  "cleanupPeriodDays": 30,
  "outputStyle": "Explanatory",
  "language": "japanese"
}
```

### Worktree 配置

```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache", ".git"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

---

## 九、敏感文件排除

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./build)"
    ]
  }
}
```

---

## 十、Hook 配置

### HTTP Hook URL 限制

```json
{
  "allowedHttpHookUrls": [
    "https://hooks.example.com/*",
    "http://localhost:*"
  ]
}
```

### HTTP Hook 环境变量限制

```json
{
  "httpHookAllowedEnvVars": ["MY_TOKEN", "HOOK_SECRET", "API_KEY"]
}
```

---

## 十一、验证配置

在 Claude Code 内运行：

```bash
/status
```

输出显示各配置层的来源（Managed、User、Project）。

---

## 十二、核心命令速查

| 命令 | 说明 |
|------|------|
| `/config` | 打开配置界面 |
| `/status` | 查看活跃配置来源 |
| `/plugin` | 管理插件 |
| `/agents` | 查看子代理 |
| `/reload-plugins` | 重载插件 |
| `/model` | 切换模型 |
| `/effort` | 调整努力程度 |
| `/focus` | 切换视图模式 |
| `/tui` | 切换终端渲染器 |

---

## 十三、文件位置总结

| 配置类型 | User 位置 | Project 位置 | Local 位置 |
|----------|-----------|--------------|------------|
| **Settings** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **Subagents** | `~/.claude/agents/` | `.claude/agents/` | - |
| **MCP servers** | `~/.claude.json` | `.mcp.json` | `~/.claude.json` |
| **Plugins** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `CLAUDE.md` | `CLAUDE.local.md` |
| **Global config** | `~/.claude.json` | - | - |

---

## 十四、示例配置模板

### 团队共享配置（`.claude/settings.json`）

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run build)"
    ],
    "deny": [
      "Bash(curl *)",
      "Bash(wget *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../docs/"]
  },
  "env": {
    "TEAM_ID": "platform-team",
    "CI_ENDPOINT": "https://ci.company.com"
  },
  "companyAnnouncements": [
    "欢迎加入平台团队！查看代码指南：docs.company.com"
  ],
  "attribution": {
    "commit": "🤖 Generated with Claude Code",
    "pr": ""
  }
}
```

### 个人配置（`.claude/settings.local.json`）

```json
{
  "model": "claude-opus-4-7",
  "effortLevel": "xhigh",
  "alwaysThinkingEnabled": true,
  "viewMode": "verbose",
  "disableAutoMode": "disable"
}
```
