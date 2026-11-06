---
title: "Cloudflare Agents SDK — PiHarness (structured notes)"
author: "Cloudflare"
source_url: "https://developers.cloudflare.com/agents/harnesses/pi/"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Cloudflare Agents documentation: running Pi Durable via `PiHarness` Lifecycle capability (page last updated 2026-11-03 in fetch). Prefer this when you are already on the Agents SDK rather than hand-rolling DO alarms.

## Background / How it works

Pi runs the agent loop (transcript, inbox of follow-ups/steers, model calls, tools, retries, crash recovery). `PiHarness` supplies SQLite storage in the Durable Object and keeps the object awake when Pi has work.

Lifecycle concept: a reusable DO piece that receives SQLite + a durable job queue. `PiHarness` uses them for:

- **Storage** — Pi tables prefixed `pi_` so they do not collide with yours.
- **Wake-up** — Pi’s scheduler is in-memory; an evicted object has none. `PiHarness` schedules one Lifecycle job from Pi’s `wake()`: heartbeat while active, due when the next sleep wakes, complete when idle. Eviction mid-run → job alarm restarts the object → Pi reopens storage and continues.

`PiHarness` does not dictate transport; you watch session state and send it over WebSockets, HTTP, or RPC.

## Install / version coupling

```
npm i agents @earendil-works/pi-durable @earendil-works/pi-ai @earendil-works/pi-schema @earendil-works/chord
```

Pi packages are optional peers of `agents` at ~1.5.0. Agents SDK **0.33+** supports Pi 1.5. A newer Pi minor can write storage an older minor cannot read—**roll forward**, do not roll back after conversations ran on 1.5. Wrangler needs `nodejs_compat`, AI binding (if using Workers AI), and a SQLite-backed DO migration (`new_sqlite_classes`).

## Key APIs

Factory on a plain `DurableObject`: build `PiHarness` with `harness: ({ storage, context, onWake }) => Harness.open(...)`, `defaults: { model, thinkingLevel, maxTurns }`, then `Lifecycle.install(this).use(harness)`.

- `prompt` / `submit`+`wait` with **`operationId`** (idempotent submit; SDK-level cousin of core `requestId`).
- Busy: queued follow-up by default; `whenBusy: "steer"`; `whenIdle: "reject"`.
- Label machine-originated input with custom `entry.kind` / `data` (never `pi.*` prefix)—model still sees text as user message.
- `settings.afterFailedRun`: `keep` (default) / `continue` / `withdraw`.
- Sessions: root id `"1"`; `sessions.create` / `fork` / `list`. One DO per user/chat if you want isolation at the object boundary.
- `watch` for UI; after object restart open a new watch and resend first value.
- `pending()`, `continue()`, `abort()`, `reset(handoff)`, `setModel`, `messages` / `entries`.

Recovery: eviction/crash/CPU-memory → next heartbeat (~30s while working). Streaming cut-off → aborted partial + retry policy. Safe tools rerun; others interrupted. `restartLimit` (default 5) ends crash loops. Progress deferred up to `persistWithinMs` (~1s) by default.

## Takeaways for an in-house harness

- Prefer Lifecycle-integrated wake over reinventing alarm math when on Cloudflare Agents.
- Pin `agents` and Pi minor **together**; document that storage format can be forward-only across minors.
- In wrappers, name SDK `operationId` vs core `requestId` explicitly to avoid dual abstractions leaking into product code.
- Tools that must not double-fire still need app idempotency even when marked unsafe—models may re-issue calls after an interrupted result.

## Limitations (documented)

No built-in tool approval UI. `abort()` waits until idle—tools ignoring abort keep the session busy. One model stream longer than ~15 minutes can be cut by alarm invocation limits. Background completion noticed up to ~30s late. No conversation deletion yet. Running work prevents graceful drain/eviction.

## Quote

> “Pi’s scheduler runs in memory, and an evicted object has none. PiHarness schedules one Lifecycle job for the object, driven by Pi’s wake().”
> — https://developers.cloudflare.com/agents/harnesses/pi/
