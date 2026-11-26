---
title: "Harness-Delta Attribution: Dissecting Gains From Harness Evolution (full-text reprint)"
author: "Wenxuan Ding, Qin Lu, Changlong Yu, Sha Li, Shuowei Jin, Xin Liu, Zhenyu Shi, and Greg Durrett"
source_url: "https://arxiv.org/abs/2610.22466"
arxiv_id: "2610.22466"
arxiv_version: "v1"
published_at: "2026-10-16"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-reprint"
list_index: 5
archive_note: "Full-text reprint of a work whose license allows redistribution and translation, plus a Chinese translation in article_zh.md. Not an official translation."
---
# Harness-Delta Attribution: Dissecting Gains From Harness Evolution

**Authors:** Wenxuan Ding, Qin Lu, Changlong Yu, Sha Li, Shuowei Jin, Xin Liu, Zhenyu Shi, and Greg Durrett

**Source:** [2610.22466](https://arxiv.org/abs/2610.22466) (v1). First posted 2026-10-16 01:39 (Asia/Shanghai); arXiv published 2026-10-15T17:39:38Z. PDF: https://arxiv.org/pdf/2610.22466. HTML: https://arxiv.org/html/2610.22466.

**License:** This paper is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). This archive reprints the full text and provides a Chinese translation with attribution to the original authors. This is not an official or author-approved translation.

**Fetched:** Successfully read from public arXiv sources on 2026-11-25 (Asia/Shanghai). Listed as item 5 in [elvis / @omarsar0, “My Top 10 RSI Papers of 2026”](https://x.com/omarsar0/status/2125234668396023943).

---
# Harness-Delta Attribution: Dissecting Gains From Harness Evolution

Wenxuan Ding Qin Lu Changlong Yu Sha Li  Affiliation: Amazon New York University  Email: [wd2403@nyu.edu](<mailto:>) Shuowei Jin Xin Liu Zhenyu Shi Greg Durrett  Affiliation: Amazon New York University

## Abstract
Harness evolution automatically optimizes the prompts, skills, tools, and orchestration code around a fixed-weight model, offering a way to improve agentic systems without updating model weights. The agent harness is iteratively optimized to achieve higher performance on the search set. But how much of this gain reflects benchmark-specific overfitting or additional inference-time compute, and how much may generalize beyond the search set? In this work, we introduce Harness-Delta Attribution (HDA), an evaluation protocol that dissects an evolution gain into artifact-driven overfitting, gains explained by test-time scaling, and the remainder, which gives an upper bound on residual generalizable improvement. We apply HDA to 11 executor models and three harness-evolution frameworks across math, coding, creativity, and agentic tasks. In 21 of these settings that show statistically significant search-set gains, we find that a large share of the improvement is explained by overfitting or test-time scaling. Specifically, among these 21 settings, overfitting and test-time scaling account for 54% and 14% of the gain on average; their combined share exceeds 50% in 14 settings. The residual and held-out transfer vary substantially across tasks, models, and frameworks. Importantly, search-set gains often fail to transfer to held-out tasks, and provide limited evidence that they will persist under practical distribution shifts. Moreover, validation-based signals are insufficient to prevent overfitting. These results motivate attribution-aware evaluation in future harness evolution research. 11 1 Code: <https://github.com/Wenwen-D/HarnessDeltaAttribution> Blog: <https://wenwen-d.github.io/blog/harness-delta-attribution/>

---

Figure 1: An evolved harness may improve the performance by overfitting to benchmark artifacts, scaling test-time compute, or making generalizable improvements (left). HDA asks where the observed gain comes from and attributes it to these three sources (right). Overfitting and test-time scaling explain a substantial share across settings.

## 1 Introduction

An agent system can be viewed as an LLM paired with a harness: the prompts, tools, memory, skills, orchestration code, and control flow that shape how a model interacts with a task. Even without updating the model weights, changes to the harness can alter system performance by a wide margin, as observed on benchmarks such as Terminal-Bench (Merrill et al., 2026) and SWE-bench (Jimenez et al., 2024). This has motivated recent work on harness evolution, a form of text-space optimization that iteratively updates the non-parametric components around a fixed-weight model using task feedback (Lee et al., 2026; Lin et al., 2026a; Lin et al., 2026b). Specifically, in each iteration, an executor model attempts the tasks with the current harness. A proposer agent then inspects the trajectories and evaluation results to propose edits to the harness. Repeating this process produces a sequence of harnesses optimized for performance on the search set (training split).

Recent harness-evolution frameworks have reported notable gains from evolved harness code (Lee et al., 2026; Ning et al., 2026), skill files (Ni et al., 2026; Yang et al., 2026; Tang et al., 2026), and updated tool-use or orchestration patterns (Chen et al., 2026). However, a higher aggregate score does not reveal what changed in the system or whether the gains will persist beyond the search set. Prior work does not always evaluate evolved harnesses on held-out instances. Even when held-out results are reported, the search and test splits may share the same task construction and dataset artifacts. Such evaluations measure transfer within a benchmark, but provide limited evidence that the gain will persist in new tasks or practical distribution shifts.

In this work, we introduce Harness-Delta Attribution (HDA), an evaluation protocol that attributes harness evolution gains to three different sources. Some gains may reflect real harness improvements that generalize beyond the search set. However, we identify two additional sources of gain that lead us to view a harness optimization’s result in a different light. First, an evolved harness may scale test-time compute: for instance, it may sample five responses in parallel and select a majority vote. Second, an evolved harness may exploit dataset shortcuts by hardcoding a recurring answer pattern from the search set. These mechanisms can produce similar benchmark gains while having very different implications for cost and generalization.

Our method, HDA, compares the baseline and evolved harnesses using a compute-matched baseline and an artifact-neutralized version of the evolved harness. This allows us to estimate a lower bound on detected overfitting, isolate gains reproducible through additional inference compute, and measure the remaining residual. We also evaluate held-out performance to understand whether the evolved harness transfers beyond the search set.

Our experiments cover three different frameworks, including Meta-Harness (Lee et al., 2026), SkillOpt (Yang et al., 2026), and AHE (Lin et al., 2026a). We use them to evolve the harnesses of 11 executor models on benchmarks covering mathematical reasoning (LiveMath (Mahdavi et al., 2025)), software engineering (SWE-bench Verified (Jimenez et al., 2024)), creative generation (CREATE (Wadhwa et al., 2026)), and embodied planning (ALFWorld (Shridhar et al., 2021)). In total, we apply HDA to 25 settings.

Figure 1 previews our main finding: across various configurations, overfitting and test-time scaling explain a large share of the search-set gains, while the residual upper bound on generalizable improvement is often small. Among the 21 settings with statistically significant gains, HDA attributes at least 54%54\% of the gain to overfitting and 14%14\% to test-time scaling on average; their combined share exceeds 50%50\% in 14 settings. The breakdowns of these improvements can also reveal additional patterns. First, different frameworks repeatedly discover the same benchmark-specific shortcuts, such as a recurring correct answer choice in LiveMath. Second, in ALFWorld, some evolved harness eliminate LLM calls entirely by hardcoding deterministic solutions on the synthetic benchmark.

Our contributions are: (1) Identifying three sources of harness evolution gains (overfitting, test-time scaling, and generalizable improvement); (2) The HDA method, which dissects the harness evolution gains into 3 categories, and reveals findings that held-out evaluation alone does not show; (3) a reusable HDA skill and evaluation suite for applying this analysis to new benchmarks and harness-evolution frameworks.

## 2 Preliminaries

Let MM denote the fixed executor model whose harness is being optimized, and HH its harness; together, (M,H)(M,H) form the agent. HH comprises the non-weight components that govern how the executor handles a task, such as its prompts, skills, tools, memory, and orchestration logic (Weng, 2026). Different harness-evolution methods optimize different subsets of these components while keeping MM fixed. We use ℋ\mathcal{H} to denote the framework-specific space of harnesses that may be explored.

Given a task set 𝒳\mathcal{X} and task instance x∈𝒳x\in\mathcal{X}, the executor model produces an execution trajectory

| τH,x∼M(⋅∣x;H).\tau_{H,x}\sim M(\cdot\mid x;H). |
---|---|---

This trajectory includes the model outputs, tool calls, environment interactions, and other intermediate states produced during execution. Let r⁡(x,τH,x)r(x,\tau_{H,x}) denote the task-specific reward assigned to the trajectory. We define the score of harness HH on 𝒳\mathcal{X} as

| S𝒳​(H)=1|𝒳|​∑x∈𝒳r⁡(x,τH,x).S_{\mathcal{X}}(H)=\frac{1}{|\mathcal{X}|}\sum_{x\in\mathcal{X}}r(x,\tau_{H,x}). |
---|---|---

#### Evolution loop.

Starting from a baseline harness H0H_{0}, a proposer agent 𝒫\mathcal{P} iteratively edits the harness to improve its performance, producing an evolved harness H^\hat{H}. The proposer agent may be the executor agent itself, as in self-evolution, or a separate and potentially stronger agent. Let 𝒳search\mathcal{X}_{\mathrm{search}} denote the search set used during evolution. The framework maintains a filesystem-backed history FtF_{t} of harness versions, trajectories, evaluation results, and other artifacts through iteration tt, with F0F_{0} containing the baseline evaluation. At iteration tt, the proposer uses Ft−1F_{t-1} to generate a set ℋt\mathcal{H}_{t} of kk candidate harnesses:

| ℋt:={Ht(j)}j=1k=Propose⁡(𝒫;Ft−1).\mathcal{H}_{t}:=\\{H_{t}^{(j)}\\}_{j=1}^{k}=\operatorname{Propose}(\mathcal{P};F_{t-1}). |
---|---|---

Each candidate is evaluated on 𝒳search\mathcal{X}_{\mathrm{search}}, and ft​(H)f_{t}(H) denotes its feedback, which may include its score, per-task rewards, trajectories, and execution logs. The accumulated history is then updated as

| Ft=Ft−1∪{ft​(H):H∈ℋt}.F_{t}=F_{t-1}\cup\\{f_{t}(H):H\in\mathcal{H}_{t}\\}. |
---|---|---

Some frameworks expose the persisted artifacts directly to the proposer, while others insert selected records into a structured prompt or optimizer state. Across NN iterations, the evaluated harness pool is

| ℋeval={H0}∪⋃t=1Nℋt.\mathcal{H}_{\mathrm{eval}}=\\{H_{0}\\}\cup\bigcup_{t=1}^{N}\mathcal{H}_{t}. |
---|---|---

The selected evolved harness is

| H^=arg​maxH∈ℋeval⁡S𝒳search​(H).\hat{H}=\operatorname*{arg\,max}_{H\in\mathcal{H}_{\mathrm{eval}}}S_{\mathcal{X}_{\mathrm{search}}}(H). |
---|---|---

An optional validation set 𝒳val\mathcal{X}_{\mathrm{val}} may be used to gate updates or select harnesses. Algorithm 1 summarizes this standard evolution loop.

## 3 Harness-Delta Attribution: Dissecting Gains From Harness Evolution

Figure 2: Illustration of Harness-Delta Attribution (HDA). Comparing S⁡(H0)\smash{S(H_{0})} and S⁡(H^)\smash{S(\hat{H})} gives the observed gain Δ​S\Delta S. Running H0\smash{H_{0}} with H^\smash{\hat{H}}’s compute budget gives the compute-matched baseline H0scale\smash{H_{0}^{\mathrm{scale}}}; its gain over H0\smash{H_{0}} measures Test-Time Scaling. Disabling detected shortcuts in H^\smash{\hat{H}} gives H^neutral\smash{\hat{H}^{\mathrm{neutral}}}; the resulting drop measures Overfitting. The remaining gap estimates Generalizable Improvement.

Harness evolution returns an evolved harness H^\hat{H} with a higher search set score than the baseline H0H_{0}, but the difference Δ​S=S⁡(H^)−S⁡(H0)\Delta S=S(\hat{H})-S(H_{0}) does not explain why performance improved. For example, in one Meta-Harness ALFWorld run , the harness for the 0.8B executor grew from 66 to 503 non-blank, non-comment lines (7.6×7.6\times) and reached 100%100\% search-set success, but became a deterministic controller by hard-coding benchmark priors and made no LLM calls (Figure 3). In other settings, evolution increases inference compute, while scaling the baseline directly may produce similar gains without incurring the search cost of evolution. Figure 1 (left) illustrates these cases.

We introduce Harness-Delta Attribution (HDA) to distinguish these sources of gain. The key idea is to compare the evolved harness with two controlled variants: a scaled baseline that identifies gains recoverable through test-time scaling alone without harness evolution, and a neutralized evolved harness with detected artifact-driven mechanisms removed to detect overfitting.

We implement HDA as a reusable skill that can be applied across frameworks and tasks. An analysis agent follows it to perform the attribution, and a human reviewer checks the agent’s proposal at two task-dependent checkpoints: the compute-matching setup and the overfitting neutralization. This design keeps most of the workflow automated and provides a task-aware quality-control layer, analogous to human evaluation of automated methods. We provide the HDA skill in Appendix C.

Before attribution, we apply a paired bootstrap significance test to the scores of H0H_{0} and H^\hat{H} on the search set. If Δ​S\Delta S is insignificant, we report it as within noise and do not assign an attribution. Otherwise, the HDA agent constructs a compute-matched baseline H0scaleH_{0}^{\mathrm{scale}} and a neutralized evolved harness H^neutral\hat{H}^{\mathrm{neutral}}, as described below. HDA then defines three attribution components:

| T\displaystyle T | =S⁡(H0scale)−S⁡(H0),\displaystyle=S(H_{0}^{\mathrm{scale}})-S(H_{0}), |
---|---|---|---
| O\displaystyle O | =S⁡(H^)−S⁡(H^neutral),\displaystyle=S(\hat{H})-S(\hat{H}^{\mathrm{neutral}}), |
| G\displaystyle G | =S⁡(H^neutral)−S⁡(H0scale).\displaystyle=S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}}). |

#### TTest-Time Scaling: Compute matching.

The compute-matched baseline H0scaleH_{0}^{\mathrm{scale}} preserves H0H_{0} while scaling it to match the average inference budget of H^\hat{H}. Specifically, the HDA agent compares their inference costs using measures such as LLM calls or token usage. It then proposes a task-appropriate compute measure and procedure for constructing H0scaleH_{0}^{\mathrm{scale}} for human approval.

For example, if H^\hat{H} uses ten times as many executor calls per task as H0H_{0}, H0scaleH_{0}^{\mathrm{scale}} may run H0H_{0} ten times on each task and combine its outputs using majority vote, pass@10, or another task-specific aggregation rule. If H^\hat{H} does not use more inference compute than H0H_{0}, we set H0scale=H0H_{0}^{\mathrm{scale}}=H_{0}.

The difference T=S⁡(H0scale)−S⁡(H0)T=S(H_{0}^{\mathrm{scale}})-S(H_{0}) measures how much of the observed improvement can be reproduced solely through test-time scaling rather than improved harness design. Directly scaling H0H_{0} also avoids the search cost of harness evolution. Although such gains may transfer to held-out tasks, their practical value depends on whether the additional latency and inference cost are acceptable for the target application.

#### OOverfitting: Detection and neutralization.

The HDA agent constructs the neutralized harness H^neutral\hat{H}^{\mathrm{neutral}} by removing detected overfitting mechanisms from H^\hat{H}. We classify an evolved mechanism as overfitting when it improves performance by exploiting a pattern specific to the benchmark or search set, rather than improving the agent’s ability to achieve the intended task objective more generally. Its benefit may disappear on new instances that preserve the task objective but not that pattern.

Guided by the HDA skill, the agent audits the harness code, prompts, and available execution evidence for two broad forms of overfitting:

- Direct shortcuts: mechanisms that bypass or predetermine the executor’s behavior, including eliminating LLM calls, constant answers, item-specific gates, hard-coded lookup tables, and string-matching rules.

- Benchmark-derived heuristics: mechanisms that exploit patterns specific to how the benchmark instances are constructed, rather than properties required to solve the underlying task. Examples include thresholds fitted to benchmark statistics, reverse-engineered construction rules, dataset-specific priors, and hard-coded solution recipes.

For each detected mechanism, the HDA agent makes the smallest intervention that disables the shortcut to get H^neutral\hat{H}^{\mathrm{neutral}}, while preserving all unrelated evolved components.

Figure 3: Example of overfitting in harness evolution. As illustrated in this excerpt, the ALFWorld harness evolved by Meta-Harness for Qwen3.5-0.8B overfits to benchmark-specific patterns by hard-coding a deterministic controller that makes 0 LLM calls.

To support a reliable audit, the HDA skill provides shared definitions, examples and evidence requirements. Prior work also shows that frontier agents can identify benchmark defects and reward-hacking opportunities (Tu et al., 2026; Wang et al., 2026a). Human review also adds a safeguard tailored to the task and intended use.

Note that HDA neutralizes only detected mechanisms that can be disabled without affecting unrelated parts of the harness. We therefore interpret O=S⁡(H^)−S⁡(H^neutral)O=S(\hat{H})-S(\hat{H}^{\mathrm{neutral}}) as a lower-bound estimate of overfitting, since implicit or entangled mechanisms may remain in H^neutral\hat{H}^{\mathrm{neutral}}. Despite this, as we show later in results, the share of overfitting is still substantial.

#### GGeneralizable Improvements: Residual upper bound.

The residual GG is the gain that remains after accounting for detected overfitting and test-time scaling. By construction, G=Δ​S−O−T=S⁡(H^neutral)−S⁡(H0scale)G=\Delta S-O-T=S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}}). Because H^neutral\hat{H}^{\mathrm{neutral}} may still contain unneutralized overfitting, we interpret GG as an upper bound on generalizable improvement rather than evidence of transfer. We additionally assess generalizability directly by comparing H0H_{0} and H^\hat{H} on held-out tasks in Section 5.

For reporting, we normalize each component by Δ​S\Delta S, cap the OO share to [0%,100%][0\%,100\%], cap the TT share to the remaining percentage, and assign the remainder to GG. These caps affect only the reported percentages; the raw signed components still satisfy Δ​S=O+T+G\Delta S=O+T+G.

## 4 Method and Experiments

#### Benchmarks

We evaluate across four benchmarks spanning single-turn reasoning and long-horizon agentic tasks and various domains: mathematical reasoning (LiveMath (Mahdavi et al., 2025)), creative generation (CREATE (Wadhwa et al., 2026)), interactive embodied planning (ALFWorld (Shridhar et al., 2021)), and software coding (SWE-bench Verified (Jimenez et al., 2024)).

#### Evolution Frameworks

We evaluate three harness evolution frameworks with different optimization scopes and proposer–executor configurations, including self-evolution and using a stronger agent as proposer:

- Meta-Harness (Lee et al., 2026) searches harness code around fixed LLMs. The proposer can access the source code, scores, and execution traces of prior history via a filesystem.

- SkillOpt (Yang et al., 2026) optimizes the skill of a frozen agent, where a separate optimizer model turns scored rollouts into bounded edits on the skill document.

- AHE (Lin et al., 2026a) (Agentic Harness Engineering) evolves multiple harness components, including prompts, tools, skills, and memory, through an observability-driven loop. Following the original paper, we use its self-evolution setting, with Opus 4.8 as both proposer and executor.

For Meta-Harness, we additionally provide aggregate validation scores during evolution compared to the original implementation, while keeping per-instance validation results and trajectories hidden. We retain both the search-selected and validation-selected harnesses and evaluate them on the held-out test set. More details are in Appendix B.3.

#### Models

We evaluate 11 executor models from the Qwen and Claude model families. The Qwen executors span Qwen3, Qwen3.5, and Qwen3.6, ranging from 0.8B to 35B, and include both general-purpose and coding-specialized models (Yang et al., 2025; Team, 2026; Cao et al., 2026; Qwen Team, 2026). We also evaluate Claude Haiku 4.5 and Claude Opus 4.8 (Anthropic, 2025; Anthropic, 2026b) on some settings. Meta-Harness and SkillOpt use Claude Opus 4.8 as the proposer agent (Anthropic, 2026a), whereas AHE uses Opus 4.8 as both proposer and executor. The exact executor set varies by benchmark; see more details in Table 1 and Appendix B.

## 5 Results

#### Most gains come from overfitting or test-time scaling.

Table 1: HDA results for all 25 model–benchmark–framework settings. Δ​S=S⁡(H^)−S⁡(H0)\Delta S=S(\hat{H})-S(H_{0}) is the search-set gain. OO, TT, and GG are percentages of Δ​S\Delta S, capped to [0%,100%][0\%,100\%]. †{\color[rgb]{0.418,0.4453,0.5}\dagger} marks search-set gains that are not statistically significant, for which HDA reports no attribution. ⋆{\color[rgb]{0.707,0.1367,0.0938}\star} marks settings where the evolved harness performs worse than baseline on the held-out test set. CU denotes Creativity Utility, the metric used for CREATE benchmark.

Framework | Benchmark | Executor | Δ​S\Delta S | O↓O\downarrow (%) | T↓T\downarrow (%) | G↑G\uparrow (%)
---|---|---|---|---|---|---
Meta-Harness | ALFWorld | Qwen3.5-0.8B | +100.0%+100.0\% | 100 | 0 | 0
Qwen3.5-2B | +100.0%+100.0\% | 100 | 0 | 0
Qwen3.5-4B | +60.0%+60.0\% | 93 | 0 | 7
Qwen3-8B | +64.0%+64.0\% | 100 | 0 | 0
Qwen3.6-35B-A3B⋆ | +38.0%+38.0\% | 95 | 0 | 5
LiveMath | Qwen3.5-0.8B | +77.1%+77.1\% | 91 | 9 | 0
Qwen3.5-2B | +40.0%+40.0\% | 64 | 0 | 36
Qwen3.5-4B | +51.4%+51.4\% | 78 | 11 | 11
Qwen3-8B⋆ | +28.6%+28.6\% | 100 | 0 | 0
Qwen3.6-35B-A3B⋆ | +20.0%+20.0\% | 57 | 29 | 14
CREATE | Claude Haiku 4.5 | +37.9+37.9 CU | 0 | 49 | 51
Qwen3.6-35B-A3B | +7.5+7.5 CU | 0 | 100 | 0
SWE-bench | Qwen3-4B-Instruct-2507 | +27.1%+27.1\% | 23 | 23 | 54
Qwen3-30B-A3B-Instruct-2507 | +33.3%+33.3\% | 0 | 6 | 94
Qwen3.6-27B⋆ | +8.3%+8.3\% | 0 | 0 | 100
Qwen3-Coder-30B-A3B-Instruct⋆ | +12.5%+12.5\% | 0 | 0 | 100
SkillOpt | ALFWorld | Qwen3.5-4B | +46.2%+46.2\% | 44 | 0 | 56
LiveMath | Qwen3.5-4B | +45.7%+45.7\% | 88 | 0 | 12
CREATE | Qwen3.6-35B-A3B | −0.3-0.3 CU† | – | – | –
Claude Haiku 4.5 | +0.4+0.4 CU† | – | – | –
SWE-bench | Qwen3-4B-Instruct-2507 | +10.4%+10.4\% | 0 | 0 | 100
Qwen3.6-27B | 0†0^{{\color[rgb]{0.418,0.4453,0.5}\dagger}} | – | – | –
AHE | LiveMath | Claude Opus 4.8 | +48.6%+48.6\% | 100 | 0 | 0
CREATE | Claude Opus 4.8 | +6.9+6.9 CU | 0 | 75 | 25
SWE-bench | Claude Opus 4.8 | 0†0^{{\color[rgb]{0.418,0.4453,0.5}\dagger}} | – | – | –

In many settings, a large share of the observed gain reflects overfitting or more test-time compute rather than a more capable executor. As shown in Table 1, O+TO+T accounts for more than half of the attributed gain in 14 of the 21 settings with significant search-set gains. Across these settings, OO and TT explain 54%54\% and 14%14\% of the gain on average, respectively. We clip each normalized percentage to the range [0%,100%][0\%,100\%]. The dominant source varies by benchmark: ALFWorld and LiveMath are dominated by overfitting, CREATE by test-time scaling, and SWE-bench shows clearer residual generalizable improvement. Appendix G contrasts a benchmark-fitted patch-size heuristic with a reusable action-feedback guardrail on SWE-bench.

For example, on ALFWorld, Meta-Harness uses Opus 4.8 as the proposer to optimize harnesses for five executors ranging from Qwen3.5-0.8B to Qwen3.6-35B-A3B separately. All five reach 100%100\% search-set success after evolution, while HDA attributes 98%98\% of their gain to overfitting. As shown in Figure 3, smaller-model (0.8B, 2B) harnesses become rule-based controllers that make no LLM calls, while larger-model (4B, 35B-A3B) harnesses encode similar benchmark-specific knowledge through prompts, templates, and deterministic rules.

CREATE provides a representative example of test-time scaling. It evaluates associative creativity by asking models to produce a diverse set of high-quality responses; its Creative Utility (CU) metric rewards larger answer pools containing strong and diverse answers (Wadhwa et al., 2026). As shown in Table 1, under Meta-Harness, a compute-matched baseline recovers 49%49\% of the gain for Claude Haiku 4.5 and even outperforms the evolved Qwen3.6-35B-A3B harness. Similarly, in AHE, test-time scaling contributes +5.1+5.1 CU, accounting for 75%75\% of observed evolution gain. In contrast, SkillOpt only edits the skill file while keeping the control flow fixed, and we observe H0scale=H0H_{0}^{\mathrm{scale}}=H_{0} and T=0T=0. However, neither SkillOpt CREATE setting produces a statistically significant gain. Overall, in frameworks where the orchestration code can be flexibly updated, the gains are more substantial but also more prone to test-time scaling. We show example harnesses in Appendix F.

#### Different frameworks rediscover the same overfitting and scaling strategies.

LiveMath is a multiple-choice benchmark for research-level mathematical reasoning. In 21 of the 35 search-set questions, one answer choice is the same meta-level statement: “one of the remaining options is correct, but a stronger result can be proven.” This option is correct in all 21 cases.

Our HDA analysis shows that all three frameworks independently discover this shortcut. Meta-Harness implements a deterministic string-matching branch to catch this option, SkillOpt turns the option into a default decision rule, and AHE adds an explicit instruction to select it without solving the problem. The same shortcut appears across code, skill, and prompt evolution, rather than being specific to one framework. We show the snippets of evolved harnesses in Appendix E.

Similarly, on ALFWorld, Meta-Harness and SkillOpt encode similar task recipes, appliance mappings, and object-location priors, but represent them as executable controllers and natural-language skills, respectively. On CREATE, both Meta-Harness and AHE resort to scaling strategy to increase the potential coverage and diversity of answers. More details in Appendix D.

Such shortcut-driven gains make it difficult to assess the performance of harness evolution frameworks or make fair comparison across such frameworks. This suggests the need for rich and low-artifact benchmarks.

#### Search-set gains often fail to generalize, and even validation signals do not prevent overfitting.

We test generalizability directly on held-out test sets. As shown in Figure 4, held-out gains are generally smaller than training gains. Excluding ALFWorld, which we discuss below in § 5, the held-out gain is smaller than the search-set gain in 13 of 15 settings and less than half the search-set gain in 8. In 4 settings, the evolved harness even performs worse than the baseline on test set. These results confirm that large training gains often come from overfitting rather than generalizable improvement.

Having observed substantial overfitting, we next ask how easily it can be reduced using a standard safeguard: held-out validation signals. We therefore augment Meta-Harness with validation-based selection. Then we evaluate and compare train-best harness (H^\hat{H}) and val-best harness (H^val\hat{H}^{\mathrm{val}}) on the held-out test set.

As shown in Figure 5, validation selection increases mean held-out performance by 8.08.0 percentage points on ALFWorld, 0.80.8 points on LiveMath, and 8.58.5 points on SWE-bench Verified, while decreasing CREATE performance by 0.40.4 CU. Across the 16 model–benchmark settings, it improves the held-out score in 10 settings but reduces it in 6. More importantly, validation selection does not close the transfer gap: in 14 of 16 settings, the validation-selected harness’s held-out gain remains below the search-set gain of the search-selected harness. Thus, validation selection helps in some settings but does not reliably prevent overfitting. Validation selection therefore improves transfer in some settings but it remains non-trivial to further close the train-to-test gap.

Figure 4: Search-set gains do not reliably predict held-out transfer. For each setting, the open circle is the search-set gain of the evolved harness (Δ​Ss​e​a​r​c​h\Delta S_{search}) and the filled circle its held-out gain (Δ​St​e​s​t\Delta S_{test}); a red marker flags a negative held-out gain. The length of the line denotes the gap between gain on search set and on held-out set. Diamond means H^=H0\hat{H}=H_{0}; ×\times marks a non-significant gain. Units are pp, except CREATE (CU). Figure 5: Effect of validation selection across 16 Meta-Harness settings. Open circles show the search-set gain of the search-selected harness H^\hat{H}, gray circles show its held-out test gain, and blue diamonds show the held-out gain of the validation-selected harness H^val\hat{H}^{\mathrm{val}}. Gains are reported in percentage points, except for CREATE, which uses Creative Utility (CU).

#### Why held-out evaluation alone is not enough

Figure 4 also illustrates why held-out evaluation alone can miss overfitting. ALFWorld initially appears to be a strong transfer case: its test set gains are large and close to its search-set gains. HDA reveals, however, that 89%89\% of the harness evolution gain comes from overfitting on average: the evolved harnesses may hardcode the deterministic solutions or recipes that exploit how the benchmark instances are constructed, and even eliminate the LLM calls in the most extreme cases (Figure 3). Although we deliberately withhold several task types and scenarios from the search set, the evolved harnesses still achieve near-perfect performance on test set.

This turns out to highlight another cause of overfitting: distillation of benchmark-specific knowledge from a stronger proposer. On this relatively well-known benchmark, the strong proposer may already encode its information in the parametric knowledge, and the generated deterministic harnesses cover instances beyond search set. These results thus provide limited evidence of generalizable improvement. On tasks that are completely unfamiliar by the proposer-executor system, such recipes may fail under even small changes because they rely on string matching and fixed benchmark patterns.

LiveMath shows a similar limitation. The same meta-option artifact occurs across the search, validation, and test sets. For the Meta-Harness 0.8B executor, test accuracy is 100%100\% on questions containing the meta-option (53/5353/53), but only 31%31\% on the other questions (22/7122/71). Thus, a positive held-out gain can reflect an artifact shared across splits rather than broader mathematical reasoning.

CREATE shows the corresponding limitation for test-time scaling: extra model calls can improve both search- and test-set scores, but held-out evaluation alone does not distinguish scaling from an improvement at a fixed inference budget. HDA complements held-out evaluation by identifying overfitting, test-time scaling, and the residual generalizable improvement.

## 6 Related Work

#### Agentic Harness and Harness Evolution.

An agentic harness comprises the prompts, skills, tools, memory, context, and orchestration logic around a fixed model (Weng, 2026). Early agent systems relied largely on manual harness engineering (Yao et al., 2023; Yang et al., 2024), while initial approaches to automated optimization focused on individual components, particularly prompts (Khattab et al., 2024; Agrawal et al., 2026). More recent methods expand the scope to skills (Ni et al., 2026; Tang et al., 2026; Ke et al., 2026), executable harness code (Hu et al., 2025; Novikov et al., 2025; Karten et al., 2026; Ning et al., 2026; Zhang et al., 2026d), tools and workflows (Zhang et al., 2025; Chen et al., 2025), memory and context (Liu et al., 2026; Zhang et al., 2026f; Wu et al., 2026; Ouyang et al., 2026), environments (Huang et al., 2026a), and more (Ye et al., 2026; Zhang et al., 2026g; Zhang et al., 2026e). Self-evolving agent frameworks use the agent itself to improve its harness (Zhang et al., 2026b; Zhang et al., 2026a; Zhang et al., 2026c; Luo et al., 2026). Other frameworks use a separate proposer agent or optimizer to update the harness from task feedback (Lee et al., 2026; Yang et al., 2026). These methods report substantial gains without updating the executor’s model weights. Our work instead studies where these gains come from and whether they reflect meaningful, generalizable improvements.

#### Evaluating Harness Optimization.

Some recent harness evolution frameworks use benchmark tasks to search for improved harnesses and then report final performance on the in-distribution benchmark set (Wang et al., 2026b; Lin et al., 2026a; Zhang et al., 2026d). This protocol makes search-set optimization difficult to distinguish from generalizable improvement. Wang et al. (2026b) observe that harness evolution does not reliably beat parallel sampling or sequential refinement under unified budgets. Huang et al. (2026b) address the same evaluation concern by constructing separate auxiliary evolution and evaluation suites designed to reduce task-specific overfitting. Other work studies different aspects of harness benefit. Lin et al. (2026b) distinguish an agent’s ability to update a harness from the ability to follow and benefit from it, using an in-situ evaluation protocol. Concurrent work on RRSI regularizes harness proposal and selection to reduce overfitting and improve transfer (Xia et al., 2026). Our work complements these approaches by attributing an observed gain to detected overfitting, test-time scaling, and the remainder, which gives an upper bound on generalizable improvement.

## 7 Conclusion

In this work, we study where the gains of harness evolution come from. We introduce Harness-Delta Attribution (HDA), a portable, semi-automated evaluation protocol that attributes an observed gain to overfitting, test-time scaling, and a residual upper bound on generalizable improvement. We find that overfitting or test-time scaling explains a large share of the gain across many settings. We further show that held-out evaluation alone is not enough to assess the practical utility of an evolved harness. We suggest that future harness-evolution research measure overfitting using HDA together with held-out evaluation; account for test-time scaling by analyzing its contribution during evaluation as well as explicitly expressing the cost constraints during evolution. Finally, benchmark selection is critical for meaningful evaluation and comparison: rich, low-artifact benchmarks with meaningful distribution shifts are needed to discourage shortcuts, reward transferable harness improvements, and reveal differences among evolution methods.

## Acknowledgments

This work was partially supported by the Sloan Foundation, NSF CAREER Award IIS-2145280, NSF grant IIS-2433071, and the NSF under Cooperative Agreement 2421782 and the Simons Foundation grant MPS-AI-00010515 awarded to the NSF-Simons AI Institute for Cosmic Origins — CosmicAI, <https://www.cosmicai.org/>.

## References

  * Agrawal et al. (2026) L. A. Agrawal, S. Tan, D. Soylu, N. Ziems, R. Khare, K. Opsahl-Ong, A. Singhvi, H. Shandilya, M. J. Ryan, M. Jiang, et al. Gepa: reflective prompt evolution can outperform reinforcement learning.  In International Conference on Learning Representations,  Vol. 2026, pp. 8479–8565.  Cited by: §6.
  * Anthropic (2025) Anthropic Claude Haiku 4.5.  Note: <https://www.anthropic.com/news/claude-haiku-4-5>Accessed: 2026-09-25 Cited by: §B.1, §4.
  * Anthropic (2026a) Anthropic Claude Code.  Note: <https://claude.com/product/claude-code>Accessed: 2026-09-20 Cited by: §4.
  * Anthropic (2026b) Anthropic Claude Opus 4.8.  Note: <https://www.anthropic.com/news/claude-opus-4-8>Accessed: 2026-09-25 Cited by: §B.1, §4.
  * Cao et al. (2026) R. Cao, M. Chen, J. Chen, Z. Cui, Y. Feng, B. Hui, Y. Jing, K. Li, M. Li, J. Lin, Z. Ma, K. Shum, X. Wang, J. Wei, J. Yang, J. Zhang, L. Zhang, Z. Zhang, W. Zhao, and F. Zhou Qwen3-coder-next technical report.  arXiv preprint arXiv:2603.00729.  Cited by: §B.1, §4.
  * Chen et al. (2025) G. Chen, Z. Zhang, X. Cong, F. Guo, Y. Wu, Y. Lin, W. Feng, and Y. Wang Learning evolving tools for large language models.  In The Thirteenth International Conference on Learning Representations,  External Links: [Link](<https://openreview.net/forum?id=wtrDLMFU9v>) Cited by: §6.
  * Chen et al. (2026) T. Chen, S. Lu, K. Zhao, W. Meng, H. Teng, T. Li, C. Li, X. Liu, J. Liang, Z. Zhang, et al. Harnessx: a composable, adaptive, and evolvable agent harness foundry.  arXiv preprint arXiv:2606.14249.  Cited by: §1.
  * Hu et al. (2025) S. Hu, C. Lu, and J. Clune Automated design of agentic systems.  In International Conference on Learning Representations,  Vol. 2025, pp. 21344–21377.  Cited by: §6.
  * Huang et al. (2026a) C. Huang, Z. Wang, R. Han, J. Yan, Y. Chen, Z. CuiZhu, K. Jiang, P. Xia, H. Yu, Y. Zhuang, et al. Envharness: awakening static worlds for agent learning.  arXiv preprint arXiv:2608.19880.  Cited by: §6.
  * Huang et al. (2026b) L. Huang, C. Yang, H. Zhou, H. Song, Z. Chen, R. Le, Y. Song, W. X. Zhao, and T. Zhang Evo-bench: can language models improve agent harness?.  arXiv preprint arXiv:2608.09096.  Cited by: §6.
  * Jimenez et al. (2024) C. E. Jimenez, J. Yang, A. Wettig, S. Yao, K. Pei, O. Press, and K. R. Narasimhan SWE-bench: can language models resolve real-world github issues?.  In The Twelfth International Conference on Learning Representations,  External Links: [Link](<https://openreview.net/forum?id=VTF8yNQM66>) Cited by: §B.2, Appendix G, §1, §1, §4.
  * Karten et al. (2026) S. Karten, J. Zhang, T. Upaa Jr, R. Feng, W. Li, C. Shi, C. Jin, and K. Vodrahalli Continual harness: online adaptation for self-improving foundation agents.  arXiv preprint arXiv:2605.09998.  Cited by: §6.
  * Ke et al. (2026) Z. Ke, V. Patil, H. Shi, Y. Li, Y. Liu, S. Shekkizhar, A. Koul, J. Wang, X. P. Nguyen, S. Yavuz, et al. EVOHARNESSBENCH: can your agents keep pace with an evolving harness?.  arXiv preprint arXiv:2609.04280.  Cited by: §6.
  * Khattab et al. (2024) O. Khattab, A. Singhvi, P. Maheshwari, Z. Zhang, K. Santhanam, S. Vardhamanan, S. Haq, A. Sharma, T. T. Joshi, H. Moazam, H. Miller, M. Zaharia, and C. Potts DSPy: compiling declarative language model calls into self-improving pipelines.  Cited by: §6.
  * Lee et al. (2026) Y. Lee, R. Nair, Q. Zhang, K. Lee, O. Khattab, and C. Finn Meta-harness: end-to-end optimization of model harnesses.  arXiv preprint arXiv:2603.28052.  Cited by: §1, §1, §1, 1st item, §6.
  * Lin et al. (2026a) J. Lin, S. Liu, C. Pan, L. Lin, S. Dou, Z. Xi, X. Huang, H. Yan, Z. Han, T. Gui, et al. Agentic harness engineering: observability-driven automatic evolution of coding-agent harnesses.  arXiv preprint arXiv:2604.25850.  Cited by: §1, §1, 3rd item, §6.
  * Lin et al. (2026b) M. Lin, J. Wu, Z. Wang, Z. Shi, Y. Sang, B. He, Z. Liu, T. Wei, Z. Wu, Z. Zhang, et al. Harness updating is not harness benefit: disentangling evolution capabilities in self-evolving llm agents.  arXiv preprint arXiv:2605.30621.  Cited by: §1, §6.
  * Liu et al. (2026) J. Liu, X. Ye, P. Xia, Z. Zheng, C. Xie, M. Ding, and H. Yao Evolvemem: self-evolving memory architecture via autoresearch for llm agents.  arXiv preprint arXiv:2605.13941.  Cited by: §6.
  * Luo et al. (2026) X. Luo, F. Wang, C. Hu, D. Xue, and Y. Deng Self-evolving agent harnesses via gated semantic quality-diversity.  arXiv preprint arXiv:2607.13683.  Cited by: §6.
  * Mahdavi et al. (2025) S. Mahdavi, M. Li, K. Liu, C. Thrampoulidis, L. Sigal, and R. Liao Leveraging online olympiad-level math problems for llms training and contamination-resistant evaluation.  arXiv preprint arXiv:2501.14275.  Cited by: Appendix E, §1, §4.
  * Merrill et al. (2026) M. Merrill, A. Shaw, N. Carlini, B. Li, H. Raj, I. Bercovich, L. Shi, J. Shin, T. Walshe, E. K. Buchanan, et al. Terminal-bench: benchmarking agents on hard, realistic tasks in command line interfaces.  In International Conference on Learning Representations,  Vol. 2026, pp. 40903–40986.  Cited by: §1.
  * Ni et al. (2026) J. Ni, Y. Liu, X. Liu, Y. Sun, M. Zhou, P. Cheng, D. Wang, E. Zhao, X. Jiang, and G. Jiang Trace2skill: distill trajectory-local lessons into transferable agent skills.  arXiv preprint arXiv:2603.25158.  Cited by: §1, §6.
  * Ning et al. (2026) X. Ning, K. Tieu, D. Fu, T. Wei, Z. Li, Y. Bei, J. Zou, M. Ai, Z. Liu, T. Li, et al. Code as agent harness.  arXiv preprint arXiv:2605.18747.  Cited by: §1, §6.
  * Novikov et al. (2025) A. Novikov, N. Vũ, M. Eisenberger, E. Dupont, P. Huang, A. Z. Wagner, S. Shirobokov, B. Kozlovskii, F. J. Ruiz, A. Mehrabian, et al. Alphaevolve: a coding agent for scientific and algorithmic discovery.  arXiv preprint arXiv:2506.13131.  Cited by: §6.
  * Ouyang et al. (2026) S. Ouyang, J. Yan, I. Hsu, Y. Chen, K. Jiang, Z. Wang, R. Han, L. Le, S. Daruki, X. Tang, et al. Reasoningbank: scaling agent self-evolving with reasoning memory.  In International Conference on Learning Representations,  Vol. 2026, pp. 94327–94354.  Cited by: §6.
  * Qwen Team (2026) Qwen Team Qwen3.6-27B: flagship-level coding in a 27B dense model.  External Links: [Link](<https://qwen.ai/blog?id=qwen3.6-27b>) Cited by: §B.1, §4.
  * Shridhar et al. (2021) M. Shridhar, X. Yuan, M. Côté, Y. Bisk, A. Trischler, and M. Hausknecht ALFWorld: Aligning Text and Embodied Environments for Interactive Learning.  In Proceedings of the International Conference on Learning Representations (ICLR),  External Links: [Link](<https://arxiv.org/abs/2010.03768>) Cited by: Appendix D, §1, §4.
  * Tang et al. (2026) L. Tang, C. Rashtchian, C. Ferng, A. Tomkins, D. Juan, and T. Vu WikiSkill: compiling agent experience into persistent knowledge for skill evolution.  arXiv preprint arXiv:2608.27454.  Cited by: §1, §6.
  * Team (2026) Q. Team Qwen3. 5-omni technical report.  arXiv preprint arXiv:2604.15804.  Cited by: §B.1, §4.
  * Tu et al. (2026) X. Tu, T. Wang, K. Huang, Y. Qu, S. Mostafavi, et al. Benchguard: who guards the benchmarks? automated auditing of llm agent benchmarks.  arXiv preprint arXiv:2604.24955.  Cited by: §3.
  * Wadhwa et al. (2026) M. Wadhwa, T. S. Roy, H. Lederman, J. J. Li, and G. Durrett Create: testing llms for associative creativity.  arXiv preprint arXiv:2603.09970.  Cited by: §B.2, Appendix F, §1, §4, §5.
  * Wang et al. (2026a) J. Wang, F. Bianchi, S. Zhu, F. Nie, Y. Kwon, B. Dhingra, and J. Zou Automated benchmark auditing for ai agents and large language models.  arXiv preprint arXiv:2605.26079.  Cited by: §3.
  * Wang et al. (2026b) Y. Wang, H. Zhu, Z. Hu, Y. Yuan, Z. Chen, S. Senthil, H. Hajishirzi, Y. Tsvetkov, P. Dasigi, and T. Xiao Rethinking the evaluation of harness evolution for agents.  Cited by: §6.
  * Weng (2026) L. Weng Harness engineering for self-improvement.  lilianweng.github.io.  External Links: [Link](<https://lilianweng.github.io/posts/2026-07-04-harness/>) Cited by: §2, §6.
  * Wu et al. (2026) S. Wu, H. Zhu, Y. Zhang, X. Wang, and S. Yeung-Levy AutoMem: automated learning of memory as a cognitive skill.  arXiv preprint arXiv:2607.01224.  Cited by: §6.
  * Xia et al. (2026) P. Xia, R. Han, Z. Wang, Y. Chen, Y. Zhuang, Y. Lee, C. Huang, H. Yu, Z. CuiZhu, Y. Ming, H. Yao, B. Gokturk, T. Pfister, and C. Lee RRSI: regularized recursive self-improvement of agent harnesses.  arXiv preprint arXiv:2609.24972.  Cited by: §6.
  * Yang et al. (2025) A. Yang, A. Li, B. Yang, B. Zhang, B. Hui, B. Zheng, B. Yu, C. Gao, C. Huang, C. Lv, et al. Qwen3 technical report.  arXiv preprint arXiv:2505.09388.  Cited by: §B.1, §4.
  * Yang et al. (2024) J. Yang, C. Jimenez, A. Wettig, K. Lieret, S. Yao, K. Narasimhan, and O. Press Swe-agent: agent-computer interfaces enable automated software engineering.  Advances in Neural Information Processing Systems 37, pp. 50528–50652.  Cited by: §6.
  * Yang et al. (2026) Y. Yang, Z. Gong, W. Huang, Q. Yang, Z. Zhou, Z. Huang, Y. Li, X. Gao, Q. Dai, B. Liu, et al. Skillopt: executive strategy for self-evolving agent skills.  arXiv preprint arXiv:2605.23904.  Cited by: §B.2, §1, §1, 2nd item, §6.
  * Yao et al. (2023) S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. R. Narasimhan, and Y. Cao ReAct: synergizing reasoning and acting in language models.  In The Eleventh International Conference on Learning Representations,  External Links: [Link](<https://openreview.net/forum?id=WE_vluYUL-X>) Cited by: §6.
  * Ye et al. (2026) H. Ye, X. He, V. Arak, H. Dong, and G. Song Meta context engineering via agentic skill evolution.  In Forty-third International Conference on Machine Learning,  External Links: [Link](<https://openreview.net/forum?id=P1jHroBS5E>) Cited by: §6.
  * Zhang et al. (2026a) H. Zhang, S. Zhang, K. Li, C. Zhang, Y. Chen, Y. Zhang, L. Bai, and S. Hu Self-harness: harnesses that improve themselves.  arXiv preprint arXiv:2606.09498.  Cited by: §6.
  * Zhang et al. (2026b) J. Zhang, S. Hu, C. Lu, R. Lange, and J. Clune Darwin gödel machine: open-ended evolution of self-improving agents.  In International Conference on Learning Representations,  Vol. 2026, pp. 104223–104294.  Cited by: §6.
  * Zhang et al. (2026c) J. Zhang, B. Zhao, W. Yang, J. Foerster, J. Clune, M. Jiang, S. Devlin, and T. Shavrina Hyperagents.  arXiv preprint arXiv:2603.19461.  Cited by: §6.
  * Zhang et al. (2026d) J. Zhang, Y. Gu, J. Ruan, M. Song, Y. Peng, Z. Han, J. Xiang, Z. Wang, C. Yang, Y. Ouyang, et al. Harnessing agentic evolution.  arXiv preprint arXiv:2605.13821.  Cited by: §6, §6.
  * Zhang et al. (2025) J. Zhang, J. Xiang, Z. Yu, F. Teng, X. Chen, J. Chen, M. Zhuge, X. Cheng, S. Hong, J. Wang, et al. Aflow: automating agentic workflow generation.  In International Conference on Learning Representations,  Vol. 2025, pp. 34040–34077.  Cited by: §6.
  * Zhang et al. (2026e) L. Zhang, R. Zhou, D. Song, Z. Chen, Y. Tian, J. Yang, H. Ma, C. Li, G. Feng, X. Li, et al. Harnesscompass: guiding automatic harness evolution toward generalizable and effective agent harnesses.  arXiv preprint arXiv:2608.01918.  Cited by: §6.
  * Zhang et al. (2026f) Q. Zhang, C. Hu, S. Upasani, B. Ma, F. Hong, V. Kamanuru, J. Rainton, C. Wu, M. Ji, H. Li, et al. Agentic context engineering: evolving contexts for self-improving language models.  In International Conference on Learning Representations,  Vol. 2026, pp. 86069–86100.  Cited by: §6.
  * Zhang et al. (2026g) Y. Zhang, Y. Dai, J. Tan, L. Yang, R. Mullur, T. Hoang, Z. Hu, J. Zhu, P. Mui, S. Savarese, et al. DarwinX: evolving agent harnesses through natural selection.  arXiv preprint arXiv:2608.07545.  Cited by: §6.

## Appendix A Harness Evolution Details

Algorithm 1 summarizes the standard harness evolution loop.

Algorithm 1 Standard Harness Evolution Loop

1: Input: baseline harness H0H_{0}, executor model MM, proposer agent 𝒫\mathcal{P}, task sets 𝒳search\mathcal{X}_{\mathrm{search}} and optional 𝒳val\mathcal{X}_{\mathrm{val}}, iterations NN

2: F0←{Evaluate​(H0,M,𝒳search,𝒳val)}F_{0}\leftarrow\\{\textsc{Evaluate}(H_{0},M,\mathcal{X}_{\mathrm{search}},\mathcal{X}_{\mathrm{val}})\\}

3: for t=1,…,Nt=1,\ldots,N do

4: ℋt←Propose​(𝒫,Ft−1)\mathcal{H}_{t}\leftarrow\textsc{Propose}(\mathcal{P},F_{t-1})

5: Ft←Ft−1F_{t}\leftarrow F_{t-1}

6: for all H∈Valid​(ℋt)H\in\textsc{Valid}(\mathcal{H}_{t}) do

7: ft​(H)←Evaluate​(H,M,𝒳search,𝒳val)f_{t}(H)\leftarrow\textsc{Evaluate}(H,M,\mathcal{X}_{\mathrm{search}},\mathcal{X}_{\mathrm{val}})

8: Ft←Ft∪{ft​(H)}F_{t}\leftarrow F_{t}\cup\\{f_{t}(H)\\}

9: H^←Select​(FN,𝒳sel)\hat{H}\leftarrow\textsc{Select}(F_{N},\mathcal{X}_{\mathrm{sel}})

10: return H^\hat{H}

## Appendix B Experiment Setup Details

This appendix reports the model coverage and data splits used in our experiments.

### B.1 Executor models

We evaluate 11 executor models: nine open-weight models from the Qwen3, Qwen3.5, and Qwen3.6 families (Yang et al., 2025; Team, 2026; Cao et al., 2026; Qwen Team, 2026), together with Claude Haiku 4.5 and Claude Opus 4.8 (Anthropic, 2025; Anthropic, 2026b). The 25 model–benchmark–framework settings comprise 16 Meta-Harness, 6 SkillOpt, and 3 AHE settings. Table 1 gives the complete mapping.

### B.2 Benchmarks and splits

For LiveMath, we follow the 35/18/124 train/validation/test split used by SkillOpt (Yang et al., 2026). All splits follow the same question construction, and performance is measured by accuracy.

We split ALFWorld along both task-type and room axes. The 50 search games cover pick_and_place_simple, pick_clean_then_place_in_recep, and pick_heat_then_place_in_recep in kitchens, living rooms, and bathrooms. The 36 validation games include all six task types but only selected task-type–room combinations; the test set contains fresh, disjoint games with broader combination coverage. The 120 test games comprise 25 new instances of the search task types, 20 pick-and-place games in the held-out bedroom, and 25 games each for pick_cool_then_place_in_recep, pick_two_obj_and_place, and look_at_obj_in_light. Performance is measured by episode success within 50 environment steps.

We split CREATE by relation type and domain. The 120-item search split covers five common encyclopedic relations: occupation, place of birth, educated at, award received, and country of citizenship. The 80-item validation split includes new instances of these relations and four additional relations. The 561-item test set contains 107 new instances of the search relations, 263 instances from held-out biographical, family, and career relations, and 191 instances from held-out scientific and biomedical relations. Performance is measured by Creative Utility (CU), a continuous and unbounded metric that rewards valid and diverse answers (Wadhwa et al., 2026).

We split SWE-bench Verified into 48 search, 24 validation, and 50 test instances (Jimenez et al., 2024). The instances are sampled with seed 42 after filtering for tasks resolvable within one hour, while preserving the repository distribution across partitions. Performance is measured by success rate.

### B.3 Validation-based signal for Meta-Harness

For each candidate harness, the search-set evaluation exposes per-instance scores, execution trajectories, and logs to the proposer through the shared filesystem. We additionally evaluate candidates on a validation set, but expose only the aggregate validation score; per-instance validation results and trajectories remain hidden. We retain two harnesses from the evaluated candidates: the harness with the highest search score, H^\hat{H}, and the harness with the highest validation score, H^val\hat{H}^{\mathrm{val}}. Both are evaluated on the held-out test set only after evolution. This setup tests whether a standard validation signal improves transfer without revealing validation examples or trajectories.

## Appendix C The HDA Skill

Figure 6 summarizes the HDA skill used by the analysis agent. The complete machine-readable skill is included in the supplementary materials.

HDA Skill: Summarized Workflow Input: a baseline harness H0H_{0}, an evolved harness H^\hat{H}, and one evaluation set. 1. Select H0H_{0}, H^\hat{H}, and the evaluation set, and compute Δ​S=S⁡(H^)−S⁡(H0)\Delta S=S(\hat{H})-S(H_{0}). 2. Specify the task objective, score aggregation, inference-cost measure, and task-specific evaluation hooks. 3. Apply a paired significance test. If Δ​S\Delta S is not statistically significant, report it as within noise and do not assign an attribution. 4. Construct the compute-matched baseline H0scaleH_{0}^{\mathrm{scale}} and compute T=S⁡(H0scale)−S⁡(H0)T=S(H_{0}^{\mathrm{scale}})-S(H_{0}). Human checkpoint 1: review the compute measure and matching procedure. 5. Detect and neutralize supported overfitting mechanisms in H^\hat{H} to construct H^neutral\hat{H}^{\mathrm{neutral}}, then compute O=S⁡(H^)−S⁡(H^neutral)O=S(\hat{H})-S(\hat{H}^{\mathrm{neutral}}). Human checkpoint 2: review the evidence and neutralization. 6. Compute the residual G=Δ​S−O−T=S⁡(H^neutral)−S⁡(H0scale),G=\Delta S-O-T=S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}}), and report the raw components, normalized percentages, evidence, and limitations. Interpretation: OO is a lower bound on detected overfitting, and GG is an upper bound on generalizable improvement. Held-out transfer is evaluated separately. Figure 6: Overview of the HDA skill. The skill standardizes the attribution procedure while retaining human review at the two task-dependent decisions: compute matching and overfitting neutralization. Figure 7: Overview of representative evolved-harness mechanisms across the four benchmarks (left four cards, Meta-Harness): hard-coded object-location priors and task recipes on ALFWorld, the meta-option string match on LiveMath, parallel generation frames on CREATE (a test-time-scaling counter-case), and a patch-size prior on SWE-bench. The two right cards show the _same_ LiveMath meta-option artifact independently rediscovered by SkillOpt (as a skill default rule) and AHE (as an explicit prompt rule). Red marks overfitting evidence; yellow marks test-time-scaling evidence; each card’s OO/TT chip is the attribution of the specific Table 1 row whose winner is excerpted — not a benchmark-level average (ALFWorld and LiveMath: Qwen3.5-0.8B; CREATE: Claude Haiku 4.5; SWE-bench: Qwen3-4B-Instruct-2507; SkillOpt: LiveMath Qwen3.5-4B; AHE: LiveMath Claude Opus 4.8). The following sections (E–G) present each mechanism in detail with source excerpts.

## Appendix D ALFWorld: Benchmark-Specific Procedural Scaffolds

On ALFWorld (Shridhar et al., 2021), Meta-Harness and SkillOpt encode similar simulator-specific knowledge, including task recipes, appliance mappings, object-location priors, and environment rules. Figures 8 and 9 show one representative harness from each framework. Red highlights mark mechanisms identified as overfitting by HDA.

Meta-Harness ⋅\cdot ALFWorld ⋅\cdot Qwen3.5-0.8B: zero-LLM controller [⬇](<) LOCATION_PRIORS = { ’mug’: [’coffeemachine’, ’countertop’, ’cabinet’, ’shelf’], ’tomato’: [’fridge’, ’countertop’, ’diningtable’], ’desklamp’: [’desk’, ’sidetable’], ... # object-to-receptacle priors } class AdaptiveContainerDepthAgent(AgentHarness): @staticmethod def _parse_task(task_description: str): ... if ’cool’ in task: process_steps.append((’cool’, ’fridge’)) elif ’heat’ in task: process_steps.append((’heat’, ’microwave’)) elif ’clean’ in task: process_steps.append((’clean’, ’sinkbasin’)) def step(self, observation, admissible_actions): self._update_state(observation, admissible_actions) action = self._pick_action(observation, admissible_actions) return StepResult(action=action, thought=f"phase={self.phase}") Figure 8: Meta-Harness ALFWorld winner for Qwen3.5-0.8B. The highlighted tables and parser rules encode benchmark-specific object locations and task procedures. The full harness contains no llm_call, so all actions come from the deterministic controller. HDA: O/T/G=100/0/0O/T/G=100/0/0. SkillOpt ⋅\cdot ALFWorld ⋅\cdot Qwen3.5-4B: the scaffold as a natural-language skill [⬇](<) ## Task Procedures - **pick_heat_then_place**: take <obj> -> go to microwave 1 -> heat <obj> with microwave 1 -> go to <recep> -> place. - **pick_cool_then_place**: take <obj> -> go to fridge 1 -> cool <obj> with fridge 1 -> go to <recep> -> place. - **pick_clean_then_place**: take <obj> -> go to sinkbasin 1 -> clean <obj> with sinkbasin 1 -> go to <recep> -> place. ## Simulator Rules and Search Priors If an action returns "Nothing happens", do not repeat it: a precondition is unmet. There is no terminate action. Closed cabinets, drawers, fridges, and microwaves hide their contents. For food and produce, search countertops, dining tables, sinkbasins, and fridges before cabinets or drawers; this is exactly how the lettuce task was lost. Figure 9: SkillOpt ALFWorld skill for Qwen3.5-4B. The highlighted instructions encode the same appliance recipes, simulator rules, and location priors as the Meta-Harness controller, including a rule distilled from a specific search-set failure. HDA: O/T/G=44/0/56O/T/G=44/0/56.

The two frameworks therefore rediscover similar ALFWorld-specific procedures but represent them differently: executable controller logic in Meta-Harness and natural-language instructions in SkillOpt. The zero-LLM controller still reaches 93.3%93.3\% success on held-out ALFWorld games because the same simulator ontology and rules persist across splits, illustrating why held-out performance alone may not reveal benchmark-specific overfitting. Full harnesses are included in the supplementary materials.

## Appendix E LiveMath: The Shared Meta-Option Shortcut

LiveMath (Mahdavi et al., 2025) contains a recurring answer choice: “one of the remaining options is correct, but a stronger result can be proven.” It appears in 21 of the 35 search questions and is correct in all 21. The same pattern occurs in 53 of the 124 test questions and is again always correct, so exploiting it can also appear to transfer. All three frameworks discover this shortcut. Figure 10 shows how each framework encodes it; red highlights mark overfitting mechanisms and yellow highlights mark test-time scaling.

Meta-Harness ⋅\cdot LiveMath ⋅\cdot Qwen3.5-0.8B: string matching with a multi-call fallback [⬇](<) META_MARKERS = [ "one of the remaining options is correct", "stronger result can be proven", ] def solve(self, item): meta_label = self._find_meta(item["choices"]) if meta_label: return SolveResult(predicted_label=meta_label) votes = [] for prompt in generation_prompts: generation = self.llm_call(prompt) votes.append(self._match_choice(generation)) return majority_vote(votes)

SkillOpt ⋅\cdot LiveMath ⋅\cdot Qwen3.5-4B: the same shortcut as a skill rule [⬇](<) ## Option Comparison - Treat a meta-option such as "one of the remaining options is correct, but a stronger result can be proven" as a first-class candidate, never a throwaway. - If the best concrete option is true but improvable, the meta-option almost always beats it. ### Meta-option decision rule The meta-option is your working default. Reject it only when a concrete option exactly matches the strongest known result and no stronger true statement can be added; otherwise, the meta-option wins.

AHE ⋅\cdot LiveMath ⋅\cdot Claude Opus 4.8: an explicit prompt rule [⬇](<) ## Handling the "meta" option This meta-option is a FIRST-CLASS candidate, not filler. If a strictly stronger true statement exists beyond the best concrete option, the meta-option is correct. Choose it.

Figure 10: Three representations of the same LiveMath artifact. Meta-Harness directly returns the matching option before invoking the executor and otherwise uses multiple model calls (O/T/G=91/9/0O/T/G=91/9/0). SkillOpt encodes it as a default decision rule in the skill document (88/0/1288/0/12). AHE writes an explicit selection rule into the system prompt (100/0/0100/0/0).

The shortcut therefore recurs across code, skill, and prompt evolution. Because the artifact is shared across the search and test splits, held-out gains within LiveMath do not by themselves establish improved mathematical reasoning. Full harnesses for all three frameworks are included in the supplementary materials.

## Appendix F CREATE: Metric-Driven Test-Time Scaling

CREATE rewards answer pools containing many valid and diverse responses through its Creative Utility (CU) metric (Wadhwa et al., 2026). HDA finds no detected overfitting in the attributed CREATE settings (O=0O=0); instead, frameworks that can modify orchestration increase the number of model calls and token usage. Figure 11 shows the parallel strategy discovered by Meta-Harness, the sequential strategy discovered by AHE, and SkillOpt’s fixed single-call control flow. Yellow highlights mark test-time-scaling mechanisms.

Meta-Harness ⋅\cdot CREATE ⋅\cdot Claude Haiku 4.5: parallel generation [⬇](<) # Six independent first-wave frames for frame_name, template, temp in wave1_frames: response = self.llm_call(messages=[...], temperature=temp) all_paths.extend(extract_paths(response)) # Fourteen additional diversity frames for frame_cfg in _WAVE2_FRAMES: response = self.llm_call(messages=[...], temperature=frame_cfg["temp"]) all_paths.extend(extract_paths(response))

AHE ⋅\cdot CREATE ⋅\cdot Claude Opus 4.8: sequential expansion [⬇](<) You MUST generate as many distinct valid paths as possible. The final answer should contain MANY distinct paths (aim for 8+). MIN_PATHS = 5 if has_answer_block and n_paths < MIN_PATHS: feedback = _build_abstention_feedback(n_paths) return HookResult.with_modifications( messages=messages + [feedback], force_continue=True, # force another model turn )

SkillOpt ⋅\cdot CREATE: the skill changes, but the one-call rollout remains fixed [⬇](<) system = _build_system(skill_content) # inject the evolved skill user = _build_user(item) response, _ = chat_target( system=system, user=user, max_completion_tokens=max_completion_tokens, ) result["n_turns"] = 1

Figure 11: CREATE orchestration across the three frameworks. Meta-Harness makes 20 calls per item and obtains O/T/G=0/49/51O/T/G=0/49/51. AHE increases the mean from 1.04 to 2.46 calls per item through forced expansion turns; using the paper’s k=3k=3 compute match gives 0/75/250/75/25. SkillOpt can edit only the injected skill, while the rollout remains one call; both SkillOpt CREATE gains are statistically insignificant, so HDA assigns no attribution.

The compute-matched baseline therefore recovers a substantial share of the gains when orchestration is evolvable, and even outperforms the evolved Meta-Harness Qwen3.6-35B-A3B harness. SkillOpt provides the complementary case: restricting evolution to the skill prevents additional calls, but neither evaluated setting produces a significant gain. Full harnesses are included in the supplementary materials.

## Appendix G SWE-bench Verified: Mixed Gains and Reusable Guardrails

SWE-bench Verified (Jimenez et al., 2024) shows the clearest residual improvements in our study, but its gains can still combine overfitting and test-time scaling. Figure 12 contrasts two Meta-Harness examples. For Qwen3-4B-Instruct-2507, the evolved harness combines general edit guardrails with a patch-size rule fitted to search-set trajectories. It also uses 1.44×1.44\times as many LLM calls as the baseline, so HDA attributes its gain as O/T/G=23/23/54O/T/G=23/23/54. The Qwen3-30B-A3B-Instruct-2507 harness instead detects a general parser failure and has O/T/G=0/6/94O/T/G=0/6/94.

Meta-Harness ⋅\cdot SWE-bench ⋅\cdot Qwen3-4B: generic edit feedback plus a benchmark-fitted patch envelope [⬇](<) # General guardrail: make failed edits visible. _NOOP_LINES = [ "This command did NOT modify any file.", "Do NOT assume the edit worked and do NOT submit.", ] # Patch-size thresholds inferred from resolved and failed search trajectories. ADD_T, DEL_T, DUP_T = 12, 8, 3 if adds >= ADD_T or dels >= DEL_T or maxdup >= DUP_T: warn("Your accumulated patch is LARGE for a bug fix.") warn("Resolved SWE-bench fixes are almost always a FEW lines.") warn("Trim the patch before submitting.")

Meta-Harness ⋅\cdot SWE-bench ⋅\cdot Qwen3-30B-A3B: content-independent action feedback [⬇](<) def _count_action_blocks(response): edits = len(re.findall(r"‘‘‘edit\b", response)) commands = len(re.findall(r"‘‘‘(?:bash|sh|mswea_bash_command)\b", response)) return edits + commands # The parser executes only the first action block. if self.dropped_actions > 0: observation += ( "Only the first action block was executed. " f"{self.dropped_actions} block(s) were not run. " "Re-issue each required action and inspect its actual output." )

Figure 12: Examples of mixed and residual-dominated gains on SWE-bench Verified. Red marks the Qwen3-4B harness’s benchmark-derived patch-size heuristic: the +12/−8+12/-8-line and repetition thresholds encode the search set’s tendency toward small fixes and may reject valid larger patches elsewhere. Disabling only this mechanism accounts for O=23%O=23\%. Matching the evolved harness’s higher LLM inference cost raises the baseline from pass@2 to pass@3 and accounts for T=23%T=23\%; the remaining G=54%G=54\% comes from mechanisms such as edit-effect feedback. The Qwen3-30B-A3B action-block guardrail uses no benchmark-specific content, yielding O/T/G=0/6/94O/T/G=0/6/94.

Thus, SWE-bench contains more reusable harness improvements than the other benchmarks, but the source remains setting-dependent. The weaker Meta-Harness executors transfer positively, whereas Qwen3.6-27B and Qwen3-Coder-30B-A3B regress on the held-out test set despite 0/0/1000/0/100 search-set attributions. This confirms that GG is an upper bound rather than a guarantee of transfer. Full harnesses and additional settings are included in the supplementary materials.
