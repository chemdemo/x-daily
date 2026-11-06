---
title: "Cloudflare Agents SDK — PiHarness（结构化笔记）"
author: "Cloudflare"
source_url: "https://developers.cloudflare.com/agents/harnesses/pi/"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Cloudflare Agents 文档：用 `PiHarness` Lifecycle 能力跑 Pi Durable（抓取时页面更新于 2026-11-03）。已在 Agents SDK 上时优先看这篇，而不是手写 DO alarm。

## 背景 / 如何工作

Pi 跑 agent 循环（转录本、follow-up/steer 收件箱、模型调用、工具、重试、崩溃恢复）。`PiHarness` 在 Durable Object 里提供 SQLite 存储，并在 Pi 有活时保活对象。

Lifecycle：可复用的 DO 片段，获得 SQLite + 耐久任务队列。`PiHarness` 用途：

- **存储** — Pi 表前缀 `pi_`，避免与业务表冲突。
- **唤醒** — Pi 调度器在内存里；被驱逐的对象没有调度器。`PiHarness` 根据 Pi 的 `wake()` 调度一个 Lifecycle job：活跃时心跳、下次睡眠到期时到期、空闲时完成。中途驱逐 → job alarm 重启对象 → Pi 重开存储继续。

`PiHarness` 不规定传输；你 watch 会话状态，再用 WebSocket / HTTP / RPC 发出去。

## 安装 / 版本耦合

安装 `agents` 与 Pi 相关包。Pi 是 `agents` 的可选 peer（~1.5.0）。Agents SDK **0.33+** 支持 Pi 1.5。更新的 Pi minor 可能写入旧 minor 读不了的格式——会话跑过 1.5 后应**向前修**，不要回滚。Wrangler 需要 `nodejs_compat`、（若用 Workers AI）AI binding，以及 SQLite DO 迁移（`new_sqlite_classes`）。

## 关键 API

在普通 `DurableObject` 上工厂创建 `PiHarness`（传入 `onWake`），再 `Lifecycle.install(this).use(harness)`。

- `prompt` / `submit`+`wait`，用 **`operationId`** 做幂等（SDK 层对应核心 `requestId` 的亲戚）。
- Busy：默认 follow-up；`steer`；`reject`。
- 机器来源输入用自定义 `entry.kind`（不要用 `pi.*` 前缀）。
- `afterFailedRun`：`keep` / `continue` / `withdraw`。
- 会话：根 id `"1"`；create / fork / list。若要按用户隔离，可一对象一会话。
- `watch`；对象重启后新开 watch 并重发首值。

恢复：驱逐/崩溃 → 下次心跳；流式中断 → 部分答案 + 重试；safe 工具重跑否则 interrupted；`restartLimit` 终止死循环。

## 对自研 harness 的启示

- 在 Cloudflare Agents 上优先用 Lifecycle 集成唤醒，少重造 alarm 算术。
- **一起钉** `agents` 与 Pi minor；写明存储格式可能跨 minor 只进不退。
- 封装里明确区分 SDK `operationId` 与核心 `requestId`，避免双重抽象泄漏进产品代码。
- 即使标 unsafe，模型仍可能在 interrupted 后再发调用——禁止双触发的工具仍需应用层幂等。

## 局限（文档写明）

无内置工具审批 UI。`abort()` 等到 idle。单次模型流超过约 15 分钟可能被 alarm 上限切断。后台完成最多晚约 30s。尚不能删除会话。进行中工作会阻止优雅排空/驱逐。

## 摘录

> “Pi’s scheduler runs in memory, and an evicted object has none. PiHarness schedules one Lifecycle job for the object, driven by Pi’s wake().”
> — https://developers.cloudflare.com/agents/harnesses/pi/
