---
title: "Bokvi — Pi Durable survives its own crashes (structured notes)"
author: "Tomáš (blog.bokvi.com)"
source_url: "https://blog.bokvi.com/blog/pi-durable-agent-harness/"
published_at: "2026-10-03"
archived_at: "2026-11-06"
category: "practice"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Tomáš, blog.bokvi.com (2026-10-03, ~12 min). Combines close reading of Earendil’s announcement/README, frame-by-frame study of the vacation-planner demo recording, and an original `kill -9` experiment. MIT-licensed package context; Pi Durable shipped with Pi 1.0 on 2026-10-01.

## Why the author cares

Pi the coding agent stays a one-person terminal tool: if it dies, you look and continue. Pi Durable targets agents you leave running for weeks (Slack/triage/research). The missing piece between babysat coding agents and service-shaped agents is checkpointed tasks—including mid-tool-call death.

## Kill -9 experiment (author’s script)

Faux provider (no API key). Model requests two tools at once: `search` with `replay: "safe"` (2s fake search) and `book` without replay (2s fake card charge). Submit with `requestId: "job-42"`. Kill at 1.5s while both run. Reopen same SQLite:

1. Resubmit job-42 → same submission id (exactly-once admission).
2. Search reruns from scratch.
3. Book does **not** rerun; model gets interrupted (“may have partially run”) and must decide.

Interpretation: safe ⇒ at-least-once inside the harness; default ⇒ at-most-once auto-rerun; outside effects are exactly-once only if the far side is idempotent. Earendil’s checkout example passes `payment-${task.id}` to the bank for the same reason.

## Themes reinforced from announcement/demo

~15k LOC agent-readable source. Vacation demo: subagent parallel searches; process dies mid-trains; `--continue` only reruns unfinished safe work. Hooks use `api.memo` so Slack deploy approval is not asked twice after crash. Subagents/forks/steer/multiplayer are conversations + ownership + committed views. Compaction is a background task; older messages stay searchable after handoff/`reset`. Hot reload replaces extensions by name.

## Limits section (author’s “where it stops”)

Experimental APIs change without notice. Durable here means crash survival, **not** distributed execution—no cross-process lock (demo uses lockfile); rolling deploys need care. Crash means process crash; NORMAL SQLite can lose newest on power loss; JSONL needs `fsync: true` for flush-before-marker. Replay safety is your promise. TypeScript-first. API is low-level on purpose.

## Takeaways for an in-house harness

Reproduce the search/book kill test in CI with a faux model. Use the article’s wording when educating PMs: durable ≠ distributed ≠ exactly-once. Steal the memo-approval pattern for any human gate that must survive restarts.

## Quote

> “replay: "safe" makes a tool at-least-once and the default makes it at-most-once; an effect outside the harness happens exactly once only if the far side is idempotent.”
> — https://blog.bokvi.com/blog/pi-durable-agent-harness/
