# Claude Code Routines 自动化例行任务指南

> 基于官方文档 (https://code.claude.com/docs/en/routines) 整理
> 
> 最后更新：2026 年 4 月 19 日
> 
> **注意**：Routines 目前处于研究预览阶段，行为、限制和 API 可能会变化。

---

## 目录

- [概述](#概述)
- [示例用例](#示例用例)
- [创建 Routine](#创建-routine)
- [配置触发器](#配置触发器)
- [管理 Routines](#管理-routines)
- [使用与限制](#使用与限制)

---

## 概述

**Routine** 是保存的 Claude Code 配置：包含 prompt、仓库和 connectors，打包后自动运行。Routines 在 Anthropic 管理的云端基础设施执行，笔记本电脑关闭时也能继续工作。

### 触发器类型

| 类型 | 说明 |
|------|------|
| **Scheduled** | 定时运行：每小时、每天、每周 |
| **API** | HTTP POST 请求触发，带 bearer token |
| **GitHub** | 响应仓库事件：PR、Release |

单个 routine 可组合多种触发器。例如 PR review routine 可定时运行、从 deploy 脚本触发、并响应每个新 PR。

### 可用性

- Pro、Max、Team、Enterprise 计划
- 启用 [Claude Code on the web](/en/claude-code-on-the-web)
- 管理地址：[claude.ai/code/routines](https://claude.ai/code/routines)
- CLI：`/schedule`

---

## 示例用例

| 场景 | 触发器 | 工作内容 |
|------|--------|----------|
| **Backlog 维护** | Scheduled（每工作日） | 读取新 issues，应用标签，分配 owner，发 Slack 总结 |
| **Alert 分诊** | API（监控工具调用） | 拉取堆栈，关联 commits，开 draft PR 提议修复 |
| **定制 Code Review** | GitHub（PR opened） | 应用团队检查清单，留 inline 评论，发总结评论 |
| **Deploy 验证** | API（CD pipeline 调用） | 运行 smoke checks，扫描错误日志，发布 go/no-go |
| **Docs Drift** | Scheduled（每周） | 扫描合并 PR，标记过期文档，开更新 PR |
| **Library Port** | GitHub（PR closed/merged） | 将变更移植到另一语言的 SDK，开匹配 PR |

---

## 创建 Routine

从 Web、Desktop app 或 CLI 创建。所有方式写入同一云端账户。

### 从 Web 创建

| 步骤 | 内容 |
|------|------|
| **1. 打开创建表单** | 访问 claude.ai/code/routines，点击 **New routine** |
| **2. 名称和 Prompt** | 描述性名称 + 自包含的 prompt（必须明确目标和成功标准） |
| **3. 选择仓库** | 添加 GitHub 仓库，运行时从默认分支克隆 |
| **4. 选择环境** | Cloud environment：网络访问、环境变量、setup script |
| **5. 选择触发器** | Schedule / GitHub event / API |
| **6. 检查 Connectors** | 默认包含所有 MCP connectors，移除不需要的 |
| **7. 创建** | 点击 **Create**，点击 **Run now** 立即运行 |

### 从 CLI 创建

```bash
/schedule                        # 对话式创建
/schedule daily PR review at 9am # 直接描述创建
/schedule list                   # 列出所有 routines
/schedule update                 # 修改 routine
/schedule run                    # 立即触发
```

> CLI `/schedule` 仅创建 scheduled routines。添加 API 或 GitHub 触发器需在 Web 编辑。

### 从 Desktop App 创建

点击 **New task** → **New remote task**

> **New local task** 是本地定时任务，不是 routine。

---

## 配置触发器

### Schedule Trigger

**预设频率**：hourly、daily、weekdays、weekly

**时间处理**：
- 输入本地时间，自动转换
- 运行可能延迟几分钟（stagger）

**自定义 Cron**：
- 用 `/schedule update` 设置
- 最小间隔：1 小时

### API Trigger

从 Web 添加到现有 routine：

| 步骤 | 内容 |
|------|------|
| **1. 编辑 Routine** | 点击 routine → pencil icon |
| **2. 添加 API Trigger** | **Add another trigger** → **API** |
| **3. 复制 URL 和 Token** | 复制 URL，点击 **Generate token**（仅显示一次） |

#### 触发 API

```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCDEFGHJKLMNOPQRSTUVW/fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod."}'
```

**返回**：

```json
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

> **注意**：`/fire` 端点使用 `experimental-cc-routine-2026-04-01` beta header，可能变化。

**Token 管理**：
- 每个 routine 有独立 token
- **Regenerate** 或 **Revoke** 可在 Web 操作

### GitHub Trigger

从 Web UI 配置：

| 步骤 | 内容 |
|------|------|
| **1. 编辑 Routine** | 点击 routine → pencil icon |
| **2. 添加 GitHub Trigger** | **Add another trigger** → **GitHub event** |
| **3. 安装 Claude GitHub App** | 必须安装才能接收 webhook |
| **4. 配置触发器** | 选择仓库、事件、可选过滤器 |

#### 支持的事件

| 事件 | 触发时机 |
|------|----------|
| **Pull request** | opened、closed、assigned、labeled、synchronized 等 |
| **Release** | created、published、edited、deleted |

#### PR 过滤器

| 过滤字段 | 匹配内容 |
|----------|----------|
| Author | PR 作者 GitHub 用户名 |
| Title | PR 标题 |
| Body | PR 描述 |
| Base branch | 目标分支 |
| Head branch | 来源分支 |
| Labels | 应用标签 |
| Is draft | 是否草稿 |
| Is merged | 是否已合并 |

**操作符**：equals、contains、starts with、is one of、matches regex

> `matches regex` 测试整个字段值，非子串。匹配 `hotfix` 需写 `.*hotfix.*`。

**示例组合**：
- Base branch `main` + Head branch contains `auth-provider`
- Is draft `false`（仅 ready-for-review）
- Labels include `needs-backport`

#### Session 映射

每个匹配 GitHub 事件启动新 session。不跨事件复用 session。

---

## 管理 Routines

### 查看和操作 Runs

点击 run 打开完整 session：
- 查看 Claude 做了什么
- 审查更改
- 创建 PR
- 继续对话

### 编辑和控制

| 操作 | 说明 |
|------|------|
| **Run now** | 立即运行 |
| **Toggle Repeats** | 暂停/恢复定时 |
| **Pencil icon** | 编辑：名称、prompt、仓库、环境、connectors、触发器 |
| **Delete icon** | 删除 routine（past sessions 保留） |

### 仓库和分支权限

| 设置 | 说明 |
|------|------|
| **默认** | 仅推送 `claude/` 前缀分支 |
| **Allow unrestricted branch pushes** | 允许推送任意分支 |

### Connectors

默认包含所有 MCP connectors。移除不需要的以限制工具访问。

从 **Settings > Connectors** 或 `/schedule update` 管理。

### Environments

Cloud environment 控制：
- **Network access**：网络访问级别
- **Environment variables**：API keys、tokens
- **Setup script**：安装依赖（结果缓存）

参见 [Cloud environment](/en/claude-code-on-the-web#the-cloud-environment)。

---

## 使用与限制

### 消耗机制

- 与交互式 session 相同方式消耗订阅用量
- 每账户每日运行次数上限
- 查看：claude.ai/code/routines 或 claude.ai/settings/usage

### 超额处理

| 情况 | 结果 |
|------|------|
| **启用 Extra usage** | 按 metered overage 继续运行 |
| **未启用 Extra usage** | 拒绝额外运行直到窗口重置 |

从 **Settings > Billing** 启用 extra usage。

---

## 相关资源

- [`/loop` 和 in-session scheduling](/en/scheduled-tasks) — CLI session 内的本地定时任务
- [Desktop scheduled tasks](/en/desktop-scheduled-tasks) — 本机运行的本地定时任务
- [Cloud environment](/en/claude-code-on-the-web#the-cloud-environment) — 云 session 运行环境
- [MCP connectors](/en/mcp) — 连接 Slack、Linear、Google Drive 等外部服务
- [GitHub Actions](/en/github-actions) — CI pipeline 中运行 Claude