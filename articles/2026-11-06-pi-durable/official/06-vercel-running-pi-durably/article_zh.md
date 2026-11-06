---
title: "Vercel — 在 Vercel 上耐久运行 Pi（结构化笔记）"
author: "Vercel"
source_url: "https://vercel.com/docs/agent-harnesses/pi"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Vercel 文档：用 `durable-harness` 在 Vercel Functions 上耐久运行 Pi。有两种变体；文档推荐完整 Pi Durable 路径。

## 选择变体

| | `durable-harness/pi-durable`（推荐） | `durable-harness/pi` |
|---|---|---|
| 最适合 | 大多数 Pi agent | 只要 Pi 的 agent 循环 |
| 你写什么 | 在 `openWorldStorage()` 上写自己的 Pi 代码 | `pi({ model, tools })` |
| 能力 | 完整 Harness：扩展、compaction、fork | Agent 循环 + 工具 |
| 速度 | 每步等日志（更慢） | 耐久延迟更小 |
| Steering | 另一请求可接管 session | 任意请求可重定向进行中的 turn |
| 重连 | 从进行中响应开头重发 | 从上次事件 cursor 续传 |

若已有 Pi Durable 代码或需要 fork/compaction/扩展，选 `pi-durable`——只换存储。

## 推荐路径（`pi-durable`）

1. 安装 `durable-harness`、`pi-durable`、`chord`、`pi-ai`。
2. 模块里 `resumeWith((storage) => Harness.open(...))`——若应用打开了 pi-durable session 却没有 `resumeWith`，Vercel 构建会失败。
3. 用 `openWorldStorage(sessionId)` 替代 SQLite/JSONL/memory。
4. 加 `chatRoute`（POST 提交 + SSE；GET 重连），包上 session 所有权 / `authorize`。生产环境无 `authorize` 会 401；公开访问需显式标志。
5. 队列路由：`flowHandler` + `vercel.json` / `harnessFunctions()` / Next 的 `withDurableHarness`。
6. 工具：用 `CodingTools` 配 `virtualEnv()` 或 `sandboxEnv()`——**不要**把 Function 自己的文件系统/进程交给 `env`。可安全重跑的工具标 `replay: "safe"`。

## 与 Pi 原生存储的差异

- 打开 session 即**接管**；前一个打开者在下次写入处停下。用同一 `requestId` 重试。
- 后台工作在关闭 Harness 后由队列收尾；`chatRoute` 会等待。
- 请求在工作期间保活；若 `recoverAfterMs`（默认 **120s**）无 commit 则可能结束——安静的长工具需提高该值。
- 崩溃后：日志中的已完成步骤；模型调用与 safe 工具重跑；unsafe 返回 interrupted。

Steer：聊天路由 `"kind": "steer"` 或服务端 `whenBusy: "steer"`。重连：pi-durable 的 GET 无 cursor（从进行中响应开头重放）；瘦 `pi` 变体用事件 cursor。

## 对自研 harness 的启示

- 在 Vercel 上，耐久是**日志 + 队列接管**，不是单个长寿进程——据此设计工具与 env。
- 工作区必须随 session 迁移（`virtual` / Sandbox），符合「控制面 ≠ 工具沙箱」。
- 多 agent 应用要在 `resumeWith` / `pi({ name })` 上使用稳定 `name`。
- 自托管：通过 `WORKFLOW_TARGET_WORLD` 等使用另一 World（如 Postgres）。

## 局限 / 注意

鉴权与所有权存储自负。公开路由消耗 AI Gateway 额度。选变体时仔细比较重连体验。

## 摘录

> “Give Pi’s tools a workspace, not your function.”
> — https://vercel.com/docs/agent-harnesses/pi
