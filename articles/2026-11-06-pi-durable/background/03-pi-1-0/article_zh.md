---
title: "Pi 1.0 发布（结构化笔记）"
author: "Earendil"
source_url: "https://earendil.com/posts/pi-1-0/"
published_at: "2026-10-01"
archived_at: "2026-11-06"
category: "background"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Earendil（2026-10-01）：Pi 1.0 发布帖，与 Pi Durable 同日。把 coding agent 定位为经硬化、极简、可扩展、个人与企业可依赖的 harness，并解释为何把耐久工作拆进独立实验包。

## Pi 1.0 产品立场

每周数十万用户；数月 issue/PR 硬化。极简是刻意的：agent 工具每周都在变；Pi 等功能证明自己后再接受复杂度（并指向当周早些时候关于 codemode/MCP 的讨论）。

## 1.0 点名的功能

- Codemode（原生 MCP；非 LLM 模型如 Jev 与图像模型）
- 虚拟模型的扩展支持
- 延迟工具加载
- Anthropic 模型的缓存预热
- 会话中系统消息（转录本感知的提示/工具变更）
- 新 TUI 主题；默认全屏

## 为何 Pi Durable 作为兄弟存在

Earendil 希望把 Pi 的极简与「驾驭 AI」精神带到编码与终端**之外**：其他界面、更长跑的会话/任务——也就是更耐久。与其把 Pi 扭成它不是的东西，他们交付了共享极简与超可塑性的新基底 **Pi Durable**。细节指向 Mario 的 Durable 设计帖。

## 安装

Pi：`pi.dev` 的 curl/PowerShell 安装脚本。Durable：`npm install` 三件套。皆 MIT；文档 pi.dev；代码 `earendil-works/pi`。

## 对自研 harness 的启示

产品线要清晰：**稳定交互式 Pi** vs **实验性耐久 harness**。押注 Durable 的生产项目必须假定 API 变动并积极钉版本；勿把 Pi 1.0 CLI 稳定性与 Durable 稳定性混为一谈。向管理层解释「为何不直接给 Pi 加耐久」被有意拒绝时，用此帖。

## 与 Durable 发布帖对照阅读

先读 Pi 1.0 了解产品哲学与 coding-agent 范围，再读 Durable 发布帖了解实验性长跑基底。二者一起回答「为何两个包？」：让终端 agent 保持稳定极简，同时探索耐久/多人/边缘托管，而不强迫每位 Pi 用户上实验性存储/task 运行时。对 harness 工程师，这种拆分是模板——稳定交互循环，在耐久控制面做实验，二者共享模型层（`pi-ai`）。


文中演示帧还展示了 codemode 汇总一周提交、经 Jev 在 Opus 与 GPT 间切换的虚拟路由模型，以及 `/session` 成本拆解——有助于理解 Durable 为何共享 `pi-ai` 却不共享以 TUI 为中心的 coding-agent UX。

## 摘录

> “Rather than stray from our minimal roots and try to make Pi something it wasn’t, we combined that work into a new experimental package that we are shipping today, Pi Durable.”
> — （同上）
