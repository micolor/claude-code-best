# Claude Code Hooks 官方完整指南

> 基于官方文档 (https://code.claude.com/docs/en/hooks) 的完整参考与实战笔记

---

## 目录

- [什么是 Hooks](#什么是-hooks)
- [Hook 生命周期](#hook-生命周期)
- [快速开始](#快速开始)
- [配置详解](#配置详解)
- [Hook 事件详解](#hook-事件详解)
- [四种 Hook 类型](#四种-hook-类型)
- [输入输出规范](#输入输出规范)
- [实战示例](#实战示例)
- [调试与故障排除](#调试与故障排除)
- [最佳实践](#最佳实践)

---

## 什么是 Hooks

**Hooks** 是自动化工具，在 Claude Code 会话的特定事件发生时执行自定义操作。

### 核心优势

| 优势 | 说明 |
|------|------|
| **自动化工作流** | 文件保存后自动运行测试、格式化代码 |
| **安全检查** | 在执行危险命令前拦截并验证 |
| **质量门禁** | 提交前运行 lint、验证命名规范 |
| **上下文注入** | 会话开始时自动加载项目状态 |

### 何时使用 Hooks

| 场景 | 推荐事件 |
|------|----------|
| 保存文件后自动测试 | `PostToolUse` (matcher: `Write\|Edit`) |
| 阻止危险命令 | `PreToolUse` (matcher: `Bash`) |
| Prompt 过滤 | `UserPromptSubmit` |
| 阻止 Claude 停止 | `Stop` |
| 会话开始加载上下文 | `SessionStart` |
| 配置变更审计 | `ConfigChange` |

<Note>
Hooks 运行在你的系统上，拥有你用户的完整权限。始终审查和测试 hook 命令。
</Note>

---

## Hook 生命周期

Hooks 在 Claude Code 会话的三个主要阶段触发：

```
SessionStart → [UserPromptSubmit → [PreToolUse → PermissionRequest → PostToolUse] × N → Stop] × N → SessionEnd
```

### 事件频率分类

| 频率 | 事件 |
|------|------|
| **每会话一次** | `SessionStart`、`SessionEnd` |
| **每轮一次** | `UserPromptSubmit`、`Stop`、`StopFailure` |
| **每工具调用** | `PreToolUse`、`PostToolUse`、`PostToolUseFailure`、`PermissionRequest`、`PermissionDenied` |

### 完整事件列表

| 事件 | 触发时机 | 可阻塞 |
|------|----------|--------|
| `SessionStart` | 会话开始或恢复 | ❌ |
| `UserPromptSubmit` | 用户提交 prompt 后 | ✅ |
| `PreToolUse` | 工具执行前 | ✅ |
| `PermissionRequest` | 权限对话框出现时 | ✅ |
| `PermissionDenied` | auto 模式拒绝权限时 | ❌ |
| `PostToolUse` | 工具执行成功后 | ✅ |
| `PostToolUseFailure` | 工具执行失败后 | ❌ |
| `Notification` | 发送通知时 | ❌ |
| `SubagentStart` | 子 agent 启动时 | ❌ |
| `SubagentStop` | 子 agent 完成时 | ✅ |
| `TaskCreated` | 创建任务时 | ✅ |
| `TaskCompleted` | 完成任务时 | ✅ |
| `Stop` | Claude 完成响应时 | ✅ |
| `StopFailure` | API 错误导致停止时 | ❌ |
| `TeammateIdle` | Agent 队友即将空闲时 | ✅ |
| `InstructionsLoaded` | 加载指令文件时 | ❌ |
| `ConfigChange` | 配置文件变更时 | ✅ |
| `CwdChanged` | 工作目录变更时 | ❌ |
| `FileChanged` | 监听的文件变更时 | ❌ |
| `WorktreeCreate` | 创建工作树时 | ✅ |
| `WorktreeRemove` | 移除工作树时 | ❌ |
| `PreCompact` | 上下文压缩前 | ✅ |
| `PostCompact` | 上下文压缩后 | ❌ |
| `Elicitation` | MCP 请求用户输入时 | ✅ |
| `ElicitationResult` | 用户响应 MCP 后 | ✅ |

---

## 快速开始

### 5 分钟上手

**步骤 1**：在项目根目录创建 `.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo '文件已修改'"
          }
        ]
      }
    ]
  }
}
```

**步骤 2**：启动 Claude Code 并编辑文件

```bash
claude
# 然后让 Claude 修改一个文件
```

**步骤 3**：看到输出

```
[hook: PostToolUse] 文件已修改
```

### 第一个实用 Hook：阻止 rm -rf

```bash
# 创建脚本
mkdir -p .claude/hooks
cat > .claude/hooks/block-rm.sh << 'EOF'
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command' < /dev/stdin)

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    "hookSpecificOutput": {
      "hookEventName": "PreToolUse",
      "permissionDecision": "deny",
      "permissionDecisionReason": "禁止使用 rm -rf"
    }
  }'
  exit 2
fi
exit 0
EOF
chmod +x .claude/hooks/block-rm.sh
```

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

---

## 配置详解

### 三层结构

```json
{
  "hooks": {
    "事件名": [              // 第一层：选择事件
      {
        "matcher": "模式",    // 第二层：过滤器
        "hooks": [           // 第三层：处理器数组
          {
            "type": "command",
            "command": "脚本"
          }
        ]
      }
    ]
  }
}
```

### Matcher 匹配规则

matcher 的解析方式取决于其字符：

| Matcher 值 | 解析方式 | 示例 |
|------------|----------|------|
| `"*"`、`""` 或省略 | 匹配全部 | 所有事件 |
| 仅字母、数字、`_`、`\|` | 精确匹配或列表 | `Bash`、`Edit\|Write` |
| 包含其他字符 | JavaScript 正则 | `^Notebook.*`、`mcp__.*__write` |

### 各事件的 Matcher 值

| 事件 | Matcher 匹配对象 | 示例值 |
|------|------------------|--------|
| `PreToolUse`、`PostToolUse`、`PermissionRequest` 等 | 工具名称 | `Bash`、`Edit\|Write`、`mcp__memory__.*` |
| `SessionStart` | 启动方式 | `startup`、`resume`、`clear`、`compact` |
| `SessionEnd` | 退出原因 | `clear`、`resume`、`logout`、`other` |
| `Notification` | 通知类型 | `permission_prompt`、`idle_prompt`、`auth_success` |
| `SubagentStart/Stop` | Agent 类型 | `Bash`、`Explore`、`Plan` |
| `PreCompact/PostCompact` | 触发方式 | `manual`、`auto` |
| `ConfigChange` | 配置来源 | `user_settings`、`project_settings`、`local_settings` |
| `FileChanged` | 监听的文件名 | `.envrc\|.env` |
| `StopFailure` | 错误类型 | `rate_limit`、`authentication_failed`、`server_error` |
| `InstructionsLoaded` | 加载原因 | `session_start`、`nested_traversal`、`path_glob_match` |

<Note>
`UserPromptSubmit`、`Stop`、`TeammateIdle`、`TaskCreated`、`TaskCompleted`、`WorktreeCreate`、`WorktreeRemove`、`CwdChanged` 不支持 matcher。
</Note>

### Hook 配置位置

| 位置 | 作用域 | 是否可分享 |
|------|--------|------------|
| `~/.claude/settings.json` | 所有项目 | ❌ |
| `.claude/settings.json` | 单项目 | ✅ (可提交) |
| `.claude/settings.local.json` | 单项目 | ❌ (gitignore) |
| 插件 `hooks/hooks.json` | 插件启用时 | ✅ |
| Skill frontmatter | Skill 激活时 | ✅ |

---

## Hook 事件详解

### PreToolUse

**触发时机**：工具调用创建后，执行前

**Matcher**：工具名称 (`Bash`、`Write`、`Edit`、`Read`、`Glob`、`Grep`、`Agent`、`WebFetch`、`WebSearch`、`AskUserQuestion`、`ExitPlanMode`、MCP 工具)

**输入示例**：
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test",
    "description": "运行测试套件"
  },
  "tool_use_id": "toolu_01ABC123..."
}
```

**决策控制**（使用 `hookSpecificOutput`）：

| 字段 | 值 | 说明 |
|------|-----|------|
| `permissionDecision` | `allow` | 跳过权限提示，自动执行 |
| `permissionDecision` | `deny` | 阻止工具调用 |
| `permissionDecision` | `ask` | 向用户确认 |
| `permissionDecision` | `defer` | 延迟执行（非交互模式） |
| `permissionDecisionReason` | string | 原因说明 |
| `updatedInput` | object | 修改工具输入参数 |
| `additionalContext` | string | 添加到上下文的额外信息 |

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "已验证命令安全"
  }
}
```

---

### PostToolUse

**触发时机**：工具执行成功后

**Matcher**：同 PreToolUse

**输入示例**：
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PostToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.txt",
    "content": "file content"
  },
  "tool_response": {
    "filePath": "/path/to/file.txt",
    "success": true
  },
  "tool_use_id": "toolu_01ABC123..."
}
```

**决策控制**（顶层字段）：

| 字段 | 值 | 说明 |
|------|-----|------|
| `decision` | `block` | 阻塞并提供反馈给 Claude |
| `reason` | string | 阻塞原因 |
| `additionalContext` | string | 额外上下文 |
| `updatedMCPToolOutput` | object | 替换 MCP 工具输出 |

```json
{
  "decision": "block",
  "reason": "测试失败，修复后再继续",
  "additionalContext": "失败的测试：test_login"
}
```

---

### UserPromptSubmit

**触发时机**：用户提交 prompt 后，Claude 处理前

**Matcher**：不支持

**输入示例**：
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "UserPromptSubmit",
  "prompt": "Write a function to calculate factorial"
}
```

**决策控制**（顶层字段）：

| 字段 | 值 | 说明 |
|------|-----|------|
| `decision` | `block` | 阻止 prompt 处理并擦除 |
| `reason` | string | 阻塞原因（对用户显示） |
| `additionalContext` | string | 添加到上下文的字符串 |
| `sessionTitle` | string | 设置会话标题 |

---

### Stop

**触发时机**：Claude 完成响应时

**Matcher**：不支持

**输入示例**：
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "Stop",
  "stop_hook_active": false,
  "last_assistant_message": "I've completed the refactoring..."
}
```

**决策控制**（顶层字段）：

| 字段 | 值 | 说明 |
|------|-----|------|
| `decision` | `block` | 阻止 Claude 停止 |
| `reason` | string | 继续工作的原因 |

```json
{
  "decision": "block",
  "reason": "测试还未通过，请继续修复"
}
```

---

### SessionStart

**触发时机**：会话开始或恢复时

**Matcher**：`startup`、`resume`、`clear`、`compact`

**输入示例**：
```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "SessionStart",
  "source": "startup",
  "model": "claude-sonnet-4-6"
}
```

**特殊能力**：可设置环境变量

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export PATH="$PATH:./node_modules/.bin"' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

---

## 四种 Hook 类型

### Command Hook（最常用）

执行 shell 脚本。

```json
{
  "type": "command",
  "command": "/path/to/script.sh",
  "timeout": 60,
  "async": false,
  "asyncRewake": false,
  "statusMessage": "正在运行..."
}
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ | `"command"` |
| `command` | ✅ | 执行的 shell 命令 |
| `timeout` | ❌ | 超时秒数（默认 600） |
| `async` | ❌ | 后台执行不阻塞 |
| `asyncRewake` | ❌ | 失败时唤醒 Claude |
| `statusMessage` | ❌ | 自定义加载消息 |

---

### HTTP Hook

发送 HTTP POST 请求。

```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/pre-tool-use",
  "headers": {
    "Authorization": "Bearer $MY_TOKEN"
  },
  "allowedEnvVars": ["MY_TOKEN"],
  "timeout": 30
}
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ | `"http"` |
| `url` | ✅ | POST 目标 URL |
| `headers` | ❌ | 请求头（支持 `$VAR` 插值） |
| `allowedEnvVars` | ❌ | 允许插值的变量列表 |
| `timeout` | ❌ | 超时秒数（默认 30） |

<Note>
非 2xx 响应、连接失败、超时都是非阻塞错误。要阻塞工具调用，返回 2xx + JSON body `{"decision": "block"}`。
</Note>

---

### Prompt Hook

使用 LLM 单次判断。

```json
{
  "type": "prompt",
  "prompt": "判断是否应该停止：$ARGUMENTS。检查所有任务是否完成。",
  "model": "haiku",
  "timeout": 30
}
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ | `"prompt"` |
| `prompt` | ✅ | 发送给 LLM 的 prompt（`$ARGUMENTS` 插入输入 JSON） |
| `model` | ❌ | 使用的模型（默认快速模型） |
| `timeout` | ❌ | 超时秒数（默认 30） |

**LLM 返回格式**：
```json
{
  "ok": true,
  "reason": "所有任务已完成"
}
```

---

### Agent Hook

使用 agent 多轮验证（可访问工具）。

```json
{
  "type": "agent",
  "prompt": "验证所有单元测试是否通过。运行测试套件并检查结果。$ARGUMENTS",
  "timeout": 120
}
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ | `"agent"` |
| `prompt` | ✅ | agent 的任务描述 |
| `timeout` | ❌ | 超时秒数（默认 60） |

Agent 可使用 Read、Grep、Glob 等工具验证条件。

---

## 输入输出规范

### 退出码含义

| 退出码 | 含义 | 效果 |
|--------|------|------|
| `0` | 成功 | 允许操作，处理 stdout JSON |
| `2` | 阻塞错误 | 阻止操作，显示 stderr 给用户 |
| 其他 | 非阻塞错误 | 继续执行，记录 debug 日志 |

<Warning>
只有退出码 2 能阻塞操作。exit 1 被视为非阻塞错误，Claude 继续执行。
</Warning>

### JSON 输出格式

**通用控制字段**：

```json
{
  "continue": false,
  "stopReason": "构建失败",
  "suppressOutput": false,
  "systemMessage": "警告信息"
}
```

**PreToolUse 专用**（嵌套在 `hookSpecificOutput` 中）：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "原因"
  }
}
```

**PostToolUse/Stop 专用**（顶层字段）：

```json
{
  "decision": "block",
  "reason": "原因"
}
```

---

## 实战示例

### 1. 文件保存后异步运行测试

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/run-tests.sh",
            "async": true,
            "timeout": 300
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/run-tests.sh
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# 只测试源文件
if [[ "$FILE_PATH" != *.ts && "$FILE_PATH" != *.js ]]; then
  exit 0
fi

# 运行测试
RESULT=$(npm test 2>&1)
EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
  echo "{\"systemMessage\": \"测试通过\"}"
else
  echo "{\"systemMessage\": \"测试失败：$RESULT\"}"
fi
```

---

### 2. 安全拦截危险命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command' < /dev/stdin)

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    "hookSpecificOutput": {
      "hookEventName": "PreToolUse",
      "permissionDecision": "deny",
      "permissionDecisionReason": "禁止使用 rm -rf"
    }
  }'
  exit 2
fi
exit 0
```

---

### 3. Stop Hook 验证任务完成

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "验证所有单元测试是否通过。运行测试套件并检查结果。$ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

---

### 4. 匹配 MCP 工具

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Memory operation' >> ~/mcp-operations.log"
          }
        ]
      },
      {
        "matcher": "mcp__.*__write.*",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/validate-write.py"
          }
        ]
      }
    ]
  }
}
```

---

### 5. 会话开始加载上下文

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/load-context.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/load-context.sh
# 读取最近的 git commit
LAST_COMMIT=$(git log -1 --oneline 2>/dev/null)
echo "最近提交：$LAST_COMMIT"

# 设置环境变量
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo "export CURRENT_BRANCH=$(git branch --show-current)" >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

---

### 6. Skill 中定义 Hook

```yaml
---
name: secure-operations
description: 执行安全操作检查
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---

此 Skill 在每个 Bash 命令前运行安全检查脚本。
```

---

## 调试与故障排除

### 启用调试日志

```bash
# 输出到指定文件
claude --debug-file /tmp/claude-debug.log

# 或使用默认位置
claude --debug

# 日志位置：~/.claude/debug/<session-id>.txt
```

### 详细日志级别

```bash
CLAUDE_CODE_DEBUG_LOG_LEVEL=verbose claude
```

### 调试输出示例

```text
[DEBUG] Executing hooks for PostToolUse:Write
[DEBUG] Found 1 hook commands to execute
[DEBUG] Executing hook command: /path/to/script.sh with timeout 600000ms
[DEBUG] Hook command completed with status 0: output here
```

### 常见问题

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| Hook 不触发 | matcher 不匹配 | 检查事件名和 matcher 值 |
| JSON 解析失败 | stdout 有其他输出 | 确保只输出 JSON（其他输出到 stderr） |
| 无限 Stop 循环 | 没有检查 `stop_hook_active` | 在脚本中检查该字段 |
| 环境变量不生效 | 没有使用 `CLAUDE_ENV_FILE` | SessionStart/CwdChanged/FileChanged 使用该文件持久化变量 |
| 权限不足 | 脚本没有执行权限 | `chmod +x script.sh` |

---

## 最佳实践

### 脚本组织

```
.claude/
├── settings.json          # Hook 配置
├── settings.local.json    # 本地配置（不提交）
└── hooks/
    ├── block-rm.sh        # 安全拦截
    ├── run-tests.sh       # 自动测试
    ├── format-code.sh     # 代码格式化
    └── audit-changes.sh   # 审计日志
```

### 安全建议

1. **验证输入**：不信任任何输入数据
2. **引用变量**：使用 `"$VAR"` 而非 `$VAR`
3. **使用绝对路径**：用 `$CLAUDE_PROJECT_DIR` 引用项目根目录
4. **路径检查**：检查 `..` 防止目录遍历
5. **避免敏感文件**：不要读取 `.env`、`.git/`、密钥等

### 性能建议

1. **保持 Hook 快速**：SessionStart 等高频事件尤其重要
2. **使用 async**：长时间任务使用异步执行
3. **使用 `if` 过滤**：减少不必要的脚本执行
4. **超时设置**：为所有 Hook 设置合理的 timeout

### 测试 Hook

```bash
# 手动测试脚本
echo '{"tool_input":{"command":"rm -rf /tmp"}}' | ./block-rm.sh

# 查看输出
cat

# 检查退出码
echo $?
```

---

## 附录：速查表

### 事件速查

| 事件 | 可阻塞 | 决策字段位置 | 典型用途 |
|------|--------|--------------|----------|
| `PreToolUse` | ✅ | `hookSpecificOutput` | 安全校验 |
| `PostToolUse` | ✅ | 顶层 `decision` | 质量检查 |
| `UserPromptSubmit` | ✅ | 顶层 `decision` | Prompt 过滤 |
| `Stop` | ✅ | 顶层 `decision` | 完成验证 |
| `SessionStart` | ❌ | `additionalContext` | 加载上下文 |
| `SessionEnd` | ❌ | 无 | 清理/日志 |

### 退出码速查

```
exit 0  → 成功，允许操作
exit 2  → 阻塞，阻止操作
exit 1  → 非阻塞错误（继续执行）
```

### 环境变量

| 变量 | 说明 |
|------|------|
| `$CLAUDE_PROJECT_DIR` | 项目根目录 |
| `$CLAUDE_PLUGIN_ROOT` | 插件目录 |
| `$CLAUDE_CODE_REMOTE` | 远程环境标识 |
| `$CLAUDE_ENV_FILE` | 环境变量持久化文件（SessionStart/CwdChanged/FileChanged） |

---

*基于官方 Hooks Reference 整理 | 适用于 Claude Code v2.1.89+*
