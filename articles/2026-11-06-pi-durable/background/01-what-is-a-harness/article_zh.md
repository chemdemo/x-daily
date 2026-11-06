---
title: "什么是 Harness？（结构化笔记）"
author: "Earendil Product"
source_url: "https://earendil.com/posts/what-is-a-harness/"
published_at: "2026-08-20"
archived_at: "2026-11-06"
category: "background"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Earendil Product 随笔（2026-08-20），写给在 AI 信息流里听过「agent harness」却不好意思问含义的人。通篇用攀岩安全带隐喻。早于 Pi Durable，但提供 Durable 文默认读者已懂的词汇。

## 隐喻

攀岩安全带支撑你，把你连到绳索/快挂以防止坠落并约束路线，还能挂工具。你可带到不同山峰并按地形改装备环。Agent harness 在结构与功能上类似：支撑、控制、挂工具、可携带/可改造。

## 定义

Agent harness 是为 AI 模型提供运行环境的软件。与多数模型不同，终端用户可以**拥有**自己的 harness。业界简化公式：Agent = Model + Harness。最早流行应用是 coding agent；如今各类 agent 核心都有 harness。界面各异：终端（Pi）、iMessage/聊天、邮件（文中提及 Lefos）等。

## 四件事

1. **系统提示** — 每轮注入的指令（比训练期「灵魂文档」更外挂；像新人手册）。  
2. **工具** — 模型可调用的代码能力；harness 描述并实现，通常不硬编码何时使用。  
3. **Agentic loop** — 观察 → 决定 → 工具 → 观察 → … 直到完成（文中用邮件比较本地学校的例子）。  
4. **翻译层** — 对接多家模型供应商（甚至在同一循环混用）；相对实验室锁定应用，保留用户选择与本地会话主权。

## 让 harness 成为你的 / Pi 立场

与模型不同，harness 可被拥有与改造。Pi 极简（短提示、少工具），开源跑在笔记本上，有数千共享扩展。Earendil 把中立开源 harness 框为**人的能动性**工具——驾驭技术，而非被技术驾驭。

## 与 Pi Durable 读者的关系

Durable 说「harness」时，指上述栈**加上**耐久存储、tasks、commits 与多客户端机械——不只是交互式 coding 循环。用此文给只懂聊天 UI 的干系人扫盲。

## 摘录

> “An agent harness is a piece of software that provides an environment for an AI model to operate within. Unlike most AI models, you as an end user can own your own agent harness.”
> — （同上）
