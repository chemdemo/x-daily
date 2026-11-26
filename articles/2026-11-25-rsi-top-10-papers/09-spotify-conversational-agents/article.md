---
title: "Bootstrapping Conversational Recommendation Agents At Spotify (structured notes)"
author: "Enrico Palumbo, Alexandre Tamborrino, Victor Ode, Ben Lacker, Adrià Casas Escoda, Jeremy Hopple, Marcus Better, James Leoni, Hugo Galvão, Hugues Bouchard, Mounia Lalmas, José Luis Redondo García, Abenezer Abebe, Ann Clifton, Anton Blomberg, Henrik Lindström, Dani Doro, and Christine Doig Cardet"
source_url: "https://arxiv.org/abs/2609.30297"
arxiv_id: "2609.30297"
arxiv_version: "v1"
doi: "10.1145/3773078.3831910"
published_at: "2026-09-16"
archived_at: "2026-11-25"
license: "CC BY-NC-ND 4.0"
license_url: "https://creativecommons.org/licenses/by-nc-nd/4.0/"
kind: "structured-notes"
list_index: 9
archive_note: "Structured research notes written after reading the paper. This is not a reprint and not a full translation. CC BY-NC-ND 4.0 allows non-commercial sharing of the unmodified work but does not allow derivative works such as a translation, so this archive keeps original-length notes only."
---

## Source

Enrico Palumbo et al. (Spotify). *Bootstrapping Conversational Recommendation Agents At Spotify: Synthetic Data Generation and Self-Improvement Loops*. arXiv:2609.30297v1, first posted 2026-09-16 22:48 (Asia/Shanghai). Also forthcoming at RecSys ’26 (Minneapolis, 2026-09-27 to 2026-10-02), DOI [10.1145/3773078.3831910](https://doi.org/10.1145/3773078.3831910). PDF: https://arxiv.org/pdf/2609.30297. HTML: https://arxiv.org/html/2609.30297. 9 pages. Listed as item 9 in [elvis / @omarsar0, “My Top 10 RSI Papers of 2026”](https://x.com/omarsar0/status/2125234668396023943).

**Fetch status:** Full paper successfully read from the public arXiv HTML and PDF on 2026-11-25 (Asia/Shanghai).

**License / why notes only:** The abstract page states [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/). The ND term forbids derivative works, including a full translation, so this archive does not reprint or fully translate the paper.

## Background

The paper describes how Spotify built a conversational music recommendation agent in a cold-start setting, before real user conversations existed. The hard part is agent planning: which tools to call, in what order, and how to keep constraints across turns.

## Method (in brief)

A synthetic-data pipeline turns single-turn prompts into multi-turn user–agent dialogues (a UserLLM adapts later turns to the evolving conversation). Human reviewers rated a 100-conversation sample on four binary dimensions. A self-improvement loop then runs test conversations repeatedly, uses variance to find unstable planning or tool use, and has a coding agent propose prompt and tool-description fixes in mini-batches of 8 over a set of about 1,000 examples. Every proposed change is reviewed as a pull request before merge. The judge model is chosen to be different from the agent model.

## Findings

Human scores were above 90% on all four dimensions (task demonstration 98%, realism 93%, adaptation quality 94%, judge accuracy 97%). Instruction retention and content refinement were the hardest capabilities; pass@1 stayed stable through four turns and dropped from the fifth turn on. The self-improvement loop added an +8% relative pass@1 gain on the full test set on top of an already heavily tuned manual prompt (pairwise sign test, p=0.01). In a two-week online A/B test across several markets, with about 15 million users, the new multi-turn agent increased listening by 14%, weekly active users by 5%, and reduced skip rate by 5% versus a prior experience limited to session refinement, after which the agent was rolled out.

## Takeaway

In production, start the self-improvement loop narrow, keep a person reviewing each change, and measure the result with real users. Variance-based contrastive analysis helped the authors avoid noisy updates from infrastructure or judge failures. The authors note remaining work on long-context compaction and on expanding improvement from prompts and tool descriptions to the broader harness.

## Quote

> “outputs are always reviewed by humans before deployment”
> — https://arxiv.org/abs/2609.30297
