# Claude Code Commands 完整命令参考指南

> 基于官方文档 (https://code.claude.com/docs/en/commands) 的完整学习笔记

---

## 目录

- [命令概述](#命令概述)
- [完整命令列表](#完整命令列表)
- [命令分类详解](#命令分类详解)
- [Bundled Skills](#bundled-skills)
- [MCP 命令](#mcp-命令)
- [使用技巧](#使用技巧)

---

## 命令概述

**Commands** 从会话内部控制 Claude Code，提供快速方式来：
- 切换模型
- 管理权限
- 清空上下文
- 运行工作流
- 更多功能

**使用方法**：
- 输入 `/` 查看所有可用命令
- 输入 `/` + 字母进行过滤

**注意**：
- 不是所有命令都对每个用户可见
- 可用性取决于平台、套餐和环境
- `/desktop` 仅在 macOS 和 Windows 上显示
- `/upgrade` 仅在 Pro 和 Max 套餐上显示

---

## 完整命令列表

### A

| 命令 | 作用 |
|------|------|
| `/add-dir <path>` | 添加工作目录以在当前会话中访问文件 |
| `/agents` | 管理 [agent](/en/sub-agents) 配置 |

### B

| 命令 | 作用 |
|------|------|
| `/batch <instruction>` | **[Skill]** 并行编排代码库的大规模变更 |
| `/branch [name]` | 在当前点创建对话分支，别名 `/fork` |
| `/btw <question>` | 询问快速副作用问题，不添加到对话历史 |

### C

| 命令 | 作用 |
|------|------|
| `/chrome` | 配置 [Claude in Chrome](/en/chrome) 设置 |
| `/claude-api` | **[Skill]** 加载 Claude API 参考材料 |
| `/clear` | 开始新对话，清空上下文，别名 `/reset`、`/new` |
| `/color [color\|default]` | 设置提示栏颜色 |
| `/compact [instructions]` | 压缩对话，可选focus指令 |
| `/config` | 打开设置界面，别名 `/settings` |
| `/context` | 将当前上下文使用可视化为彩色网格 |
| `/copy [N]` | 复制最后助手响应到剪贴板 |
| `/cost` | 显示 token 使用统计 |

### D

| 命令 | 作用 |
|------|------|
| `/debug [description]` | **[Skill]** 启用调试日志并排查问题 |
| `/desktop` | 在 Claude Code Desktop 应用中继续当前会话，别名 `/app` |
| `/diff` | 打开交互式 diff 查看器 |
| `/doctor` | 诊断和验证 Claude Code 安装和设置 |

### E

| 命令 | 作用 |
|------|------|
| `/effort [level\|auto]` | 设置模型努力程度 |
| `/exit` | 退出 CLI，别名 `/quit` |
| `/export [filename]` | 导出当前对话为纯文本 |
| `/extra-usage` | 配置额外使用量以在达到限制时继续工作 |

### F

| 命令 | 作用 |
|------|------|
| `/fast [on\|off]` | 切换快速模式 |
| `/feedback [report]` | 提交关于 Claude Code 的反馈，别名 `/bug` |
| `/focus` | 切换 focus 视图 |

### H

| 命令 | 作用 |
|------|------|
| `/heapdump` | 写入 JavaScript 堆快照以诊断高内存使用 |
| `/help` | 显示帮助和可用命令 |
| `/hooks` | 查看钩子配置 |

### I

| 命令 | 作用 |
|------|------|
| `/ide` | 管理 IDE 集成并显示状态 |
| `/init` | 使用 `CLAUDE.md` 指南初始化项目 |
| `/insights` | 生成分析你的 Claude Code 会话的报告 |
| `/install-github-app` | 为仓库设置 Claude GitHub Actions 应用 |
| `/install-slack-app` | 安装 Claude Slack 应用 |

### K

| 命令 | 作用 |
|------|------|
| `/keybindings` | 打开或创建键盘绑定配置文件 |

### L

| 命令 | 作用 |
|------|------|
| `/less-permission-prompts` | **[Skill]** 扫描并添加允许列表以减少权限提示 |
| `/login` | 登录到你的 Anthropic 账户 |
| `/logout` | 登出 Anthropic 账户 |
| `/loop [interval] [prompt]` | **[Skill]** 重复运行提示，别名 `/proactive` |

### M

| 命令 | 作用 |
|------|------|
| `/mcp` | 管理 MCP 服务器连接和 OAuth 认证 |
| `/memory` | 编辑 `CLAUDE.md` 记忆文件 |
| `/mobile` | 显示下载 Claude 移动应用的二维码 |
| `/model [model]` | 选择或更改 AI 模型 |

### P

| 命令 | 作用 |
|------|------|
| `/passes` | 分享免费一周的 Claude Code 给朋友 |
| `/permissions` | 管理工具权限规则，别名 `/allowed-tools` |
| `/plan [description]` | 直接进入计划模式 |
| `/plugin` | 管理 Claude Code 插件 |
| `/powerup` | 通过快速互动课程发现 Claude Code 功能 |
| `/pr-comments [PR]` | {/* max-version: 2.1.90 */}已移除 |
| `/privacy-settings` | 查看和更新隐私设置 |

### R

| 命令 | 作用 |
|------|------|
| `/recap` | 按需生成当前会话的单行摘要 |
| `/release-notes` | 在交互式版本查看器中查看变更日志 |
| `/reload-plugins` | 重新加载所有活动插件 |
| `/remote-control` | 使会话可从 claude.ai 远程控制，别名 `/rc` |
| `/remote-env` | 配置默认远程环境 |
| `/rename [name]` | 重命名当前会话 |
| `/resume [session]` | 恢复对话，别名 `/continue` |
| `/review [PR]` | 在本地会话中审查 pull request |
| `/rewind` | 回滚对话和/或代码，别名 `/checkpoint`、`/undo` |

### S

| 命令 | 作用 |
|------|------|
| `/sandbox` | 切换沙盒模式 |
| `/schedule [description]` | 创建、更新、列出或运行常规任务，别名 `/routines` |
| `/security-review` | 分析当前分支的待处理变更以查找安全漏洞 |
| `/setup-bedrock` | 配置 Amazon Bedrock 认证 |
| `/setup-vertex` | 配置 Google Vertex AI 认证 |
| `/simplify [focus]` | **[Skill]** 审查最近变更的文件并修复问题 |
| `/skills` | 列出可用 skills |
| `/stats` | 可视化每日使用情况 |
| `/status` | 打开设置界面（状态标签） |
| `/statusline` | 配置 Claude Code 的状态行 |
| `/stickers` | 订购 Claude Code 贴纸 |

### T

| 命令 | 作用 |
|------|------|
| `/tasks` | 列出和管理后台任务，别名 `/bashes` |
| `/team-onboarding` | 从你的使用历史生成团队入职指南 |
| `/teleport` | 将 Web 会话拉入此终端，别名 `/tp` |
| `/terminal-setup` | 配置终端键盘绑定 |
| `/theme` | 更改颜色主题 |
| `/tui [default\|fullscreen]` | 设置终端 UI 渲染器 |

### U

| 命令 | 作用 |
|------|------|
| `/ultraplan <prompt>` | 在 ultraplan 会话中起草计划 |
| `/ultrareview [PR]` | 在云沙盒中运行深度多 agent 代码审查 |
| `/upgrade` | 打开升级页面 |
| `/usage` | 显示套餐使用限制和速率限制状态 |

### V

| 命令 | 作用 |
|------|------|
| `/vim` | {/* max-version: 2.1.91 */}已移除 |
| `/voice` | 切换语音输入 |

### W

| 命令 | 作用 |
|------|------|
| `/web-setup` | 使用本地 `gh` CLI 凭据连接 GitHub 账户 |

---

## 命令分类详解

### 会话管理

| 命令 | 作用 |
|------|------|
| `/clear` | 清空上下文开始新对话 |
| `/compact` | 压缩对话历史保留关键信息 |
| `/branch` | 创建对话分支 |
| `/resume` | 恢复之前的对话 |
| `/rewind` | 回滚到之前的点 |
| `/exit` | 退出 CLI |
| `/rename` | 重命名会话 |
| `/recap` | 生成会话摘要 |

### 模型配置

| 命令 | 作用 |
|------|------|
| `/model` | 切换 AI 模型 |
| `/effort` | 设置努力程度（low/medium/high/xhigh/max） |
| `/fast` | 切换快速模式 |
| `/color` | 设置提示栏颜色 |

### 权限与安全

| 命令 | 作用 |
|------|------|
| `/permissions` | 管理工具权限规则 |
| `/sandbox` | 切换沙盒模式 |
| `/security-review` | 安全漏洞审查 |
| `/privacy-settings` | 隐私设置（Pro/Max） |

### 项目初始化

| 命令 | 作用 |
|------|------|
| `/init` | 初始化项目与 CLAUDE.md |
| `/add-dir` | 添加工作目录 |
| `/memory` | 编辑记忆文件 |

### 开发与调试

| 命令 | 作用 |
|------|------|
| `/debug` | 调试日志和故障排除 |
| `/review` | PR 审查 |
| `/ultrareview` | 深度多 agent 代码审查 |
| `/batch` | 大规模并行变更 |
| `/simplify` | 代码简化和优化 |

### 状态与统计

| 命令 | 作用 |
|------|------|
| `/stats` | 使用情况可视化 |
| `/cost` | Token 使用统计 |
| `/context` | 上下文使用可视化 |
| `/usage` | 套餐使用限制 |
| `/status` | 版本、模型、账户状态 |

### 扩展与集成

| 命令 | 作用 |
|------|------|
| `/agents` | 管理子 agent 配置 |
| `/skills` | 列出可用 skills |
| `/hooks` | 查看钩子配置 |
| `/mcp` | 管理 MCP 服务器 |
| `/plugin` | 管理插件 |
| `/reload-plugins` | 重新加载插件 |
| `/ide` | IDE 集成 |

### 自动化

| 命令 | 作用 |
|------|------|
| `/loop` | 重复运行提示 |
| `/schedule` | 创建和管理常规任务 |

### 远程与同步

| 命令 | 作用 |
|------|------|
| `/remote-control` | 远程控制 |
| `/teleport` | 拉取 Web 会话 |
| `/desktop` | 在 Desktop 应用继续 |
| `/remote-env` | 配置远程环境 |

### 账户与设置

| 命令 | 作用 |
|------|------|
| `/login` | 登录 |
| `/logout` | 登出 |
| `/config` | 打开设置 |
| `/theme` | 更改主题 |
| `/keybindings` | 键盘绑定配置 |
| `/upgrade` | 升级套餐 |
| `/passes` | 分享免费周 |

### 帮助与诊断

| 命令 | 作用 |
|------|------|
| `/help` | 显示帮助 |
| `/doctor` | 诊断安装 |
| `/feedback` | 提交反馈 |
| `/release-notes` | 变更日志 |
| `/insights` | 使用分析报告 |
| `/team-onboarding` | 团队入职指南 |

---

## Bundled Skills

以下命令是**Bundled Skills**，不是内置命令：

| Skill | 作用 |
|-------|------|
| `/batch` | 并行编排大规模代码库变更 |
| `/claude-api` | 加载 Claude API 参考材料 |
| `/debug` | 调试日志和故障排除 |
| `/less-permission-prompts` | 扫描并添加允许列表 |
| `/loop` | 重复运行提示 |
| `/simplify` | 代码审查和优化 |

**特点**：
- 使用与自定义 skill 相同的机制
- 基于提示的执行（交给 Claude 编排）
- Claude 可在相关时自动调用
- 可以在 `/skills` 中查看

---

## MCP 命令

MCP 服务器可以暴露作为命令显示的提示：

**格式**：`/mcp__<server>__<prompt>`

**示例**：
```
/mcp__github__search-repos
/mcp__playwright__navigate
```

这些命令从连接的服务器动态发现。

---

## 使用技巧

### 快速过滤

输入 `/` 后继续输入字母快速过滤命令：
```
/mo   → 显示匹配 model 的命令
/cl   → 显示匹配 clear、color、compact 等的命令
```

### 常用命令组合

**开始新项目**：
```
/init → 初始化项目
/memory → 添加项目约定
```

**调试问题**：
```
/debug → 启用调试日志
/doctor → 诊断安装
```

**代码审查**：
```
/review [PR] → 本地 PR 审查
/ultrareview [PR] → 深度云审查
/security-review → 安全漏洞分析
```

**大规模变更**：
```
/batch migrate src/ from Solid to React
/simplify focus on memory efficiency
```

**并行工作**：
```
/loop 5m check if the deploy finished
```

### 参数格式说明

| 格式 | 说明 |
|------|------|
| `<arg>` | 必需参数 |
| `[arg]` | 可选参数 |
| `[color\|default]` | 可选参数，多个选项 |

### 别名速查

| 命令 | 别名 |
|------|------|
| `/clear` | `/reset`、`/new` |
| `/config` | `/settings` |
| `/exit` | `/quit` |
| `/resume` | `/continue` |
| `/rewind` | `/checkpoint`、`/undo` |
| `/desktop` | `/app` |
| `/remote-control` | `/rc` |
| `/teleport` | `/tp` |
| `/schedule` | `/routines` |
| `/loop` | `/proactive` |
| `/tasks` | `/bashes` |
| `/feedback` | `/bug` |
| `/branch` | `/fork` |
| `/mobile` | `/ios`、`/android` |

---

## 相关资源

- **[Skills](/en/skills)** - 创建自定义命令
- **[Interactive mode](/en/interactive-mode)** - 键盘快捷键、Vim 模式、命令历史
- **[CLI reference](/en/cli-reference)** - 启动时标志
- **[Sub-agents](/en/sub-agents)** - 创建和使用子 agent
- **[Hooks](/en/hooks)** - 自动化工作流

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-04-18 | 1.0.0 | 基于官方文档整理 |

---

## 已移除命令

| 命令 | 移除版本 | 替代方案 |
|------|----------|----------|
| `/autofix-pr [PR]` | 未知 | 手动审查 PR 或使用 `/review` |
| `/pr-comments [PR]` | v2.1.91 | 直接让 Claude 查看 PR 评论 |
| `/vim` | v2.1.92 | 使用 `/config` → Editor mode |

---

*本文档基于官方文档 https://code.claude.com/docs/en/commands 整理。*
