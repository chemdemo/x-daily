---
title: "Vercel — Running Pi Durably (structured notes)"
author: "Vercel"
source_url: "https://vercel.com/docs/agent-harnesses/pi"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Vercel documentation for running Pi durably on Vercel Functions using the `durable-harness` package. Two variants exist; the docs recommend the full Pi Durable path.

## Choose a variant

| | `durable-harness/pi-durable` (recommended) | `durable-harness/pi` |
|---|---|---|
| Best for | Most Pi agents | Only need Pi’s agent loop |
| What you write | Your Pi code on `openWorldStorage()` | `pi({ model, tools })` |
| Features | Full Harness: extensions, compaction, forks | Agent loop + tools |
| Speed | Each step waits on the log (slower) | Less durability latency |
| Steering | Another request can take the session over | Redirect a running turn from any request |
| Reconnecting | Resends in-progress response from its start | Continues from last event cursor |

Pick `pi-durable` if you already have Pi Durable code or need forks/compaction/extensions—only the storage changes.

## Recommended path (`pi-durable`)

1. Install `durable-harness`, `@earendil-works/pi-durable`, `chord`, `pi-ai`.
2. Module that calls `resumeWith((storage) => Harness.open(storage, { models, registry }, ctx))`—Vercel builds fail if the app opens pi-durable sessions without `resumeWith`.
3. Replace SQLite/JSONL/memory with `openWorldStorage(sessionId)` (creates a new session if id omitted).
4. Add `chatRoute` (POST submit + SSE; GET reconnect) wrapped in session ownership / `authorize`. Production without `authorize` returns 401; public access needs explicit flags (and sandbox budget rules if using Sandbox).
5. Queue route via `flowHandler` + `vercel.json` / `harnessFunctions()` / Next.js `withDurableHarness`.
6. Tools: install `CodingTools` with `virtualEnv()` or `sandboxEnv()`—**do not** point `env` at the Function’s own filesystem/processes. Mark tools `replay: "safe"` when re-running is harmless.

## Differences from Pi’s own storage

- Opening a session **takes it over**; the previous opener stops at its next write. Retry with the same `requestId`. `chatRoute` handles loss/retry.
- Background work (compaction, background subagents) finishes via the queue after you close the Harness; `chatRoute` waits for it.
- A request stays alive while work runs, ending when done or after `recoverAfterMs` (default **120s**) with no commit—quiet long tools need a raised limit.
- On crash: finished steps from the log; model calls and safe tools rerun; unsafe tools return interrupted.

Steer with `"kind": "steer"` on the chat route or `whenBusy: "steer"` in server code. Reconnect: pi-durable GET without cursor (replay in-progress response from start; handle `rewind`); thin `pi` variant uses event cursors.

## Takeaways for an in-house harness

- On Vercel, durability is **log + queue takeover**, not a single long-lived process—design tools and env accordingly.
- Workspace must travel with the session (`virtual` / Sandbox), matching “control plane ≠ tool sandbox.”
- Multi-agent apps need stable `name` on `resumeWith` / `pi({ name })`.
- Self-host: another World (e.g. Postgres) via `WORKFLOW_TARGET_WORLD` + `registerQueue` / `world.start()`.

## Limitations / caveats

Auth and ownership stores are your responsibility. Public routes spend AI Gateway credits. Compare reconnect UX carefully when choosing variants.

## Quote

> “Give Pi’s tools a workspace, not your function.”
> — https://vercel.com/docs/agent-harnesses/pi
