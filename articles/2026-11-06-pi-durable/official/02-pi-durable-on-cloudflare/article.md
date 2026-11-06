---
title: "Pi Durable on Cloudflare (structured notes)"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/pi-durable-on-cloudflare/"
published_at: "2026-10-16"
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Earendil Engineering’s second official Pi Durable post (2026-10-16). Companion runnable sample: https://github.com/earendil-works/pi-durable-cloudflare. Read after the announcement when you care about serverless hosting.

## Background / What it is

Many agents respond to an event, work for a while, then idle. Paying for a always-on machine is wasteful. Cloudflare Durable Objects (DO) hibernate when idle and wake on HTTP, WebSocket, or alarm. This post shows a **multiplayer chat agent** that stores **one Pi Durable session per DO**, uses DO SQLite, hibernates cleanly, wakes for scheduled work, and searches across conversations with a subagent.

Architecture sketch: React client → PartySocket → Worker (route by session ID) → SQLite-backed DO (hibernatable WebSockets, embedded SQLite, Pi Harness → model provider). Shareable session URLs; same URL → same object; different IDs → isolated state.

## Key ideas

1. **Session boundary = DO instance**, not “one giant transcript.” Inside a session you create independent conversations and forks from committed entries.
2. **Storage adapter**: `openDurableObjectSqliteStorage(ctx.storage)` over DO `SqlStorage`. Bring-your-own backends remain possible; run `registerStorageConformance` with Workers Vitest integration for DO adapters.
3. **Lifecycle**: DO constructor runs on create and after hibernation/eviction. Use `blockConcurrencyWhile` to initialize harness before events proceed. On harness `closed` with failure, `ctx.abort` so the next event reopens storage.
4. **Idempotent conversation create/fork**: store a receipt document keyed by client `requestId` in the same commit that creates the conversation—critical after WebSocket drops.
5. **Streaming**: per socket, `watchEvents` sends snapshot then commit deltas (transcript, streaming output, tools, compaction, resets, inbox). Client keeps a session snapshot for the sidebar separately.
6. **Wake model**: restore streams from socket attachments (username + conversationId); clients resend unconfirmed work under the same requestId. Project Pi deadlines onto the **single** DO alarm; while work is in progress, set a short recovery alarm so eviction still comes back. Alarm handlers must be idempotent (at-least-once).
7. **Search subagent**: `search_history` is `replay: "safe"`, owns a child conversation, searches other top-level transcripts (not its own / not prior search artifacts), returns only the final answer to the parent. Reuse child by `ownerTaskId` on replay.

## Methods / how-to (useful level)

Local run needs Node 22.19+, pnpm 11, OpenAI key: clone sample, `pnpm install`, `.dev.vars`, `pnpm dev`. Suggested try sequence: state a fact in conversation A; ask about it in conversation B; inspect search subagent; schedule weather in two minutes and leave.

Schedule tool → durable Pi sleep task → DO alarm. Refresh alarm from `harness.inspect()` tasks each wake.

## Takeaways for an in-house coding-agent harness

- If your control plane can sleep, you **must** externalize the wake signal (alarm/queue/cron). In-memory Pi timers alone are insufficient on hibernating hosts.
- Multi-conversation products need fork + receipt + ownership patterns, not only “save messages.”
- Treat Activity / task-graph visibility as a first-class operator UI (the sample’s Activity panel).
- Conformance tests for storage adapters are part of production readiness, not optional polish.

## Limitations / caveats

Sample has **no authentication**—anyone who can hit a deployed Worker spends your model budget. Alarms are at-least-once. Demo search is simple substring matching. Pin experimental Pi Durable versions when copying patterns.

## Quote

> “Pi can track any number of deadlines, but a Durable Object has only one alarm.”
> — https://earendil.com/posts/pi-durable-on-cloudflare/
