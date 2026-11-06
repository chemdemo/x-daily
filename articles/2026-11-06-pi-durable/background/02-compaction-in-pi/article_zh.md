---
title: "Pi 中的 Compaction 如何工作（结构化笔记）"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/compaction-in-pi/"
published_at: "2026-08-13"
archived_at: "2026-11-06"
category: "background"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Earendil Engineering（2026-08-13）：Pi **coding agent** 中 compaction 如何工作。理解 Pi Durable compaction **task** 的必要背景——问题框架相同，但带耐久调度。

## 问题

LLM 上下文窗口有限。coding-agent 回合会累积系统提示、工具、用户/助手消息与工具结果。最终供应商会因过大拒绝请求。

## 溢出的两种应对

1. 开全新空会话——丢掉先前决策与未完成工作（有时因长上下文质量下降而可取）。  
2. **Compact**：用更小表示替换旧历史。实践中用 LLM 摘要请求，而非纯确定性保留/丢弃函数。

## Pi 的实现

- 触发：接近上限时在回合后自动；溢出错误时中途；手动 `/compact`。  
- 按 **token 预算**保留近讯（默认约 20k tokens ≈ 约 5–20 回合）；更早回合被序列化并摘要。  
- Compaction 是**独立**请求：不同系统提示（「上下文摘要助手」）、交接班式结构化段落（目标、进展、关键决策），可选用不同模型。  
- 结果以纯文本 compaction entry 存入 session（可读、可跨模型移植）。  
- **Prompt caching**：插入摘要后在首个变化 token 处打断精确前缀匹配；后续回合重建缓存。

## 桥接到 Pi Durable

在 Durable 中，compaction 是一等耐久 task：在硬 reserve 之前后台摘要，仅当下一请求装不下才阻塞等待，溢出时 compact 并重试一次，可手动 `compact(instructions)`，旧 entry 仍在存储，`beforeCompact` 等 hooks 可提供/拒绝摘要。`reset`/handoff 刷新模型上下文，同时历史仍可通过工具搜索。

## 对自研 harness 的启示

把摘要提示与主 agent 人设分开。显式预算近讯 token。预期 compact 后的缓存失效成本。有耐久存储时，勿把「模型看到的已压缩视图」当成「审计历史已删除」——必留结构化事实放 documents，而不只靠散文摘要。

## 把 compaction 思路迁到自研时的实践清单

为自研 harness 设计 compaction 时写明：(1) 谁触发（自动阈值 vs 手动 vs 溢出），(2) 何为原文保留 vs 摘要，(3) 摘要请求是否用不同模型/提示，(4) 摘要在存储中如何寻址（entry 种类、head/指针），(5) prompt cache 如何失效，(6) 哪些事实绝不能只依赖摘要、必须活在类型化 documents 里。Pi Durable 的 settings（`reserveTokens`、`backgroundTokens`、`keepRecentTokens`、按模型覆盖）是该清单的具体参考实现——不是唯一可能。

## 摘录

> “The ideal outcome of a good summarization for a coding agent is like a handoff briefing from one shift to the next.”
> — （同上）
