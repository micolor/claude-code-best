# MCP 使用指南

## 一、什么是 MCP

MCP（Model Context Protocol）让 Claude Code 能连接外部工具和数据源，无需手动复制粘贴数据。

## 二、添加 MCP 服务器

### 三种连接方式

```bash
# 1. HTTP 服务器（推荐）
claude mcp add --transport http <名称> <URL>
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 2. SSE 服务器（已弃用）
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 3. 本地 stdio 服务器
claude mcp add --transport stdio --env API_KEY=xxx airtable -- npx -y airtable-mcp-server
```

### Windows 注意事项
```bash
# Windows 原生需要使用 cmd /c 包装 npx
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

## 三、管理服务器

```bash
# 列出所有服务器
claude mcp list

# 查看服务器详情
claude mcp get github

# 删除服务器
claude mcp remove github

# 在 Claude Code 内管理
/mcp
```

## 四、配置作用域

| 作用域 | 存储位置 | 共享 | 用途 |
|--------|----------|------|------|
| local（默认） | `~/.claude.json` | 否 | 个人开发服务器 |
| project | `.mcp.json` | 是 | 团队协作 |
| user | `~/.claude.json` | 否 | 跨项目个人工具 |

```bash
# 指定作用域
claude mcp add --transport http stripe --scope user https://mcp.stripe.com
```

## 五、认证（OAuth 2.0）

```bash
# 1. 添加服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 在 Claude Code 内认证
/mcp

# 3. 使用固定回调端口
claude mcp add --transport http --callback-port 8080 my-server https://mcp.example.com

# 4. 使用预配置凭证
claude mcp add --transport http --client-id xxx --client-secret --callback-port 8080 my-server https://mcp.example.com
```

## 六、常用示例

```bash
# GitHub
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Sentry
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# PostgreSQL
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub --dsn "postgresql://user:pass@host:5432/db"
```

## 七、高级功能

### 1. 动态 Headers
```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-auth-headers.sh"
    }
  }
}
```

### 2. MCP 提示词作为命令
```bash
# 执行 MCP 提示词
/mcp__github__list_prs
/mcp__github__pr_review 456
```

### 3. 引用 MCP 资源
```text
@mcp__server://resource/path
```

### 4. 从 Claude Desktop 导入
```bash
claude mcp add-from-claude-desktop
```

### 5. JSON 配置添加
```bash
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp"}'
```

## 八、输出限制

```bash
# 默认限制：25,000 tokens
# 调整限制
export MAX_MCP_OUTPUT_TOKENS=50000
```

## 九、企业级管理

### 方式 1：独占控制（managed-mcp.json）
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `/etc/claude-code/managed-mcp.json`
- Windows: `C:\Program Files\ClaudeCode\managed-mcp.json`

### 方式 2：策略控制（允许/拒绝列表）
```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverUrl": "https://*.company.com/*" }
  ],
  "deniedMcpServers": []
}
```

## 十、核心命令速查

| 命令 | 说明 |
|------|------|
| `claude mcp add --transport <类型> <名称> <URL/命令>` | 添加服务器 |
| `claude mcp list` | 列出所有服务器 |
| `claude mcp get <名称>` | 查看服务器详情 |
| `claude mcp remove <名称>` | 删除服务器 |
| `/mcp` | 在 Claude Code 内认证/管理 |
| `claude mcp add-json <名称> '<json>'` | 从 JSON 配置添加 |
| `claude mcp add-from-claude-desktop` | 从 Claude Desktop 导入 |
