---
title: "Harness Evolution Hits a Ceiling: When Weight Training Should Begin (structured notes)"
author: "Yuan Tian, Bing Hu, Hao Wang, Binghang Lu, and Fang Wu"
source_url: "https://arxiv.org/abs/2610.11655"
arxiv_id: "2610.11655"
arxiv_version: "v1"
published_at: "2026-10-08"
archived_at: "2026-11-25"
license: "arXiv.org perpetual, non-exclusive license 1.0"
license_url: "https://arxiv.org/licenses/nonexclusive-distrib/1.0/"
kind: "structured-notes"
list_index: 4
archive_note: "Structured research notes written after reading the paper. This is not a reprint, not a close paraphrase of the full text, and not a full translation. The arXiv non-exclusive license grants rights to arXiv, not a third-party right to redistribute or translate the work."
---

## Source

Yuan Tian, Bing Hu, Hao Wang, Binghang Lu, and Fang Wu. *Harness Evolution Hits a Ceiling: When Weight Training Should Begin*. arXiv:2610.11655v1, first posted 2026-10-08 18:33 (Asia/Shanghai). PDF: https://arxiv.org/pdf/2610.11655. HTML: https://arxiv.org/html/2610.11655. 21 pages. Listed as item 4 in [elvis / @omarsar0, “My Top 10 RSI Papers of 2026”](https://x.com/omarsar0/status/2125234668396023943).

**Fetch status:** Full paper successfully read from the public arXiv HTML and PDF on 2026-11-25 (Asia/Shanghai).

**License / why notes only:** The abstract page states the [arXiv.org perpetual, non-exclusive license 1.0](https://arxiv.org/licenses/nonexclusive-distrib/1.0/). That license does not authorize third-party full-text redistribution or translation. These notes are an original short summary, significantly shorter than the paper, with no close quotation beyond a short attributed phrase.

## Background

The paper treats two levers for improving a long-horizon agent: evolving the *harness* around a frozen model, and training the model’s *weights*. The authors run a self-evolving harness first, then cross seed vs evolved harnesses with base vs trained weights, and argue that the right lever can be read from how failures are composed.

## Method (in brief)

On DeepPlanning, a travel-planning benchmark with programmatic scoring, they use 48 development tasks for every decision and 72 held-out tasks the loop never sees. Failed trajectories are labelled by the first signal that fires: unbacked claims (F1), harness friction (F2), loops or exhausted step budgets (F3), or a delivered but poor plan (F4). They also track delivery rate and quality among delivered plans. An LLM proposes harness edits; a judge verifies them. Scores are four-rollout means, with differences below about 0.035 treated as noise.

## Findings

Harness evolution lifts held-out scores of Qwen3.5-4B from 0.16 to 0.30 and Qwen3.5-9B from 0.32 to 0.44. For 4B, delivery rises from 55% to 90% and loop (F3) failures fall; content failures (F4) are not reduced. LoRA adapters trained on evolved-harness trajectories add about +0.13 under the original harness for both sizes. On 4B the adapter stacks with the evolved harness; on 9B the adapter alone matches the evolution line and cuts content failures from about a quarter of held-out trajectories to one in twenty. A placebo adapter trained on answer-shuffled trajectories falls below the base model, so the gain is in the trajectory content. On WebArena-Lite the same loop gains +0.09 on 117 unseen tasks, largely from keeping two full pages of history instead of one; adapters trained on those trajectories do not add to that gain.

## Takeaway

Diagnose first: use harness changes for process failures and for edits that change what the model *sees*; use weight training for content failures and for edits that change what the model *writes*. The loop stops at F4 partly because its evidence package does not surface the scorer’s content failures clearly.

## Quote

> “evolve the harness for process failures and train the weights for content failures”
> — https://arxiv.org/abs/2610.11655
