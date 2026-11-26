---
title: "Harness Evolution Hits a Ceiling：何时该开始训练权重（结构化笔记）"
author: "Yuan Tian, Bing Hu, Hao Wang, Binghang Lu, and Fang Wu"
source_url: "https://arxiv.org/abs/2610.11655"
arxiv_id: "2610.11655"
arxiv_version: "v1"
published_at: "2026-10-08"
archived_at: "2026-11-25"
license: "arXiv.org perpetual, non-exclusive license 1.0"
license_url: "https://arxiv.org/licenses/nonexclusive-distrib/1.0/"
kind: "structured-notes"
list_index: 4
archive_note: "在通读论文后撰写的结构化研究笔记。不是原文转载，不是全文近译，也不是逐句翻译。arXiv 非独占许可只授权给 arXiv，并不授予第三方全文再分发或翻译权。"
---

## 来源

Yuan Tian、Bing Hu、Hao Wang、Binghang Lu、Fang Wu。《Harness Evolution Hits a Ceiling: When Weight Training Should Begin》。arXiv:2610.11655v1，首次发布 2026-10-08 18:33（Asia/Shanghai）。PDF：https://arxiv.org/pdf/2610.11655。HTML：https://arxiv.org/html/2610.11655。21 页。列入 [elvis / @omarsar0，《My Top 10 RSI Papers of 2026》](https://x.com/omarsar0/status/2125234668396023943) 第 4 项。

**抓取状态：** 已于 2026-11-25（Asia/Shanghai）从公开的 arXiv HTML 与 PDF 通读全文。

**许可 / 为何只做笔记：** 摘要页标明 [arXiv.org 永久非独占许可 1.0](https://arxiv.org/licenses/nonexclusive-distrib/1.0/)。该许可不授予第三方全文再分发或翻译权。本篇为显著短于原文的独立摘要，仅保留一处带出处的短引。

## 背景

论文比较提升长程 agent 的两种杠杆：在冻结模型周围演化 *harness*，以及训练模型 *权重*。作者先跑自演化 harness，再把「种子 / 演化后 harness」与「基座 / 训练后权重」交叉比较，主张正确杠杆可以从失败构成中读出。

## 方法（提要）

在可程序化打分的行程规划基准 DeepPlanning 上，用 48 道开发题做全部决策，72 道留出题循环从未见过。失败轨迹按最先触发的信号标注：无依据断言（F1）、harness 自身摩擦（F2）、循环或步数耗尽（F3）、已提交但质量差的方案（F4）。同时记录交付率与已交付方案的质量。由 LLM 提出 harness 修改，再由评判器核验。分数为四次 rollout 均值，约 0.035 以下的差异视为噪声。

## 主要发现

harness 演化把 Qwen3.5-4B 的留出分从 0.16 提到 0.30，把 Qwen3.5-9B 从 0.32 提到 0.44。4B 的交付率从 55% 升至 90%，循环类（F3）失败下降，内容失败（F4）并未减少。用演化后 harness 的轨迹训练 LoRA，在原版 harness 下两种规模都约 +0.13。4B 上 adapter 可与演化 harness 叠加；9B 上单用 adapter 即可追平演化线，并把内容失败从约四分之一降到二十分之一。对打乱答案的轨迹训练的安慰剂 adapter 低于基座模型，说明增益来自轨迹内容。同一循环在 WebArena-Lite 的 117 道未见题上 +0.09，主要来自上下文保留两页完整历史而非一页；在这些轨迹上训练的 adapter 并不能再现该增益。

## 收获

先诊断再干预：过程失败、以及改变模型「看见什么」的修改，用 harness；内容失败、以及改变模型「写出什么」的修改，用权重训练。循环在 F4 处停止，部分是因为给提案模型的证据包没有把评分器看到的内容失败清楚展示出来。

## 摘录

> “evolve the harness for process failures and train the weights for content failures”
> — https://arxiv.org/abs/2610.11655
