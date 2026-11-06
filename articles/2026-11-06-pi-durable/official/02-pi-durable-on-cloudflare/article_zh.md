---
title: "Pi Durable 在 Cloudflare 上（结构化笔记）"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/pi-durable-on-cloudflare/"
published_at: "2026-10-16"
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Earendil Engineering 第二篇官方 Pi Durable 文章（2026-10-16）。配套可运行样例：https://github.com/earendil-works/pi-durable-cloudflare。在发布帖之后、关心 serverless 托管时阅读。

## 背景 / 是什么

许多 agent 响应事件、工作一段时间后空闲。常开机器浪费。Cloudflare Durable Objects（DO）空闲可休眠，由 HTTP、WebSocket 或 alarm 唤醒。本文展示一个**多人聊天 agent**：每个 DO 存**一个 Pi Durable session**，用 DO SQLite，干净休眠，为定时工作唤醒，并用子 agent 跨会话搜索。

架构：React 客户端 → PartySocket → Worker（按 session ID 路由）→ 带 SQLite 的 DO（可休眠 WebSocket、嵌入式 SQLite、Pi Harness → 模型供应商）。可分享 session URL；同一 URL → 同一对象；不同 ID → 状态隔离。

## 关键概念

1. **Session 边界 = DO 实例**，不是「一个巨大转录本」。session 内可创建独立会话，并从已提交 entry fork。
2. **存储适配**：`openDurableObjectSqliteStorage(ctx.storage)`。仍可自带后端；DO 适配器用 Workers Vitest 跑 `registerStorageConformance`。
3. **生命周期**：构造函数在创建与休眠/驱逐后都会跑。用 `blockConcurrencyWhile` 先初始化 harness。harness 因失败关闭时 `ctx.abort`，下次事件再打开。
4. **幂等创建/fork**：在创建会话的同一 commit 里，用客户端 `requestId` 写收据文档——WebSocket 掉线后尤其关键。
5. **流式**：每条 socket 用 `watchEvents` 先发快照再发增量。侧边栏用单独的 session 快照。
6. **唤醒模型**：从 socket attachment 恢复流；客户端用同一 requestId 重发未确认工作。把多个 Pi 截止时间投影到**唯一** DO alarm；进行中工作时设短恢复 alarm。Alarm 处理必须幂等（至少一次）。
7. **搜索子 agent**：`search_history` 标 `replay: "safe"`，拥有子会话，搜索其他顶层转录本，只把最终答案回给父会话；重放时按 `ownerTaskId` 复用子会话。

## 方法 / 上手要点

本地需 Node 22.19+、pnpm 11、OpenAI key：克隆样例、`pnpm install`、配 `.dev.vars`、`pnpm dev`。建议试：在会话 A 陈述事实；在 B 询问；查看搜索子 agent；两分钟后查天气并离开。

schedule 工具 → 耐久 Pi sleep task → DO alarm。每次唤醒用 `harness.inspect()` 刷新 alarm。

## 对自研 coding-agent harness 的启示

- 控制面可休眠时，必须外置唤醒信号（alarm/队列/cron）。仅靠 Pi 内存定时器不够。
- 多会话产品需要 fork + 收据 + 所有权，而不只是「存消息」。
- 把 Activity / task-graph 可见性当作一等运维 UI。
- 存储适配器的一致性测试是生产就绪的一部分。

## 局限 / 注意

样例**无鉴权**——谁都能打到已部署 Worker 就会烧你的模型额度。Alarm 至少一次。演示搜索是简单子串匹配。复制模式时钉住实验性 Pi Durable 版本。

## 摘录

> “Pi can track any number of deadlines, but a Durable Object has only one alarm.”
> — https://earendil.com/posts/pi-durable-on-cloudflare/
