---
title: "Pi Durable cheatsheet — luongnv (structured notes)"
author: "luongnv.com"
source_url: "https://luongnv.com/awesome-cheatsheets/cheatsheets/pi-durable/"
published_at: null
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Community cheatsheet at luongnv.com aggregating Pi Durable quick starts, API tables, Cloudflare `PiHarness` notes, proactive-agent patterns, and pointers to the `disler/durable-pi` demo network. Use as a **map and checklist**, not as a substitute for the official README—APIs drift; some snippets still show older field names (e.g. tool result `content` vs newer `output`).

## Prerequisites recalled

Node ≥22.19 (or Bun/DO with adapters); ESM TypeScript; provider key; optional Pi checkout for 30+ examples; optional Cloudflare Workers SQLite DO; optional community demo (Node 24, `just`, OpenRouter). Durable does not replace `pi-dev` for short supervised terminal sessions. Package is experimental—pin exact versions and re-read README for your installed version.

## Step ladder (nine rungs)

1. MemoryStorage quickstart Q&A (`submit`/`wait`/`AssistantEntry`)
2. SQLite + `resume` + `requestId` + reopen on `harness.closed` failed
3. CodingTools + `NodeExecutionEnv`; `defineTool` with replay policy; wrap/override by name
4. Per-conversation `configure` / fork / `whenBusy` steer|followUp|reject; harness `settings`
5. Hooks + `api.memo` approvals; subagent ownership; `defineTask` waiting/`failFast`; background tasks
6. Nested `executeTool`, `callers`/`modelTools`, codemode sandbox, deferred `tool_search`
7. `defineDoc`, compaction/reset/handoff, hot `registry.install` same name
8. Multiplayer `watch`/`watchEvents`; Cloudflare `PiHarness` + `operationId`
9. Proactive: event fingerprint → commit incident+conversation → approval child task that sleeps/polls a decision document

## Best practices vs don’ts (distilled)

Do: treat `replay: "safe"` as a contract; derive requestId/operationId from stable ids; memo hook decisions; keep sections stable for prompt cache; prefer documents over side tables; choose document `fork` policy deliberately; give subagents cheaper models and strip recursive extensions; raise progress intervals on remote storage; check usage docs; restrict recursive `callers`; put cheap classifiers in front of proactive agents; decide fail-open vs fail-closed for guard hooks.

Don’t: dual-open one storage; put wall-clock in sections; assume `abort()` returns immediately; confuse PiHarness API with core API; rely on MemoryStorage outside tests; float versions; treat classifier guards as sandboxes or device tokens as less than shell.

## Takeaways for an in-house harness

Keep this page beside the official README as an operator checklist. Every copied snippet must be diffed against the installed package version before landing in production templates. The proactive “approval as durable child task” pattern is especially relevant for coding agents that deploy.

## Caveat on community demos

The cheatsheet references `disler/durable-pi` and proactive `just` recipes. Those are **community** demos with their own pins (often older pi-durable), unsandboxed bash, and localhost-only assumptions. Steal patterns (event fingerprint → one conversation; approval as child task) but re-implement against your pinned official package version and your security boundary—do not paste demo tokens or fail-open bash gates into production.

## Quote

> “The package is experimental: ‘The API changes without notice between releases.’ Pin exact versions…”
> — https://luongnv.com/awesome-cheatsheets/cheatsheets/pi-durable/
