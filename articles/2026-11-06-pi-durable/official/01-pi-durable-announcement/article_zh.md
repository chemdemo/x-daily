---
title: "Pi Durable — 官方发布帖（结构化笔记）"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/pi-durable/"
published_at: "2026-10-01"
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Earendil Engineering 发布帖，与 Pi 1.0 同日推出 Pi Durable。设计全景的一手文档。

## 背景 / 是什么

Pi Durable 是**实验性** TypeScript 包（`@earendil-works/pi-durable`），面向长跑、多客户端的 agent 应用。它**不替代** Pi coding agent（终端、单人驱动），而是独立的 harness：存储 + 并行跑 LLM 会话的机械装置，含工具与执行环境。

与 Pi 共享：`pi-ai`、极简与可塑。包约 15k 行（不含测试，约 150k–250k tokens），小到可让 agent 通读。

## 关键概念

- **Harness**：一份已打开的存储 + 运行时。变更经原子 **commit**；未入库不展示。
- **Conversation**：不可变 **entry** 转录本。
- **Task**：带 **checkpoint** 的持久状态机。模型调用、工具调用、compaction、自定义 `defineTask` 都是 task。
- **Document**：与转录本同库的类型化 JSON（`defineDoc`）。
- **Submission** + 可选 **`requestId`**：输入恰好一次受理（重试返回同一 submission）。
- **Extension / Registry**：可插拔工具、sections、hooks、wraps、tasks；会话只存**名称**不存代码。

## 方法 / 设计要点

1. 打开存储 → `Harness.open` → `root()` → `submit` → `wait`。
2. 重启：同一存储 + `harness.resume()`，未完成 task 从上次 checkpoint 继续。
3. 工具声明 `replay: "safe"`（崩溃后可重跑）或默认 unsafe（向模型报告 interrupted）。
4. fork 共享父历史至某 entry；多会话并发；每会话独立 agent 配置。
5. 子 agent = 由工具 task 拥有的普通会话。
6. Compaction 是后台 task；旧消息仍在存储；`reset`/handoff 后仍可搜索。
7. 多客户端：`viewState` / `watch` / `watchEvents`；`whenBusy: "steer"` 转向。
8. 热更新：同名 `registry.install` 替换；进行中的调用用旧代码收尾。

## 对自研 harness 的启示

- 「先提交再展示」+ task checkpoint 才是耐久核心，不是「把聊天 JSON 存盘」。
- 区分 **输入幂等**（`requestId`）与 **外部副作用幂等**（工具/供应商 key）。
- 所有权树提供干净的 abort/清理语义。
- 可插拔存储与执行环境，让控制面与工具沙箱可分机。

## 局限 / 注意

- API 实验性；同一存储同一时间仅一个进程拥有（无跨进程锁）。
- Durable ≠ 外部副作用恰好一次；支付/部署仍需供应商幂等键。
- SQLite `synchronous = NORMAL`：进程崩溃可恢复；断电/主机故障可能丢最新 commit。

## 摘录

> “Conversations, model turns, tool calls, and your own state are committed to storage before anything is shown. If the process dies mid-turn, reopening the storage picks the work up where it stopped.”
> — https://earendil.com/posts/pi-durable/
