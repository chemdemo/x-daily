---
title: "daily.dev / Flavio — 用 Pi Durable 熬过崩溃（结构化笔记）"
author: "Flavio Copes (via daily.dev link)"
source_url: "https://daily.dev/posts/how-to-build-agents-that-survive-crashes-with-pi-durable-zt8jcyleg"
published_at: "2026-11-04"
archived_at: "2026-11-06"
category: "practice"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Flavio Copes 在 daily.dev 的帖子，外链规范长教程 https://flaviocopes.com/pi-durable（2026-11-04 更新）。daily.dev 页是摘要 Q&A；Flavio 文是带交互图的完整演练。两 URL 都归档：此处作发现卡，实质来自 Flavio。

## 演练构建什么

项目 `pi-durable-demo`，含 `agent.ts` + `tools.ts`。工具 `service_status` 读 fixture（website healthy / checkout degraded），等待 10 秒（`context.abortSignal`），返回 JSON，标 `replay: 'safe'`。Agent 打开 SQLite、安装扩展、`resume()`、提交 CLI `requestId`+prompt、打印助手文本、在 `finally` 关闭。强杀会跳过 `finally`——正是测试点。

## 崩溃与重试实验

1. 工具中途 `kill -9` → 同命令重启 → 同一 submission id；工具从头再跑（结果从未提交，不是恢复的 JS 栈）。  
2. 对已完成 requestId 再跑 → 同一 submission/答案；工具不再启动。  
3. 思想实验：换成邮件发送且供应商已接受、本地提交前崩溃 → 自动重跑可能重复；默认 unsafe 报 interrupted；应用仍须阻止模型发出新的有害调用。

## 更广的系统笔记

多客户端：一个所有者进程；`viewState`；`whenBusy: "steer"`。部署：挂持久卷；把库当敏感应用数据。长聊天用 compaction；关键事故字段放 documents。子 agent 需自建。走向 Sentry 类：校验 webhook → 映射调查 → 稳定 id → 窄工具 → PR → 对账。并提到 Cloudflare Workflows、Vercel Workflow 等替代耐久运行时需另作选择。

## 对自研 harness 的启示

出色的入门实验：两个文件、一次杀测、一次幂等重试测。教 unsafe 写入时与 WithOne Gmail 文配对。实质引用 Flavio；本集合保留 daily.dev 作索引入口。

## 与本集合其余篇目如何衔接

在 Agent-Engineering 第 2–3 篇之后做 Flavio 的两文件实验，再升到 Joulyan 生产清单与 WithOne 的 Gmail 写入测试，然后才给任何会突变的工具标 `replay: "safe"`。daily.dev 卡片适合在 Slack 分享短链；需要完整杀测步骤与插图时，把工程师指向 flaviocopes URL。

## 摘录

> “Pi Durable resumes from durable checkpoints. It doesn’t restore a suspended JavaScript stack at the exact instruction that was interrupted.”
> — https://flaviocopes.com/pi-durable
