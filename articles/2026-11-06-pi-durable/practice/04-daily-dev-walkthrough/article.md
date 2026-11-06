---
title: "daily.dev / Flavio — Survive crashes with Pi Durable (structured notes)"
author: "Flavio Copes (via daily.dev link)"
source_url: "https://daily.dev/posts/how-to-build-agents-that-survive-crashes-with-pi-durable-zt8jcyleg"
published_at: "2026-11-04"
archived_at: "2026-11-06"
category: "practice"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

daily.dev post by Flavio Copes linking to the canonical long tutorial at https://flaviocopes.com/pi-durable (updated 2026-11-04). The daily.dev page summarizes Q&A; the Flavio article is the full walkthrough with interactive figures. Archive both URLs: discovery card here, substance from Flavio.

## What the walkthrough builds

Project `pi-durable-demo` with `agent.ts` + `tools.ts`. Tool `service_status` reads a fixture (website healthy / checkout degraded), waits 10 seconds (`context.abortSignal`), returns JSON, marked `replay: 'safe'`. Agent opens SQLite `./agent.sqlite`, installs extension, `resume()`, submits CLI `requestId` + prompt, prints assistant text, closes in `finally`. Hard kill skips `finally`—exactly the test case.

## Crash and retry labs

1. Mid-tool `kill -9` → restart same command → same submission id; tool restarts from top because the result was never committed (not a restored JS stack).
2. Re-run completed requestId → same submission/answer; tool does not start again.
3. Thought experiment: replace fixture with email sender accepted by provider then crash before local commit → auto-rerun could duplicate; default unsafe reports interrupted; app must still stop the model from issuing a fresh harmful call.

## Broader systems notes

Multi-client: one owner process; others connect to it; `viewState` for UIs; `whenBusy: "steer"` joins at tool-round boundary. Deploy: mount persistent volume; treat DB as sensitive application data. Compaction for long chats; keep critical incident fields in documents. Subagents require your own composition from conversations/ownership/tasks/forks. Path to Sentry-like agents: verify webhook → map investigation → stable id → narrow tools → PR → reconcile. Mentions alternative durable runners (Cloudflare Workflows, Vercel Workflow) as a separate choice.

## Takeaways for an in-house harness

Excellent onboarding lab: two files, one kill test, one idempotent retry test. Pair with WithOne’s Gmail article when teaching unsafe writes. Cite Flavio for substance; keep daily.dev as the indexed entry in this collection.

## How this fits the rest of the collection

Use Flavio’s two-file lab after Agent-Engineering articles 2–3, then escalate to Joulyan’s production checklist and WithOne’s Gmail write test before you mark any mutating tool `replay: "safe"`. The daily.dev card is useful for sharing a short link in Slack; point engineers at the flaviocopes URL when they need the full kill-test procedure and figures.

## Quote

> “Pi Durable resumes from durable checkpoints. It doesn’t restore a suspended JavaScript stack at the exact instruction that was interrupted.”
> — https://flaviocopes.com/pi-durable
