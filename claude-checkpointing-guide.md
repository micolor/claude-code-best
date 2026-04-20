# Claude Code Checkpointing 会话检查点指南

> 基于官方文档 (https://code.claude.com/docs/en/checkpointing) 的完整学习笔记

---

## 目录

- [概述](#概述)
- [检查点工作机制](#检查点工作机制)
- [回滚与压缩功能](#回滚与压缩功能)
- [常用场景](#常用场景)
- [限制说明](#限制说明)

---

## 概述

**Checkpointing** 是 Claude Code 的自动状态追踪系统，可追踪 Claude 的文件编辑操作，让你能够：

- 快速撤销更改
- 回滚到之前的状态
- 压缩对话历史释放上下文空间

这为大胆尝试大规模任务提供了安全保障。

---

## 检查点工作机制

### 自动追踪

Claude Code 自动追踪所有通过文件编辑工具进行的更改：

| 特性 | 说明 |
|------|------|
| **检查点创建** | 每个用户提示创建一个新检查点 |
| **持久化** | 检查点跨会话保留，可在恢复的对话中访问 |
| **自动清理** | 会话在 30 天后自动清理（可配置） |

### 追踪范围

| 操作类型 | 是否追踪 |
|----------|----------|
| Claude 的 Edit/Write 工具编辑 | ✅ 追踪 |
| Bash 命令修改文件 (rm/mv/cp) | ❌ 不追踪 |
| 手动编辑文件 | ❌ 不追踪 |
| 其他会话的编辑 | ❌ 不追踪（除非修改了相同文件） |

---

## 回滚与压缩功能

### 打开回滚菜单

两种方式：
- 按 `Esc` 两次 (`Esc` + `Esc`)
- 使用 `/rewind` 命令

### 操作选项

选择一个检查点后，可执行以下操作：

| 操作 | 说明 |
|------|------|
| **Restore code and conversation** | 恢复代码和对话到该点 |
| **Restore conversation** | 回滚对话，保留当前代码 |
| **Restore code** | 恢复文件更改，保留对话 |
| **Summarize from here** | 压缩该点之后的对话为摘要 |
| **Never mind** | 返回消息列表不做更改 |

### Restore vs Summarize 区别

| 操作类型 | 效果 |
|----------|------|
| **Restore** | 回滚状态：撤销代码更改或对话历史 |
| **Summarize** | 压缩上下文：保留早期上下文完整，只压缩后续部分 |

**Summarize 工作原理**：
- 选中消息之前的消息保持完整
- 选中消息及后续消息被替换为 AI 生成的摘要
- 不改变磁盘上的文件
- 原始消息保存在会话 transcript 中，需要时 Claude 可引用

> **提示**：如果想保留原会话完整并尝试不同方案，使用 [fork](/en/how-claude-code-works#resume-or-fork-sessions) (`claude --continue --fork-session`) 而不是 summarize。

---

## 常用场景

| 场景 | 使用方式 |
|------|----------|
| **探索替代方案** | 尝试不同实现方法，不丢失起点 |
| **从错误中恢复** | 快速撤销引入 bug 的更改 |
| **迭代功能** | 实验不同变体，可随时回退 |
| **释放上下文空间** | 从中间点压缩冗长的调试会话 |

---

## 限制说明

### 不追踪的更改

以下类型的文件更改无法通过回滚撤销：

```bash
# 这些 Bash 命令的修改不追踪
rm file.txt
mv old.txt new.txt
cp source.txt dest.txt
```

### 不是版本控制的替代品

| 系统 | 用途 |
|------|------|
| **Checkpointing** | 快速、会话级恢复（本地撤销） |
| **Git 等版本控制** | 永久历史记录和协作 |

**建议**：继续使用 Git 进行 commits、branches 和长期历史，Checkpointing 作为"本地撤销"补充。

---

## 相关链接

- [Interactive mode](/en/interactive-mode) - 键盘快捷键和会话控制
- [Commands](/en/commands) - 使用 `/rewind` 访问检查点
- [CLI reference](/en/cli-reference) - 命令行选项