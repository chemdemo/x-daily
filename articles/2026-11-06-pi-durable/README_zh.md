# Pi Durable 阅读笔记（2026-11-06）

面向 **x-daily** 归档的独立专题目录：关于 **Pi Durable**（Earendil 实验性耐久 agent harness）的双语结构化研究笔记，供自研 coding-agent harness 的工程师使用。

**不是全文转载。** 每篇为压缩笔记（背景、关键概念、方法、对 harness 的启示、局限、少量带出处摘录）。英文：`article.md`。中文：`article_zh.md`（对上述笔记的忠实翻译）。每篇另有 `meta.json`。

归档日期：**2026-11-06**（Asia/Shanghai 日历日）。

## 建议阅读顺序

1. [官方发布帖](official/01-pi-durable-announcement/article_zh.md) — 设计全景  
2. Agent-Engineering 前三篇：[导论](tutorials/01-agent-engineering-intro/article_zh.md) → [第一次会话](tutorials/02-agent-engineering-first-conversation/article_zh.md) → [SQLite / resume](tutorials/03-agent-engineering-sqlite-resume/article_zh.md)  
3. [Cloudflare 专文](official/02-pi-durable-on-cloudflare/article_zh.md) + [样例仓](official/04-pi-durable-cloudflare-sample/article_zh.md)  
4. [WithOne Gmail 幂等](practice/03-withone-gmail-idempotency/article_zh.md) — 外部副作用  

其余按 **official / tutorials / practice / background** 按需浏览。贯穿主题：**durable ≠ 外部副作用恰好一次**；`requestId` 只去重输入受理。

系列总览（笔记中提及，不单设目录）：https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero

## 目录

| 标题 | 笔记 | 作者 | 分类 |
|---|---|---|---|
| Pi Durable — 官方发布帖 | [official/01-pi-durable-announcement/article.md](official/01-pi-durable-announcement/article.md) · [article_zh.md](official/01-pi-durable-announcement/article_zh.md) | Earendil Engineering | `official` |
| Pi Durable 在 Cloudflare 上 | [official/02-pi-durable-on-cloudflare/article.md](official/02-pi-durable-on-cloudflare/article.md) · [article_zh.md](official/02-pi-durable-on-cloudflare/article_zh.md) | Earendil Engineering | `official` |
| @earendil-works/pi-durable README | [official/03-durable-package-readme/article.md](official/03-durable-package-readme/article.md) · [article_zh.md](official/03-durable-package-readme/article_zh.md) | Earendil / pi-durable maintainers | `official` |
| pi-durable-cloudflare 样例仓库 | [official/04-pi-durable-cloudflare-sample/article.md](official/04-pi-durable-cloudflare-sample/article.md) · [article_zh.md](official/04-pi-durable-cloudflare-sample/article_zh.md) | earendil-works | `official` |
| Cloudflare Agents SDK — PiHarness | [official/05-cloudflare-agents-pi/article.md](official/05-cloudflare-agents-pi/article.md) · [article_zh.md](official/05-cloudflare-agents-pi/article_zh.md) | Cloudflare | `official` |
| Vercel — 在 Vercel 上耐久运行 Pi | [official/06-vercel-running-pi-durably/article.md](official/06-vercel-running-pi-durably/article.md) · [article_zh.md](official/06-vercel-running-pi-durably/article_zh.md) | Vercel | `official` |
| Agent-Engineering — 导论 | [tutorials/01-agent-engineering-intro/article.md](tutorials/01-agent-engineering-intro/article.md) · [article_zh.md](tutorials/01-agent-engineering-intro/article_zh.md) | Nelson Lin | `tutorials` |
| Agent-Engineering — 第一次会话 | [tutorials/02-agent-engineering-first-conversation/article.md](tutorials/02-agent-engineering-first-conversation/article.md) · [article_zh.md](tutorials/02-agent-engineering-first-conversation/article_zh.md) | Nelson Lin | `tutorials` |
| Agent-Engineering — SQLite 与 resume() | [tutorials/03-agent-engineering-sqlite-resume/article.md](tutorials/03-agent-engineering-sqlite-resume/article.md) · [article_zh.md](tutorials/03-agent-engineering-sqlite-resume/article_zh.md) | Nelson Lin | `tutorials` |
| 从零理解 pi-durable | [tutorials/04-from-the-ground-up/article.md](tutorials/04-from-the-ground-up/article.md) · [article_zh.md](tutorials/04-from-the-ground-up/article_zh.md) | Akshay Parkhi | `tutorials` |
| Pi Durable 实战要点 (Joulyan) | [tutorials/05-joulyan-in-practice/article.md](tutorials/05-joulyan-in-practice/article.md) · [article_zh.md](tutorials/05-joulyan-in-practice/article_zh.md) | Joulyan IT | `tutorials` |
| Pi Durable 速查表 | [tutorials/06-cheatsheet/article.md](tutorials/06-cheatsheet/article.md) · [article_zh.md](tutorials/06-cheatsheet/article_zh.md) | luongnv.com | `tutorials` |
| Orbi — harness vs Durable 与 kill -9 | [practice/01-orbi-harness-vs-durable/article.md](practice/01-orbi-harness-vs-durable/article.md) · [article_zh.md](practice/01-orbi-harness-vs-durable/article_zh.md) | Larry (Orbi) | `practice` |
| Bokvi — 在自身崩溃后继续 | [practice/02-bokvi-survives-crashes/article.md](practice/02-bokvi-survives-crashes/article.md) · [article_zh.md](practice/02-bokvi-survives-crashes/article_zh.md) | Tomáš | `practice` |
| WithOne — 崩溃安全的 Gmail 草稿 | [practice/03-withone-gmail-idempotency/article.md](practice/03-withone-gmail-idempotency/article.md) · [article_zh.md](practice/03-withone-gmail-idempotency/article_zh.md) | One Team | `practice` |
| daily.dev / Flavio 演练 | [practice/04-daily-dev-walkthrough/article.md](practice/04-daily-dev-walkthrough/article.md) · [article_zh.md](practice/04-daily-dev-walkthrough/article_zh.md) | Flavio Copes | `practice` |
| 什么是 Harness？ | [background/01-what-is-a-harness/article.md](background/01-what-is-a-harness/article.md) · [article_zh.md](background/01-what-is-a-harness/article_zh.md) | Earendil Product | `background` |
| Pi 中的 Compaction 如何工作 | [background/02-compaction-in-pi/article.md](background/02-compaction-in-pi/article.md) · [article_zh.md](background/02-compaction-in-pi/article_zh.md) | Earendil Engineering | `background` |
| Pi 1.0 发布 | [background/03-pi-1-0/article.md](background/03-pi-1-0/article.md) · [article_zh.md](background/03-pi-1-0/article_zh.md) | Earendil | `background` |

## 目录结构

```
articles/2026-11-06-pi-durable/
  README.md / README_zh.md / meta.json
  official/…/  tutorials/…/  practice/…/  background/…/
    article.md  article_zh.md  meta.json
```

## 抓取状态

本归档所列源 URL 均已成功抓取（GitHub 样例用 README raw；daily.dev 以链接的 Flavio 全文为准）。无「抓取失败」占位笔记。
