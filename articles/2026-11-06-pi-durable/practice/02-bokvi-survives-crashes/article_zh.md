---
title: "Bokvi — Pi Durable 在自身崩溃后继续（结构化笔记）"
author: "Tomáš (blog.bokvi.com)"
source_url: "https://blog.bokvi.com/blog/pi-durable-agent-harness/"
published_at: "2026-10-03"
archived_at: "2026-11-06"
category: "practice"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Tomáš，blog.bokvi.com（2026-10-03，约 12 分钟）。结合通读 Earendil 发布帖/README、逐帧度假规划 demo，以及原创 `kill -9` 实验。包为 MIT；Pi Durable 与 Pi 1.0 同日发布于 2026-10-01。

## 作者为何关心

Pi coding agent 仍是单人终端工具：死了就人看了继续。Pi Durable 面向你想丢下跑数周的 agent（Slack/分诊/调研）。夹在「有人看管的 coding agent」与「服务形 agent」之间缺的，是带 checkpoint 的 task——包括工具调用中途死亡。

## Kill -9 实验（作者脚本）

Faux provider（无 API key）。模型同时要两个工具：`search`（`replay: "safe"`）与 `book`（无 replay）。用 `requestId: "job-42"` 提交。1.5 秒时杀掉。重开同一 SQLite：

1. 再交 job-42 → 同一 submission id（受理恰好一次）。  
2. Search 从头重跑。  
3. Book **不**重跑；模型收到 interrupted，自行决定。

解读：safe ⇒ harness 内至少一次；默认 ⇒ 不自动重跑（至多一次自动重跑）；外部效果恰好一次仅当远端幂等。Earendil 结账例向银行传 `payment-${task.id}` 同理。

## 从发布帖/demo 强化的主题

约 15k 行可读源码。度假 demo：子 agent 并行搜索；火车搜索中途死；`--continue` 只重跑未完成的 safe 工作。Hooks 用 `api.memo`。子 agent/fork/steer/多人 = 会话 + 所有权 + 已提交视图。Compaction 是后台 task。热更新按名替换。

## 局限（作者「止步处」）

实验性 API。这里的 durable 指扛崩溃，**不是**分布式——无跨进程锁。崩溃指进程崩溃；NORMAL 断电可能丢最新。Replay 是你的承诺。偏 TS。API 故意偏底层。

## 对自研 harness 的启示

用 faux 模型在 CI 复现 search/book 杀测。用该文措辞给 PM 扫盲：durable ≠ 分布式 ≠ 恰好一次。把 memo 审批模式用于任何必须跨重启的人工门闸。

## 摘录

> “replay: "safe" makes a tool at-least-once and the default makes it at-most-once; an effect outside the harness happens exactly once only if the far side is idempotent.”
> — （同上）
