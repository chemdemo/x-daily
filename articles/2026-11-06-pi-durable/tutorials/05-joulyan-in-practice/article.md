---
title: "Pi Durable in Practice — Joulyan (structured notes)"
author: "Joulyan IT"
source_url: "https://joulyan.com/en/blog/pi-durable-in-practice-building-agents-that-pick-up-where-they-left-off"
published_at: "2026-10-17"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Joulyan IT blog (2026-10-17, ~9 min): production-minded Pi Durable practice. Opening line sets the tone—Pi Durable helps resume after crashes, but saving a conversation will not stop a duplicate invoice.

## Recovery needs more than a transcript

Example: agent 40 minutes into maintenance, plan recorded, deploy started, container replaced. A transcript shows intent; recovery also needs which work completed, which state committed, and whether the deploy happened. Pi Durable provides persistence + execution machinery (Harness, Conversation, Task, Document via `defineDoc`, Submission) with atomic commits across those local pieces—**stopping at the storage boundary**. It cannot make a DB commit and an external email one indivisible operation.

## Startup and ownership checklist

- `npm install --save-exact` for `pi-durable`, `pi-ai`, `chord`; commit lockfile (experimental API).
- Put `./agent.sqlite` on a **persistent volume**, not container ephemeral disk.
- Exactly one process owner per storage; other clients attach to that owner—not multi-writer NFS.
- `harness.resume` on every startup before accepting new input.
- Stable `requestId` from webhook event ids (fresh UUID per attempt defeats dedupe).

## Classify tools before writing prompts

| Action | Replay | Why |
|---|---|---|
| Read config / query status | Often `replay: 'safe'` | Harmless / read-only |
| Start deploy / send invoice | Leave unmarked | Prior call may have already mutated the world |

Warning emphasized: Submission `requestId` dedupes **conversation input**, not payments/emails/deploys. Use provider idempotency or reconcile; Documents can store external op ids but cannot close the accept-before-local-commit crash window alone. Prefer workflows: start deploy → retain external op id → separate status check → require reconciliation/approval if start result unknown.

## Ops and capabilities

systemd `Restart=always` or container restart policy + volume. Horizontal scale = separate storage per agent/tenant. CodingTools + NodeExecutionEnv = your FS/shell permissions—constrain with `configure({ tools: { remove: [deploy] } })` and wrappers/hooks, not prompt hope. Compaction example settings: `reserveTokens: 16384`, `backgroundTokens: 32768`. No built-in subagents—build via ownership APIs. Hot `loadExtension` deserves recovery tests too.

## Prove before unattended / when to skip

Best test: interrupt **after external accept, before local result commit**. Also double-submit same requestId; kill mid-model and mid-tool. Skip Durable for one-shot scripts, stateless chat, or teams needing a frozen API this quarter. You still own credentials, UI, backups, supervision, external safeguards.

## Takeaways for an in-house harness

Make “tool contracts determine whether recovery is safe” an explicit design review gate. Ship one bounded agent with persistent SQLite and a deliberate crash test around the riskiest external action before expanding autonomy.

## Quote

> “A Submission’s requestId deduplicates conversation input. It does not make external actions happen exactly once.”
> — https://joulyan.com/en/blog/pi-durable-in-practice-building-agents-that-pick-up-where-they-left-off
