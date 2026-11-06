---
title: "pi-durable-cloudflare 样例仓库（结构化笔记）"
author: "earendil-works"
source_url: "https://github.com/earendil-works/pi-durable-cloudflare"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

在 Cloudflare Workers / Durable Objects 上跑 Pi Durable，并带 React 多人客户端的官方样例仓 README。设计叙事见 Earendil「Pi Durable on Cloudflare」；本仓是具体集成。

## 背景 / 是什么

**可运行示例，不是生产服务**：共享聊天 session、多并发会话、从更早消息 fork、重连不丢已提交转录本或排队提示、一次性/周期定时 follow-up、历史搜索子 agent，以及 **Activity** 面板（查看任务所有权、等待、睡眠、工具调用、后台工作，并重试因反复崩溃而停住的任务）。

**故意无鉴权、无限流。** 公开部署等于让陌生人烧你的模型凭证。

## 关键布局（看哪里）

| 路径 | 作用 |
|---|---|
| `src/chat.ts` | 模型、扩展、工具、默认 agent |
| `src/session.ts` | 在内置 DO SQLite 上托管 harness；连接可休眠 WebSocket 与 alarm |
| `src/tools.ts` / `schedule.ts` / `search.ts` | 示例工具与耐久任务 |
| `src/client/` | React 客户端与事件驱动会话状态 |
| `test/` | 应用流：重启恢复、调度、fork、inbox；DO 存储适配器在对象内测试 |

README 立场：**代码（尤其测试）才是文档**，用于说明重启/调度/fork/inbox 行为。

## 方法 / 如何运行

要求：Node.js **22.19+**、**pnpm 11**、OpenAI API key。

```
pnpm install
cp .dev.vars.example .dev.vars   # 设置 OPENAI_API_KEY
pnpm dev                         # Worker + Vite 客户端
```

打开 Vite URL，创建 session，用第二个浏览器打开同一 URL 试多人。另有 `pnpm test` / `check` / `build` / `preview`。

部署：`wrangler login` → `secret put OPENAI_API_KEY` → `pnpm deploy`。对外暴露前先加鉴权与限流。

## 对自研 harness 的启示

- 复制结构模式（session DO、alarm 刷新、事件流 attachment、Activity UI），别把 demo 当产品骨架。
- 像该仓一样投入**重启测试**——博客会漂，测试更贴近 API 现实。
- 分清「笔记本上的 demo 密钥」与「租户隔离的生产」：样例证明耐久机制，不证明多租户。

## 局限 / 注意

实验性 Pi Durable API 可能变更。无鉴权。README 还指向发布帖、`packages/durable` 文档/示例、Cloudflare DO 文档。

## 摘录

> “This is a sample, not a production service. It intentionally has no authentication or rate limiting.”
> — https://github.com/earendil-works/pi-durable-cloudflare
