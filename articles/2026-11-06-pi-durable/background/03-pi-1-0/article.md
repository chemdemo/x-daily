---
title: "Pi 1.0 announcement (structured notes)"
author: "Earendil"
source_url: "https://earendil.com/posts/pi-1-0/"
published_at: "2026-10-01"
archived_at: "2026-11-06"
category: "background"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Earendil (2026-10-01): Pi 1.0 launch post, published the same day as Pi Durable. Positions the coding agent as a hardened, minimal, extensible harness people and businesses can depend on, and explains why durability work was split into a separate experimental package.

## Pi 1.0 product stance

Hundreds of thousands of weekly users; months of hardening from issues/PRs. Minimalism is intentional: agent tooling churns weekly; Pi waits until features prove themselves before accepting complexity (codemode/MCP discussion referenced from earlier in the week).

## Features called out for 1.0

- Codemode (native MCP; non-LLM models like Jev and image models)
- Extension support for virtual models
- Deferred tool loading
- Cache warming for Anthropic models
- Mid-conversation system messages (transcript-aware prompt/tool changes)
- New TUI theme; full-screen mode by default

## Why Pi Durable exists as a sibling

Earendil wants Pi’s minimalism and “wield AI” ethos **outside** coding and **outside** the terminal: other surfaces, longer-running conversations/tasks—i.e. more durable. Rather than warp Pi into something it is not, they shipped **Pi Durable** as a new substrate sharing minimalism and supermalleability. Points readers to Mario’s Durable design post for details.

## Install

Pi: curl/PowerShell installers from `pi.dev`. Durable: `npm install @earendil-works/pi-durable @earendil-works/pi-ai @earendil-works/chord`. Both MIT; docs at pi.dev; code at `earendil-works/pi`.

## Takeaways for an in-house harness

Product-line clarity: **stable interactive Pi** vs **experimental durable harness**. Production bets on Durable must assume API churn and pin aggressively; do not confuse Pi 1.0 CLI stability with Durable stability. Use this post when explaining to leadership why “just add durability to Pi” was intentionally rejected.

## Reading this beside the Durable announcement

Read Pi 1.0 first for product philosophy and coding-agent scope, then the Durable announcement for the experimental long-running substrate. Together they answer “why two packages?”: keep the terminal agent stable and minimal, explore durability/multiplayer/edge hosting without forcing every Pi user onto an experimental storage/task runtime. For harness engineers, that split is a template—stabilize the interactive loop, experiment on the durable control plane, share the model layer (`pi-ai`) between them.


Demo frames in the post also show codemode summarizing a week of commits, a virtual router model switching between Opus and GPT via Jev, and `/session` cost breakdowns—useful context for why Durable shares `pi-ai` but not the TUI-centric coding-agent UX.

## Quote

> “Rather than stray from our minimal roots and try to make Pi something it wasn’t, we combined that work into a new experimental package that we are shipping today, Pi Durable.”
> — https://earendil.com/posts/pi-1-0/
