---
title: "@earendil-works/pi-durable README（结构化笔记）"
author: "Earendil / pi-durable maintainers"
source_url: "https://github.com/earendil-works/pi/blob/main/packages/durable/README.md"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

官方包 README（raw 同上）。规范文档见 `docs/spec.md`；`test/examples` 有 30+ 可运行示例。

## 背景

实验性 harness 的权威 API 词典。安装：`npm i @earendil-works/pi-durable @earendil-works/pi-ai @earendil-works/chord`。

## 关键概念（压缩）

Harness、Conversation、Entry、Commit、Document、Task、Submission、Turn/Run、Extension、Registry、Agent——与发布帖一致，并补充：adapter 不进 Context；取消 wait ≠ 取消工作；遥测 span 不记录消息正文。

## 持久化与恢复

- SQLite + `resume()`。
- 重复 `requestId` 返回已有 submission；`harness.submission(id)` 重启后重获。
- Sleep：以 phase 结束存储睡眠，禁止 phase 中途 sleep。
- 可驱逐宿主：`onWake` + 由 `wake()` 布 alarm，alarm 时 `resume()`。
- 崩溃循环：`restartLimit`（默认 5）；失败 run 后队列行为由 `afterFailedRun` 决定。
- 旁观者用 `readOnly: true`。

## Extensions / 工具 / code mode

- 工具：执行前提交 intent；`replay: "safe"` 或 interrupted；三通道 output/structuredOutput/details。
- 嵌套 `executeTool`、callers/modelTools、deferred + tool_search。
- Codemode：QuickJS + journal 确定性恢复。
- Hooks、inbox（followUp/steer/reject）、`submit(tx, draft)`、continue。

## 存储后端

Memory / IndexedDB / Node SQLite / JSONL / Cloudflare DO SQLite；自定义走一致性套件。可设表前缀如 `pi_`。

## 启示

以 README + examples 为契约；钉版本。工具围绕 replay、structuredOutput、所有权设计。Serverless 认真实现 `onWake` 与失败重开。

## 局限

API 无预告变更；单进程拥有存储；延迟 progress 最多丢约 1s；abort 等到 idle。

## 摘录

> “A durable agent harness. Conversations, model turns, tool calls, and your own state are committed to storage before anything is shown.”
> — https://github.com/earendil-works/pi/blob/main/packages/durable/README.md
