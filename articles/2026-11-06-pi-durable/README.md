# Pi Durable reading notes (2026-11-06)

Independent topic collection for the **x-daily** archive: structured bilingual research notes on **Pi Durable** (Earendil’s experimental durable agent harness), aimed at engineers building an in-house coding-agent harness.

**Not full-article reprints.** Each piece is a condensed note (background, key ideas, methods, harness takeaways, caveats, short attributed quotes only). English: `article.md`. Chinese: `article_zh.md` (faithful translation of those notes). `meta.json` per piece.

Archived at: **2026-11-06** (Asia/Shanghai calendar day of this collection).

## Suggested reading order

1. [Official announcement](official/01-pi-durable-announcement/article.md) — design overview  
2. Agent-Engineering first three tutorials: [intro](tutorials/01-agent-engineering-intro/article.md) → [first conversation](tutorials/02-agent-engineering-first-conversation/article.md) → [SQLite / resume](tutorials/03-agent-engineering-sqlite-resume/article.md)  
3. [Pi Durable on Cloudflare](official/02-pi-durable-on-cloudflare/article.md) + [sample repo](official/04-pi-durable-cloudflare-sample/article.md)  
4. [WithOne Gmail idempotency](practice/03-withone-gmail-idempotency/article.md) — external side effects  

Then skim remaining **official** / **tutorials** / **practice** / **background** as needed. Cross-cutting theme: **durable ≠ exactly-once for external side effects**; `requestId` dedupes input admission only.

Series hub (mentioned in notes, no separate folder): https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero

## Index

| Title | Notes | Author | Category |
|---|---|---|---|
| Pi Durable — official announcement | [official/01-pi-durable-announcement/article.md](official/01-pi-durable-announcement/article.md) · [article_zh.md](official/01-pi-durable-announcement/article_zh.md) | Earendil Engineering | `official` |
| Pi Durable on Cloudflare | [official/02-pi-durable-on-cloudflare/article.md](official/02-pi-durable-on-cloudflare/article.md) · [article_zh.md](official/02-pi-durable-on-cloudflare/article_zh.md) | Earendil Engineering | `official` |
| @earendil-works/pi-durable README | [official/03-durable-package-readme/article.md](official/03-durable-package-readme/article.md) · [article_zh.md](official/03-durable-package-readme/article_zh.md) | Earendil / pi-durable maintainers | `official` |
| pi-durable-cloudflare sample repo | [official/04-pi-durable-cloudflare-sample/article.md](official/04-pi-durable-cloudflare-sample/article.md) · [article_zh.md](official/04-pi-durable-cloudflare-sample/article_zh.md) | earendil-works | `official` |
| Cloudflare Agents SDK — PiHarness | [official/05-cloudflare-agents-pi/article.md](official/05-cloudflare-agents-pi/article.md) · [article_zh.md](official/05-cloudflare-agents-pi/article_zh.md) | Cloudflare | `official` |
| Vercel — Running Pi Durably | [official/06-vercel-running-pi-durably/article.md](official/06-vercel-running-pi-durably/article.md) · [article_zh.md](official/06-vercel-running-pi-durably/article_zh.md) | Vercel | `official` |
| Agent-Engineering — Introduction | [tutorials/01-agent-engineering-intro/article.md](tutorials/01-agent-engineering-intro/article.md) · [article_zh.md](tutorials/01-agent-engineering-intro/article_zh.md) | Nelson Lin | `tutorials` |
| Agent-Engineering — First conversation | [tutorials/02-agent-engineering-first-conversation/article.md](tutorials/02-agent-engineering-first-conversation/article.md) · [article_zh.md](tutorials/02-agent-engineering-first-conversation/article_zh.md) | Nelson Lin | `tutorials` |
| Agent-Engineering — SQLite & resume() | [tutorials/03-agent-engineering-sqlite-resume/article.md](tutorials/03-agent-engineering-sqlite-resume/article.md) · [article_zh.md](tutorials/03-agent-engineering-sqlite-resume/article_zh.md) | Nelson Lin | `tutorials` |
| pi-durable, from the ground up | [tutorials/04-from-the-ground-up/article.md](tutorials/04-from-the-ground-up/article.md) · [article_zh.md](tutorials/04-from-the-ground-up/article_zh.md) | Akshay Parkhi | `tutorials` |
| Pi Durable in Practice (Joulyan) | [tutorials/05-joulyan-in-practice/article.md](tutorials/05-joulyan-in-practice/article.md) · [article_zh.md](tutorials/05-joulyan-in-practice/article_zh.md) | Joulyan IT | `tutorials` |
| Pi Durable cheatsheet | [tutorials/06-cheatsheet/article.md](tutorials/06-cheatsheet/article.md) · [article_zh.md](tutorials/06-cheatsheet/article_zh.md) | luongnv.com | `tutorials` |
| Orbi — harness vs Durable vs kill -9 | [practice/01-orbi-harness-vs-durable/article.md](practice/01-orbi-harness-vs-durable/article.md) · [article_zh.md](practice/01-orbi-harness-vs-durable/article_zh.md) | Larry (Orbi) | `practice` |
| Bokvi — survives its own crashes | [practice/02-bokvi-survives-crashes/article.md](practice/02-bokvi-survives-crashes/article.md) · [article_zh.md](practice/02-bokvi-survives-crashes/article_zh.md) | Tomáš | `practice` |
| WithOne — crash-safe Gmail draft | [practice/03-withone-gmail-idempotency/article.md](practice/03-withone-gmail-idempotency/article.md) · [article_zh.md](practice/03-withone-gmail-idempotency/article_zh.md) | One Team | `practice` |
| daily.dev / Flavio walkthrough | [practice/04-daily-dev-walkthrough/article.md](practice/04-daily-dev-walkthrough/article.md) · [article_zh.md](practice/04-daily-dev-walkthrough/article_zh.md) | Flavio Copes | `practice` |
| What is a Harness? | [background/01-what-is-a-harness/article.md](background/01-what-is-a-harness/article.md) · [article_zh.md](background/01-what-is-a-harness/article_zh.md) | Earendil Product | `background` |
| How Compaction Works in Pi | [background/02-compaction-in-pi/article.md](background/02-compaction-in-pi/article.md) · [article_zh.md](background/02-compaction-in-pi/article_zh.md) | Earendil Engineering | `background` |
| Pi 1.0 announcement | [background/03-pi-1-0/article.md](background/03-pi-1-0/article.md) · [article_zh.md](background/03-pi-1-0/article_zh.md) | Earendil | `background` |

## Layout

```
articles/2026-11-06-pi-durable/
  README.md / README_zh.md / meta.json
  official/…/  tutorials/…/  practice/…/  background/…/
    article.md  article_zh.md  meta.json
```

## Fetch status

All listed source URLs were fetched successfully for this archive (GitHub sample via README raw; daily.dev via linked Flavio full article). No stub “fetch failed” notes.
