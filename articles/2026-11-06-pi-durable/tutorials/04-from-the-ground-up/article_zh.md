---
title: "从零理解 pi-durable（结构化笔记）"
author: "Akshay Parkhi"
source_url: "https://www.akshayparkhi.net/2026/Oct/20/pi-durable-from-the-ground-up/"
published_at: "2026-10-20"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Akshay Parkhi 博客（2026-10-20），配套可运行仓 https://github.com/avparkhi/pi-durable-hello——五个脚本、一份 SQLite、默认免费 `faux` 模型（无需服务器/UI/云）。示例钉 pi-durable 1.0.3。

## 问题与想法

普通 agent 循环把工作放在进程内存。聊天 resume 只恢复文本，不恢复哪个工具在跑、已改了磁盘什么、队列里还有什么、应用当时相信什么。Pi Durable 每步先提交再展示——进程变得可抛弃。一旦 run 能熬过进程，agent 就能活在机器上（而非终端窗口），多客户端可观看/转向，agent 可等人类数小时。一句话：**agent 比客户端活得久**。

## 积木（真实类型）

Storage；Harness；Conversation；Entry 种类；Commit；Document；Task；Submission（+ 可选 `requestId`）；Extension；Watch（UI 只是观察者；晚到者拿快照）。

数据流：submit → user → generation → assistant（工具调用）→ tool tasks → results → generation → answer。每行都是 commit/checkpoint。

## 崩溃安全规则

工具声明 replay。`safe` → 崩溃后重跑；否则 → interrupted 结果，模型决定，harness 不静默重复副作用。诚实边界：**durable = 有 checkpoint，不是恰好一次**。支付/部署需自备幂等键。

## 五课

hello / crash / resume / twice / watch——直接映射到 Durable Pi 类 HTTP 应用（每设备 SQLite、带 requestId 的 POST、SSE、设备重启 resume、Steer 按钮）。

## 解锁模式 / 局限

主动 agent；多客户端；人在环审批作耐久等待。局限：非恰好一次；单进程；bash 权限；实验性钉版本。

## 对自研 harness 的启示

可克隆运行的教学仓比幻灯片更能上手。把五课变成内部验收测试。前端设计评审保留「UI 是已提交状态的观察者」口号。

## 摘录

> “Durable means checkpointed, not exactly once. A payment or a deploy still needs its own idempotency key.”
> — （同上）
