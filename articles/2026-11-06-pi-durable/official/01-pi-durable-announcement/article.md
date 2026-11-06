---
title: "Pi Durable — official announcement (structured notes)"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/pi-durable/"
published_at: "2026-10-01"
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Earendil Engineering announcement shipping Pi Durable alongside Pi 1.0. Primary design overview.

## Background / What it is

Pi Durable is an **experimental** TypeScript package (`@earendil-works/pi-durable`) for long-running, multi-client agent applications. It does **not** replace the Pi coding agent (terminal, one person). It is a separate harness: storage + machinery to run conversations with LLMs in parallel, with tools and execution environments.

Shared with Pi: `pi-ai`, minimalism, malleability. Package size ~15k lines without tests (~150k–250k tokens)—small enough for an agent to read.

## Key ideas

- **Harness** = one open storage + runtime. All changes go through atomic **commits**; nothing is shown before it is stored.
- **Conversation** = immutable transcript of **entries** (user, assistant, tool-result, system, reset, compaction, custom kinds).
- **Task** = durable state machine with a **checkpoint** before each step. Model calls (`pi.generation`), tool calls (`pi.tool`), compaction, and custom `defineTask` work are all tasks.
- **Document** = typed JSON next to the transcript (`defineDoc`), changed in the same commits.
- **Submission** + optional **`requestId`** = exactly-once admission of input (retries return the same submission).
- **Extension / Registry** = pluggable tools, sections, hooks, wraps, tasks; conversations store **names**, not code.

## Methods / design points

1. Open storage (`MemoryStorage`, SQLite, JSONL, or custom) → `Harness.open` → `root()` → `submit` → `wait`.
2. On restart: reopen same storage, `harness.resume()`, unfinished tasks continue from last checkpoint.
3. Tools declare `replay: "safe"` (rerun after crash) or default unsafe (model gets interrupted result).
4. Forks share parent history up to an entry without copying; concurrent conversations; per-conversation agent config.
5. Subagents = ordinary conversations owned by a tool task (`ownership: { kind: "task", taskId }`).
6. Compaction is a background task; older messages stay in storage; `reset` / handoff keep history searchable.
7. Multiplayer: clients attach via `viewState` / `watch` / `watchEvents`; steer with `whenBusy: "steer"`.
8. Hot reload: `registry.install` same name replaces extension; running calls finish on old code.

## Takeaways for an in-house harness

- Commit-before-show + task checkpoints are the durability core—not “save the chat JSON.”
- Separate **input idempotency** (`requestId`) from **external side-effect idempotency** (tool/provider keys).
- Ownership trees (task → child tasks/conversations) give clean abort/cleanup semantics.
- Pluggable storage + execution env lets the control plane and tool sandbox live on different machines.

## Limitations / caveats

- Experimental API; one process owns a storage (no cross-process lock).
- Durable ≠ exactly-once for external effects; bank/deploy examples still need provider idempotency keys.
- SQLite `synchronous = NORMAL`: process crash OK; power/host failure may lose newest commit.

## Quote

> “Conversations, model turns, tool calls, and your own state are committed to storage before anything is shown. If the process dies mid-turn, reopening the storage picks the work up where it stopped.”
> — https://earendil.com/posts/pi-durable/
