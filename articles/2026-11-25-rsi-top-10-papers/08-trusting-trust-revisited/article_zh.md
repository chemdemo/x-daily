---
title: "Reflections on Trusting Trust, Revisited（结构化笔记）"
author: "Franziska Roesner and Tadayoshi Kohno"
source_url: "https://arxiv.org/abs/2609.17817"
arxiv_id: "2609.17817"
arxiv_version: "v1"
published_at: "2026-09-16"
archived_at: "2026-11-25"
license: "arXiv.org perpetual, non-exclusive license 1.0"
license_url: "https://arxiv.org/licenses/nonexclusive-distrib/1.0/"
kind: "structured-notes"
list_index: 8
archive_note: "在通读论文后撰写的结构化研究笔记。不是原文转载，不是全文近译，也不是逐句翻译。arXiv 非独占许可只授权给 arXiv，并不授予第三方全文再分发或翻译权。笔记只在高层次概述发现与防御，不复述攻击步骤或概念验证代码。"
---

## 来源

Franziska Roesner（华盛顿大学）、Tadayoshi Kohno（乔治城大学）。《Reflections on Trusting Trust, Revisited: Contaminating Self-Modifying AI Coding Agents with Poisoned Benchmarks》。arXiv:2609.17817v1，首次发布 2026-09-16 04:32（Asia/Shanghai）。PDF：https://arxiv.org/pdf/2609.17817。HTML：https://arxiv.org/html/2609.17817。22 页。列入 [elvis / @omarsar0，《My Top 10 RSI Papers of 2026》](https://x.com/omarsar0/status/2125234668396023943) 第 8 项。

**抓取状态：** 已于 2026-11-25（Asia/Shanghai）从公开的 arXiv HTML 与 PDF 通读全文。

**许可 / 为何只做笔记：** 摘要页标明 [arXiv.org 永久非独占许可 1.0](https://arxiv.org/licenses/nonexclusive-distrib/1.0/)。该许可不授予第三方全文再分发或翻译权。

## 背景

论文把 Thompson 经典的「信任信任」观察放到自修改 AI 编程 agent 上：此时的「编译器」会自评并改写自己。攻击者不能改种子 agent 或底层模型，但可以提供用于自评与自我改进的基准。问题是：这是否足以让后续版本的 agent 在干净的留出任务上写出不安全代码。

## 方法（提要）

作者考察三个近期系统：Darwin Gödel Machine（对其自我改进提示做了实验性修改）、SICA，以及 Hyperagents（后两者基本未改）。他们构造会在自我改进过程中把 agent 推向不安全编码模式的评测集，再测量演化后的 agent 是否仍在中性留出任务上复现这些模式，并在干净或偏安全的集合上继续演化，以检验污染是否残留。

## 主要发现

三个系统上都报告了概念验证：在恶意评测集上演化之后，agent 可能在后续干净任务上写出不安全模式（例如关闭 TLS 证书校验，或削弱令牌签名校验）。随后即使在配套干净集或未能对齐毒化线索的通用安全基准上继续演化，污染也常常仍在。专门对齐这些线索的去污集，在部分系统上只部分有效，在所报告的 Hyperagents 实验中则去掉了有害指令。作者还归纳了漏洞、基准、模型与 harness 中使演示成立的性质，并讨论预防、检测与恢复方向。

## 收获

要把自我评测集当作代码一样审查，也要审查 agent 为自己写下的工具与规则。自我改进循环若缺少安全提示、或并不劝阻针对基准的特化，系统就更容易被带偏。仅在「干净」任务上继续演化并不可靠，如果这些任务从不惩罚不安全模式的话。

## 摘录

> “self-modifying coding agents must be designed to be more resilient to such attacks”
> — https://arxiv.org/abs/2609.17817
