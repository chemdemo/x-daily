---
title: "Agent-Engineering — Pi Durable 导论（结构化笔记）"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/introduction-to-pi-durable-building-agents-that-survive-crashes-and-run-for-hours"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Nelson Lin，Agent-Engineering.dev——「Comprehensive Pi Durable Agent From Zero to Hero」系列第一篇（2026-11-03）。系列总览（本归档不单设目录）：https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero。纯心智模型，尚无生产代码。

## 背景 / 问题

多数 agent 是短暂的：状态活在进程内存。进程一死（崩溃、部署、笔记本休眠、serverless 超时），你或许还留着部分聊天记录，但会丢掉进行中的工具调用、排队消息，以及应用层对「当时何为真」的信念。**恢复转录本 ≠ 恢复工作**——系统不知道哪个工具在跑、是否已改变真实世界、收件箱里还有什么。

对有人盯着的交互式编码可以接受；对过夜调研、on-call 机器人、多客户支持、无人值守边缘宿主则不行。

## Pi Durable 是什么

Earendil 的实验性耐久 agent harness（与 Pi 1.0 同日于 2026-10-01 发布）。系列统一定义：**harness = 存储 + 并行跑一个或多个 LLM 会话的机械装置**，并提供工具与执行环境。

不替代 Pi coding agent（终端、单人驱动）。是需要耐久性的任意 agent 应用的独立库。共享 `pi-ai` 与极简/可塑哲学。

核心承诺（据 Earendil 意译）：会话、模型回合、工具调用与应用状态**先提交再展示**；重开同一存储即可继续中途工作。

## 关键概念表（系列词汇）

Harness；Conversation（不可变转录本）；Entry；Task（耐久状态机）；Checkpoint；Document（旁路类型化 JSON）；Commit（原子全有或全无）；Submission（可等待的输入）；Extension；Registry。

恢复素描：每步是 task → 前进前 checkpoint → 新进程找到未完成 task → 继续；工具声明可重放或报 interrupted；部分模型答案保留为 aborted。

## Pi coding agent vs Pi Durable

| 方面 | Coding agent | Durable |
|---|---|---|
| 用途 | 交互式终端 | 长跑应用 |
| 崩溃恢复 | 手动继续 | 从 checkpoint 自动恢复 |
| 存储 | Session 文件 | Memory / SQLite / JSONL / 自定义 |
| 并发会话 / 多客户端 | 有限 / 无 | 一等公民 / 有 |
| 状态 | 稳定 1.0 | 实验性 |

## 谁该关心 / 路线图

适合过夜 agent、事故机器人、多方协作、Cloudflare 类宿主、事件驱动主动 agent。短时有人监督的终端会话较不必要。

系列后续：搭建 → SQLite/resume → 工具/replay → 多会话 → documents → 子 agent → 多客户端 → 部署 → 端到端主动 agent。包：`pi-durable`（+ `pi-ai`、`chord`）；钉版本。

## 对自研 harness 的启示

尽早采用这套词汇，让设计评审谈 tasks/checkpoints/documents，而不是「把聊天存起来」。把 Durable 当作**面向 LLM agent 的耐久工作流引擎**，而不是外挂数据库的聊天 SDK。

## 摘录

> “Resuming a transcript is not the same as resuming the work.”
> — （同上 URL）
