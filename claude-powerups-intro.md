# Claude Powerups 入门教程

## 1. @ 引用代码

**简介**  
不用复制粘贴代码，用 `@` 直接附加文件到对话，Claude 会自动读取。

**具体使用**
```
@src/auth.ts          # 附加单个文件
@src/app.ts:42        # 引用特定行
@src/app.ts:42-58     # 引用段落（行范围）
@src/components/      # 附加整个目录
```

**引用段落的其他方式**
```
@src/app.ts 看第 3 段的认证逻辑     # @ 文件 + 文字说明
@authenticateUser                  # 直接引用函数名（如果支持）
```

---

## 2. 权限模式

**简介**  
控制 Claude 动手前的询问程度，平衡安全与效率。

**具体使用**  
按 `Shift+Tab` 循环切换：

| 模式 | 行为 | 场景 |
|------|------|------|
| `default` | 每次编辑都问 | 日常开发 |
| `accept edits` | 改代码不问，执行命令才问 | 信任的迭代 |
| `plan` | 只出方案，不动手 | 大重构前先审查 |
| `auto` | Claude 自己判断 | 无人值守任务 |

---

## 3. 撤销与分支

**简介**  
走错路随时回滚，想试方案可分叉对话。

**具体使用**

| 命令 | 作用 |
|------|------|
| `Esc Esc` | 打开 `/rewind`，选时间点回滚 |
| `/branch` | 分叉当前对话，保留原分支 |
| `/resume <id>` | 切回指定分支 |
| `/clear` | 清空对话，保留文件改动 |

**回滚选项**（`/rewind`）：

1. **Restore code and conversation** — 代码和对话全回滚
2. **Restore conversation** — 只回滚对话，代码不变
3. **Restore code** — 只回滚代码，对话不变
4. **Summarize from here** — 从该点总结上下文
5. **Never mind** — 取消，不做任何操作

---

## 4. 后台任务

**简介**  
长任务后台跑，不阻塞对话，完成自动通知。

**具体使用**
```bash
npm test &          # 后台运行测试
bun build &         # 后台构建
/tasks              # 查看所有任务状态
```

---

## 5. CLAUDE.md 项目规则

**简介**  
项目级的"使用说明书"，Claude 每次会话自动读取。

**具体使用**
```bash
/memory             # 打开编辑
/init               # 自动生成模板
```

**内容示例**
```markdown
- 测试命令：bun test
- 禁止修改：src/legacy/
- 代码风格：优先函数式组件
```

**作用域**：项目级 → 用户级 (`~/.claude/`) → 目录级

---

## 6. 自定义技能

**简介**  
把常用提示词存成命令，一键调用。

**具体使用**
```bash
# 创建
mkdir -p .claude/skills/deploy
echo "1. 构建 2. 测试 3. 部署" > .claude/skills/deploy/SKILL.md

# 使用
/deploy             # 自动执行整套流程
/skills             # 查看已安装技能
```

---

## 7. MCP 工具扩展

**简介**  
连接外部服务（数据库、Slack、浏览器），让 Claude 获得新能力。

**具体使用**
```bash
/mcp                            # 交互管理
claude mcp add 名字 -- npx 包   # 命令行添加
```

**示例**
```bash
claude mcp add playwright -- npx playwright-mcp
claude mcp add postgres -- npx postgres-mcp
```

---

## 8. 子代理

**简介**  
Claude 派出多个"分身"并行工作，适合多目录搜索、多任务并发。

**具体使用**
```
用户：用子代理搜索这 5 个目录，找所有 API 端点
→ 自动分派多个代理并行执行
```

**自定义代理**
- 在 `.claude/agents/` 下创建
- 例如：`测试员.md`、`审查员.md`、`文档员.md`
- `/agents` 查看管理

---

## 9. 模型与设置

**简介**  
根据任务难度选择模型，平衡速度与深度。

**具体使用**

| 命令 | 作用 |
|------|------|
| `/model` | 切换 Opus(难)/Sonnet(日常)/Haiku(快) |
| `/effort` | 调整思考深度 (low/medium/high) |
| `/fast` | 切换快速输出模式 |

---

## 10. 自动化 Hooks

**简介**  
配置事件触发的自动脚本，如工具调用前、会话开始时。

**具体使用**
```bash
/hooks    # 查看已配置的钩子
```

**配置位置**
- `~/.claude/settings.json`（全局）
- `.claude/settings.local.json`（项目级）

---

## 11. 远程控制

**简介**  
会话同步到手机/网页，多设备无缝切换。

**具体使用**
```bash
/remote-control       # 生成远程访问链接
/teleport             # 把网页会话拉回当前终端
```

---

## 命令速查表

| 命令 | 作用 |
|------|------|
| `@文件` | 附加文件到对话 |
| `Shift+Tab` | 切换权限模式 |
| `Esc Esc` | 打开回滚界面 |
| `/branch` | 分叉对话 |
| `/resume <id>` | 切换分支 |
| `/clear` | 清空对话历史 |
| `/tasks` | 查看后台任务 |
| `/model` | 切换模型 |
| `/effort` | 调整思考深度 |
| `/fast` | 快速模式开关 |
| `/memory` | 编辑 CLAUDE.md |
| `/init` | 生成 CLAUDE.md 模板 |
| `/permissions` | 预设允许的命令 |
| `/hooks` | 查看自动化钩子 |
| `/skills` | 查看自定义技能 |
| `/agents` | 管理子代理 |
| `/mcp` | 管理 MCP 服务器 |
| `/remote-control` | 远程访问会话 |
| `/teleport` | 转移会话到当前终端 |

---

## 典型工作流

| 场景 | 做法 |
|------|------|
| 新人入职 | `/init` 生成项目约定 |
| 日常开发 | `@文件` 引用 + default 模式 |
| 大重构 | plan 模式先出方案 |
| 跑测试 | `npm test &` 后台运行 |
| 改错了 | `Esc Esc` 回滚 |
| 试方案 | `/branch` 分叉对比 |
| 复杂调试 | `/effort high` + Opus 模型 |
| 快速修改 | Haiku 模型 + /fast |
