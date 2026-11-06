---
title: "WithOne — Pi Durable 崩溃后让 Gmail 草稿可安全重跑（结构化笔记）"
author: "One Team"
source_url: "https://www.withone.ai/blog/pi-durable-crash-safe-one-cli"
published_at: "2026-10-02"
archived_at: "2026-11-06"
category: "practice"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

withone.ai / One Team（2026-10-02）。实践文章：在 Gmail 创建草稿一返回就杀掉 Pi Durable agent，再 resume。工具封装 One CLI（HN 读 + Gmail 草稿）。说明为何「只有 replay 策略」对写入不够。

## 设置素描

经 One CLI 连接 Gmail + HN；导出 Anthropic 与连接密钥。Agent 读 HN 故事/评论后保存恰好一封摘要草稿。`CRASH_AFTER_DRAFT` 在 Gmail 应答后立刻 SIGKILL——于是 Gmail 已有草稿，而 Pi 已提交 intent 但无工具结果。存储：按模式分 SQLite；`harness.resume()`；固定 `requestId`。

## 观察到的行为（朴素草稿工具，默认 unsafe）

| 模型 | 崩溃后行为 | Gmail 草稿 | 独自跑完？ |
|---|---|---|---|
| Sonnet 5.5 | 拒重试；请人检查 | 1 | 否 |
| Haiku 4.5 | 重试工具；声称存了一封 | **2** | 是，但错了 |

转录本里已有的 safe HN 工具不会重抓。`requestId` 去重有效。样本小（各 2 次）但重复稳定。

## 带键的 safe 改写

仅在 execute 幂等后标 `replay: "safe"`。键来自 **`api.callId`**——作者首版用数字 `taskId` 导致 Haiku 类型错误，且 task id 跨库文件会碰撞。草稿正文放 `piref`+hash(callId)。流程：memo 命中 → 返回；否则搜 Gmail；命中复用；否则创建（崩溃点）；再 memo。恢复日志：「已在 Gmail，不再创建」。转录本呈现一次调用/一个结果。

## 仍然脆弱

创建后不久的搜索 401；未测索引延迟（漏检 ⇒ 重复）；未测发送；其他写入默认仍 unsafe。仅当**已存与当前**策略都说 safe 时才重跑（文中提及 harness/tool.ts）。

## 对自研 harness 的启示

标准化「写工具清单」：callId 键、外部唯一性查询、memo、供应商成功后的崩溃测试。没有该清单就不要给邮件/部署/扣款标 `replay: "safe"`。把 interrupted unsafe 下的模型性格当作产品风险，而非实现细节。

## 摘录

> “A tool call that was cut off reruns if it is safe to; otherwise the model is told it was interrupted.”
> — Earendil 语句，引用于上 URL
