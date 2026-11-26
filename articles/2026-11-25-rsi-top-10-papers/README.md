# Top 10 RSI papers of 2026 (2026-11-25)

Independent topic collection for the **x-daily** archive, taken from elvis (@omarsar0)’s X Article [“My Top 10 RSI Papers of 2026”](https://x.com/omarsar0/status/2125234668396023943) (X Article: https://x.com/i/article/2125233081082613761), posted 2026-11-24 22:26 (Asia/Shanghai).

The ten items below are the author’s numbered “favorite RSI papers from this year,” in the list order given in that post. A survey recommended *before* the ten picks is noted at the end and is **not** one of the ten.

Archived at: **2026-11-25** (Asia/Shanghai calendar day of this collection).

## License and what is stored

Each item is `article.md` + `article_zh.md` + `meta.json`.

- **Full-text reprint + Chinese translation** only when the arXiv abstract page states a license that clearly allows redistribution *and* translation (CC BY 4.0, or CC BY-NC-SA 4.0 with the NC and SA conditions). English `article.md` is a cleaned full-text reprint from the public HTML or PDF, from the abstract through the appendices. Chinese `article_zh.md` is an unofficial, unabridged translation of that text: every body paragraph from the abstract through the conclusion, plus tables, formulas, footnotes, and appendix prose. Code, prompts the paper quotes verbatim, and bibliography entries stay in English. These are not official or author-approved translations. A compressed summary is not labeled as a full translation.
- **Structured research notes only** when the work is under the arXiv.org non-exclusive license (no third-party right to redistribute or translate) or under CC BY-NC-ND (no derivatives, including a full translation). Those notes are original short summaries, far shorter than the source, and are clearly labelled as *not* a reprint or a full translation. Short attributed quotations stay under 25 English words.

Open-source *code* licenses are not treated as covering the paper text.

## Suggested reading order

This follows the grouping in the source post.

1. [Meta-Harness](01-meta-harness/article.md) — what to change: the harness, in a search *before* deployment.
2. [Exo](02-exo/article.md) — what to change: the harness *during* use, behind a protected layer.
3. [Hyperagents](03-hyperagents/article.md) — what to change: the improvement procedure itself.
4. [Harness Evolution Hits a Ceiling](04-harness-evolution-ceiling/article.md) — when to switch from harness edits to weight training.
5. [Harness-Delta Attribution](05-harness-delta-attribution/article.md) — split a reported harness gain into overfitting, extra test-time compute, and a residual.
6. [The Winner’s Curse](06-winners-curse/article.md) — keep-if-better scores on a small reused set overstate true gains.
7. [Who Verifies the Verifier?](07-who-verifies-the-verifier/article.md) — a rising agent score cannot certify the grader.
8. [Trusting Trust, Revisited](08-trusting-trust-revisited/article.md) — a poisoned self-evaluation set can steer what a self-modifying agent keeps.
9. [Spotify conversational agents](09-spotify-conversational-agents/article.md) — RSI in a production loop with human review of each change.
10. [No Second Run](10-no-second-run/article.md) — RSI at frontier pre-training by splitting the unrepeatable run into smaller loops.

If you adopt only one practice from the list: set aside a held-out test set for evaluation only, and score both the starting version and the current version on it before trusting a gain.

## Index

| # | Title | Files | Authors | License | Stored as |
|---|---|---|---|---|---|
| 1 | Meta-Harness: End-to-End Optimization of Model Harnesses | [article.md](01-meta-harness/article.md) · [article_zh.md](01-meta-harness/article_zh.md) | Yoonho Lee et al. | CC BY 4.0 | Full text + full translation |
| 2 | Exo: A Harness Architecture for Runtime Self-Improvement | [article.md](02-exo/article.md) · [article_zh.md](02-exo/article_zh.md) | Alexander Krentsel et al. | CC BY 4.0 | Full text + full translation |
| 3 | Hyperagents | [article.md](03-hyperagents/article.md) · [article_zh.md](03-hyperagents/article_zh.md) | Jenny Zhang et al. | CC BY 4.0 | Full text + full translation (v2) |
| 4 | Harness Evolution Hits a Ceiling: When Weight Training Should Begin | [article.md](04-harness-evolution-ceiling/article.md) · [article_zh.md](04-harness-evolution-ceiling/article_zh.md) | Yuan Tian et al. | arXiv non-exclusive 1.0 | Notes only |
| 5 | Harness-Delta Attribution: Dissecting Gains From Harness Evolution | [article.md](05-harness-delta-attribution/article.md) · [article_zh.md](05-harness-delta-attribution/article_zh.md) | Wenxuan Ding et al. | CC BY 4.0 | Full text + full translation |
| 6 | The Winner’s Curse in LLM Self-Improvement Loops | [article.md](06-winners-curse/article.md) · [article_zh.md](06-winners-curse/article_zh.md) | Litao Hu and Yutong Tang | CC BY 4.0 | Full text + full translation |
| 7 | Who Verifies the Verifier? | [article.md](07-who-verifies-the-verifier/article.md) · [article_zh.md](07-who-verifies-the-verifier/article_zh.md) | Xing Zhang et al. | CC BY-NC-SA 4.0 | Full text + full translation (NC, SA) |
| 8 | Reflections on Trusting Trust, Revisited | [article.md](08-trusting-trust-revisited/article.md) · [article_zh.md](08-trusting-trust-revisited/article_zh.md) | Franziska Roesner and Tadayoshi Kohno | arXiv non-exclusive 1.0 | Notes only |
| 9 | Bootstrapping Conversational Recommendation Agents At Spotify | [article.md](09-spotify-conversational-agents/article.md) · [article_zh.md](09-spotify-conversational-agents/article_zh.md) | Enrico Palumbo et al. | CC BY-NC-ND 4.0 | Notes only (ND: no translation) |
| 10 | No Second Run: Does RSI Stop at Frontier Pre-training? | [article.md](10-no-second-run/article.md) · [article_zh.md](10-no-second-run/article_zh.md) | Xiaojie Cai et al. (alphabetical) | CC BY 4.0 | Full text + full translation |

Item 2’s arXiv abstract-page title is “Exo: An Architecture for Self-Improving Systems at Runtime” (also the wording in the source X list). The paper body, which this archive reprints, is titled “Exo: A Harness Architecture for Runtime Self-Improvement”.

## English and Chinese scope

| # | English `article.md` | Chinese `article_zh.md` | Left in English on purpose |
|---|---|---|---|
| 1 | Full reprint, abstract through Appendix E | Unabridged translation of that text, including Appendices A–E | Bibliography |
| 2 | Full reprint. Official HTML ends during §5.4; from the rest of §5.4 through Appendix B the text is from the PDF, including §6, the reference list, the AI usage disclosure, and Appendices A–B. Table 1’s body was empty in the HTML and is filled from the PDF. Shading in Table A1 is not reproduced | Unabridged translation of that text, including both appendices and Table 1 | Bibliography |
| 3 | Full v2 reprint, abstract through Appendix F (ar5iv HTML; official `/html/2603.19461` returned 404) | Unabridged translation of the prose through Appendix F | Code, prompts, pseudocode, diffs, and the bibliography. One bibliography name is copied as the ar5iv HTML spells it (`Armengol-EstapÃŠ`) |
| 4 | Short original notes. Not a reprint | Chinese notes of those notes. Not a translation of the paper | — |
| 5 | Full reprint, abstract through Appendix G | Unabridged translation, including Appendices A–G and the proofs’ surrounding prose | Code listings and the bibliography |
| 6 | Full reprint, abstract through Appendix D | Unabridged translation, including proofs and result tables | Prompts the paper marks as verbatim, and the bibliography |
| 7 | Full reprint, abstract through Appendix G. License remains CC BY-NC-SA 4.0 | Unabridged translation under the same license (non-commercial, share-alike), stated in the file | Verbatim verifier expressions and the bibliography |
| 8 | Short high-level notes. Not a reprint. No attack procedure | Chinese notes of those notes. Not a translation of the paper | — |
| 9 | Short original notes. Not a reprint. ND forbids a translation | Chinese notes of those notes. Not a translation of the paper | — |
| 10 | Full reprint, abstract through Appendix D | Unabridged translation, including Appendices A–D and the result tables | Shell commands and the bibliography |

Figures are kept as captions, not redrawn. Doubled HTML math such as `4×\times` is a single LaTeX form in the Chinese files.

## Takeaways

- Give a harness-search agent the full execution traces, not only a score (Meta-Harness).
- If an agent may edit itself in production, first freeze a protected layer the agent cannot touch (Exo).
- Treat the procedure that proposes changes as something you can improve too, and log those changes separately (Hyperagents).
- Sort failures before choosing a fix: harness changes for process failures; weight training for content failures (Ceiling).
- Before trusting a harness gain, run the old harness at the new one’s compute budget, then test both on held-out tasks (HDA).
- Score the starting and current versions on a held-out set never used for selection (Winner’s Curse; 64 items removed average overstatement in that paper).
- Keep a small set of human-checked examples and measure the grader against them, separately from the agent score (Who Verifies).
- Review what goes into the self-evaluation set as you would review code; also review tools and rules the agent writes for itself (Trusting Trust). Notes for item 8 stay high-level and do not reproduce attack procedures.
- In a product loop, start narrow, keep a person reviewing each change, and measure with real users (Spotify).
- If the main loop is too slow or expensive to repeat, split it into smaller loops with a quickly measurable metric (No Second Run).

## Layout

```
articles/2026-11-25-rsi-top-10-papers/
  README.md / README_zh.md / meta.json
  01-meta-harness/ … 10-no-second-run/
    article.md  article_zh.md  meta.json
```

The repo index [`../README.md`](../README.md) links to this collection.

## Fetch status

All ten papers were successfully read from public arXiv sources on 2026-11-25 (Asia/Shanghai). No item is a “fetch failed” stub.

| # | arXiv | Read from | Notes |
|---|---|---|---|
| 1 | [2603.28052](https://arxiv.org/abs/2603.28052) | official HTML + PDF | |
| 2 | [2611.00463](https://arxiv.org/abs/2611.00463) | official HTML + PDF | Official HTML ends during §5.4. From the rest of §5.4 through Appendix B, text is transcribed from the PDF, including §6, references, the AI usage disclosure, and Appendices A–B |
| 3 | [2603.19461](https://arxiv.org/abs/2603.19461) v2 | ar5iv HTML + PDF | Official `/html/2603.19461` returned 404 |
| 4 | [2610.11655](https://arxiv.org/abs/2610.11655) | official HTML + PDF | Notes only (license) |
| 5 | [2610.22466](https://arxiv.org/abs/2610.22466) | official HTML + PDF | |
| 6 | [2610.09239](https://arxiv.org/abs/2610.09239) | official HTML + PDF | |
| 7 | [2610.11464](https://arxiv.org/abs/2610.11464) | official HTML + PDF | |
| 8 | [2609.17817](https://arxiv.org/abs/2609.17817) | official HTML + PDF | Notes only (license); no attack procedures reproduced |
| 9 | [2609.30297](https://arxiv.org/abs/2609.30297) | official HTML + PDF | Notes only (CC BY-NC-ND) |
| 10 | [2611.10719](https://arxiv.org/abs/2611.10719) | official HTML + PDF | |

No videos were downloaded. Licenses were taken from each paper’s arXiv abstract page, not inferred from a code repository.

## Recommended pre-reading (not one of the ten)

The source post asks readers to start with the survey *The Last AI Built by Humans: Toward Genuine Recursive Self-Improvement*, Yi Duan et al., 2026, [arXiv:2609.11873](https://arxiv.org/abs/2609.11873). It is **not** one of the ten numbered picks. Its abstract page states CC BY-NC-ND 4.0, so this collection does not reprint or translate it. The post also points to the DAIR collection https://academy.dair.ai/papers/collections/self-improving-agents and, inside the discussion of item 8, to several 2026 benchmarks (PostTrainBench, SEAGym, Flex-RSI, Singularity-Bench, FrontierRSI) that are context, not members of the ten.
