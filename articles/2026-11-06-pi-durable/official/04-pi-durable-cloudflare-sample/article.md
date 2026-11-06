---
title: "pi-durable-cloudflare sample repo (structured notes)"
author: "earendil-works"
source_url: "https://github.com/earendil-works/pi-durable-cloudflare"
published_at: null
archived_at: "2026-11-06"
category: "official"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Official sample repository README for running Pi Durable inside Cloudflare Workers / Durable Objects with a React multiplayer client. Design narrative lives in the Earendil “Pi Durable on Cloudflare” post; this repo is the concrete integration.

## Background / What it is

A **working example**, not a production service: shared chat sessions, multiple concurrent conversations, forks from earlier messages, reconnect without losing committed transcripts or queued prompts, one-time/recurring scheduled follow-ups, a history-search subagent, and an **Activity** panel to inspect live task ownership, waits, sleeps, tool calls, and background work—and to retry tasks parked after repeated crashes.

**Intentionally no authentication or rate limiting.** Deploying publicly would let strangers spend your model credentials.

## Key layout (where to look)

| Path | Role |
|---|---|
| `src/chat.ts` | Models, extensions, tools, default agent |
| `src/session.ts` | Hosts harness on built-in DO SQLite; wires hibernating WebSockets + alarms |
| `src/tools.ts`, `src/schedule.ts`, `src/search.ts` | Example tools and durable tasks |
| `src/client/` | React client + event-driven session state |
| `test/` | App flows: restart recovery, scheduling, forks, inboxes; DO storage adapter tested inside a Durable Object |

The README’s stance: **the code (especially tests) is the documentation** for restart/schedule/fork/inbox behavior.

## Methods / how to run

Requirements: Node.js **22.19+**, **pnpm 11**, OpenAI API key.

```
pnpm install
cp .dev.vars.example .dev.vars   # set OPENAI_API_KEY
pnpm dev                         # Worker + Vite client
```

Open the Vite URL, create a session, open the same URL in a second browser for multiplayer. Commands also include `pnpm test`, `pnpm check`, `pnpm build`, `pnpm preview`.

Deploy path: `wrangler login` → `wrangler secret put OPENAI_API_KEY` → `pnpm deploy`. Add authz and rate limits before any public exposure.

## Takeaways for an in-house harness

- Copy structural patterns (session DO, alarm refresh, event stream attachments, Activity UI) rather than treating the demo as a product skeleton.
- Invest in **restart tests** like this repo’s suite—blog posts drift; tests track API reality.
- Separate “demo credentials on a laptop” from “tenant-isolated production”: this sample proves durability mechanics, not tenancy.

## Limitations / caveats

Experimental Pi Durable APIs may change. No auth. Further reading pointed by README: announcement, `packages/durable` docs/examples, Cloudflare Durable Objects docs.

## Quote

> “This is a sample, not a production service. It intentionally has no authentication or rate limiting.”
> — https://github.com/earendil-works/pi-durable-cloudflare
