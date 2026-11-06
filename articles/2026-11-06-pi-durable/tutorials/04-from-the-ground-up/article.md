---
title: "pi-durable, from the ground up (structured notes)"
author: "Akshay Parkhi"
source_url: "https://www.akshayparkhi.net/2026/Oct/20/pi-durable-from-the-ground-up/"
published_at: "2026-10-20"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Akshay Parkhi weblog post (2026-10-20) with runnable companion repo https://github.com/avparkhi/pi-durable-hello — five scripts, one SQLite file, free `faux` model by default (no server/UI/cloud required). Example pins pi-durable 1.0.3.

## Problem and idea

A normal agent loop keeps work in process memory. Chat resume restores text, not which tool was running, what it already changed on disk, what waited in the queue, or what the app believed. Pi Durable commits every step before showing it—the process becomes disposable. Once a run survives its process, the agent can live on a machine (not a terminal window), many clients can watch/steer, and the agent can wait hours for a human. One-liner: **the agent outlives the client**.

## Building blocks (real types)

Storage (SQLite/JSONL/memory; one process owns a file); Harness (atomic commit line); Conversation (`root`); Entry kinds (`pi.user`, `pi.assistant`, `pi.tool-result`, `pi.system`, `pi.reset`, `pi.compaction`); Commit; Document; Task (checkpointed state machine); Submission (+ optional `requestId`); Extension (conversations store names); Watch (UI is only a watcher; late joiners get a snapshot).

Flow: submit → `pi.user` → generation → assistant (tool calls) → tool tasks → results → generation → answer → submission done. Each line is a commit/checkpoint—crash at any line is recoverable in storage terms.

## The crash-safety rule

Tools declare replay policy. `replay: "safe"` → rerun after crash (e.g. file read). Otherwise → interrupted result (“may have partially run”); model decides; harness does not silently repeat side effects. Honest limit: **durable = checkpointed, not exactly-once**. Payments/deploys need their own idempotency keys.

## Five lessons

1. **hello** — store Q&A in `.data/agent.sqlite`; second run still shows first entries; `pi.system` stores prompt/tools, later only diffs (prompt-cache friendly).
2. **crash** — `!sleep 4 && echo FINISHED` → bash runs → `process.exit(1)` mid-tool; user/assistant/tool task already durable.
3. **resume** — `harness.resume()`; bash not safe → interrupted tool-result → model answers; `harness.submission(id)` waits from another process.
4. **twice** — same `requestId` twice → same submission id (`same=true`).
5. **watch** — `watchEvents()` + steer while busy (inbox_update → turn_start → steer answer).

Maps to Durable Pi HTTP apps: per-device SQLite, POST messages with requestId, SSE snapshot/deltas, resume on device restart, Steer button = `whenBusy: "steer"`.

## Patterns unlocked / limits

Proactive agents (events start runs); multi-client agents; human-near-loop approvals as durable waiting tasks. Limits: not exactly-once; one process per file; bash = full user power; experimental pin.

## Takeaways for an in-house harness

Clone-and-run teaching repos beat slideware for onboarding. Mirror the five lessons as internal acceptance tests. Keep the “UI is a watcher of committed state” slogan in frontend design reviews.

## Quote

> “Durable means checkpointed, not exactly once. A payment or a deploy still needs its own idempotency key.”
> — https://www.akshayparkhi.net/2026/Oct/20/pi-durable-from-the-ground-up/
