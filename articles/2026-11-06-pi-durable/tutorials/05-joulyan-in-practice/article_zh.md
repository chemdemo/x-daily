---
title: "Pi Durable 实战要点 — Joulyan（结构化笔记）"
author: "Joulyan IT"
source_url: "https://joulyan.com/en/blog/pi-durable-in-practice-building-agents-that-pick-up-where-they-left-off"
published_at: "2026-10-17"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Joulyan IT 博客（2026-10-17，约 9 分钟）：偏生产的 Pi Durable 实践。开篇定调——Pi Durable 有助于崩溃后续跑，但保存对话挡不住重复发票。

## 恢复需要的不只是转录本

例：agent 维护进行到 40 分钟，计划已记录，部署已开始，容器被替换。转录本显示意图；恢复还需要哪些工作已完成、哪些状态已提交、部署是否已发生。Pi Durable 提供持久化 + 执行机械（Harness、Conversation、Task、`defineDoc` Document、Submission），本地片段可原子提交——**止于存储边界**。不能把库提交与外部发信合成一次不可分操作。

## 启动与所有权清单

- `--save-exact` 安装并提交 lockfile（实验性 API）。
- `./agent.sqlite` 放**持久卷**，不是容器临时盘。
- 每份存储恰好一个进程所有者；其他客户端挂到该所有者——不要多写者 NFS。
- 每次启动在接受新输入前 `harness.resume`。
- `requestId` 来自 webhook 事件稳定 id（每次投递新 UUID 会毁掉去重）。

## 写提示词前先给工具分类

| 动作 | Replay | 原因 |
|---|---|---|
| 读配置 / 查状态 | 常 `replay: 'safe'` | 无害 / 只读 |
| 启动部署 / 发发票 | 保持未标记 | 先前调用可能已改变世界 |

强调警告：Submission `requestId` 去重的是**会话输入**，不是支付/邮件/部署。用供应商幂等或对账；Document 可存外部 op id，但单独无法关闭「外部已接受 / 本地结果未提交」窗口。推荐流程：启动部署 → 保留外部 op id → 单独查状态 → 启动结果不明时要求对账/审批。

## 运维与能力

systemd `Restart=always` 或容器重启策略 + 卷。水平扩展 = 每 agent/租户独立存储。CodingTools + NodeExecutionEnv = 你的 FS/shell 权限——用 `configure` 移除 deploy、用 wrapper/hooks 约束，别靠提示词。Compaction 示例：`reserveTokens: 16384`，`backgroundTokens: 32768`。无内置子 agent——用所有权 API 自建。热 `loadExtension` 也要做恢复测试。

## 无人值守前验证 / 何时跳过

最好的测试：在**外部已接受、本地结果未提交**时打断。并测双提交同一 requestId；模型中途与工具中途杀进程。一次性脚本、无状态聊天、或本季度需要冻结 API 的团队可跳过 Durable。凭证、UI、备份、监督、外部防护仍由你负责。

## 对自研 harness 的启示

把「工具契约决定恢复是否安全」设为显式设计评审门槛。先交付一个有持久 SQLite、并对最危险外部动作做刻意崩溃测试的有界 agent，再扩大自主权。

## 摘录

> “A Submission’s requestId deduplicates conversation input. It does not make external actions happen exactly once.”
> — （同上）
