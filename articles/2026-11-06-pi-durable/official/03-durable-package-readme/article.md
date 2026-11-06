---
title: "@earendil-works/pi-durable README (structured notes)"
author: "Earendil / pi-durable maintainers"
source_url: "https://github.com/earendil-works/pi/blob/main/packages/durable/README.md"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Package README (also raw): https://raw.githubusercontent.com/earendil-works/pi/main/packages/durable/README.md  
Normative companion: `packages/durable/docs/spec.md`. Over 30 examples under `test/examples`.

## Background

Authoritative API dictionary for the experimental harness. Install: `npm i @earendil-works/pi-durable @earendil-works/pi-ai @earendil-works/chord`.

## Key concepts (compact)

Harness, Conversation, Entry, Commit, Document, Task, Submission, Turn/Run, Extension, Registry, Agent—as in the announcement, with operational detail: adapters never stored in Context; cancel wait ≠ cancel work; telemetry spans (`pi.durable.*`) avoid logging message bodies.

## Persist and resume

- SQLite: `openNodeSqliteStorage("./session.sqlite")` then `resume()`.
- Duplicate `requestId` returns existing submission; `harness.submission(id)` reacquires after restart.
- Sleep: end phase with `sleepUntil` / sleeping commit—no sleep mid-phase (phases rerun from top).
- Hosts that hibernate: pass `onWake`; host arms alarm from `wake()` (`active` / `at`); call `resume()` on alarm. Default setTimer declines memory-only timers so Workers can hibernate.
- Crash loops: model request cut off counts as failed attempt under `settings.retry`; other tasks accumulate restarts up to `settings.restartLimit` (default 5) then fault/interrupt with report.
- `settings.afterFailedRun`: keep (default) / continue / withdraw queued inputs.
- `readOnly: true` for spectators that must not resume or write.

## Extensions, tools, code mode

- Tools: intent committed before execute; `replay: "safe"` or interrupted; channels `output` / `structuredOutput` / `details`.
- Nested `api.executeTool` with keys for replay-safe reattachment; `callers` / `modelTools` / deferred + `tool_search`.
- Codemode: QuickJS sandbox, journaled results for deterministic resume; Cloudflare uses `RemoteCodemodeSandbox`.
- Hooks: beforeRequest, beforeTool (memo for decisions), afterTool, onYield, afterRun (run-end child tasks), etc.
- Busy inbox: followUp (default), steer, reject; `submit(tx, draft)` admits inside host commit; continue after failed run.

## Storage backends

| Backend | Notes |
|---|---|
| Memory | Tests |
| IndexedDB | Browser + Web Lock |
| SQLite Node | WAL, NORMAL; working set in RAM |
| JSONL | Append-only; whole history in RAM—poor for infinite agents |
| Cloudflare DO SQLite | `openDurableObjectSqliteStorage` |

Custom backends: `registerStorageConformance`. Optional table `prefix` (e.g. `pi_`).

## Takeaways

- Treat README + examples as the contract; pin versions.
- Design tools around replay policy, structuredOutput, and ownership.
- For serverless: implement `onWake` + reopen-on-`closed.failed` carefully.

## Limitations

API changes without notice. One process owns storage. Deferred progress commits lose up to `persistWithinMs` (~1s) on crash. Abort waits for idle; tools ignoring abort delay it.

## Quote

> “A durable agent harness. Conversations, model turns, tool calls, and your own state are committed to storage before anything is shown.”
> — https://github.com/earendil-works/pi/blob/main/packages/durable/README.md
