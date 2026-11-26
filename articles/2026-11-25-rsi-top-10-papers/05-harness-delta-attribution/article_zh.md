---
title: "Harness-Delta Attribution：拆解 harness 演化带来的增益"
author: "Wenxuan Ding, Qin Lu, Changlong Yu, Sha Li, Shuowei Jin, Xin Liu, Zhenyu Shi, and Greg Durrett"
source_url: "https://arxiv.org/abs/2610.22466"
arxiv_id: "2610.22466"
arxiv_version: "v1"
published_at: "2026-10-16"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-translation"
unofficial_translation: true
translation_scope: "摘要至结论的全部正文，含表格、公式、脚注与附录。代码保留英文。参考文献保留英文。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名与 CC BY 4.0。"
---

**非官方中文翻译声明。** 本文是对 Wenxuan Ding、Qin Lu、Changlong Yu、Sha Li、Shuowei Jin、Xin Liu、Zhenyu Shi 与 Greg Durrett 所著论文的非官方中文译文，原文网址为 <https://arxiv.org/abs/2610.22466>，采用 CC BY 4.0 许可（<https://creativecommons.org/licenses/by/4.0/>）。本译文未经作者审定。相对原文的改动仅限于语言（由英文译为中文）以及 Markdown 排版（公式与表格改为 Markdown）；科学内容未作改动。使用本译文须保留原作者署名与 CC BY 4.0 许可。

# Harness-Delta Attribution：拆解 harness 演化带来的增益

Wenxuan Ding、Qin Lu、Changlong Yu、Sha Li　　单位：Amazon、New York University　　邮箱：wd2403@nyu.edu

Shuowei Jin、Xin Liu、Zhenyu Shi、Greg Durrett　　单位：Amazon、New York University

## 摘要

Harness 演化自动优化固定权重模型周围的提示、技能、工具与编排代码，从而提供一种无需更新模型权重即可改进智能体系统的途径。智能体 harness 被迭代优化，以求在搜索集上取得更高表现。但这种增益中有多少反映的是特定于基准的过拟合或额外的推理时计算，又有多少可以泛化到搜索集之外？在本工作中，我们提出 Harness-Delta Attribution（HDA），一种评估协议，它将一次演化增益拆解为工件驱动的过拟合、可由测试时缩放解释的增益，以及其余部分；其余部分给出残差可泛化改进的上界。我们将 HDA 应用于 11 个执行器模型与三个 harness 演化框架，任务覆盖数学、编程、创造力与智能体任务。在其中 21 个呈现出统计显著搜索集增益的设定中，我们发现改进的很大一部分可由过拟合或测试时缩放解释。具体而言，在这 21 个设定中，过拟合与测试时缩放平均分别占增益的 $54\%$ 与 $14\%$；二者合计份额在 14 个设定中超过 $50\%$。残差与留出迁移在不同任务、模型与框架之间差异很大。重要的是，搜索集增益常常无法迁移到留出任务，并且对于这些增益能否在实际分布偏移下持续，只能提供有限的证据。此外，基于验证的信号不足以防止过拟合。这些结果促使未来的 harness 演化研究采用归因感知的评估。[^1]

[^1]: 代码：<https://github.com/Wenwen-D/HarnessDeltaAttribution>　博客：<https://wenwen-d.github.io/blog/harness-delta-attribution/>

---

图 1：演化后的 harness 可能通过过拟合于基准工件、扩展测试时计算量，或做出可泛化改进来提升表现（左）。HDA 追问观测到的增益来自何处，并将其归因于这三类来源（右）。在各个设定中，过拟合与测试时缩放解释了相当大的份额。

## 1 引言

一个智能体系统可以看作一个 LLM 与一个 harness 的配对：提示、工具、记忆、技能、编排代码以及控制流，它们共同塑造模型如何与任务交互。即便不更新模型权重，对 harness 的改动也可以在很大幅度上改变系统表现，正如在 Terminal-Bench (Merrill et al., 2026)与 SWE-bench (Jimenez et al., 2024)等基准上所观察到的那样。这推动了近期关于 harness 演化的工作。Harness 演化是一种文本空间优化，它利用任务反馈，迭代更新固定权重模型周围的非参数组件 (Lee et al., 2026; Lin et al., 2026a; Lin et al., 2026b)。具体而言，在每一轮迭代中，执行器模型使用当前 harness 尝试这些任务。随后，一个提议者智能体检查轨迹与评估结果，并对 harness 提出编辑。重复这一过程，会产生一串针对搜索集（训练划分）上的表现而优化的 harness。

近期的 harness 演化框架报告了来自演化后 harness 代码 (Lee et al., 2026; Ning et al., 2026)、技能文件 (Ni et al., 2026; Yang et al., 2026; Tang et al., 2026)以及更新后的工具使用或编排模式 (Chen et al., 2026)的显著增益。然而，更高的汇总分数并不会揭示系统中究竟改变了什么，也不会揭示这些增益能否在搜索集之外持续。先前工作并不总是在留出实例上评估演化后的 harness。即便报告了留出结果，搜索划分与测试划分仍可能共享相同的任务构造与数据集工件。此类评估衡量的是基准内部的迁移，但对于增益能否在新任务或实际分布偏移下持续，只能提供有限的证据。

在本工作中，我们提出 Harness-Delta Attribution（HDA），一种把 harness 演化增益归因于三种不同来源的评估协议。有些增益可能反映真正的 harness 改进，并且这些改进能够泛化到搜索集之外。然而，我们识别出另外两种增益来源，它们促使我们以不同的眼光来看待 harness 优化的结果。第一，演化后的 harness 可能扩展测试时计算：例如，它可能并行采样五个响应，并选出多数投票的结果。第二，演化后的 harness 可能利用数据集捷径，把搜索集中反复出现的答案模式硬编码下来。这些机制可以产生相似的基准增益，但对成本与泛化的含义却大不相同。

我们的方法 HDA 使用一个计算量匹配的基线，以及演化后 harness 的一个工件中和版本，来比较基线 harness 与演化后的 harness。这使得我们能够估计已检测过拟合的下界，分离出通过额外推理计算即可复现的增益，并度量剩余的残差。我们还评估留出表现，以了解演化后的 harness 是否迁移到搜索集之外。

我们的实验覆盖三个不同框架，包括 Meta-Harness (Lee et al., 2026)、SkillOpt (Yang et al., 2026)与 AHE (Lin et al., 2026a)。我们用它们在若干基准上演化 11 个执行器模型的 harness，这些基准覆盖数学推理 (LiveMath (Mahdavi et al., 2025))、软件工程 (SWE-bench Verified (Jimenez et al., 2024))、创造性生成 (CREATE (Wadhwa et al., 2026))与具身规划 (ALFWorld (Shridhar et al., 2021))。总计，我们将 HDA 应用于 25 个设定。

图 1 预先展示了我们的主要发现：在各种配置中，过拟合与测试时缩放解释了搜索集增益的很大份额，而可泛化改进的残差上界往往很小。在 21 个具有统计显著增益的设定中，HDA 平均将至少 $54\%$ 的增益归因于过拟合，并将 $14\%$ 归因于测试时缩放；二者合计份额在 14 个设定中超过 $50\%$。这些改进的分解还可以揭示更多模式。第一，不同框架反复发现相同的、特定于基准的捷径，例如 LiveMath 中反复出现的那个正确选项。第二，在 ALFWorld 中，一些演化后的 harness 通过在该合成基准上硬编码确定性解，完全消除了 LLM 调用。

我们的贡献是：（1）识别 harness 演化增益的三种来源（过拟合、测试时缩放与可泛化改进）；（2）HDA 方法，它将 harness 演化增益拆解为 3 类，并揭示仅靠留出评估所不能显示的发现；（3）一套可复用的 HDA 技能与评估套件，用于把这一分析应用到新的基准与 harness 演化框架上。

## 2 预备知识

设 $M$ 表示其 harness 正在被优化的固定执行器模型，$H$ 表示其 harness；二者合在一起，$(M,H)$ 构成智能体。$H$ 包含支配执行器如何处理任务的那些非权重组件，例如其提示、技能、工具、记忆与编排逻辑 (Weng, 2026)。不同的 harness 演化方法在保持 $M$ 固定的同时，优化这些组件的不同子集。我们用 $\mathcal{H}$ 表示可能被探索的、特定于框架的 harness 空间。

给定任务集 $\mathcal{X}$ 与任务实例 $x\in\mathcal{X}$，执行器模型产生一条执行轨迹

$$\tau_{H,x}\sim M(\cdot\mid x;H).$$

该轨迹包括执行过程中产生的模型输出、工具调用、环境交互以及其他中间状态。设 $r(x,\tau_{H,x})$ 表示赋予该轨迹的、特定于任务的奖励。我们将 harness $H$ 在 $\mathcal{X}$ 上的分数定义为

$$S_{\mathcal{X}}(H)=\frac{1}{|\mathcal{X}|}\sum_{x\in\mathcal{X}}r(x,\tau_{H,x}).$$

#### 演化循环

从基线 harness $H_{0}$ 出发，提议者智能体 $\mathcal{P}$ 迭代编辑该 harness 以提升其表现，从而产生演化后的 harness $\hat{H}$。提议者智能体可以就是执行器智能体本身，如自演化中那样，也可以是一个单独的、并且可能更强的智能体。设 $\mathcal{X}_{\mathrm{search}}$ 表示演化期间使用的搜索集。该框架维护一份以文件系统为后端的历史 $F_{t}$，记录直到迭代 $t$ 为止的 harness 版本、轨迹、评估结果以及其他工件，其中 $F_{0}$ 包含基线评估。在迭代 $t$，提议者使用 $F_{t-1}$ 生成一个由 $k$ 个候选 harness 组成的集合 $\mathcal{H}_{t}$：

$$\mathcal{H}_{t}:=\{H_{t}^{(j)}\}_{j=1}^{k}=\operatorname{Propose}(\mathcal{P};F_{t-1}).$$

每个候选都在 $\mathcal{X}_{\mathrm{search}}$ 上加以评估，$f_{t}(H)$ 表示其反馈，其中可以包括它的分数、逐任务奖励、轨迹与执行日志。累积历史随后更新为

$$F_{t}=F_{t-1}\cup\{f_{t}(H):H\in\mathcal{H}_{t}\}.$$

一些框架把持久化的工件直接暴露给提议者，另一些框架则把选定的记录插入结构化提示或优化器状态之中。经过 $N$ 次迭代，已评估的 harness 池为

$$\mathcal{H}_{\mathrm{eval}}=\{H_{0}\}\cup\bigcup_{t=1}^{N}\mathcal{H}_{t}.$$

被选中的演化后 harness 为

$$\hat{H}=\operatorname*{arg\,max}_{H\in\mathcal{H}_{\mathrm{eval}}}S_{\mathcal{X}_{\mathrm{search}}}(H).$$

可选的验证集 $\mathcal{X}_{\mathrm{val}}$ 可用于把关更新或选择 harness。算法 1 概括了这一标准演化循环。

## 3 Harness-Delta Attribution：拆解 harness 演化带来的增益

图 2：Harness-Delta Attribution（HDA）示意图。比较 $S(H_{0})$ 与 $S(\hat{H})$ 得到观测增益 $\Delta S$。以 $\hat{H}$ 的计算预算运行 $H_{0}$，得到计算量匹配基线 $H_{0}^{\mathrm{scale}}$；它相对 $H_{0}$ 的增益度量测试时缩放。禁用 $\hat{H}$ 中已检测到的捷径，得到 $\hat{H}^{\mathrm{neutral}}$；由此产生的下降度量过拟合。剩余的差距估计可泛化改进。

Harness 演化返回一个演化后的 harness $\hat{H}$，其搜索集分数高于基线 $H_{0}$，但差值 $\Delta S=S(\hat{H})-S(H_{0})$ 并不能解释表现为何提高。例如，在一次 Meta-Harness 的 ALFWorld 运行中，面向 0.8B 执行器的 harness 从 66 行增长到 503 行非空、非注释代码（$7.6\times$），并达到 $100\%$ 的搜索集成功率，但它通过硬编码基准先验变成了一个确定性控制器，并且不再进行任何 LLM 调用（图 3）。在其他设定中，演化增加了推理计算，而直接缩放基线也可能产生相似的增益，却不必付出演化的搜索成本。图 1（左）展示了这些情形。

我们引入 Harness-Delta Attribution（HDA），以区分这些增益来源。其核心思想是将演化后的 harness 与两个受控变体进行比较：一个经缩放的基线，用于识别仅靠测试时缩放、而无须 harness 演化即可恢复的增益；以及一个中和后的演化 harness，其中已检测到的、由工件驱动的机制被移除，用以检测过拟合。

我们将 HDA 实现为一个可复用的技能，从而可以跨框架与任务加以应用。一个分析智能体遵循该技能执行归因，人类审阅者在两个依赖于任务的检查点上检查该智能体提出的方案：计算量匹配的设置，以及过拟合的中和。这一设计使工作流的大部分保持自动化，并提供一个具备任务意识的质量控制层，类似于对自动化方法所作的人工评估。我们在附录 C 中给出 HDA 技能。

在归因之前，我们对 $H_{0}$ 与 $\hat{H}$ 在搜索集上的分数做成对 bootstrap 显著性检验。若 $\Delta S$ 不显著，我们将其报告为处于噪声范围之内，并且不赋予归因。否则，HDA 智能体按下文所述构造计算量匹配基线 $H_{0}^{\mathrm{scale}}$ 与中和后的演化 harness $\hat{H}^{\mathrm{neutral}}$。HDA 随后定义三个归因分量：

$$
\begin{aligned}
T &= S(H_{0}^{\mathrm{scale}})-S(H_{0}), \\
O &= S(\hat{H})-S(\hat{H}^{\mathrm{neutral}}), \\
G &= S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}}).
\end{aligned}
$$

#### $T$ 测试时缩放：计算量匹配

计算量匹配基线 $H_{0}^{\mathrm{scale}}$ 保留 $H_{0}$，并将其缩放到与 $\hat{H}$ 的平均推理预算相匹配。具体而言，HDA 智能体使用 LLM 调用次数或 token 用量等度量来比较二者的推理成本。随后，它提出一种适合该任务的计算度量，以及构造 $H_{0}^{\mathrm{scale}}$ 的过程，供人工批准。

例如，若 $\hat{H}$ 在每个任务上使用的执行器调用次数是 $H_{0}$ 的十倍，则 $H_{0}^{\mathrm{scale}}$ 可以在每个任务上把 $H_{0}$ 运行十次，并用多数投票、pass@10 或其他特定于任务的聚合规则合并其输出。若 $\hat{H}$ 使用的推理计算并不多于 $H_{0}$，我们令 $H_{0}^{\mathrm{scale}}=H_{0}$。

差值 $T=S(H_{0}^{\mathrm{scale}})-S(H_{0})$ 度量的是：观测到的改进中有多少可以仅通过测试时缩放加以复现，而不是来自改进后的 harness 设计。直接缩放 $H_{0}$ 也避免了 harness 演化的搜索成本。尽管此类增益可能迁移到留出任务，其实际价值取决于额外的延迟与推理成本对于目标应用而言是否可以接受。

#### $O$ 过拟合：检测与中和

HDA 智能体通过从 $\hat{H}$ 中移除已检测到的过拟合机制，来构造中和后的 harness $\hat{H}^{\mathrm{neutral}}$。当一个演化出来的机制是靠利用特定于该基准或搜索集的某种模式来提高表现，而不是更一般地提高智能体达成预期任务目标的能力时，我们将其归类为过拟合。在那些保留任务目标、但不保留该模式的新实例上，其收益可能会消失。

在 HDA 技能的引导下，该智能体审计 harness 代码、提示以及可用的执行证据，查找两类宽泛形式的过拟合：

- 直接捷径：绕过或预先决定执行器行为的机制，包括消除 LLM 调用、常量答案、针对特定条目的门控、硬编码查找表，以及字符串匹配规则。

- 源自基准的启发式：利用基准实例之构造方式所特有的模式、而不是利用求解底层任务所必需之性质的机制。例子包括拟合于基准统计量的阈值、经逆向工程得到的构造规则、数据集特定的先验，以及硬编码的解题配方。

对于每一个检测到的机制，HDA 智能体做出能够禁用该捷径的最小干预，从而得到 $\hat{H}^{\mathrm{neutral}}$，同时保留所有与此无关的演化组件。

图 3：harness 演化中过拟合的例子。如这段摘录所示，Meta-Harness 为 Qwen3.5-0.8B 演化出的 ALFWorld harness，通过硬编码一个进行 0 次 LLM 调用的确定性控制器，过拟合于特定基准的模式。

为支持可靠的审计，HDA 技能提供共享的定义、示例与证据要求。先前工作也表明，前沿智能体能够识别基准缺陷与奖励黑客机会 (Tu et al., 2026; Wang et al., 2026a)。人工审阅也增加了一道针对该任务与预期用途而定制的保障。

需要注意的是，HDA 只中和那些可以在不影响 harness 中无关部分的前提下被禁用的已检测机制。因此，我们将 $O=S(\hat{H})-S(\hat{H}^{\mathrm{neutral}})$ 解释为过拟合的下界估计，因为隐式的或相互纠缠的机制可能仍然留在 $\hat{H}^{\mathrm{neutral}}$ 之中。尽管如此，正如我们稍后在结果中所表明的，过拟合所占的份额仍然相当大。

#### $G$ 可泛化改进：残差上界

残差 $G$ 是在计入已检测过拟合与测试时缩放之后仍然留下的增益。按构造，$G=\Delta S-O-T=S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}})$。因为 $\hat{H}^{\mathrm{neutral}}$ 之中可能仍然含有未被中和的过拟合，我们将 $G$ 解释为可泛化改进的上界，而不是迁移已经发生的证据。我们在第 5 节中通过在留出任务上比较 $H_{0}$ 与 $\hat{H}$，另外直接评估可泛化性。

在报告时，我们以 $\Delta S$ 对每个分量做归一化，将 $O$ 的份额截断到 $[0\%,100\%]$，将 $T$ 的份额截断到剩余的百分比，并把剩余部分分配给 $G$。这些截断只影响所报告的百分比；原始的带符号分量仍然满足 $\Delta S=O+T+G$。

## 4 方法与实验

#### 基准

我们在四个基准上进行评估，它们跨越单轮推理与长程智能体任务，并覆盖多个领域：数学推理 (LiveMath (Mahdavi et al., 2025))、创造性生成 (CREATE (Wadhwa et al., 2026))、交互式具身规划 (ALFWorld (Shridhar et al., 2021))，以及软件编程 (SWE-bench Verified (Jimenez et al., 2024))。

#### 演化框架

我们评估三个 harness 演化框架，它们的优化范围以及提议者–执行器配置各不相同，其中包括自演化，以及使用更强的智能体作为提议者：

- Meta-Harness (Lee et al., 2026)在固定的 LLM 周围搜索 harness 代码。提议者可以通过文件系统访问先前历史的源代码、分数与执行轨迹。

- SkillOpt (Yang et al., 2026)优化一个被冻结的智能体的技能：一个单独的优化器模型把带分数的 rollout 转化为对技能文档的有界编辑。

- AHE (Lin et al., 2026a)（Agentic Harness Engineering）通过一个由可观测性驱动的循环，演化多个 harness 组件，包括提示、工具、技能与记忆。遵循原文，我们使用其自演化设定，以 Opus 4.8 同时作为提议者与执行器。

对于 Meta-Harness，相对于原始实现，我们在演化期间额外提供汇总的验证分数，同时隐藏逐实例的验证结果与轨迹。我们同时保留按搜索集选出的 harness 与按验证集选出的 harness，并在留出测试集上评估它们。更多细节见附录 B.3。

#### 模型

我们评估来自 Qwen 与 Claude 模型族的 11 个执行器模型。Qwen 执行器跨越 Qwen3、Qwen3.5 与 Qwen3.6，规模从 0.8B 到 35B，并且既包括通用模型，也包括编程专用模型 (Yang et al., 2025; Team, 2026; Cao et al., 2026; Qwen Team, 2026)。我们还在部分设定上评估 Claude Haiku 4.5 与 Claude Opus 4.8 (Anthropic, 2025; Anthropic, 2026b)。Meta-Harness 与 SkillOpt 使用 Claude Opus 4.8 作为提议者智能体 (Anthropic, 2026a)，而 AHE 使用 Opus 4.8 同时作为提议者与执行器。确切的执行器集合因基准而异；更多细节见表 1 与附录 B。

## 5 结果

#### 大多数增益来自过拟合或测试时缩放

表 1：全部 25 个模型–基准–框架设定的 HDA 结果。$\Delta S=S(\hat{H})-S(H_{0})$ 是搜索集增益。$O$、$T$ 与 $G$ 是 $\Delta S$ 的百分比，并截断到 $[0\%,100\%]$。$\dagger$ 标记不具有统计显著性的搜索集增益，对此 HDA 不报告归因。$\star$ 标记演化后的 harness 在留出测试集上表现差于基线的设定。CU 表示 Creativity Utility（创造性效用），即 CREATE 基准所使用的度量。

| 框架 | 基准 | 执行器 | $\Delta S$ | $O\downarrow$（%） | $T\downarrow$（%） | $G\uparrow$（%） |
| --- | --- | --- | --- | --- | --- | --- |
| Meta-Harness | ALFWorld | Qwen3.5-0.8B | $+100.0\%$ | 100 | 0 | 0 |
| Meta-Harness | ALFWorld | Qwen3.5-2B | $+100.0\%$ | 100 | 0 | 0 |
| Meta-Harness | ALFWorld | Qwen3.5-4B | $+60.0\%$ | 93 | 0 | 7 |
| Meta-Harness | ALFWorld | Qwen3-8B | $+64.0\%$ | 100 | 0 | 0 |
| Meta-Harness | ALFWorld | Qwen3.6-35B-A3B$^{\star}$ | $+38.0\%$ | 95 | 0 | 5 |
| Meta-Harness | LiveMath | Qwen3.5-0.8B | $+77.1\%$ | 91 | 9 | 0 |
| Meta-Harness | LiveMath | Qwen3.5-2B | $+40.0\%$ | 64 | 0 | 36 |
| Meta-Harness | LiveMath | Qwen3.5-4B | $+51.4\%$ | 78 | 11 | 11 |
| Meta-Harness | LiveMath | Qwen3-8B$^{\star}$ | $+28.6\%$ | 100 | 0 | 0 |
| Meta-Harness | LiveMath | Qwen3.6-35B-A3B$^{\star}$ | $+20.0\%$ | 57 | 29 | 14 |
| Meta-Harness | CREATE | Claude Haiku 4.5 | $+37.9$ CU | 0 | 49 | 51 |
| Meta-Harness | CREATE | Qwen3.6-35B-A3B | $+7.5$ CU | 0 | 100 | 0 |
| Meta-Harness | SWE-bench | Qwen3-4B-Instruct-2507 | $+27.1\%$ | 23 | 23 | 54 |
| Meta-Harness | SWE-bench | Qwen3-30B-A3B-Instruct-2507 | $+33.3\%$ | 0 | 6 | 94 |
| Meta-Harness | SWE-bench | Qwen3.6-27B$^{\star}$ | $+8.3\%$ | 0 | 0 | 100 |
| Meta-Harness | SWE-bench | Qwen3-Coder-30B-A3B-Instruct$^{\star}$ | $+12.5\%$ | 0 | 0 | 100 |
| SkillOpt | ALFWorld | Qwen3.5-4B | $+46.2\%$ | 44 | 0 | 56 |
| SkillOpt | LiveMath | Qwen3.5-4B | $+45.7\%$ | 88 | 0 | 12 |
| SkillOpt | CREATE | Qwen3.6-35B-A3B | $-0.3$ CU$^{\dagger}$ | – | – | – |
| SkillOpt | CREATE | Claude Haiku 4.5 | $+0.4$ CU$^{\dagger}$ | – | – | – |
| SkillOpt | SWE-bench | Qwen3-4B-Instruct-2507 | $+10.4\%$ | 0 | 0 | 100 |
| SkillOpt | SWE-bench | Qwen3.6-27B | $0^{\dagger}$ | – | – | – |
| AHE | LiveMath | Claude Opus 4.8 | $+48.6\%$ | 100 | 0 | 0 |
| AHE | CREATE | Claude Opus 4.8 | $+6.9$ CU | 0 | 75 | 25 |
| AHE | SWE-bench | Claude Opus 4.8 | $0^{\dagger}$ | – | – | – |

在许多设定中，观测增益的很大份额反映的是过拟合或更多的测试时计算，而不是一个能力更强的执行器。如表 1 所示，在 21 个具有显著搜索集增益的设定中，有 14 个设定里 $O+T$ 占被归因增益的一半以上。在这些设定中，$O$ 与 $T$ 平均分别解释增益的 $54\%$ 与 $14\%$。我们将每个归一化百分比裁剪到区间 $[0\%,100\%]$。占主导地位的来源因基准而异：ALFWorld 与 LiveMath 由过拟合主导，CREATE 由测试时缩放主导，而 SWE-bench 则显示出更清晰的残差可泛化改进。附录 G 在 SWE-bench 上，把一种拟合于基准的补丁大小启发式与一种可复用的动作反馈护栏加以对照。

例如，在 ALFWorld 上，Meta-Harness 使用 Opus 4.8 作为提议者，分别为从 Qwen3.5-0.8B 到 Qwen3.6-35B-A3B 的五个执行器各自优化 harness。五个执行器在演化之后都达到 $100\%$ 的搜索集成功率，而 HDA 将它们增益的 $98\%$ 归因于过拟合。如图 3 所示，较小模型（0.8B、2B）的 harness 变成不进行 LLM 调用的、基于规则的控制器，而较大模型（4B、35B-A3B）的 harness 则通过提示、模板与确定性规则，编码类似的、特定于基准的知识。

CREATE 提供了测试时缩放的一个代表性例子。它通过要求模型产生一组多样且高质量的响应来评估联想创造力；其 Creative Utility（创造性效用，CU）度量奖励那些包含强且多样的答案的、更大的答案池 (Wadhwa et al., 2026)。如表 1 所示，在 Meta-Harness 下，计算量匹配的基线为 Claude Haiku 4.5 恢复了增益的 $49\%$，甚至优于演化后的 Qwen3.6-35B-A3B harness。类似地，在 AHE 中，测试时缩放贡献 $+5.1$ CU，占观测到的演化增益的 $75\%$。相比之下，SkillOpt 只编辑技能文件，同时保持控制流固定，并且我们观察到 $H_{0}^{\mathrm{scale}}=H_{0}$ 以及 $T=0$。然而，SkillOpt 的两个 CREATE 设定都没有产生统计显著的增益。总体而言，在编排代码可以被灵活更新的框架中，增益更为可观，但也更容易出现测试时缩放。我们在附录 F 中展示示例 harness。

#### 不同框架重新发现相同的过拟合与缩放策略

LiveMath 是一个面向研究级数学推理的选择题基准。在 35 道搜索集题目中的 21 道里，有一个选项是同一句元层面陈述：“one of the remaining options is correct, but a stronger result can be proven.”（其余选项之一是正确的，但可以证明一个更强的结果。）该选项在全部 21 个情形中都是正确的。

我们的 HDA 分析表明，三个框架都独立地发现了这一捷径。Meta-Harness 实现了一个确定性的字符串匹配分支来捕捉该选项，SkillOpt 把该选项变成一条默认决策规则，AHE 则加入一条显式指令，要求在不求解该问题的情况下选择它。同一条捷径出现在代码、技能与提示这几种演化之中，而不是某一个框架所特有的。我们在附录 E 中展示演化后 harness 的片段。

类似地，在 ALFWorld 上，Meta-Harness 与 SkillOpt 编码了相似的任务配方、器具映射与物体位置先验，但分别把它们表示为可执行控制器与自然语言技能。在 CREATE 上，Meta-Harness 与 AHE 都诉诸缩放策略，以提高答案的潜在覆盖度与多样性。更多细节见附录 D。

此类由捷径驱动的增益，使得人们难以评估 harness 演化框架的表现，也难以在此类框架之间进行公平比较。这表明需要丰富且低工件的基准。

#### 搜索集增益常常无法泛化，即便验证信号也不足以阻止过拟合

我们在留出测试集上直接检验可泛化性。如图 4 所示，留出增益总体上小于训练增益。排除我们将在下文 §5 中讨论的 ALFWorld，在 15 个设定中的 13 个里，留出增益小于搜索集增益，并且在 8 个设定中不到搜索集增益的一半。在 4 个设定中，演化后的 harness 在测试集上的表现甚至差于基线。这些结果证实，很大的训练增益常常来自过拟合，而不是可泛化改进。

在观察到大量过拟合之后，我们接下来追问：使用一种标准保障——留出验证信号——能够多容易地减少过拟合。因此，我们为 Meta-Harness 增加基于验证的选择。然后，我们在留出测试集上评估并比较训练集最优 harness（$\hat{H}$）与验证集最优 harness（$\hat{H}^{\mathrm{val}}$）。

如图 5 所示，验证选择使平均留出表现在 ALFWorld 上提高 $8.0$ 个百分点，在 LiveMath 上提高 $0.8$ 个点，在 SWE-bench Verified 上提高 $8.5$ 个点，同时使 CREATE 的表现下降 $0.4$ CU。在 16 个模型–基准设定中，它在 10 个设定上提高了留出分数，但在 6 个设定上降低了留出分数。更重要的是，验证选择并没有弥合迁移差距：在 16 个设定中的 14 个里，按验证集选出的 harness 的留出增益，仍然低于按搜索集选出的 harness 的搜索集增益。因此，验证选择在某些设定中有所帮助，但并不能可靠地防止过拟合。因而，验证选择在某些设定中改善了迁移，但要进一步缩小从训练到测试的差距，仍然并非易事。

图 4：搜索集增益并不能可靠地预测留出迁移。对每个设定而言，空心圆是演化后 harness 的搜索集增益（$\Delta S_{search}$），实心圆是其留出增益（$\Delta S_{test}$）；红色标记标出负的留出增益。线段长度表示搜索集上的增益与留出集上的增益之间的差距。菱形表示 $\hat{H}=H_{0}$；$\times$ 标记不显著的增益。单位为 pp，CREATE 除外（单位为 CU）。

图 5：验证选择在 16 个 Meta-Harness 设定上的效果。空心圆表示按搜索集选出的 harness $\hat{H}$ 的搜索集增益，灰色圆表示其留出测试增益，蓝色菱形表示按验证集选出的 harness $\hat{H}^{\mathrm{val}}$ 的留出增益。增益以百分点报告，CREATE 除外，后者使用 Creative Utility（CU）。

#### 为何仅靠留出评估并不充分

图 4 也说明了为何仅靠留出评估会漏掉过拟合。ALFWorld 起初看起来是一个很强的迁移案例：其测试集增益很大，并且接近其搜索集增益。然而 HDA 揭示，harness 演化增益平均有 $89\%$ 来自过拟合：演化后的 harness 可能把确定性的解或配方硬编码下来，这些解或配方利用的是基准实例的构造方式，并且在最极端的情形中甚至消除了 LLM 调用（图 3）。尽管我们有意将若干任务类型与场景排除在搜索集之外，演化后的 harness 在测试集上仍然达到近乎完美的表现。

这恰恰凸显了过拟合的另一个原因：从更强的提议者那里蒸馏出特定于基准的知识。在这个相对广为人知的基准上，强提议者可能已经把相关信息编码在参数化知识之中，而所生成的确定性 harness 覆盖了搜索集之外的实例。因此，这些结果对可泛化改进只能提供有限的证据。在提议者–执行器系统完全不熟悉的任务上，此类配方甚至可能在很小的改动下就失效，因为它们依赖于字符串匹配与固定的基准模式。

LiveMath 表现出类似的局限。同一个元选项工件出现在搜索集、验证集与测试集之中。对于 Meta-Harness 的 0.8B 执行器，在包含该元选项的题目上，测试准确率为 $100\%$（$53/53$），但在其余题目上只有 $31\%$（$22/71$）。因此，正的留出增益所反映的，可能是跨划分共享的工件，而不是更广泛的数学推理。

CREATE 展示了测试时缩放所对应的局限：额外的模型调用可以同时提高搜索集与测试集的分数，但仅靠留出评估并不能把缩放与固定推理预算下的改进区分开来。HDA 通过识别过拟合、测试时缩放以及残差可泛化改进，补充了留出评估。

## 6 相关工作

#### 智能体 harness 与 harness 演化

智能体 harness 由固定模型周围的提示、技能、工具、记忆、上下文与编排逻辑组成 (Weng, 2026)。早期的智能体系统在很大程度上依赖人工的 harness 工程 (Yao et al., 2023; Yang et al., 2024)，而自动化优化的最初方法聚焦于单个组件，尤其是提示 (Khattab et al., 2024; Agrawal et al., 2026)。更为近期的方法把范围扩展到技能 (Ni et al., 2026; Tang et al., 2026; Ke et al., 2026)、可执行的 harness 代码 (Hu et al., 2025; Novikov et al., 2025; Karten et al., 2026; Ning et al., 2026; Zhang et al., 2026d)、工具与工作流 (Zhang et al., 2025; Chen et al., 2025)、记忆与上下文 (Liu et al., 2026; Zhang et al., 2026f; Wu et al., 2026; Ouyang et al., 2026)、环境 (Huang et al., 2026a)，以及更多方面 (Ye et al., 2026; Zhang et al., 2026g; Zhang et al., 2026e)。自演化的智能体框架使用智能体自身来改进其 harness (Zhang et al., 2026b; Zhang et al., 2026a; Zhang et al., 2026c; Luo et al., 2026)。其他框架使用单独的提议者智能体或优化器，依据任务反馈来更新 harness (Lee et al., 2026; Yang et al., 2026)。这些方法在不更新执行器模型权重的情况下报告了可观的增益。我们的工作则研究这些增益从何而来，以及它们是否反映有意义的、可泛化的改进。

#### 评估 harness 优化

一些近期的 harness 演化框架使用基准任务来搜索改进后的 harness，然后在同分布的基准集上报告最终表现 (Wang et al., 2026b; Lin et al., 2026a; Zhang et al., 2026d)。这一协议使得搜索集优化难以与可泛化改进区分开来。Wang et al. (2026b) 观察到，在统一预算下，harness 演化并不能可靠地胜过并行采样或顺序精炼。Huang et al. (2026b) 通过构造相互分离的辅助演化套件与评估套件来应对同一评估关切，这些套件旨在减少特定于任务的过拟合。其他工作研究 harness 收益的不同方面。Lin et al. (2026b) 使用一种原位评估协议，把智能体更新 harness 的能力，与遵循 harness 并从中受益的能力区分开来。关于 RRSI 的同期工作对 harness 的提议与选择加以正则化，以减少过拟合并改善迁移 (Xia et al., 2026)。我们的工作通过把观测到的增益归因于已检测到的过拟合、测试时缩放以及其余部分，来补充这些方法；其余部分给出可泛化改进的上界。

## 7 结论

在本工作中，我们研究 harness 演化的增益从何而来。我们提出 Harness-Delta Attribution（HDA），一种可移植的、半自动化的评估协议，它把观测到的增益归因于过拟合、测试时缩放，以及可泛化改进的残差上界。我们发现，在许多设定中，过拟合或测试时缩放解释了增益的很大份额。我们进一步表明，仅靠留出评估并不足以评估一个演化后 harness 的实际效用。我们建议，未来的 harness 演化研究把 HDA 与留出评估结合起来度量过拟合；并通过在评估期间分析测试时缩放的贡献，以及在演化期间显式表达成本约束，来计入测试时缩放。最后，基准选择对于有意义的评估与比较至关重要：需要丰富、低工件、并且带有有意义分布偏移的基准，以便抑制捷径、奖励可迁移的 harness 改进，并揭示各演化方法之间的差异。

## 致谢

本工作得到 Sloan Foundation、NSF CAREER Award IIS-2145280、NSF 项目 IIS-2433071 的部分资助，并得到 NSF 在合作协议 2421782 项下的资助，以及授予 NSF-Simons AI Institute for Cosmic Origins — CosmicAI（<https://www.cosmicai.org/>）的 Simons Foundation 项目 MPS-AI-00010515 的资助。

## 参考文献

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

## 附录 A 驾驭层演化细节

算法 1 概括了标准的驾驭层演化循环。

**算法 1** 标准驾驭层演化循环

1. 输入：基线驾驭层 $H_{0}$，执行模型 $M$，提议智能体 $\mathcal{P}$，任务集 $\mathcal{X}_{\mathrm{search}}$ 与可选的 $\mathcal{X}_{\mathrm{val}}$，迭代次数 $N$
2. $F_{0}\leftarrow\{\textsc{Evaluate}(H_{0},M,\mathcal{X}_{\mathrm{search}},\mathcal{X}_{\mathrm{val}})\}$
3. 对于 $t=1,\ldots,N$，执行
4. $\mathcal{H}_{t}\leftarrow\textsc{Propose}(\mathcal{P},F_{t-1})$
5. $F_{t}\leftarrow F_{t-1}$
6. 对于所有 $H\in\textsc{Valid}(\mathcal{H}_{t})$，执行
7. $f_{t}(H)\leftarrow\textsc{Evaluate}(H,M,\mathcal{X}_{\mathrm{search}},\mathcal{X}_{\mathrm{val}})$
8. $F_{t}\leftarrow F_{t}\cup\{f_{t}(H)\}$
9. $\hat{H}\leftarrow\textsc{Select}(F_{N},\mathcal{X}_{\mathrm{sel}})$
10. 返回 $\hat{H}$

## 附录 B 实验设置细节

本附录报告实验所用的模型覆盖范围与数据划分。

### B.1 执行模型

我们评估 11 个执行模型：来自 Qwen3、Qwen3.5 与 Qwen3.6 系列的 9 个开放权重模型（Yang 等，2025；Team，2026；Cao 等，2026；Qwen Team，2026），以及 Claude Haiku 4.5 与 Claude Opus 4.8（Anthropic，2025；Anthropic，2026b）。25 组模型–基准–框架设置包括 16 组 Meta-Harness、6 组 SkillOpt 与 3 组 AHE。表 1 给出完整对应关系。

### B.2 基准与划分

对于 LiveMath，我们沿用 SkillOpt 所使用的 35/18/124 训练/验证/测试划分（Yang 等，2026）。所有划分遵循相同的题目构造，性能以准确率衡量。

我们同时沿任务类型与房间两个轴划分 ALFWorld。50 局搜索游戏覆盖厨房、客厅与浴室中的 pick_and_place_simple、pick_clean_then_place_in_recep 与 pick_heat_then_place_in_recep。36 局验证游戏包含全部六种任务类型，但只包含选定的任务类型–房间组合；测试集包含全新且互不相交的游戏，组合覆盖更广。120 局测试游戏包括：搜索任务类型的 25 个新实例、留出卧室中的 20 局拾取并放置游戏，以及 pick_cool_then_place_in_recep、pick_two_obj_and_place 与 look_at_obj_in_light 各 25 局。性能以 50 个环境步之内的回合成功来衡量。

我们按关系类型与领域划分 CREATE。含 120 个条目的搜索划分覆盖五种常见百科关系：职业（occupation）、出生地（place of birth）、受教育于（educated at）、所获奖项（award received）与国籍国（country of citizenship）。含 80 个条目的验证划分包括这些关系的新实例，以及另外四种关系。含 561 个条目的测试集包含：搜索关系的 107 个新实例、来自留出的传记、家庭与职业关系的 263 个实例，以及来自留出的科学与生物医学关系的 191 个实例。性能以创造性效用（Creative Utility，CU）衡量。CU 是一种连续且无上界的指标，用于奖励有效且多样的答案（Wadhwa 等，2026）。

我们将 SWE-bench Verified 划分为 48 个搜索实例、24 个验证实例与 50 个测试实例（Jimenez 等，2024）。这些实例在过滤出可于一小时内解决的任务之后，以种子 42 进行抽样，同时保持各划分之间的仓库分布。性能以成功率衡量。

### B.3 Meta-Harness 的基于验证的信号

对于每个候选驾驭层，搜索集评估会经由共享文件系统，向提议智能体暴露逐实例分数、执行轨迹与日志。我们还会在验证集上评估候选，但只暴露汇总验证分数；逐实例的验证结果与轨迹仍然隐藏。我们从已评估的候选中保留两个驾驭层：搜索分数最高的驾驭层 $\hat{H}$，以及验证分数最高的驾驭层 $\hat{H}^{\mathrm{val}}$。二者都只在演化完成之后才在留出测试集上接受评估。该设置用以检验：标准的验证信号能否在不泄露验证样本或轨迹的前提下改善迁移。

## 附录 C HDA 技能

图 6 概括了分析智能体所使用的 HDA（Harness-Delta Attribution，驾驭层增量归因）技能。完整的机器可读技能包含在补充材料中。

**HDA 技能：流程概要**

输入：基线驾驭层 $H_{0}$、演化后驾驭层 $\hat{H}$，以及一个评估集。

1. 选定 $H_{0}$、$\hat{H}$ 与评估集，并计算 $\Delta S=S(\hat{H})-S(H_{0})$。
2. 规定任务目标、分数聚合方式、推理代价度量，以及任务特定的评估钩子。
3. 实施成对显著性检验。若 $\Delta S$ 在统计上不显著，则将其报告为处于噪声范围内，并且不赋予归因。
4. 构造计算量匹配基线 $H_{0}^{\mathrm{scale}}$，并计算 $T=S(H_{0}^{\mathrm{scale}})-S(H_{0})$。人工检查点 1：审查计算度量与匹配程序。
5. 检测并中和 $\hat{H}$ 中有证据支持的过拟合机制，以构造 $\hat{H}^{\mathrm{neutral}}$，然后计算 $O=S(\hat{H})-S(\hat{H}^{\mathrm{neutral}})$。人工检查点 2：审查证据与中和操作。
6. 计算残差 $G=\Delta S-O-T=S(\hat{H}^{\mathrm{neutral}})-S(H_{0}^{\mathrm{scale}})$，并报告原始分量、归一化百分比、证据与局限性。

解释：$O$ 是已检测过拟合的下界，$G$ 是可泛化改进的上界。留出迁移另行评估。

图 6：HDA 技能概览。该技能将归因程序标准化，同时在两个依赖于任务的决策上保留人工审查：计算量匹配与过拟合中和。

图 7：四个基准上有代表性的演化后驾驭层机制概览（左侧四张卡片，Meta-Harness）：ALFWorld 上硬编码的物体位置先验与任务配方，LiveMath 上的元选项字符串匹配，CREATE 上的并行生成帧（一个测试时扩展的对照情形），以及 SWE-bench 上的补丁规模先验。右侧两张卡片表明，同一个 LiveMath 元选项伪影分别被 SkillOpt（作为技能默认规则）与 AHE（作为显式提示规则）独立地再次发现。红色标出过拟合证据；黄色标出测试时扩展证据；每张卡片上的 $O/T$ 标记，是表 1 中其优胜者被摘录的那一行的归因，而不是基准层面的平均值（ALFWorld 与 LiveMath：Qwen3.5-0.8B；CREATE：Claude Haiku 4.5；SWE-bench：Qwen3-4B-Instruct-2507；SkillOpt：LiveMath Qwen3.5-4B；AHE：LiveMath Claude Opus 4.8）。以下各节（E–G）结合源文摘录，逐一详述每种机制。

## 附录 D ALFWorld：基准特定的程序性支架

在 ALFWorld（Shridhar 等，2021）上，Meta-Harness 与 SkillOpt 编码了相似的、特定于模拟器的知识，包括任务配方、器具映射、物体位置先验与环境规则。图 8 与图 9 各展示一个框架中的代表性驾驭层。红色高亮标出被 HDA 判定为过拟合的机制。

**Meta-Harness · ALFWorld · Qwen3.5-0.8B：零 LLM 控制器** [⬇](<)

```python
LOCATION_PRIORS = {
    ’mug’: [’coffeemachine’, ’countertop’, ’cabinet’, ’shelf’],
    ’tomato’: [’fridge’, ’countertop’, ’diningtable’],
    ’desklamp’: [’desk’, ’sidetable’],
    ...  # object-to-receptacle priors
}
class AdaptiveContainerDepthAgent(AgentHarness):
    @staticmethod
    def _parse_task(task_description: str):
        ...
        if ’cool’ in task:
            process_steps.append((’cool’, ’fridge’))
        elif ’heat’ in task:
            process_steps.append((’heat’, ’microwave’))
        elif ’clean’ in task:
            process_steps.append((’clean’, ’sinkbasin’))
    def step(self, observation, admissible_actions):
        self._update_state(observation, admissible_actions)
        action = self._pick_action(observation, admissible_actions)
        return StepResult(action=action, thought=f"phase={self.phase}")
```

图 8：Qwen3.5-0.8B 的 Meta-Harness ALFWorld 优胜者。高亮的表与解析规则编码了特定于基准的物体位置与任务流程。完整驾驭层中没有 `llm_call`，因此全部动作都来自确定性控制器。HDA：$O/T/G=100/0/0$。

**SkillOpt · ALFWorld · Qwen3.5-4B：作为自然语言技能的支架** [⬇](<)

**任务流程**

- **pick_heat_then_place**：`take <obj> -> go to microwave 1 -> heat <obj> with microwave 1 -> go to <recep> -> place.`
- **pick_cool_then_place**：`take <obj> -> go to fridge 1 -> cool <obj> with fridge 1 -> go to <recep> -> place.`
- **pick_clean_then_place**：`take <obj> -> go to sinkbasin 1 -> clean <obj> with sinkbasin 1 -> go to <recep> -> place.`

**模拟器规则与搜索先验**

若某个动作返回 “Nothing happens”，不要重复它：某一前置条件尚未满足。不存在 terminate 动作。关闭着的橱柜、抽屉、冰箱和微波炉会隐藏其中的物品。对于食物和农产品，应先搜索台面（countertop）、餐桌（dining table）、水槽（sinkbasin）和冰箱（fridge），再搜索橱柜（cabinet）或抽屉（drawer）；生菜任务正是这样失败的。

图 9：Qwen3.5-4B 的 SkillOpt ALFWorld 技能。高亮说明编码了与 Meta-Harness 控制器相同的器具配方、模拟器规则与位置先验，其中包括一条从某次特定搜索集失败中提炼出的规则。HDA：$O/T/G=44/0/56$。

因此，这两个框架重新发现了相似的、特定于 ALFWorld 的流程，但表示方式不同：Meta-Harness 使用可执行的控制器逻辑，SkillOpt 使用自然语言指令。零 LLM 控制器在留出的 ALFWorld 游戏上仍达到 $93.3\%$ 的成功率，因为相同的模拟器本体与规则在各个划分中持续存在，这说明为何仅凭留出性能可能揭示不了特定于基准的过拟合。完整驾驭层包含在补充材料中。

## 附录 E LiveMath：共享的元选项捷径

LiveMath（Mahdavi 等，2025）包含一个反复出现的答案选项：“剩余选项之一是正确的，但可以证明一个更强的结果。”（“one of the remaining options is correct, but a stronger result can be proven.”）它出现在 35 道搜索题中的 21 道里，并且这 21 道全部为正确选项。同一模式出现在 124 道测试题中的 53 道里，并且同样始终正确，因此利用它也会表现为能够迁移。三个框架都发现了这一捷径。图 10 展示每个框架如何编码该捷径；红色高亮标出过拟合机制，黄色高亮标出测试时扩展。

**Meta-Harness · LiveMath · Qwen3.5-0.8B：带多次调用回退的字符串匹配** [⬇](<)

```python
META_MARKERS = [
    "one of the remaining options is correct",
    "stronger result can be proven",
]
def solve(self, item):
    meta_label = self._find_meta(item["choices"])
    if meta_label:
        return SolveResult(predicted_label=meta_label)
    votes = []
    for prompt in generation_prompts:
        generation = self.llm_call(prompt)
        votes.append(self._match_choice(generation))
    return majority_vote(votes)
```

**SkillOpt · LiveMath · Qwen3.5-4B：作为技能规则的同一捷径** [⬇](<)

**选项比较**

- 将诸如 “one of the remaining options is correct, but a stronger result can be proven” 这样的元选项视为一等候选，绝不要把它当作可丢弃项。
- 若最佳具体选项为真但仍可加强，元选项几乎总是胜过它。

**元选项决策规则**

元选项是你的工作默认项。仅当某个具体选项与已知最强结果完全一致、且不能再加入更强的真陈述时，才拒绝它；否则，元选项胜出。

**AHE · LiveMath · Claude Opus 4.8：一条显式提示规则** [⬇](<)

**处理 “meta” 选项**

该元选项是一等候选，而不是填充项。若在最佳具体选项之外还存在严格更强的真陈述，则该元选项是正确的。选择它。

图 10：同一 LiveMath 伪影的三种表示。Meta-Harness 在调用执行器之前直接返回匹配的选项，否则使用多次模型调用（$O/T/G=91/9/0$）。SkillOpt 将其编码为技能文档中的默认决策规则（$88/0/12$）。AHE 将一条显式选择规则写入系统提示（$100/0/0$）。

因此，这一捷径在代码演化、技能演化与提示演化中都会再次出现。由于该伪影在搜索划分与测试划分之间是共享的，LiveMath 内部的留出增益本身并不能证明数学推理能力得到了提升。三个框架的完整驾驭层都包含在补充材料中。

## 附录 F CREATE：由指标驱动的测试时扩展

CREATE 通过其创造性效用（Creative Utility，CU）指标，奖励那些包含许多有效且多样回答的答案池（Wadhwa 等，2026）。在已归因的 CREATE 设置中，HDA 没有发现已检测到的过拟合（$O=0$）；相反，能够修改编排的框架增加了模型调用次数与 token 用量。图 11 展示 Meta-Harness 发现的并行策略、AHE 发现的顺序策略，以及 SkillOpt 固定不变的单次调用控制流。黄色高亮标出测试时扩展机制。

**Meta-Harness · CREATE · Claude Haiku 4.5：并行生成** [⬇](<)

```python
# Six independent first-wave frames
for frame_name, template, temp in wave1_frames:
    response = self.llm_call(messages=[...], temperature=temp)
    all_paths.extend(extract_paths(response))
# Fourteen additional diversity frames
for frame_cfg in _WAVE2_FRAMES:
    response = self.llm_call(messages=[...], temperature=frame_cfg["temp"])
    all_paths.extend(extract_paths(response))
```

**AHE · CREATE · Claude Opus 4.8：顺序扩展** [⬇](<)

你必须生成尽可能多的互不相同且有效的路径。最终答案应当包含许多互不相同的路径（目标为 8 条以上）。

```python
MIN_PATHS = 5
if has_answer_block and n_paths < MIN_PATHS:
    feedback = _build_abstention_feedback(n_paths)
    return HookResult.with_modifications(
        messages=messages + [feedback],
        force_continue=True,  # force another model turn
    )
```

**SkillOpt · CREATE：技能会改变，但单次调用的展开保持固定** [⬇](<)

```python
system = _build_system(skill_content)  # inject the evolved skill
user = _build_user(item)
response, _ = chat_target(
    system=system,
    user=user,
    max_completion_tokens=max_completion_tokens,
)
result["n_turns"] = 1
```

图 11：三个框架在 CREATE 上的编排。Meta-Harness 对每个条目进行 20 次调用，得到 $O/T/G=0/49/51$。AHE 通过强制扩展轮次，把每个条目的平均调用次数从 1.04 提高到 2.46；采用该论文的 $k=3$ 计算量匹配，得到 $0/75/25$。SkillOpt 只能编辑被注入的技能，而展开仍是一次调用；SkillOpt 在 CREATE 上的两项增益在统计上均不显著，因此 HDA 不赋予归因。

因此，当编排可以演化时，计算量匹配基线能够收回相当大一部分增益，甚至超过演化后的 Meta-Harness Qwen3.6-35B-A3B 驾驭层。SkillOpt 提供了互补情形：将演化限制在技能上可以避免额外调用，但两个被评估的设置都没有产生显著增益。完整驾驭层包含在补充材料中。

## 附录 G SWE-bench Verified：混合增益与可复用防护栏

SWE-bench Verified（Jimenez 等，2024）在本研究中呈现出最清晰的残差改进，但其增益仍可能同时混有过拟合与测试时扩展。图 12 对照两个 Meta-Harness 实例。对于 Qwen3-4B-Instruct-2507，演化后的驾驭层把通用编辑防护栏与一条拟合搜索集轨迹的补丁规模规则结合在一起。它的 LLM 调用次数也达到基线的 $1.44\times$，因此 HDA 将其增益归因为 $O/T/G=23/23/54$。而 Qwen3-30B-A3B-Instruct-2507 的驾驭层检测的是一种通用解析失败，其 $O/T/G=0/6/94$。

**Meta-Harness · SWE-bench · Qwen3-4B：通用编辑反馈，外加拟合于基准的补丁包络** [⬇](<)

```python
# General guardrail: make failed edits visible.
_NOOP_LINES = [
    "This command did NOT modify any file.",
    "Do NOT assume the edit worked and do NOT submit.",
]
# Patch-size thresholds inferred from resolved and failed search trajectories.
ADD_T, DEL_T, DUP_T = 12, 8, 3
if adds >= ADD_T or dels >= DEL_T or maxdup >= DUP_T:
    warn("Your accumulated patch is LARGE for a bug fix.")
    warn("Resolved SWE-bench fixes are almost always a FEW lines.")
    warn("Trim the patch before submitting.")
```

**Meta-Harness · SWE-bench · Qwen3-30B-A3B：与内容无关的动作反馈** [⬇](<)

```python
def _count_action_blocks(response):
    edits = len(re.findall(r"‘‘‘edit\b", response))
    commands = len(re.findall(r"‘‘‘(?:bash|sh|mswea_bash_command)\b", response))
    return edits + commands
# The parser executes only the first action block.
if self.dropped_actions > 0:
    observation += (
        "Only the first action block was executed. "
        f"{self.dropped_actions} block(s) were not run. "
        "Re-issue each required action and inspect its actual output."
    )
```

图 12：SWE-bench Verified 上混合增益与残差占优增益的例子。红色标出 Qwen3-4B 驾驭层中源自基准的补丁规模启发式：$+12/-8$ 行阈值与重复阈值编码了搜索集偏向小规模修复的倾向，并可能在其他地方拒绝本属有效的更大补丁。仅禁用这一机制就对应 $O=23\%$。匹配演化后驾驭层更高的 LLM 推理代价，会把基线从 pass@2 提高到 pass@3，并对应 $T=23\%$；余下的 $G=54\%$ 来自编辑效果反馈等机制。Qwen3-30B-A3B 的动作块防护栏不使用任何特定于基准的内容，得到 $O/T/G=0/6/94$。

因此，与其他基准相比，SWE-bench 包含更多可复用的驾驭层改进，但其来源仍然因设置而异。较弱的 Meta-Harness 执行器能够正向迁移，而 Qwen3.6-27B 与 Qwen3-Coder-30B-A3B 尽管搜索集归因为 $0/0/100$，却在留出测试集上出现退步。这证实 $G$ 是上界，而不是对迁移的保证。完整驾驭层以及其他设置包含在补充材料中。
