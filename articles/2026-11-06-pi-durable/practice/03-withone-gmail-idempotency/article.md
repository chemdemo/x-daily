---
title: "WithOne — Crash-safe Gmail draft after Pi Durable kill (structured notes)"
author: "One Team"
source_url: "https://www.withone.ai/blog/pi-durable-crash-safe-one-cli"
published_at: "2026-10-02"
archived_at: "2026-11-06"
category: "practice"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

withone.ai / One Team (2026-10-02). Practical write-up of killing a Pi Durable agent the instant a Gmail draft create returns, then resuming. Tools wrap One CLI (`one --agent actions execute`) for Hacker News reads and Gmail drafts. Demonstrates why “replay policy alone” is not enough for writes.

## Setup sketch

Connect Gmail + HN via One CLI; export Anthropic + connection keys. Agent reads HN story/comments then saves exactly one draft summarizing the thread. `CRASH_AFTER_DRAFT` kills with SIGKILL right after Gmail answers—so Gmail has the draft while Pi has committed intent but no tool result. Storage: per-mode SQLite file; `harness.resume()`; fixed `requestId` (`hn-digest-…`).

## Observed behavior (naive draft tool, default unsafe)

| Model | Post-crash behavior | Gmail drafts | Finished alone? |
|---|---|---|---|
| Sonnet 5.5 | Refused retry; asked human to check | 1 | No |
| Haiku 4.5 | Retried tool; said it saved one | **2** | Yes, wrongly |

Safe HN tools already in the transcript were not refetched. Submission dedupe via `requestId` worked. Sample size small (2× each) but stable across repeats.

## Keyed safe rewrite

Mark `replay: "safe"` only after making execute idempotent. Key from **`api.callId`** (model’s tool-call id stored on the task)—author’s first attempt used numeric `taskId` and Haiku hit type errors; task ids also collide across separate DB files. Put `piref`+hash(callId) in the draft body. Flow: memo hit → return; else Gmail search for marker; if found reuse; else create (crash point); memo draftId. Resume log: “already in Gmail as …, not creating another.” Transcript shows one call/one result with no restart narrative.

## Still fragile

Gmail search 401 shortly after create on one Sonnet run; search index lag not measured (miss ⇒ duplicate); sends not tested; other writes still unsafe by default. Recovery reruns only when **both** stored and current policy say safe (per harness/tool.ts note in article).

## Takeaways for an in-house harness

Standardize a “write tool checklist”: callId key, external uniqueness query, memo, crash test after provider success. Do not ship `replay: "safe"` on mail/deploy/charge without that checklist. Treat model personality under interrupted unsafe tools as a product risk, not an implementation detail.

## Quote

> “A tool call that was cut off reruns if it is safe to; otherwise the model is told it was interrupted.”
> — Earendil line cited in https://www.withone.ai/blog/pi-durable-crash-safe-one-cli
