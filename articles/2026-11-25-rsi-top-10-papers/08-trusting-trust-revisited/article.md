---
title: "Reflections on Trusting Trust, Revisited (structured notes)"
author: "Franziska Roesner and Tadayoshi Kohno"
source_url: "https://arxiv.org/abs/2609.17817"
arxiv_id: "2609.17817"
arxiv_version: "v1"
published_at: "2026-09-16"
archived_at: "2026-11-25"
license: "arXiv.org perpetual, non-exclusive license 1.0"
license_url: "https://arxiv.org/licenses/nonexclusive-distrib/1.0/"
kind: "structured-notes"
list_index: 8
archive_note: "Structured research notes written after reading the paper. This is not a reprint, not a close paraphrase of the full text, and not a full translation. The arXiv non-exclusive license grants rights to arXiv, not a third-party right to redistribute or translate the work. These notes describe findings and defenses at a high level and do not reproduce attack procedures or proof-of-concept code."
---

## Source

Franziska Roesner (University of Washington) and Tadayoshi Kohno (Georgetown University). *Reflections on Trusting Trust, Revisited: Contaminating Self-Modifying AI Coding Agents with Poisoned Benchmarks*. arXiv:2609.17817v1, first posted 2026-09-16 04:32 (Asia/Shanghai). PDF: https://arxiv.org/pdf/2609.17817. HTML: https://arxiv.org/html/2609.17817. 22 pages. Listed as item 8 in [elvis / @omarsar0, “My Top 10 RSI Papers of 2026”](https://x.com/omarsar0/status/2125234668396023943).

**Fetch status:** Full paper successfully read from the public arXiv HTML and PDF on 2026-11-25 (Asia/Shanghai).

**License / why notes only:** The abstract page states the [arXiv.org perpetual, non-exclusive license 1.0](https://arxiv.org/licenses/nonexclusive-distrib/1.0/). That license does not authorize third-party full-text redistribution or translation.

## Background

The paper revisits Thompson’s classic “trusting trust” observation—now with the “compiler” being a self-modifying AI coding agent that evaluates and rewrites itself. The question is whether an adversary who cannot edit the seed agent or the underlying model, but can supply the benchmark used for self-evaluation, can cause later versions of the agent to emit insecure code on clean, held-out tasks.

## Method (in brief)

The authors study three recent self-modifying coding systems: the Darwin Gödel Machine (with experimental modifications to its self-improvement prompt), the Self-Improving Coding Agent (SICA), and Hyperagents (both described as substantively unmodified). They construct evaluation sets that pressure the agent toward insecure coding patterns during self-improvement, then measure whether evolved agents still produce those patterns on neutral held-out tasks. They also continue evolution on clean or security-oriented sets to test whether contamination persists.

## Findings

The paper reports successful proofs of concept on all three systems: after evolution against a malicious evaluation set, agents can write insecure patterns (for example, disabling TLS certificate validation, or weakening token-signature checks) on later clean tasks. Contamination often remains after further evolution on a clean companion set or on a generic security benchmark that does not match the poison’s cues. A purpose-built decontamination set that *does* match those cues was only partially effective on some systems and fully removed the harmful directive on Hyperagents in the reported run. The authors also distill properties of the vulnerability, the benchmark, the model, and the harness that made their demonstrations possible, and outline prevention, detection, and recovery directions.

## Takeaway

Treat the self-evaluation set like code under review, and review the tools and rules the agent writes for itself. Lack of security guidance in the self-improvement loop, and prompts that do not discourage benchmark-specific evolution, made some systems easier to steer. Continuing to evolve on “clean” tasks is not a reliable disinfectant if those tasks never penalize the insecure pattern.

## Quote

> “self-modifying coding agents must be designed to be more resilient to such attacks”
> — https://arxiv.org/abs/2609.17817
