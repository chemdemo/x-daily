---
title: "Meta-Harness：对模型 harness 的端到端优化"
author: "Yoonho Lee, Roshen Nair, Qizheng Zhang, Kangwook Lee, Omar Khattab, and Chelsea Finn"
source_url: "https://arxiv.org/abs/2603.28052"
arxiv_id: "2603.28052"
arxiv_version: "v1"
published_at: "2026-03-30"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-translation"
unofficial_translation: true
translation_scope: "摘要至结论的全部正文段落、表格、公式、脚注，以及附录散文。代码与提示词保留英文。参考文献保留英文。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名与 CC BY 4.0 许可。"
---

本文是 CC BY 4.0 论文的非官方中文翻译，原作者为 Yoonho Lee, Roshen Nair, Qizheng Zhang, Kangwook Lee, Omar Khattab, and Chelsea Finn，原文见 https://arxiv.org/abs/2603.28052 。改动仅限于语言与 Markdown 格式。

# Meta-Harness：对模型 harness 的端到端优化

Yoonho Lee  所属机构：Stanford  Roshen Nair  所属机构：Stanford  Qizheng Zhang  所属机构：Stanford  Kangwook Lee  所属机构：KRAFTON  Omar Khattab  所属机构：MIT  Chelsea Finn  所属机构：Stanford

## 摘要

大型语言模型（LLM）系统的性能不仅取决于模型权重，还取决于其 harness：即决定存储什么信息、检索什么信息以及向模型呈现什么信息的代码。然而，harness 至今仍主要靠手工设计，而现有文本优化器与这一设定匹配不佳，因为它们对反馈的压缩过于激进：它们是无记忆的，仅以标量分数为条件，或将反馈限制为简短模板或摘要。我们提出 Meta-Harness，一种在 LLM 应用的 harness 代码上进行搜索的外层循环系统。它使用一个智能体式提议器，该提议器通过文件系统访问所有先前候选的源代码、分数与执行轨迹。在在线文本分类上，Meta-Harness 比一个最先进的上下文管理系统高出 7.7 个百分点，同时所用上下文 token 少 $4\times$。在检索增强的数学推理上，单个被发现的 harness 在五个留出模型上使 200 道 IMO 级别题目的准确率平均提高 4.7 个百分点。在智能体式编程上，被发现的 harness 在 TerminalBench-2 上超越了最佳的手工构建基线。这些结果共同表明，对先前经验更丰富的访问能够使 harness 工程实现自动化。

---

项目页面（含交互式演示）：<https://yoonholee.com/meta-harness/>

优化后的 harness：<https://github.com/stanford-iris-lab/meta-harness-tbench2-artifact>

图 1：（左）在文本分类上，Meta-Harness 优于先前最佳的手工设计 harness（ACE）以及现有文本优化器（TTT-Discover、OpenEvolve），仅经过 4 次评测即达到次优方法的最终准确率。（右）在 TerminalBench-2 上，Meta-Harness 优于所有已报告的 Claude Haiku 4.5 harness。

## 1 引言

改变固定大型语言模型（LLM）周围的 harness，可以在同一基准上产生 $6\times$ 的性能差距 [47]。Harness——即决定存储什么、检索什么以及向模型展示什么的代码——其重要性往往不亚于模型本身。这种敏感性促使人们对 harness 工程的兴趣日益增长。Harness 工程是通过打磨 LLM 周围的代码来提升整个系统性能的实践 [36; 21; 10; 9]。尽管它如此重要，harness 工程在很大程度上仍是手工的：实践者检查失败、调整启发式，并在少量设计上迭代。在本文中，我们追问这一过程本身能否被自动化。

一个自然的起点是近期关于文本优化的工作，因为 harness 工程同样涉及利用先前尝试的反馈来迭代改进文本与代码产物 [38; 39; 35; 26; 1]。然而，这些方法与 harness 工程匹配不佳，因为它们通常在短视野或被严重压缩的反馈下运行：有些仅以当前候选为条件 [31; 51; 53]，另一些主要依赖标量分数 [35; 12]，还有一些把反馈限制为简短模板或由 LLM 生成的摘要 [1; 26]。这是一种出于可扩展性的务实选择，并不能作为更长程依赖不提供信息的证据。Harness 作用于长视野：关于存储什么、何时检索它、或如何呈现它的单个选择，都可能在许多推理步骤之后影响行为。被压缩的反馈常常去掉把下游失败追溯到更早 harness 决策所需的信息。在若干有代表性的文本优化器所研究的任务上，每个优化步骤可用的上下文仅为 100 到 30,000 个 token（表 1），远低于 harness 搜索的诊断足迹。更广泛地，关于检索与记忆增强语言模型的工作表明，有用的上下文往往应当被自适应地访问，而不是被整块塞进单个提示 [28; 48; 37; 56]。

图 2：Meta-Harness 搜索循环。（1）一个智能体读取包含所有先前候选的源代码、执行轨迹与分数的文件系统，并提议一个新的 harness。（2）我们在评测任务上评估所提议的 harness。（3）所有日志（所提议的代码、推理轨迹、评测分数）被存入文件系统中的一个新目录，然后循环重复。

| 方法 | 历史 | 日志内容 | MTok/iter |
| --- | --- | --- | --- |
| OPRO [51] | 窗口 | 过去的（解, 分数）对 | 0.002 |
| TextGrad [53] | 上一次 | 针对当前产物的文本反馈 | 0.015 |
| AlphaEvolve [35] | 窗口 | 程序数据库 + 评测分数 | 0.022 |
| GEPA [1] | 摘要 | 来自 rollout 轨迹的反思性反馈 | 0.008 |
| Feedback Descent [26] | 摘要 | 比较 + 文本反馈 | 0.012 |
| TTT-Discover [55] | 窗口 | 先前解的片段 | 0.026 |
| Meta-Harness | 全部 | 全部日志与分数 | 10.0 |

表 1：文本优化方法及其设定的比较。每一行表示一个已跨任务合并的方法。Mtok/iter 是我们对每篇论文所考虑的最大设定中、对一个文本产物做一次评测所产生的完整上下文的最佳估计。本文所考虑的设定，在每次产物评测上产生高出若干数量级的上下文。

我们用 Meta-Harness 应对这一局限。Meta-Harness 是一个智能体式 harness，经由端到端搜索来优化 harness（图 2）。其提议器是一个编程智能体，即一个基于语言模型、能够调用开发者工具并修改代码的系统。选择编程智能体（而不是原始 LLM）之所以重要，是因为经验量很快就会超出上下文限制，因此提议器必须决定检查什么，并通过与代码库的直接交互来验证编辑。其关键设计选择是通过文件系统暴露完整历史，从而能够对原始的先前代码与执行轨迹做选择性诊断，而不是从按候选压缩后的摘要出发进行优化。对于每一个先前的候选 harness，文件系统都存储源代码、评测分数与执行轨迹，提议器通过 grep 和 cat 等标准操作检索它们，而不是把它们作为单个提示吞入。在实践中，在我们要求最高的设定里，提议器每次迭代读取文件数的中位数为 82，并且每一步引用超过 20 个先前候选（附录 A）。在我们所研究的设定中，单次评测最多可以产生 10,000,000 个 token 的诊断信息，大约比先前文本优化设定中所使用的最大反馈预算高出三个数量级（表 1）。

我们在在线文本分类、数学推理与智能体式编程上评估 Meta-Harness。在在线文本分类上，Meta-Harness 所发现的 harness 比 Agentic Context Engineering（ACE，Zhang et al. [59]）高出 7.7 个百分点，同时使用的上下文 token 少 $4\times$，并且仅用四次提议就达到了次优文本优化器在 60 次提议之后的最终性能（图 1）。在检索增强的数学推理上，单个被发现的 harness 在五个留出模型上使 200 道 IMO 级别题目的准确率平均提高 4.7 个百分点。在 TerminalBench-2 上，所发现的 harness 超越 Terminus-KIRA，并在所有 Haiku 4.5 智能体中排名第 1。

## 2 相关工作

从较高层面看，Meta-Harness 把更广泛文献中关于信用分配与元学习的思想 [40; 46; 3; 17; 44; 2] 带入由编程智能体近期进展所开启的一种新机制之中。该系统并不更新模型权重，而是在 harness 层面分配信用：它利用过去 rollout 的经验，有意识地推理哪些步骤与组件对失败负有责任，然后改写支配未来行为的外部代码。更具体地说，该方法位于若干近期研究线索的交汇处；它与关于外部上下文的自适应访问、可执行代码搜索以及文本优化的工作关系最为直接。

外部记忆与自适应访问。若干先前工作指出，把大型知识源或长输入当作语言模型自适应访问的外部资源，而不是在单次遍历中消费它们，是有益的。具体而言，检索增强生成 [28]、交错的检索与推理 [48]、基于记忆的智能体 [37]，或递归语言模型 [56]，都是对外部上下文进行自适应访问的机制。Meta-Harness 使用类似的访问模式，但处于要求更高的 harness 工程设定中：提议器有选择地检查由代码、分数与执行轨迹构成的大量外部历史，以改进上下文管理过程本身。

可执行代码搜索。近期方法在可执行代码上搜索函数、工作流或智能体设计。早期工作提议在演化程序搜索中把大模型用作变异与交叉算子 [27]。此后的方法在固定的程序脚手架内演化指定的函数 [39]，使用元智能体依据先前发现来编写新智能体 [20]，或在智能体系统的工作流图上搜索 [58]。另一条工作线索为持续学习智能体搜索记忆设计，其中记忆跨任务流持续存在 [57; 50]。与之相对，Meta-Harness 搜索领域特定的 harness，包括提示构造、检索以及在任务之间重置的状态更新策略。其外层循环被刻意保持为最小：它不依赖固定脚手架、先前发现的档案或持久记忆机制，而是给予提议器对先前经验不受限制的文件系统访问。这使智能体能够决定检查哪些信息，并使得搜索可以覆盖完整的 harness 实现，而不是一个预定义的上下文管理过程空间。

文本优化方法。Meta-Harness 也与 ProTeGi、TextGrad、OPRO、GEPA、AlphaEvolve/OpenEvolve 以及 Feedback Descent 等方法密切相关；这些方法利用先前尝试的反馈，迭代改进提示或其他文本产物 [38; 31; 53; 51; 1; 35; 43; 26]。然而，这些方法不太适合 harness 工程。在该设定中，优化目标是一个完整的可执行过程，而相关的环境反馈分布在代码、分数与执行轨迹之中，其方式难以预先概括。Meta-Harness 中的提议器并非只对聚合分数或摘要作出反应，而是能够在失败样例及其执行轨迹上进行推理，从而提出有针对性的编辑。那些论文与本文所考虑的问题规模比较见表 1；在我们的问题设定中与 OpenEvolve、GEPA 和 TTT-Discover 的直接比较见图 1 与图 4。

## 3 Meta-Harness：用于优化 harness 的 harness

本节描述 Meta-Harness，即我们用于在任务特定 harness 上搜索的外层循环过程。Meta-Harness 建立在如下想法之上：harness 优化受益于允许提议器经由文件系统访问、有选择地检查先前代码与执行轨迹，而不是从有损摘要或额外的手工设计搜索结构出发进行优化。在较高层面上，它反复提议、评测并记录新的 harness。

Meta-Harness 本身在广义上就是一个 harness（名称即由此而来），因为它决定提议器模型在搜索期间看到什么信息。除非另有说明，我们用 _harness_ 指代正在被优化的任务特定程序。

目标。Harness 是一个有状态的程序，它包装一个语言模型，并决定模型在每一步看到什么上下文。目标很简单：找到使底层模型在目标任务分布上表现最好的 harness。形式上，令 $M$ 表示一个固定的语言模型，$\mathcal{X}$ 表示一个任务分布。对于 harness $H$ 与任务实例 $x\sim\mathcal{X}$，我们执行一条 rollout 轨迹 $\tau\sim p_{M}(H,x)$。该 harness 为 $M$ 构造提示，模型作出响应，harness 在每次交互之后更新其状态。任务特定的奖励函数 $r(\tau,x)$ 为该轨迹评分。Harness 优化的目标，是找到最大化期望最终奖励的 harness：

$$H^{*}=\arg\max_{H}\mathbb{E}_{x\sim\mathcal{X},\tau\sim p_{M}(H,x)}\;r(\tau,x),$$

当多个目标相关时（例如准确率与上下文成本），我们在 Pareto 支配下评估候选，并报告所得前沿。在实践中，这一搜索传统上由人类工程师与研究者完成，他们手工迭代地打磨提示、上下文管理规则与工具使用逻辑。

Meta-Harness 搜索循环。Meta-Harness 使用单一的编程智能体提议器，该提议器可以访问一个不断增长的文件系统 $\mathcal{D}$，此文件系统充当其反馈通道[^1]。此处，编程智能体是一个基于语言模型的系统，它能够调用开发者工具并修改代码。与把改进逻辑外化到手工设计的搜索循环之中的先前系统不同，Meta-Harness 把诊断与提议委托给编程智能体本身：它决定检查哪些先前产物、应对哪些失败模式，以及是做局部编辑还是更大幅度的重写。等价地，提议器并不是在由外层循环组装的固定提示上运行的原始下一 token 模型；它是一个智能体，把检索信息、浏览先前产物以及编辑代码作为搜索本身的一部分。每个被评测的 harness 贡献一个目录，其中包含其源代码、分数与执行轨迹（例如提示、工具调用、模型输出与状态更新）。该文件系统通常远大于提议器的上下文窗口，因此提议器通过 grep 和 cat 等终端工具查询它，而不是把它作为单个提示吞入。在每次迭代中，提议器首先检查先前的代码、分数与执行轨迹，然后在生成新的 harness 之前对可能的失败模式进行推理。

[^1]: 基于更早的探索，我们认为这一工作流直到最近才变得实用，这发生在 2026 年初前后编程智能体能力出现大幅提升之后。

Meta-Harness 维护一个种群 $\mathcal{H}$ 以及已评测 harness 上的 Pareto 前沿，但不施加父代选择规则：提议器在提议新 harness 时，可以自由检查任何先前的 harness 及其执行轨迹。我们以固定的迭代次数运行演化，并在 Pareto 前沿上进行最终的测试集评测。这种简单性是刻意的：把诊断与编辑决策留给提议器，而不是把搜索启发式硬编码进去，Meta-Harness 就能随着编程智能体变得更强而自动改进。提议器从不会看到测试集结果；它唯一的反馈来自搜索集，即在搜索期间用于评测候选 harness 并产生改进反馈信号的那一部分任务实例，以及那些搜索运行期间所记录的执行轨迹。

代码空间搜索的优势。Harness 优化发生在代码空间中。在这里，对检索、记忆或提示构造逻辑的微小改动都可能在许多步骤之后影响行为，这使得局部搜索启发式与该问题匹配不佳。通过检查执行轨迹，提议器常常能够推断一个 harness *为何* 失败，以及哪些更早的设计选择很可能促成了这次失败，而不仅仅是它 *确实* 失败了；附录 A 与 A.2 中的搜索轨迹说明了这一点。在那里，我们看到提议器广泛阅读先前的代码与日志，然后利用这些轨迹识别出混杂的编辑、分离出可能的因果改动，并在反复出现回归之后转向更安全的修改。因此，提议器可以在算法结构的层面上修改 harness，范围从对检索、记忆或提示构造逻辑的改动，直到完整的程序重写，而不是填写模板或应用预定义的变异算子。在实践中，它常常从一个较强的先前 harness 出发，但这是一种涌现出来的策略，而不是硬编码的规则。尽管搜索空间很大，把 harness 表示为程序提供了一种自然的正则化偏置：编程模型倾向于提议连贯的算法，而不是脆弱的硬编码解，从而把搜索偏向可复用的上下文管理过程。这一动作空间与前沿编程助手所受训练的读—写—执行工作流紧密对齐。

实践实现。在我们的实验中，每个 harness 都是一个单文件 Python 程序，它修改任务特定的提示、检索、记忆与编排逻辑。在我们的实验中，提议器 $P$ 是使用 Opus-4.6 的 Claude Code [5]。提议器由一份最小的领域特定技能引导，该技能描述新 harness 应写到何处、如何检查先前的 harness 及其执行轨迹，以及哪些文件可以修改、哪些不可以。基座模型 $M$ 随领域而变化，并且始终被冻结；细节见第 4 节。在我们的实验中，一次典型运行在 20 次迭代中大约评测 60 个 harness。我们在附录 D 中给出了在新领域实现 Meta-Harness 的额外建议。

算法 1　Meta-Harness 在 harness 上的外层循环

```text
1: Input: tasks $\mathcal{X}$, LLM $M$, proposer $P$, iterations $N$

2: Initialize: population $\mathcal{H}$  $\triangleright$ Initial set of valid harnesses

3: Initialize: filesystem $\mathcal{D}\leftarrow\emptyset$  $\triangleright$ stores code, scores, traces

4: for $H\in\mathcal{H}$ do

5: $E_{H}\leftarrow\textrm{Evaluate}(H,M,\mathcal{X})$

6: $\mathcal{D}\leftarrow\mathcal{D}\cup\{(H,E_{H})\}$

7: for $t=1\ldots N$ do

8: Proposer $P$ queries filesystem $\mathcal{D}$  $\triangleright$ inspects prior harnesses and scores

9: Proposer $P$ proposes $k$ new harnesses $\{H_{1},\dots,H_{k}\}$

10: for $H$ in $\{H_{1},\dots,H_{k}\}$ do

11: if $H$ passes interface validation then

12: $\mathcal{D}\leftarrow\mathcal{D}\cup\{(H,\textsc{Evaluate}(H,M,\mathcal{X}))\}$

13: return Pareto frontier of harnesses stored in $\mathcal{D}$
```

## 4 实验

我们在三个任务领域上评估 Meta-Harness：在线文本分类、数学推理与智能体式编程。在每个领域中，我们使用标准评测指标，把我们的搜索所发现的 harness 与适合该领域的基线进行比较。精确的实验设置请参见各小节。

我们与两大类方法进行比较。（1）人类设计的策略：这些是针对每个领域手工制作的 harness，代表上下文构造方面的当前最先进水平。我们在相应小节中描述这些基线。（2）程序搜索方法：这些方法利用反馈与奖励信号在候选 harness 上搜索，但其设计所针对的规模小于 harness 工程。

### 4.1 在线文本分类

我们遵循 Zhang et al. [59] 与 Ye et al. [52] 的在线文本分类设定：一个 LLM 一次接收一个带标签样例，更新其记忆，并在留出的测试集上接受评测。我们使用 GPT-OSS-120B 作为 LLM 文本分类器，并考虑为文本分类设计 harness 的问题。我们使用三个数据集，选择它们是出于难度与领域多样性：LawBench（Law）[16] 根据案情描述预测刑事罪名（215 个类别）；Symptom2Disease（S2D）[19] 根据症状描述预测疾病（22 个类别）；USPTO-50k [41] 根据产物分子预测前体反应物（180 个类别）。我们用该设定中的主要基线 harness 初始化搜索种群 $\mathcal{H}$：零样本、少样本、ACE 与 MCE。我们运行了 20 次演化迭代，每次迭代两个候选，产生 40 个候选 harness。

|  | 数据集 | 数据集 | 数据集 | 平均 | 平均 |
| --- | --- | --- | --- | --- | --- |
| Harness | USPTO | S2D | Law | 准确率 | Ctx $\downarrow$ |
| 零样本 | 12.0 | 63.2 | 7.0 | 27.4 | 0 |
| 少样本（8） | 14.0 | 67.9 | 21.0 | 34.3 | 2.0 |
| 少样本（32） | 13.0 | 72.2 | 21.0 | 35.4 | 7.9 |
| 少样本（全部） | 15.0 | 78.3 | 29.0 | 40.8 | 12.3 |
| MCE [52]† | 14.0 | 83.0 | 23.0 | 40.0 | 28.5 |
| ACE [59]† | 16.0 | 77.8 | 29.0 | 40.9 | 50.8 |
| Meta-Harness | 14.0 | 86.8 | 45.0 | 48.6 | 11.4 |

表 2：所有 harness 在三个数据集上的测试集指标。Ctx 表示上下文中的额外输入 token（以千计）。†：实现来自 Ye et al. [52]。$\downarrow$：越低越好。Meta-Harness 在使用更小输入上下文的同时，提高了在线文本分类准确率。

图 3：在线文本分类上准确率与上下文 token 的 Pareto 前沿。Meta-Harness 取得了比所有比较方法更强的准确率–上下文 Pareto 前沿。

与文本优化器的比较。我们将 Meta-Harness 与有代表性的文本优化方法进行比较。为了公平比较，我们使用相同的提议器配置（Opus-4.6，最大推理），仅依据搜索集性能选择候选，并把测试集一直留到最终评测。由于评测是主要的计算瓶颈，我们给予每种方法相同的提议 harness 评测预算。我们考虑以下比较对象：

- Best-of-N：从种子独立采样，不含搜索结构；这是一个计算量匹配的对照，用于检验搜索本身是否要紧。

- OpenEvolve [43]：以 LLM 变异对程序进行的演化搜索。

- TTT-Discover [54]：我们只使用其方法中的文本优化组件，即经由 PUCT 复用规则进行的提议选择。

在这一设定中，Meta-Harness 以 $0.1\times$ 的评测次数即达到先前最佳文本优化器（OpenEvolve、TTT-Discover）的水平，且其最终准确率超出它们 10 个百分点以上（图 1 与图 4）。我们把这一加速归因于有意的设计选择：它们对外层循环只施加最少的必要结构（第 3 节）。具体而言，Meta-Harness 用文件系统保留完整的经验历史，并允许提议器检查任何必要的内容；OpenEvolve 与 TTT-Discover 则以更为结构化、并且比完整文件系统访问受限得多的提议器输入来运行。我们注意到，在线文本分类是我们所研究的上下文最小的设定（表 1），因此如果结构较重的文本优化器在这里已经落后，它们的局限在更难的设定中可能只会进一步增大。

**Meta-Harness 快 $10\times$，并收敛到更好的 harness。** 在这一设定中，Meta-Harness 以少 $10\times$ 的完整评测即达到先前最佳文本优化器（OpenEvolve、TTT-Discover）的水平，且其最终准确率超出它们 10 个百分点以上。

| 方法 | 分数 | 代码 | 摘要 | 轨迹 | 中位数 $\uparrow$ | 最佳准确率 $\uparrow$ | >> ZS |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 仅分数 | ✓ | ✓ | × | × | 34.6 | 41.3 | 26 |
| 分数 + 摘要 | ✓ | ✓ | ✓ | × | 34.9 | 38.7 | 23 |
| Meta-Harness（完整） | ✓ | ✓ | - | ✓ | 50.0 | 56.7 | 39 |

表 3：在线文本分类中提议器可用信息的消融。>> ZS：准确率超过零样本基线的运行次数。完整的 Meta-Harness 接口大幅优于仅分数以及分数加摘要的消融。对原始执行轨迹的访问，是使 harness 搜索得以进行的关键成分。

为了分离提议器接口中哪些部分最为重要，我们在在线文本分类中比较三种条件：仅分数条件；分数加摘要条件，其中提议器收到由 LLM 生成的摘要但没有原始轨迹；以及能够访问执行轨迹的完整 Meta-Harness 接口（表 3）。结果显示，完整接口具有很大优势：仅分数达到中位数 34.6、最佳准确率 41.3，分数加摘要达到中位数 34.9、最佳 38.7。相比之下，Meta-Harness 达到中位数 50.0、最佳准确率 56.7，甚至其中位数候选也优于任一消融下找到的最佳候选。我们将此解释为如下证据：对执行轨迹的完整访问是该接口最重要的组成部分；摘要并不能恢复缺失的信号，甚至可能因为压缩掉具有诊断价值的细节而有害。

| 方法 | 中位数 | 最佳 |
| --- | --- | --- |
| GEPA [1] | 32.6 | 40.2 |
| Best-of-N | 34.0 | 44.2 |
| OpenEvolve [43] | 39.1 | 43.3 |
| TTT-Discover [54] | 34.1 | 45.6 |
| Meta-Harness | 50.0 | 56.7 |

表 4：不同文本优化器所提议 harness 的文本分类准确率（搜索集）。Meta-Harness 在 harness 优化上显著更为有效。

与最先进 harness 的比较。我们的主要比较对象是针对这一问题设定手工设计的 harness：Agentic Context Engineering（ACE，Zhang et al. [59]）使用反思性记忆策展来随时间构建上下文；Meta Context Engineering（MCE，Ye et al. [52]）维护并演化一个用于上下文构造的自然语言技能库。作为额外基线，我们评测零样本提示，以及样例数为 $N\in\{4,8,16,32,\text{all}\}$ 的少样本提示。表 2 的结果表明，Meta-Harness 相对先前手工设计的 harness 有大幅提升。被选中的 Meta-Harness[^2] 达到 48.6% 的准确率，超出 ACE 7.7 个百分点，超出 MCE 8.6 个百分点。这些收益并非来自使用更多上下文：Meta-Harness 只使用 11.4K 个上下文 token，ACE 为 50.8K，MCE 为 28.5K。

[^2]: 为简洁起见，我们略微重载了术语：在表格中，Meta-Harness 表示被发现的最佳 harness，而在其他地方它指整个 harness 搜索过程。

准确率–上下文权衡。由于 Meta-Harness 对 harness 代码进行自由形式的优化，我们可以表达对准确率与上下文成本二者的联合偏好，而不必事先承诺单一的标量目标。仅给定当前指标与所期望的权衡，提议器就能够在前沿的宽广范围内发现 harness，从而在图 3 中得到一条平滑的准确率–上下文 Pareto 曲线。这使我们能够以受控方式用额外的上下文换取更高的测试准确率，而不必承诺单一的手工设计工作点。

分布外（OOD）任务评测。我们评估被发现的 harness 能否泛化到搜索期间完全未见过的全新数据集。我们考虑九个多样化的数据集，并在第 C.1 节中详细描述它们。被选中的 Meta-Harness 系统取得最佳平均准确率（73.1%），优于 ACE（70.2%）以及所有少样本基线（表 5）。值得注意的是，我们观察到，在 32 个样例之外天真地增加更多少样本样例，会在 $7/9$ 个任务上损害性能。Meta-Harness 在 $6/9$ 个数据集上表现最高，这表明被发现的 harness 捕捉到了对文本分类普遍有效的策略，而不是过拟合到搜索期间所使用的特定数据集。

| Harness | SciC | FiNER | Amz5 | FPB | GoEmo | Bank77 | News | SciT | TwHate | 平均准确率 | Ctx $\downarrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 零样本 | 32.7 | 56.0 | 52.7 | 90.0 | 42.0 | 80.7 | 84.7 | 89.3 | 75.3 | 67.0 | - |
| 少样本（8） | 34.0 | 63.0 | 54.0 | 90.0 | 44.0 | 82.7 | 84.7 | 91.3 | 76.7 | 68.9 | 2.2 |
| 少样本（32） | 38.7 | 62.0 | 53.3 | 90.7 | 43.3 | 86.0 | 85.3 | 90.7 | 76.7 | 69.6 | 5.2 |
| 少样本（全部） | 35.3 | 61.0 | 50.0 | 93.3 | 42.7 | 80.7 | 84.0 | 90.0 | 76.7 | 68.2 | 7.4 |
| ACE [59] | 40.7 | 74.0 | 48.0 | 96.7 | 44.0 | 83.3 | 86.0 | 90.7 | 68.7 | 70.2 | 11.7 |
| Meta-Harness | 53.3 | 67.0 | 60.0 | 94.0 | 46.0 | 82.7 | 86.7 | 91.3 | 77.3 | 73.1 | 7.3 |

表 5：OOD 文本分类数据集评测。我们报告每个数据集的测试准确率，以及全部九个数据集上额外上下文 token 的平均值。在这 9 个先前未见的任务上，Meta-Harness 超出次优方法 2.9 个百分点。

### 4.2 面向检索增强推理的 harness

我们研究一种对奥林匹克数学求解而言稍非标准的设定：使模型具备从大型语料库中检索样例的能力。原则上有充分理由预期检索会有助于数学推理，因为解答常常共享可复用的证明模式，因此先前的推理轨迹包含模型在推理时可能加以利用的信息。然而，检索尚未成为这一设定中的标准成分，先前工作表明，它在推理密集的数学基准上远不如在更以事实为根基的领域中成功 [42; 49; 6]。困难在于，天真的检索很少能以正确的形式浮现出正确的轨迹。这表明，成功较少取决于加入检索本身，而更多取决于发现正确的检索策略。我们并不手工设计该策略，而是交给 Meta-Harness 一组困难的奥林匹克题目，并允许检索行为本身从搜索中涌现出来。

检索语料库包含来自八个开源数据集的 $\geq$500,000 道已解答题目。我们针对评测基准与搜索集仔细地对其进行了去重与去污染，确认留出题目在我们基于字符串的过滤器下没有精确的前缀匹配，并人工检查了留出样例排名靠前的 BM25 检索结果（第 C.2 节）。我们使用 Meta-Harness，在一个由 250 道奥林匹克难度数学题构成的搜索集（OlympiadBench + Omni-MATH hard）上，以 40 次迭代优化一个 harness，产生 109 个候选检索 harness。我们用该设定中的主要基线 harness 初始化搜索种群 $\mathcal{H}$：零样本、少样本与 ACE。我们使用 GPT-OSS-20B，依据搜索集性能选出单个 harness（第 B.2 节）。我们在取自 IMO-AnswerBench、IMO-ProofBench 与 ArXivMath [30; 6] 的 200 道先前未见的 IMO 级别题目上评测该 harness。除 GPT-OSS-20B 之外，我们还在搜索期间未见过的四个模型上评测同一个检索 harness：GPT-5.4-nano、GPT-5.4-mini、Gemini-3.1-Flash-Lite 与 Gemini-3-Flash。我们遵循先前工作 [30] 的标准评测协议，并报告每道题目在三个样本上平均后的准确率。

结果。表 6 将所发现的 harness 与无检索、使用独立嵌入模型 text-embedding-3-small 的稠密检索、随机少样本提示以及 BM25 检索进行比较。与之相对，Meta-Harness 完全在代码空间中运作，建立在与稀疏基线相同的、基于 BM25 的词法检索栈之上，而不是引入额外的稠密编码器。所发现的检索 harness 在全部五个留出模型上都优于无检索基线，平均增益为 4.7 个百分点。它在平均意义上也达到或超过最强的固定基线，总体上超出 BM25 检索 1.3 个百分点，同时避免了稠密检索与随机少样本提示在若干模型上所观察到的性能回退。

| 方法 | GPT-5.4n | GPT-5.4m | Gem-3.1FL | Gem-3F | GPT-20B | 平均 |
|---|---|---|---|---|---|---|
| 无检索器 | 23.0 | 28.8 | 28.6 | 42.6 | 47.6 | 34.1 |
| 稠密检索（$k=1$） | 27.1 (+4.1) | 24.5 (-4.3) | 31.3 (+2.7) | 42.3 (-0.3) | 46.9 (-0.7) | 34.4 (+0.3) |
| 稠密检索（$k=5$） | 31.1 (+8.1) | 28.3 (-0.5) | 37.1 (+8.5) | 47.2 (+4.6) | 46.7 (-0.9) | 38.1 (+4.0) |
| 随机少样本 | 23.1 (+0.1) | 24.5 (-4.3) | 31.0 (+2.4) | 40.4 (-2.2) | 41.8 (-5.8) | 32.2 (-1.9) |
| BM25 检索 | 30.2 (+7.2) | 29.2 (+0.4) | 32.8 (+4.2) | 46.6 (+4.0) | 48.9 (+1.3) | 37.5 (+3.4) |
| Meta-Harness | 31.7 (+8.7) | 30.4 (+1.6) | 34.9 (+6.3) | 46.3 (+3.7) | 50.6 (+3.0) | 38.8 (+4.7) |

表 6：在 200 道 IMO 级别数学题上的检索增强数学问题求解。我们给出的是每道题三次采样取平均后的 pass@1，括号内为相对基线的绝对提升。所发现的 Meta-Harness 检索策略在全部五个留出模型上都提升了这些 IMO 级别问题的推理表现，相对无检索器平均提高 4.7 个百分点。

**Meta-Harness 提升 IMO 级别数学问题上的推理。** 在检索增强的数学推理中，单个被发现的检索驾驭框架可以迁移到五个留出模型上，相对无检索平均将准确率提高 4.7 个百分点，并在所比较的方法中取得最强的总体平均。

### 4.3 在 TerminalBench-2 上评估智能体编程驾驭框架

TerminalBench-2 [33] 在 89 项具有挑战性的任务上评估 LLM 智能体。这些任务要求在复杂依赖之下进行长程、完全自主的执行，并需要大量领域知识。已有工作表明，智能体驾驭框架的选择对该基准上的表现影响很大。我们从两个较强的开源基线 Terminus 2 [33] 与 Terminus-KIRA [25] 初始化搜索。在本实验中，搜索与最终评估在同一个 89 任务基准上进行。我们将该基准用作一个发现型问题 [55]，其目标是发现一种驾驭框架配置，用以提升在一个困难且受到公开竞争的基准上的表现。这是标准做法：公开的技术记录已经描述了在 TerminalBench 本身上反复进行的、针对该基准的驾驭框架迭代 [18; 34; 25]，而且该基准规模小、评测代价高，若再引入单独的数据划分，会实质性地削弱搜索信号。我们还通过人工检查，以及基于正则表达式、针对任务特定字符串是否泄漏进演化得到的驾驭框架的审计，来检查过拟合。我们指出，尽管所得驾驭框架专门针对 TerminalBench-2 这一设定，但从单条指令出发自主完成困难的长程任务是一项核心能力，而且该基准包含许多令前沿模型与重度工程化驾驭框架仍然难以应对的任务。

| 驾驭框架 | 自动 | 通过率 (%) |
|---|---|---|
| Claude Opus 4.6 | | |
| Claude Code | × | 58.0 |
| Terminus 2 | × | 62.9 |
| Mux | × | 66.5 |
| Droid | × | 69.9 |
| TongAgents | × | 71.9 |
| MAYA-V2 | × | 72.1 |
| Terminus-KIRA | × | 74.7 |
| Capy | × | 75.3 |
| ForgeCode | × | 81.8 |
| Meta-Harness | ✓ | $\mathbf{76.4}$ |
| Claude Haiku 4.5 | | |
| OpenHands | × | 13.9 |
| Claude Code | × | 27.5 |
| Terminus 2 | × | 28.3 |
| Mini-SWE-Agent | × | 29.8 |
| Terminus-KIRA | × | 33.7 |
| Goose | × | 35.5 |
| Meta-Harness | ✓ | $\mathbf{37.6}$ |

表 7：TerminalBench-2 上的通过率。其余结果来自官方排行榜。在这一竞争性任务上，Meta-Harness 在全部 Opus-4.6 智能体中排名第 2，在全部 Haiku-4.5 智能体中排名第 1。

结果。我们在表 7 中报告完整基准上的结果，评估使用两个基座模型：Claude Opus 4.6 与 Claude Haiku 4.5。在 Opus 4.6 上，Meta-Harness 发现了一个通过率达 76.4% 的驾驭框架，超过手工设计的 Terminus-KIRA（74.7%），并在 TerminalBench-2 排行榜的全部 Opus 4.6 智能体中排名第 2。唯一得分更高的 Opus 4.6 智能体是 ForgeCode（81.8%）；然而，我们无法仅凭公开可得的代码复现其所报告的结果，这表明其排行榜分数依赖于已发布仓库之外的组件。在较弱的 Haiku 4.5 模型上，提升更大：Meta-Harness 达到 37.6%，比次优的已报告智能体（Goose，35.5%）高出 2.1 个百分点。TerminalBench-2 是一个正在被激烈争夺的基准，多个团队直接针对它进行优化，因此一种自动搜索方法能够在这一前沿取得收益，对长程文本优化循环而言是令人鼓舞的。

提议器的定性行为。驾驭框架的搜索轨迹有助于解释 Meta-Harness 为何取得这些收益；我们在附录 A 中给出详细总结。在早期迭代中，提议器把看似合理的结构性修复与提示模板编辑结合在一起，并观察到两个候选都出现了退步。它随后明确假设，这些退步受到了共享提示干预的混杂，于是把结构性改动与提示重写隔离开来，并最终转向一种更安全的加性修改，该修改成为这次运行中的最佳候选。这提供了定性证据，表明文件系统访问使提议器能够以足够的细节检视先前经验，从而形成因果假设，并据此修订驾驭框架。

**Meta-Harness 在 TerminalBench-2 上超越手工设计的智能体。** 在 TerminalBench-2 上，Meta-Harness 自动发现的驾驭框架在 Opus 4.6 上超越 Terminus-KIRA，并在全部 Haiku 4.5 智能体中排名第 1。

## 5 讨论

除了胜过现有驾驭框架之外，Meta-Harness 还有若干实用优势。所发现的驾驭框架能够泛化到分布外的分类数据集（表 5），以及数学设定中未见过的基座模型（表 6）。一次搜索运行在数小时的墙钟时间内即可完成，却能产生可读、可迁移的策略，这些策略可以跨模型复用，包括未来更强的模型。代码空间中的过拟合也更容易检查：脆弱的 `if` 链或硬编码的类别映射一经检视便能看见，权重空间中的过拟合则不然。更广泛地说，我们的结果表明，Meta-Harness 的主要优势不只是在代码上进行搜索，而是带有*对先前诊断经验的选择性访问*的搜索。提议器并不局限于标量奖励或固定摘要；它可以检视原始代码、执行轨迹和先前的失败，然后利用这些信息形成并检验关于应当改变什么的假设。第 A.2 节中的定性搜索轨迹直接说明了这一行为。

我们的发现反映了机器学习中反复出现的一种模式 [45]：一旦某个搜索空间变得可及，更强的通用智能体就可以胜过手工设计的方案。未来工作的一个自然下一步，是让驾驭框架与模型权重协同演化，使策略塑造模型所学的内容，反之亦然。尽管我们在三个彼此不同的领域上作了评估，我们的实验表明，驾驭框架搜索可以与一个特别强的编程智能体提议器（Claude Code）配合工作；该效应如何随提议器智能体的不同而变化，更广泛的研究仍留待未来工作。

## 致谢

我们感谢 KRAFTON AI 提供 API 额度支持。本工作得到 OpenAI、KFAS 以及 Schmidt Sciences AI2050 的支持。我们感谢 Anikait Singh 与 Jubayer Ibn Hamid 提供的宝贵反馈与建议，并感谢 Sienna J. Lee 在本工作早期耐心倾听 YL 那些尚未成形的想法。

## 参考文献

  * [1] L. A. Agrawal, S. Tan, D. Soylu, N. Ziems, R. Khare, K. Opsahl-Ong, A. Singhvi, H. Shandilya, M. J. Ryan, M. Jiang, et al. (2025) Gepa: reflective prompt evolution can outperform reinforcement learning.  arXiv preprint arXiv:2507.19457.  Cited by: Appendix E, Table 1, §1, §2, Table 4.
  * [2] E. Akyürek, D. Schuurmans, J. Andreas, T. Ma, and D. Zhou (2023) What learning algorithm is in-context learning? investigations with linear models.  External Links: 2211.15661, [Link](<https://arxiv.org/abs/2211.15661>) Cited by: §2.
  * [3] M. Andrychowicz, M. Denil, S. Gomez, M. W. Hoffman, D. Pfau, T. Schaul, B. Shillingford, and N. De Freitas (2016) Learning to learn by gradient descent by gradient descent.  Advances in neural information processing systems 29.  Cited by: §2.
  * [4] Anthropic and community contributors Agentskills/agentskills.  Note: GitHub repository <https://github.com/agentskills/agentskills>Specification and documentation for Agent Skills, accessed March 27, 2026 Cited by: 1st item.
  * [5] Anthropic (2025) Claude code: an agentic coding tool.  Note: <https://www.anthropic.com/claude-code> Cited by: §3.
  * [6] M. Balunović, J. Dekoninck, I. Petrov, N. Jovanović, and M. Vechev (2025) MathArena: evaluating llms on uncontaminated math competitions.  SRI Lab, ETH Zurich.  External Links: [Link](<https://matharena.ai/>) Cited by: §4.2, §4.2.
  * [7] F. Barbieri, J. Camacho-Collados, L. Neves, and L. Espinosa-Anke (2020) TweetEval: unified benchmark and comparative evaluation for tweet classification.  External Links: 2010.12421, [Link](<https://arxiv.org/abs/2010.12421>) Cited by: 9th item.
  * [8] L. Beurer-Kellner, M. Fischer, and M. Vechev (2023) Prompting is programming: a query language for large language models.  Proceedings of the ACM on Programming Languages 7 (PLDI), pp. 1946–1969.  External Links: ISSN 2475-1421, [Link](<http://dx.doi.org/10.1145/3591300>), [Document](<https://dx.doi.org/10.1145/3591300>) Cited by: Appendix E.
  * [9] B. Böckeler (2026) Harness engineering.  Note: <https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html>martinfowler.com Cited by: §1.
  * [10] C. Bölük (2026) I improved 15 LLMs at coding in one afternoon. only the harness changed..  Note: <https://blog.can.ac/2026/02/12/the-harness-problem/> Cited by: §1.
  * [11] I. Casanueva, T. Temčinas, D. Gerz, M. Henderson, and I. Vulić (2020) Efficient intent detection with dual sentence encoders.  External Links: 2003.04807, [Link](<https://arxiv.org/abs/2003.04807>) Cited by: 6th item.
  * [12] M. Cemri, S. Agrawal, A. Gupta, S. Liu, A. Cheng, Q. Mang, A. Naren, L. E. Erdogan, K. Sen, M. Zaharia, et al. (2026) AdaEvolve: adaptive llm driven zeroth-order optimization.  arXiv preprint arXiv:2602.20133.  Cited by: §1.
  * [13] LangChain Note: Software, released 2022-10-17 External Links: [Link](<https://github.com/langchain-ai/langchain>) Cited by: Appendix E.
  * [14] A. Cohan, W. Ammar, M. van Zuylen, and F. Cady (2019) Structural scaffolds for citation intent classification in scientific publications.  External Links: 1904.01608, [Link](<https://arxiv.org/abs/1904.01608>) Cited by: 1st item.
  * [15] D. Demszky, D. Movshovitz-Attias, J. Ko, A. Cowen, G. Nemade, and S. Ravi (2020) GoEmotions: a dataset of fine-grained emotions.  External Links: 2005.00547, [Link](<https://arxiv.org/abs/2005.00547>) Cited by: 5th item.
  * [16] Z. Fei, X. Shen, D. Zhu, F. Zhou, Z. Han, A. Huang, S. Zhang, K. Chen, Z. Yin, Z. Shen, et al. (2024) Lawbench: benchmarking legal knowledge of large language models.  In Proceedings of the 2024 conference on empirical methods in natural language processing,  pp. 7933–7962.  Cited by: §4.1.
  * [17] C. Finn, P. Abbeel, and S. Levine (2017) Model-agnostic meta-learning for fast adaptation of deep networks.  In International Conference on Machine Learning,  Cited by: §2.
  * [18] ForgeCode (2025) Benchmarks don’t matter.  External Links: [Link](<https://forgecode.dev/blog/benchmarks-dont-matter/>) Cited by: §4.3.
  * [19] Gretel AI (2023) Symptom to diagnosis dataset.  Note: <https://huggingface.co/datasets/gretelai/symptom_to_diagnosis>Accessed: 2026-01-22 Cited by: §4.1.
  * [20] S. Hu, C. Lu, and J. Clune (2025) Automated design of agentic systems.  In The Thirteenth International Conference on Learning Representations,  External Links: [Link](<https://openreview.net/forum?id=t9U3LW7JVX>) Cited by: §2.
  * [21] A. Justin Young (2025) Effective harnesses for long-running agents.  Note: <https://anthropic.com/engineering/effective-harnesses-for-long-running-agents>Anthropic Engineering Blog Cited by: §1.
  * [22] P. Keung, Y. Lu, G. Szarvas, and N. A. Smith (2020) The multilingual amazon reviews corpus.  External Links: 2010.02573, [Link](<https://arxiv.org/abs/2010.02573>) Cited by: 3rd item.
  * [23] O. Khattab, A. Singhvi, P. Maheshwari, Z. Zhang, K. Santhanam, S. Vardhamanan, S. Haq, A. Sharma, T. T. Joshi, H. Moazam, H. Miller, M. Zaharia, and C. Potts (2023) DSPy: compiling declarative language model calls into self-improving pipelines.  External Links: 2310.03714, [Link](<https://arxiv.org/abs/2310.03714>) Cited by: Appendix E.
  * [24] T. Khot, A. Sabharwal, and P. Clark (2018) SciTaiL: a textual entailment dataset from science question answering.  Proceedings of the AAAI Conference on Artificial Intelligence 32 (1).  External Links: [Link](<https://ojs.aaai.org/index.php/AAAI/article/view/12022>), [Document](<https://dx.doi.org/10.1609/aaai.v32i1.12022>) Cited by: 8th item.
  * [25] KRAFTON AI and Ludo Robotics (2026) Terminus-kira: boosting frontier model performance on terminal-bench with minimal harness.  External Links: [Link](<https://github.com/krafton-ai/kira>) Cited by: §B.3, §4.3.
  * [26] Y. Lee, J. Boen, and C. Finn (2025) Feedback descent: open-ended text optimization via pairwise comparison.  In arXiv preprint arXiv:2511.07919,  Cited by: Table 1, §1, §2.
  * [27] J. Lehman, J. Gordon, S. Jain, K. Ndousse, C. Yeh, and K. O. Stanley (2022) Evolution through large models.  External Links: 2206.08896, [Link](<https://arxiv.org/abs/2206.08896>) Cited by: §2.
  * [28] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal, H. Küttler, M. Lewis, W. Yih, T. Rocktäschel, et al. (2020) Retrieval-augmented generation for knowledge-intensive nlp tasks.  Advances in neural information processing systems 33, pp. 9459–9474.  Cited by: §1, §2.
  * [29] L. Loukas, M. Fergadiotis, I. Chalkidis, E. Spyropoulou, P. Malakasiotis, I. Androutsopoulos, and G. Paliouras (2022) FiNER: financial numeric entity recognition for xbrl tagging.  In Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers),  pp. 4419–4431.  External Links: [Link](<http://dx.doi.org/10.18653/v1/2022.acl-long.303>), [Document](<https://dx.doi.org/10.18653/v1/2022.acl-long.303>) Cited by: 2nd item.
  * [30] T. Luong, D. Hwang, H. H. Nguyen, G. Ghiasi, Y. Chervonyi, I. Seo, J. Kim, G. Bingham, J. Lee, S. Mishra, A. Zhai, C. H. Hu, H. Michalewski, J. Kim, J. Ahn, J. Bae, X. Song, T. H. Trinh, Q. V. Le, and J. Jung (2025) Towards robust mathematical reasoning.  In Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing,  External Links: [Link](<https://aclanthology.org/2025.emnlp-main.1794/>) Cited by: §4.2.
  * [31] A. Madaan, N. Tandon, P. Gupta, S. Hallinan, L. Gao, S. Wiegreffe, U. Alon, N. Dziri, S. Prabhumoye, Y. Yang, et al. (2023) Self-refine: iterative refinement with self-feedback.  Advances in neural information processing systems 36, pp. 46534–46594.  Cited by: §1, §2.
  * [32] P. Malo, A. Sinha, P. Takala, P. Korhonen, and J. Wallenius (2013) Good debt or bad debt: detecting semantic orientations in economic texts.  External Links: 1307.5336, [Link](<https://arxiv.org/abs/1307.5336>) Cited by: 4th item.
  * [33] M. A. Merrill, A. G. Shaw, N. Carlini, B. Li, H. Raj, I. Bercovich, L. Shi, J. Y. Shin, T. Walshe, E. K. Buchanan, et al. (2026) Terminal-bench: benchmarking agents on hard, realistic tasks in command line interfaces.  arXiv preprint arXiv:2601.11868.  Cited by: §4.3.
  * [34] J. Nichols (2025) How we scored #1 on terminal-bench (52%).  External Links: [Link](<https://www.warp.dev/blog/terminal-bench>) Cited by: §4.3.
  * [35] A. Novikov, N. Vũ, M. Eisenberger, E. Dupont, P. Huang, A. Z. Wagner, S. Shirobokov, B. Kozlovskii, F. J. Ruiz, A. Mehrabian, et al. (2025) Alphaevolve: a coding agent for scientific and algorithmic discovery.  arXiv preprint arXiv:2506.13131.  Cited by: Appendix E, Table 1, §1, §2.
  * [36] OpenAI (2026) Harness engineering: leveraging Codex in an agent-first world.  Note: <https://openai.com/index/harness-engineering/>OpenAI Blog Cited by: §1.
  * [37] C. Packer, V. Fang, S. Patil, K. Lin, S. Wooders, and J. Gonzalez (2023) MemGPT: towards llms as operating systems..  Cited by: §1, §2.
  * [38] R. Pryzant, D. Iter, J. Li, Y. T. Lee, C. Zhu, and M. Zeng (2023) Automatic prompt optimization with “gradient descent” and beam search.  arXiv preprint arXiv:2305.03495.  Cited by: §1, §2.
  * [39] B. Romera-Paredes, M. Barekatain, A. Novikov, M. Balog, M. P. Kumar, E. Dupont, F. J. Ruiz, J. S. Ellenberg, P. Wang, O. Fawzi, et al. (2024) Mathematical discoveries from program search with large language models.  Nature 625 (7995), pp. 468–475.  Cited by: §1, §2.
  * [40] J. Schmidhuber (1993) A neural network that embeds its own meta-levels.  In IEEE International Conference on Neural Networks,  Cited by: §2.
  * [41] N. Schneider, N. Stiefl, and G. A. Landrum (2016) What’s what: the (nearly) definitive guide to reaction role assignment.  Journal of chemical information and modeling 56 (12), pp. 2336–2346.  Cited by: §4.1.
  * [42] S. Shakya, A. Hartl, S. Hochreiter, and K. Pöppel (2026) Adaptive retrieval helps reasoning in llms – but mostly if it’s not used.  External Links: 2602.07213, [Link](<https://arxiv.org/abs/2602.07213>) Cited by: §4.2.
  * [43] A. Sharma (2025) OpenEvolve: an open-source evolutionary coding agent.  Note: <https://github.com/algorithmicsuperintelligence/openevolve>GitHub repository External Links: [Link](<https://github.com/algorithmicsuperintelligence/openevolve>) Cited by: Appendix E, §2, 2nd item, Table 4.
  * [44] J. Snell, K. Swersky, and R. S. Zemel (2017) Prototypical networks for few-shot learning.  In Advances in Neural Information Processing Systems,  Cited by: §2.
  * [45] R. Sutton (2019) The bitter lesson, 2019.  URL http://www. incompleteideas. net/IncIdeas/BitterLesson. html.  Cited by: §5.
  * [46] S. Thrun and L. Pratt (1998) Learning to learn: introduction and overview.  In Learning to learn,  pp. 3–17.  Cited by: §2.
  * [47] M. Tian, Z. Wang, B. Yang, Z. Tang, K. Zhu, H. Dong, H. Li, X. Xie, G. Wang, and J. You (2026) SWE-bench mobile: can large language model agents develop industry-level mobile applications?.  In arXiv preprint,  External Links: [Link](<https://api.semanticscholar.org/CorpusID:285462974>) Cited by: §1.
  * [48] H. Trivedi, N. Balasubramanian, T. Khot, and A. Sabharwal (2023) Interleaving retrieval with chain-of-thought reasoning for knowledge-intensive multi-step questions.  External Links: 2212.10509, [Link](<https://arxiv.org/abs/2212.10509>) Cited by: §1, §2.
  * [49] C. Xiao, G. T. Hudson, and N. A. Moubayed (2024) RAR-b: reasoning as retrieval benchmark.  External Links: 2404.06347, [Link](<https://arxiv.org/abs/2404.06347>) Cited by: §4.2.
  * [50] Y. Xiong, S. Hu, and J. Clune (2026) Learning to continually learn via meta-learning agentic memory designs.  In OpenReview,  External Links: [Link](<https://api.semanticscholar.org/CorpusID:285454009>) Cited by: §2.
  * [51] C. Yang, X. Wang, Y. Lu, H. Liu, Q. V. Le, D. Zhou, and X. Chen (2023) Large language models as optimizers.  In The Twelfth International Conference on Learning Representations,  Cited by: Table 1, §1, §2.
  * [52] H. Ye, X. He, V. Arak, H. Dong, and G. Song (2026) Meta context engineering via agentic skill evolution.  arXiv preprint arXiv:2601.21557.  Cited by: §4.1, §4.1, Table 2, Table 2.
  * [53] M. Yuksekgonul, F. Bianchi, J. Boen, S. Liu, Z. Huang, C. Guestrin, and J. Zou (2024) TextGrad: automatic ”differentiation” via text.  External Links: 2406.07496, [Link](<https://arxiv.org/abs/2406.07496>) Cited by: Table 1, §1, §2.
  * [54] M. Yuksekgonul, D. Koceja, X. Li, F. Bianchi, J. McCaleb, X. Wang, J. Kautz, Y. Choi, J. Zou, C. Guestrin, et al. (2026) Learning to discover at test time.  arXiv preprint arXiv:2601.16175.  Cited by: 3rd item, Table 4.
  * [55] M. Yuksekgonul, D. Koceja, X. Li, F. Bianchi, J. McCaleb, X. Wang, J. Kautz, Y. Choi, J. Zou, C. Guestrin, and Y. Sun (2026) Learning to discover at test time.  External Links: 2601.16175, [Link](<https://arxiv.org/abs/2601.16175>) Cited by: Table 1, §4.3.
  * [56] A. L. Zhang, T. Kraska, and O. Khattab (2026) Recursive language models.  External Links: 2512.24601, [Link](<https://arxiv.org/abs/2512.24601>) Cited by: §1, §2.
  * [57] G. Zhang, H. Ren, C. Zhan, Z. Zhou, J. Wang, H. Zhu, W. Zhou, and S. Yan (2025) Memevolve: meta-evolution of agent memory systems.  arXiv preprint arXiv:2512.18746.  Cited by: §2.
  * [58] J. Zhang, J. Xiang, Z. Yu, F. Teng, X. Chen, J. Chen, M. Zhuge, X. Cheng, S. Hong, J. Wang, B. Zheng, B. Liu, Y. Luo, and C. Wu (2025) AFlow: automating agentic workflow generation.  External Links: 2410.10762, [Link](<https://arxiv.org/abs/2410.10762>) Cited by: §2.
  * [59] Q. Zhang, C. Hu, S. Upasani, B. Ma, F. Hong, V. Kamanuru, J. Rainton, C. Wu, M. Ji, H. Li, U. Thakker, J. Zou, and K. Olukotun (2025) Agentic context engineering: evolving contexts for self-improving language models.  In arXiv preprint arXiv:2510.04618,  Cited by: §1, §4.1, §4.1, Table 2, Table 5.
  * [60] X. Zhang, J. Zhao, and Y. LeCun (2016) Character-level convolutional networks for text classification.  External Links: 1509.01626, [Link](<https://arxiv.org/abs/1509.01626>) Cited by: 7th item.


图 4：在线文本分类上，所有参与比较的文本优化器随评估次数变化的搜索集准确率。每个点是一个候选驾驭框架；折线追踪迄今最佳。各数据集曲线与总体曲线一并展示。Meta-Harness 在前 4 次评估内即达到 OpenEvolve 与 TTT-Discover 的最终准确率，并继续提升，最终高出所有基线 10 个百分点以上。

## 附录 A 提议器的定性行为

本节考察提议器在搜索过程中如何使用文件系统，材料取自 TerminalBench-2 上的一次运行（10 次迭代，Claude Opus 4.6）。

### A.1 文件访问统计

为核实提议器是在实质性地使用文件系统，而不是默认只做局部编辑，我们记录了每次迭代的全部文件读取。

表 8 汇总了这些结果。提议器每次迭代读取文件数的中位数为 82（范围为 69–99），大致均分于先前驾驭框架的源代码（41%）与执行轨迹（40%），其余为分数摘要（6%）以及其他文件（13%）。这证实提议器的访问模式是非马尔可夫的：它常规地检视可用历史中的大部分，而不是仅以最近的父代为条件。

| 统计量 | 取值 |
|---|---|
| 每次迭代读取的文件数（中位数） | 82 |
| 每次迭代读取的文件数（范围） | 69–99 |
| 文件类型构成 | |
| 驾驭框架源代码 | 41% |
| 执行轨迹 | 40% |
| 分数/摘要文件 | 6% |
| 其他 | 13% |

表 8：TerminalBench-2 搜索运行（10 次迭代，Claude Opus 4.6）中提议器的文件访问统计。提议器大量读取文件系统，对先前源代码与执行轨迹的关注大致相当。

### A.2 定性行为：对先前失败的因果推理

TerminalBench-2 的搜索日志呈现出一条清晰的叙事弧线，提议器在其中从自身的退步中学习。它并非在局部编辑中随机游走，而是对早期候选为何失败形成明确诊断，然后转向一种更安全的设计模式。下方日志框中的全部文字均逐字引自提议器在每次迭代中记录的推理（强调为本文所加）。

迭代 1–2：有希望的缺陷修复被提示编辑所混杂。前两次迭代都把看似合理的结构性修复与提示模板修改捆绑在一起，并且都相对 64.4% 的 Terminus-KIRA 基线大幅退步。迭代 1 针对由泄漏的终端标记造成的观察污染，并加入一个循环打断器：

假设：`__CMDEND__` 标记碎片泄漏进长时运行任务上的 LLM 观察，导致模型陷入混乱并进入无工具调用的无限循环。剥离这些标记，并加上一个循环打断器，将挽回被浪费的步骤。

该候选还引入了一个以清理为导向的新提示模板以及一份验证清单。迭代 2 提出另一种状态机修复：

双重确认的完成机制会造成验证螺旋。在一些轨迹中可以观察到，智能体很早就解决了任务，却额外消耗 15–40 步以上反复验证，因为每一条验证命令都会重置 `_pending_completion`，从而需要又一轮 `task_complete` $\rightarrow$ 清单 $\rightarrow$ 验证循环。

这第二个候选完全移除了 pending-completion 机制，同时沿用了标记剥离与新提示。它仍然退步，这使得提议器得到两个失败候选：结构性改动各不相同，但共享同一项提示干预。

迭代 3：提议器识别出混杂因素。到迭代 3，提议器明确推断，退步主要并不是由结构性缺陷修复本身造成的：

先前的尝试：`evo_marker_fix`（58.9%，-5.6 个百分点）、`evo_single_confirm`（57.8%，-6.7 个百分点）——二者都退步了。退步的根因：提示模板改动（清理指令）使智能体在任务完成之前删除了必要状态。结构性缺陷修复与有害的提示改动发生了混杂。`evo_strip_only` 把两项已被证实的结构性修复隔离开来。

这是该轨迹中的关键因果步骤。提议器注意到，前两次失败的共同因素并不是某一项特定的缺陷修复，而是偏重清理的提示重写。因此它恢复原始提示，只测试标记剥离与循环打断器。所得候选仍然略低于基线（63.3%，-1.1 个百分点），但损失远小于更早的版本，这支持了关于混杂因素的诊断。

迭代 4–6：针对已诊断失败模式的直接修复仍然退步。接下来的三次迭代继续探测设计空间的同一部分，但此时对完成逻辑为何脆弱已有更明确的理论。迭代 4 把失败归因于一个具体的状态机缺陷：验证命令会重置完成标志，并使智能体陷入重复的清单循环：

删掉那两行在中间命令运行时重置完成标志的 `self._pending_completion = False`。这修复了一个状态机缺陷，其过程为：(1) 智能体调用 `task_complete` $\rightarrow$ 看到 QA 清单，`_pending_completion = True`；(2) 智能体运行验证命令 $\rightarrow$ `_pending_completion = False`（缺陷！）；(3) 智能体再次调用 `task_complete` $\rightarrow$ *再次*看到清单 $\rightarrow$ 无限循环。

提议器甚至引用了具体的轨迹证据，指出 `configure-git-webserver` 产生了基线失败：智能体在实际上已经解决任务之后，陷入 30–60 步的验证螺旋。迭代 5 试图在保留确认的同时软化清理用语，但仍然编辑了提示，并严重退步。迭代 6 回到更安全的 `evo_strip_only` 基座，并提出一项系统级优化：

当终端输出没有变化时，空命令回合会浪费完整的 LLM 往返。在下一次 LLM 调用之前进行智能等待（对窗格轮询至多 $3\times5$s），可以在长时运行任务上节省 5–15 个回合。

这一改动同样退步。至此，提议器学到了一条具体的经验：对提示与完成流程的修改属于高风险，即便局部假设听起来合理。

迭代 7：获胜候选。在连续六次退步之后，提议器把策略从修改控制循环，转向在循环开始之前添加信息：

先前全部 6 次迭代都相对 64.4% 的基线退步，因为它们修改了完成流程、提示模板或观察处理。`evo_env_bootstrap` 采取不同的做法——纯加性。它在第一次 LLM 调用之前用单条 shell 命令采集一份环境快照，并将其附加到初始提示上。其他方法一律不改。这应当在依赖繁重的任务上消除 3–5 个被浪费的探索回合，同时不使已经通过的任务面临退步风险。

该候选是迄今最好的结果。要紧的不只是迭代 7 获胜，而是提议器阐明了它*为何*应当更安全：它避免触碰先前脆弱的完成机制，转而添加主要在困难任务上才有用的信息。

迭代 8：组合。在找到一项加性改进之后，提议器接着尝试把它与更早的一项结构性修复组合起来：

把两项正交的修复组合起来——环境快照（节省早期探索回合）加上带无工具调用循环打断器的标记剥离——将带来 +1–3 个百分点，因为它们针对相互独立的失败模式，并且不触碰提示或确认流程（这两者在先前 7 次迭代中的 5 次里造成了退步）。

迭代 10：跨运行迁移。提议器引用了另一次更早搜索运行的结果：

演化历史表明，“don’t cleanup service artifacts”价值 +18 个百分点。迭代 9（`evo_no_cleanup_directive`）针对同一想法，但在评估之前崩溃。

小结。这段搜索轨迹表明，提议器所做的不止于随机变异。在前七次迭代中，它识别出一个混杂因素，直接检验了隔离该混杂的假设，观察到控制流与提示编辑仍然脆弱，然后有意转向一项纯加性修改，而这项修改成为这次运行中的最佳候选。它随后尝试把这一获胜想法与更早的修复组合起来，甚至跨运行迁移经验。这种对先前失败的因果推理，正是完整历史的文件系统访问所促成的，也是压缩反馈优化器无法支持的。

## 附录 B 所发现的驾驭框架

Meta-Harness 发现的是切合手头这一具体问题设定的、可执行的推理时过程。这些 harness 是结构化的、领域特定的策略，常常带有非平凡的控制流，例如路由、过滤，以及有条件的上下文构造；遴选它们的唯一依据，是它们能否提升搜索集上的表现。本节以紧凑的方法式抽象，呈现若干有代表性的 harness，概括那些驱动推理时行为的主要行为与控制流决策。作为参照，每个被发现 harness 的完整实现都在 100–1000 行代码这一量级。

### B.1 文本分类 Harness

在在线文本分类中，Meta-Harness 发现的是一族基于记忆的 harness，而不是某一种单一的典范策略。表 9 报告主搜索中非支配变体的帕累托前沿；这些变体全部仅按搜索集表现选出。这里我们突出两个有代表性的端点：上下文最低的前沿点 Meta-Harness（草稿验证），以及主文所使用的、准确率最高的前沿点 Meta-Harness（标签预置查询）。

#### 概述

两种 harness 都维护一份由过往已标注示例构成、并不断增长的记忆，且在推理时依据这份记忆来构造提示。不同之处在于用以探查该记忆的控制流。Meta-Harness（草稿验证）使用两次较短的调用，并明确地用检索到的反例来检验模型的第一次猜测；Meta-Harness（标签预置查询）则把更大的单次调用预算，用于使标签空间和局部决策边界变得明确。图 5 与图 6 概括这两个程序。

#### Meta-Harness（草稿验证）

对应的被发现文件是 `draft_verification.py`。这一轻量变体把预测变成一个两次调用的过程。它首先检索 5 个最相似的已标注示例，并作出一个草稿预测。随后，它以该草稿标签为条件，再次查询同一份记忆，检索 5 个标签相同的*确认例*以及 5 个标签不同的*挑战例*，并询问模型是维持还是修订其初始答案。被发现的关键行为是：第二次检索同时依赖于查询和草稿预测，因此该 harness 能够找出针对模型当前猜测的反例，而不只是泛泛的近邻。如果已经积累的已标注示例太少，程序会退回到标准的单次调用少样本提示。

查询 + 记忆 → 检索最相似的前 5 个示例 → 草稿调用 → 初始标签 $D$ → 检索确认例（== $D$）与挑战例（$\neq D$）→ 验证调用 → 保持或修订 $D$ → 最终标签 $D$

图 5：草稿验证分类 harness。第一次调用根据一段较短的检索上下文产生草稿标签。第二次调用检索支持与反对该草稿的证据，并返回最终预测。

- 阶段 1：草稿。检索 5 个最近的已标注示例，并请求一个初始预测。

- 阶段 2：验证。以草稿标签为条件进行检索，然后在作出最终预测之前，同时展示支持性示例与挑战性示例。

- 冷启动。若可用的已标注示例少于 5 个，则跳过两阶段过程，改用标准的单次调用少样本提示。

- 为何开销低。两次调用都使用较短的检索上下文，因此即便有两次模型调用，总体上下文成本仍靠近前沿的低端。

#### Meta-Harness（标签预置查询）

对应的被发现文件是 `label_primed_query_anchored.py`。这个最强的变体使用一次更大的调用，由三个部分构成。它以一段*标签预置*开头，列出合法的输出标签；随后构造一个*覆盖*部分，每个标签放入一个与查询相关的示例；最后加入*查询锚定的对比对*，把高度相似但标签不同的示例并排放置。覆盖块展示完整的标签空间，对比块则锐化当前查询周围的局部决策边界。在代码中，该 harness 通过对过往已标注示例做 TF-IDF 检索来实现这一点，并用一条查询锚定的配对规则，从同一局部邻域中选取形成对比的示例。

查询 + 记忆 → 标签预置 → 全部合法标签 → TF-IDF 检索 → 查询锚定配对 → 覆盖块 → 每个标签的最佳示例 → 对比对 → 相似示例、不同标签 → 组装为单条提示，其中包含预置、覆盖与对比对 → 最终标签

图 6：标签预置的查询锚定分类 harness。该程序构造单条提示以暴露标签空间，再用与查询相关的覆盖示例和局部对比对将其填充。

- 标签预置。在展示任何示例之前先列出合法的输出标签，使模型一开始就看到完整的答案空间。

- 覆盖块。对每个已知标签，检索与查询最相关的已标注示例，每个类别纳入一个代表性示例。

- 对比块。构造由高度相似、但标签不同的示例组成的配对，使提示暴露出当前查询周围的局部决策边界。

- 检索规则。使用 TF-IDF 相似度以及查询锚定的配对对象选择，而不是与标签无关的最近邻。

|  | 数据集 |  |  | 平均指标 |  |
| --- | --- | --- | --- | --- | --- |
| 变体 | USPTO $\uparrow$ | Symptom $\uparrow$ | LawBench $\uparrow$ | 平均 $\uparrow$ | Ctx $\downarrow$ |
| Meta-Harness（草稿验证） | 18.0 | 85.4 | 17.0 | 40.1 | 5.4 |
| Meta-Harness（错误标注） | 9.0 | 87.7 | 24.0 | 40.2 | 22.3 |
| Meta-Harness（思维链回放） | 13.0 | 88.2 | 25.0 | 42.1 | 23.3 |
| Meta-Harness（聚类覆盖） | 12.0 | 86.8 | 33.0 | 43.9 | 31.2 |
| Meta-Harness（级联检索） | 12.0 | 86.8 | 36.0 | 44.9 | 39.2 |
| Meta-Harness（RRF + 对比） | 18.0 | 89.6 | 35.0 | 47.5 | 41.4 |
| Meta-Harness（相关性 + 对比） | 18.0 | 90.6 | 36.0 | 48.2 | 43.9 |
| Meta-Harness（标签预置查询） | 14.0 | 86.8 | 45.0 | 48.6 | 45.5 |

表 9：来自主文本分类搜索的帕累托最优被发现变体，在平均准确率与上下文成本之间权衡。主文所选系统为 Meta-Harness（标签预置查询）。Ctx 表示输入上下文中额外字符数的平均值（以千计）。［图：图 7：各数据集上被发现文本分类策略的搜索集准确率与测试准确率。每个粉色点是一条被发现的策略；基线已标注。虚线对角线为 $y = x$。］图 7：各数据集上被发现文本分类策略的搜索集准确率与测试准确率。每个粉色点是一条被发现的策略；基线已标注。虚线对角线为 $y = x$。

### B.2 数学检索 Harness

本小节描述 Meta-Harness 针对数学推理所发现的检索 harness（第 4.2 节）。最终的 harness 是一个紧凑的四路线 BM25 程序，其结构是经由搜索涌现出来的，而不是事后手工指定的。下面的全部设计选择——路由谓词、重排序项、去重阈值，以及每条路线的示例数量——都由外层循环在 40 轮演化迭代中选出。

#### 概述

在推理时，该 harness 把每道题恰好分入四条路线之一：组合数学、几何、数论，或者面向代数及其他问题的默认路线。这些门控实现为作用于题目陈述的轻量词汇谓词，包括关键词集合，以及少量针对几何记号的正则特征。该 harness 不跨路线聚合输出：一旦选定某条路线，就只有该路线为最终提示检索示例。所有路线都以 BM25 作为底层检索机制，检索范围是上文所述经过过滤的语料。BM25 索引使用面向数学的分词器，把 LaTeX 记号（例如 $\frac$、$\hat{2}$）保留为原子单元。所选 harness 是两条成功搜索谱系的合并，由提议器在搜索过程中自主组合而成：其中一条贡献了基于原始 BM25 的更强几何路线，另一条则贡献了基于去重与难度重排序的更强组合数学路线。图 8 给出最终程序的一张紧凑流程图。

查询 → 词汇路由器 → 关键词与正则线索 → 组合数学 → BM25@20 → 去重至 8 → 重排序 → 保留 3 → 几何 → 1 条固定参考 + 2 条 BM25 → 不重排序 → 数论 → BM25@12 → 重排序 → 保留 3 → 代数/其他 → BM25@10 → 重排序 → 自适应 $K$ → 构建最终提示

图 8：被发现的数学检索 harness。词汇路由器把每条查询分配给四条按学科划分的检索策略之一。被选中的策略检索示例，这些示例被插入最终提示。

- 组合数学：取回 20 个 BM25 候选，去重至 8 个，按词汇分数与难度重排序，然后返回前 3 个。这是该 harness 显式地在多样性与难题匹配之间做权衡的主要路线。

- 几何：返回 1 条困难的 NuminaMath 参考，连同 2 个原始 BM25 近邻。搜索在这里始终更偏好原始的结构匹配，而不是难度重排序。

- 数论：取回 12 个 BM25 候选，并依据词汇分数、难度，以及对很早就点明所用技巧的解答所给的少量加分进行重排序。这有利于证明策略写得明确的示例。

- 默认：取回 10 个 BM25 候选，按词汇分数与难度重排序，并根据排名靠前的检索分数有多集中，自适应地选择示例数量。

### B.3 TerminalBench-2 Harness

被发现的 TerminalBench-2 harness 建立在 Terminus-KIRA [25] 之上，继承其原生工具调用（用以取代 Terminus 2 基于 ICL 的 JSON 解析）、30KB 输出上限，以及多视角完成检查清单。Meta-Harness 所发现的主要修改是环境自举：在智能体循环开始之前，该 harness 运行一条复合 shell 命令，采集沙箱环境的一份快照，并将其注入初始提示。提议器的假设在搜索日志中有逐字记录，内容为：

假设：「在第一次 LLM 回合之前注入环境快照（操作系统、已安装语言、包管理器、/app 的内容），将在依赖较重的任务上，把浪费掉的探索回合减少 3–5 轮。」变更：「新增 `_gather_env_snapshot()`，它运行一条复合 shell 命令，用以收集工作目录、/app 列表、可用语言（python、gcc、node、java、rustc、go）、包管理器（pip、apt）［……］，并作为 [Environment Snapshot] 块注入。」

该快照包括：工作目录、/app 的列表（较大的目录截断为 20 项）、可用编程语言及其版本（Python、GCC、G++、Node、Java、Rust、Go）、已安装的包管理器（pip、apt-get），以及可用内存。这就消除了智能体通常花费在查明有哪些工具和文件可用上的 2–4 个探索回合，使模型能够立即开始有效工作。这条自举命令受到 15 秒超时的保护，并且失败时静默处理，因此不会在异常环境中破坏智能体。完整实现在 Terminus-KIRA 之上大约增加 80 行。图 9 概括了该 harness 的结构。

#### 逐任务分析

与 Terminus-KIRA 相比，被发现的 harness 在 89 项任务中的 7 项上有所提升，提升最大的是 protein-assembly 与 path-tracing。这些有所提升的任务有一个共同性质：它们需要一些无法事先假定其可用性的领域特定工具（生物信息学库、渲染流水线、国际象棋引擎、密码学工具、CoreWars 模拟器）。若没有这一自举，智能体会把最初的 2–4 个回合花在探测环境上；在回合预算很紧、或早期错误假设会级联放大的任务上，这些被浪费的回合就可能是通过与失败的差别。这表明，当环境并不显而易见，且任务要求智能体使其策略与实际所安装的内容相匹配时，自举的价值最大。

任务说明 → 环境自举 → 工作目录、文件、语言、包管理器、内存 → 初始提示 → 任务 + 快照 → 智能体循环 → 原生工具调用 → 30KB 输出上限 → 多视角完成检查清单 → 任务完成 → 通过/失败

图 9：被发现的 TerminalBench-2 harness。该 harness 继承 Terminus-KIRA 的原生工具调用、输出上限以及完成检查清单（绿色）。环境自举（红色）是 Meta-Harness 所发现的组件：它在智能体循环开始之前采集沙箱快照，从而消除早期的探索回合。

## 附录 C 数据集细节

### C.1 分布外文本分类数据集

- SciCite 是 Cohan 等人 [14] 提出的三路引用意图分类基准。每个示例由科学论文中的一段引用上下文构成，并按该引用的修辞角色标注，例如背景、方法或结果。该任务检验模型能否从局部科学上下文推断一篇论文为何引用另一篇论文。

- FiNER-139 是 Loukas 等人 [29] 提出的金融数值实体识别基准。它由金融申报文件中的词级标注构成，包含 139 种细粒度 XBRL 实体类型，因而比标准的句子级分类任务细粒度得多。该基准检验模型能否依据上下文识别并分类数值型金融实体。

- Amazon Reviews 是 Keung 等人 [22] 提出的 Multilingual Amazon Reviews Corpus 的英语部分。在我们的设定中，它被用作五路评论评分预测任务，标签对应于该评论的星级。该基准评估依据产品评论文本进行的通用领域情感与评分预测。

- Financial PhraseBank 是 Malo 等人 [32] 提出的三路金融情感基准。它由金融新闻及相关经济文本中的句子构成，这些句子按相对于市场情绪而言的正面、中性或负面加以标注。该任务评估金融领域中的领域特定情感分类。

- GoEmotions 是 Demszky 等人 [15] 提出的细粒度情绪分类基准。它包含标注了 27 个情绪类别外加一个中性类别的英语 Reddit 评论，通常被当作 28 路分类任务。该基准检验的是超出粗粒度正负情感的细微情绪识别。

- Banking77 是 Casanueva 等人 [11] 提出的细粒度意图分类基准。它包含标注有 77 种意图的网上银行用户话语，覆盖范围广泛的客户服务请求。该任务评估标签空间较大时的单领域意图检测。

- AG News 是四路新闻主题分类基准，通常与 Zhang 等人 [60] 的文本分类设定联系在一起。示例被标以宽泛的新闻类别，例如国际、体育、商业以及科学/技术。它是主题分类的一项标准通用领域基准。

- SciTail 是科学领域的文本蕴含基准，其任务是在面向科学的推理设定中，预测一个假设是否由前提句所蕴含 [24]。

- TweetEval (Hate) 是 Barbieri 等人 [7] 提出的 TweetEval 基准中的仇恨言论子集。在一套统一的社交媒体评测中，它是用于检测仇恨内容与非仇恨内容的二分类推文任务。该基准检验在嘈杂的短形式社交媒体文本上的稳健分类。

### C.2 数学检索语料库

表 10 列出第 4.2 节所使用的检索语料由哪些数据集组成。原始来源所含题目多于最终语料；合并之前施加了若干过滤步骤。NuminaMath-1.5 被过滤为竞赛数学子集（AMC/AIME、奥林匹克参考、数论、不等式及相关来源），并丢弃质量较低的网页抓取条目。OpenMathReasoning 被去重为每道题一条解答（保留在独立验证器上通过率最高的解答），并且来源与任一评测基准家族（IMO、AIME、HMMT、SMT、USAMO、Putnam）相匹配的题目，在去重之前被移除。随后，整个语料针对全部评测基准以及 harness 搜索期间所使用的搜索集做了去污染，先用精确前缀匹配，再用模糊 Jaccard 相似度（阈值 0.8）；在任一准则下与某道评测题目相匹配的语料题目都被丢弃。来自 OpenMathReasoning 与 DeepMath 的解答被截断到 5,000 个字符，以限制检索上下文的长度。在运行时，所选 harness 进一步把检索限制为解答非空且短于 4,000 个字符的条目。检索到的解答在插入提示时再次被截断到 3,000 个字符。对于几何路线，该 harness 还从难度大于 6 的 NuminaMath 题目中另行构建一个困难参考索引。

| 数据集 | 题目数 | 解答长度 | 证明 |
| --- | --- | --- | --- |
| [OpenMathReasoning](https://huggingface.co/datasets/nvidia/OpenMathReasoning) | 281,743 | 5,000† | 34% |
| [DeepMath-103K](https://huggingface.co/datasets/zwhe99/DeepMath-103K) | 103,021 | 5,000† | 0% |
| [NuminaMath-1.5](https://huggingface.co/datasets/AI-MO/NuminaMath-1.5) | 129,520 | 1,376 | 13% |
| [PolyMath](https://huggingface.co/datasets/AIMO-Corpus/PolyMath) | 11,083 | 363 | 0% |
| [Omni-MATH](https://huggingface.co/datasets/KbsdJames/Omni-MATH) | 4,289 | 829 | 0% |
| [FineProofs-SFT](https://huggingface.co/datasets/SPIderman5/FineProofs-SFT) | 4,275 | 3,977 | 100% |
| [AIME 1983–2024](https://huggingface.co/datasets/gneubig/aime-1983-2024) | 933 | — | 0% |
| [Putnam-AXIOM](https://huggingface.co/datasets/Putnam-AXIOM/putnam-axiom-dataset-v1) | 492 | 888 | 100% |
| 合计 | 535,356 | 5,000† | 22% |

† 在 5,000 个字符处截断；实际解答更长。

表 10：数学检索语料中的数据集（合计 535K 道题）。解答长度是以字符计的解答长度中位数。证明表示该数据集是否包含证明型题目（依据答案字段或题目类型字段）。

### C.3 数学 IMO 级别测试集

主文汇总的是 200 道 IMO 级别题目上的结果，这些题目取自 IMO-AnswerBench、IMO-ProofBench、ArXivMath 2025 年 12 月以及 ArXivMath 2026 年 1 月。这个 200 题的评测集，由 IMO-AnswerBench 的一个分层抽取的 100 题子集，以及另外三个基准的全部题目组成。这一按基准的分解是有用的，因为这四个数据集混合了答案式、证明式和研究式题目，而主文为求简略把它们汇总在了一起。若纳入本节的表，则应在五个留出模型上，分别就 Base 与 Meta-Harness 报告每一个基准。

| 数据集 | 题目数 |
| --- | --- |
| IMO-AnswerBench | 100 |
| IMO-ProofBench | 60 |
| ArXivMath 2025 年 12 月 | 17 |
| ArXivMath 2026 年 1 月 | 23 |
| 合计 | 200 |

表 11：200 题 IMO 级别评测集的构成。

## 附录 D 实践实现建议

Meta-Harness 在很大程度上与领域无关：我们预期，只要一个语言模型被任务特定的 harness 包裹，它就适用。然而，要把它应用到一个新领域，就需要在一种相对较新的、由 LLM 辅助编程的范式中工作：提议器以先前各次运行的长程历史为条件，并编写那些效果可能要到许多步之后才看得到的程序。在把这一工作流做到可靠的过程中，我们发现有一小套实践上的选择，在本文所研究的三个领域里都始终重要。下面的准则本身并不是关于该方法的科学主张；它们是构建并运行该系统时得到的工程经验。我们希望这些经验能让后续工作更容易在其他领域应用 Meta-Harness。

- 写好技能。技能文本是引导搜索的主要界面，它的质量是决定这个循环能否运转的最强杠杆。提议器会收到一段自然语言技能 [4]，其中定义它的角色、目录布局、CLI 命令以及输出格式。在实践中，技能应当约束的是输出以及与安全相关的行为，而不是提议器的诊断过程：它应当写明哪些事情被禁止、要产出哪些产物、以及要优化哪些目标，同时让模型按需要自由地检查分数、轨迹和先前的代码。我们检视 Meta-Harness 运行日志后的直觉是：迭代足够多次之后，累积下来的轨迹对提议器行为的塑造，往往超过技能文本本身。以我们的经验，反复打磨技能文本对搜索质量的影响，大于改变迭代次数或种群规模。在投入一次完整运行之前，应当专门做几次较短的演化运行（每次 3–5 轮迭代），用来调试并打磨这段技能。

- 从一个基线 harness、以及一个对它而言困难的搜索集开始。先写一个简单基线（例如少样本提示），然后构造搜索集：要么筛出基线做错的示例，要么选取困难实例的一个多样化子集。如果基线已经使评测饱和，搜索就几乎没有什么可优化的。把搜索集保持得足够小，使每次运行大约能做 50 次完整评测（我们的分类实验中为 50–100 个示例，数学检索为 88 道题）；快速且有区分度的评测，比规模大的评测更有价值。

- 以易于浏览的格式记录一切。评测代码应当以提议器能够可靠查询的形式，写下代码、分数和执行轨迹。在实践中，这意味着使用 JSON 等机器可读格式，按层次组织产物，选用合理且一致的文件名，并采用能让正则搜索这类简单工具良好工作的命名方案。

- 通过一个小型 CLI 使日志可以被查询（可选，但有帮助）。每个 harness 都有一个目录，其中包含源代码、分数和执行轨迹；但随着历史增长，单靠原始的文件系统访问就会变得笨拙。一个简短的 CLI，若能列出帕累托前沿、显示排名前 $k$ 的 harness，并比较成对运行之间的代码与结果，就能使经验库容易使用得多；而查询这类 CLI，与编码智能体所受训练的那些工作流是紧密对齐的。如果存在相关的离线经验（其他模型的 rollout、已解出题目的语料、相关论文），把它转换成同一种目录结构，也有助于热启动探索，并为新想法提供依据。这一层帮助提议器省下它可能浪费在导航上的词元。

- 在昂贵的基准之前做轻量验证。写一个小型验证测试：导入该模块，实例化该类，并在极小的一组示例上调用这两个方法。搜索期间被提出的 harness 应当先通过这一测试，然后再被完整评测。一个简单的测试脚本可以在数秒之内抓住大多数格式错误或无法运行的候选，并把失败的成本保持在接近于零。

- 在提议器之外自动完成评测。运行评测足够简单，不值得让提议器来做。应当由另一个 harness 为候选打分，并把结果写到文件系统。

## 附录 E 扩展相关工作

本附录扩展第 2 节中的简短讨论，并把 Meta-Harness 放在若干相邻的研究脉络之中加以定位；这些脉络我们未能在主文里详细覆盖。一个反复出现的区别是：Meta-Harness 优化的是可执行的 harness 实现，并通过文件系统向提议器提供对先前代码、分数和执行轨迹的选择性访问。

#### AlphaEvolve / OpenEvolve

AlphaEvolve [35] 与 OpenEvolve [43] 借助带有结构化反馈的、由 LLM 引导的变异来演化代码：提议器收到一个带有标量分数的程序数据库（每步 4–22K 词元；表 1），并对经锦标赛选出的父代施加固定的变异策略。这些方法是为算法发现与优化而设计的（数学猜想、调度启发式、硬件内核），其中搜索目标是单个带有干净标量目标的无状态函数，并且变异是局部的。harness 工程则是另一种范式：harness 是有状态的程序，会跨许多示例积累经验，而单个设计选择（例如在记忆里存放什么）就可能级联影响到整段评测序列。Meta-Harness 的应对办法，是给予一个不作结构化约束的编码智能体完整的文件系统访问权，让它有选择地阅读任一先前候选的源代码、执行轨迹和分数。

#### GEPA

就反馈的丰富程度而言，GEPA [1] 是最接近的文本优化器，它为每个候选提供 rollout 轨迹。它面向反馈回路较短的任务上的提示优化（数学题、指令遵循、代码优化），在这些任务中，每次 rollout 是一次 LLM 调用或一条短流水线。在这一范式中，逐候选的反思效果很好：一条提示、一个答案、一个分数。harness 工程需要同时跨许多示例、许多候选进行推理：要理解一种检索策略为何对一类问题有效、却在另一类上变差，就必须在整个种群上比较执行轨迹。GEPA 一次只处理一个候选（每步 2–8K 词元；表 1），并使用一种固定的批评格式，这种格式必须预先料到哪些信息是相关的。Meta-Harness 让提议器同时接触全部先前候选，并由智能体决定要检查什么。

#### 提示编排框架

有若干系统为组合多阶段 LLM 程序提供结构化抽象。LMQL [8]、LangChain [13] 与 DSPy [23] 通过为提示模板、控制流以及模块化 LLM 流水线提供更高层的接口，使提示工程更加系统化。这些框架帮助开发者指定并组织 LLM 程序，但通常仍要求手工设计检索策略、记忆更新以及编排逻辑。Meta-Harness 工作在另一个层面上：它在可执行代码中搜索这些策略的*实现*，把 harness 本身当作优化目标。
