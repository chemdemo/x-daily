---
title: "How Compaction Works in Pi (structured notes)"
author: "Earendil Engineering"
source_url: "https://earendil.com/posts/compaction-in-pi/"
published_at: "2026-08-13"
archived_at: "2026-11-06"
category: "background"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Earendil Engineering (2026-08-13): how compaction works in the Pi **coding agent**. Essential background for Pi Durable’s compaction **tasks**, which reuse the same problem framing with durable scheduling.

## The problem

LLM context windows are finite. A coding-agent turn accumulates system prompt, tools, user/assistant messages, and tool results. Eventually providers reject requests as too large (`Request exceeds the maximum size`).

## Two responses to overflow

1. Start a new empty conversation—lose prior decisions and unresolved work (sometimes desirable because quality can degrade with long context).
2. **Compact**: replace older history with a smaller representation. In practice, implementations use an LLM summarization request rather than a purely deterministic keep/drop function.

## Pi’s implementation

- Triggers: auto after a turn when nearing the limit; mid-turn on overflow errors; manual `/compact`.
- Retains a recent window by **token budget** (default ~20k tokens ≈ roughly 5–20 turns); older turns are serialized and summarized.
- Compaction is a **standalone** request with a different system prompt (“context summarization assistant”), structured handoff-style sections (goal, progress, key decisions), and optionally a different model—avoiding unnecessary cost on the live conversation cache.
- Result stored as plain-text compaction entry in the session (readable/portable across model switches).
- **Prompt caching**: exact prefix match breaks at the first changed token after inserting a summary; cache rebuilds on subsequent turns.

## Bridge to Pi Durable

In Durable, compaction is a first-class durable task: background summarization before the hard reserve, blocking wait only when the next request would not fit, overflow compact-and-retry once, manual `compact(instructions)`, older entries remain in storage, hooks like `beforeCompact` can supply/decline summaries. `reset`/handoff start fresh model context while history stays searchable via tools.

## Takeaways for an in-house harness

Separate summarizer prompts from the main agent persona. Budget retained recent tokens explicitly. Expect cache invalidation costs after compact. Never treat “compacted model view” as “deleted audit history” when you have durable storage—put must-keep structured facts in documents, not only in prose summaries.

## Practical checklist when porting compaction ideas

When designing compaction for an in-house harness, write down: (1) who triggers it (auto threshold vs manual vs overflow), (2) what is retained verbatim vs summarized, (3) whether the summary request uses a different model/prompt, (4) how the summary is addressed in storage (entry kind, heads/pointers), (5) how prompt caches invalidate, and (6) which facts must never depend on the summary alone because they live in typed documents. Pi Durable’s settings (`reserveTokens`, `backgroundTokens`, `keepRecentTokens`, per-model overrides) are a concrete reference implementation of that checklist—not the only possible one.

## Quote

> “The ideal outcome of a good summarization for a coding agent is like a handoff briefing from one shift to the next.”
> — https://earendil.com/posts/compaction-in-pi/
