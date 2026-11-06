---
title: "Agent-Engineering — SQLite, crash recovery, resume() (structured notes)"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/pi-durable-persistence-sqlite-crash-recovery-and-resume"
published_at: "2026-11-04"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Series article 3 (Nelson Lin, 2026-11-04): SQLite persistence, crash recovery, `resume()`, replay-safe tools, and `requestId` idempotency—complete runnable script continuing the gateway + Deepseek setup from article 2.

## What you will be able to do

Kill a running agent mid-tool-call and have a new process pick up exactly where durable state left off. Understand what is on disk vs what “resume” means vs what replay-safe means.

## Complete pattern (compressed)

- Open `./session.sqlite` via `openNodeSqliteStorage`
- Install a `service_status` tool with `replay: "safe"` and a long `setTimeout` so you can kill mid-flight
- `Harness.open` → `root` with instructions to use the tool → **`harness.resume()`** before submit
- Submit with stable `requestId` (e.g. `demo-checkout-status-1`)
- Wait; on done, read `AssistantEntry`
- Always `close` in `finally`

Kill test: run → see `Tool started: checkout` → `pkill -9 -f index.ts` → rerun same command → same submission id, tool reruns, model answers. Files: `session.sqlite` + WAL/SHM. Install `@earendil-works/pi-schema` when on pi-durable 1.5+ tool params.

## What is persisted / unit of recovery

Atomic commits hold conversations & immutable entries, full task records (phase + checkpoint), submissions, documents, provider session UUIDv7 identities. Tasks (`pi.generation`, `pi.tool`, custom `defineTask`, compaction) are the recovery unit—each checkpoints before moving on; reopen continues from last checkpoint.

## Storage table (article)

| Backend | Fit |
|---|---|
| Memory | Tests/demos |
| SQLite Node | Most long-running apps; working set in RAM |
| JSONL | Debug / small histories; whole history in RAM |
| Cloudflare DO | Edge sample path |

SQLite notes: WAL, `synchronous = NORMAL` → process crash OK; power/host failure may lose newest commit.

## `resume()` and `requestId`

`resume()` starts the scheduler and continues unfinished/sleeping-due tasks. Submit/wait also start it; calling explicitly documents the recovery path. Same `requestId` after restart returns the existing submission (exactly-once admission). `harness.submission(id)` reacquires for waiting from another process.

## Limitations called out

One process owns storage (no built-in lock). Replay safety is an unchecked promise. Experimental API—pin and re-test. External side effects need their own idempotency even with perfect internal checkpoints.

## Takeaways for an in-house harness

Golden path CI: kill mid-safe-tool and mid-unsafe-tool. Document that `requestId` ≠ payment idempotency key. Prefer SQLite for unbounded agents; treat NORMAL durability as an SLO input.

## Quote

> “Only mark a tool replay: \"safe\" when re-executing it has no harmful external side-effects (or the external system is itself idempotent).”
> — https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/pi-durable-persistence-sqlite-crash-recovery-and-resume
