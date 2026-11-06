---
title: "Agent-Engineering — Introduction to Pi Durable (structured notes)"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/introduction-to-pi-durable-building-agents-that-survive-crashes-and-run-for-hours"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Nelson Lin, Agent-Engineering.dev — first article of “Comprehensive Pi Durable Agent From Zero to Hero” (published 2026-11-03). Series hub (not a separate archive folder): https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero. Pure mental models; no production code yet.

## Background / The problem

Most agents are ephemeral: state lives in process memory. When the process dies (crash, deploy, laptop sleep, serverless timeout), you may keep a partial chat log, but you lose in-flight tool calls, queued messages, and application-level belief about what was true. **Resuming a transcript is not resuming the work**—the system does not know which tool was running, whether it already mutated the world, or what sat in the inbox.

That gap is tolerable for interactive coding with a human watching. It is not tolerable for overnight research, on-call bots, multi-customer support, or unattended edge hosts.

## What Pi Durable is

Experimental durable agent harness from Earendil (shipped with Pi 1.0 on 2026-10-01). Definition used throughout the series: **harness = storage + machinery to run one or more LLM conversations in parallel**, providing tools and execution environments.

Not a replacement for the Pi coding agent (terminal, one driver). Separate library for any agentic app that needs durability. Shares `pi-ai` and the minimalism/malleability philosophy.

Core promise (paraphrased from Earendil): conversations, model turns, tool calls, and your application state are **committed before shown**; reopen the same storage to continue mid-turn work.

## Key concepts table (series vocabulary)

Harness; Conversation (immutable transcript); Entry; Task (durable state machine); Checkpoint; Document (typed JSON beside transcript); Commit (atomic all-or-nothing write); Submission (waitable input); Extension; Registry.

Recovery sketch: every step is a task → checkpoint before proceed → new process finds unfinished tasks → continue; tools declare replay-safe or get interrupted; partial model answers kept as aborted.

## Pi coding agent vs Pi Durable

| Aspect | Coding agent | Durable |
|---|---|---|
| Use | Interactive terminal | Long-running apps |
| Crash recovery | Manual continue | Automatic from checkpoints |
| Storage | Session files | Memory / SQLite / JSONL / custom |
| Concurrent conversations / multi-client | Limited / no | First-class / yes |
| Status | Stable 1.0 | Experimental |

## Who should care / roadmap

Useful for overnight agents, incident bots, multi-party collaborative agents, Cloudflare-style hosts, event-driven proactive agents. Less necessary for short supervised terminal sessions.

Series continues: setup → SQLite/resume → tools/replay → multi-conversation → documents → subagents → multi-client → deployment → end-to-end proactive agent. Packages: `@earendil-works/pi-durable` (+ `pi-ai`, `chord`); pin versions.

## Takeaways for an in-house harness

Adopt this vocabulary early so design reviews talk about tasks/checkpoints/documents instead of “save the chat.” Treat Durable as a **durable workflow engine specialized for LLM agents**, not a chatbot SDK with a database bolted on.

## Quote

> “Resuming a transcript is not the same as resuming the work.”
> — https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/introduction-to-pi-durable-building-agents-that-survive-crashes-and-run-for-hours
