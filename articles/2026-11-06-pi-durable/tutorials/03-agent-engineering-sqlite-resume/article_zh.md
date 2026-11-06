---
title: "Agent-Engineering — SQLite、崩溃恢复与 resume()（结构化笔记）"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/pi-durable-persistence-sqlite-crash-recovery-and-resume"
published_at: "2026-11-04"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

系列第三篇（Nelson Lin，2026-11-04）：SQLite 持久化、崩溃恢复、`resume()`、replay-safe 工具与 `requestId` 幂等——完整可运行脚本，承接第二篇的网关 + Deepseek 设置。

## 你将能做到什么

在工具调用中途杀死运行中的 agent，让新进程从耐久状态留下的位置继续。理解磁盘上有什么、「resume」意味什么、replay-safe 意味什么。

## 完整模式（压缩）

- 用 `openNodeSqliteStorage` 打开 `./session.sqlite`
- 安装带 `replay: "safe"` 与长 `setTimeout` 的 `service_status`，以便中途可杀
- `Harness.open` → `root`（指示使用该工具）→ 提交前 **`harness.resume()`**
- 用稳定 `requestId` 提交
- 等待；完成后读 `AssistantEntry`
- 在 `finally` 里始终 `close`

杀测：运行 → 见 `Tool started: checkout` → `pkill -9` → 同命令重跑 → 同一 submission id、工具重跑、模型作答。文件：`session.sqlite` + WAL/SHM。pi-durable 1.5+ 工具参数需安装 `pi-schema`。

## 持久化内容 / 恢复单位

原子 commit 保存会话与不可变 entries、完整 task 记录（phase + checkpoint）、submissions、documents、provider session UUIDv7。Tasks（`pi.generation`、`pi.tool`、自定义 `defineTask`、compaction）是恢复单位——每步前进前 checkpoint；重开从上次 checkpoint 继续。

## 存储表（文中）

| 后端 | 适合 |
|---|---|
| Memory | 测试/演示 |
| SQLite Node | 多数长跑应用；工作集在内存 |
| JSONL | 调试/小历史；全历史在内存 |
| Cloudflare DO | 边缘样例路径 |

SQLite：WAL、`synchronous = NORMAL` → 进程崩溃 OK；断电/主机故障可能丢最新 commit。

## `resume()` 与 `requestId`

`resume()` 启动调度器并继续未完成/到期睡眠的任务。submit/wait 也会启动；显式调用让恢复路径可见。重启后同一 `requestId` 返回已有 submission（受理恰好一次）。`harness.submission(id)` 可在另一进程重获并等待。

## 文中局限

一进程拥有存储（无内置锁）。Replay 安全是不受检的承诺。实验性 API——钉版本并重测。即使内部 checkpoint 完美，外部副作用仍需自备幂等。

## 对自研 harness 的启示

黄金 CI：safe 工具中途杀 + unsafe 工具中途杀。写明 `requestId` ≠ 支付幂等键。无界 agent 优先 SQLite；把 NORMAL 耐久性当作 SLO 输入。

## 摘录

> “Only mark a tool replay: \"safe\" when re-executing it has no harmful external side-effects (or the external system is itself idempotent).”
> — （见 front matter URL）
