---
title: "Hyperagents"
author: "Jenny Zhang, Bingchen Zhao, Wannan Yang, Jakob Foerster, Jeff Clune, Minqi Jiang, Sam Devlin, and Tatiana Shavrina"
source_url: "https://arxiv.org/abs/2603.19461"
arxiv_id: "2603.19461"
arxiv_version: "v2"
published_at: "2026-03-20"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-translation"
unofficial_translation: true
translation_scope: "v2 正文（摘要至结论）与附录散文的忠实全文翻译。代码、提示词与伪代码保留英文。参考文献保留英文。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名与 CC BY 4.0。"
---

**非官方中文翻译声明。** 本文是 arXiv:2603.19461 **v2**《Hyperagents》的非官方中文译文，依 **CC BY 4.0** 许可翻译。原文作者为 Jenny Zhang、Bingchen Zhao、Wannan Yang、Jakob Foerster、Jeff Clune、Minqi Jiang、Sam Devlin 与 Tatiana Shavrina。原文：https://arxiv.org/abs/2603.19461 。本译文未经作者审定，不改变科学内容；代码、提示词与伪代码保留英文，参考文献保留英文。使用本译文须保留原作者署名与 CC BY 4.0 许可。许可文本：https://creativecommons.org/licenses/by/4.0/ 。

# 超智能体（Hyperagents）

Jenny Zhang，单位：不列颠哥伦比亚大学。Bingchen Zhao。Wannan Yang。Jakob Foerster，单位：Meta FAIR。Minqi Jiang，工作完成于 Meta。Sam Devlin，单位：Meta 超级智能实验室。Tatiana Shavrina，单位：Meta FAIR；单位：Vector 研究所；单位：爱丁堡大学；单位：加拿大 CIFAR AI 讲席。工作完成于在 Meta 实习期间。

## 摘要

自我改进的人工智能系统旨在通过学习改进其自身的学习过程与问题求解过程，减少对人类工程的依赖。现有的递归自我改进方法通常依赖固定的、手工设计的元层面机制，这从根本上限制了此类系统能够改进的速度。达尔文-哥德尔机（Darwin Gödel Machine，DGM）(Zhang et al., 2025b) 表明，开放式自我改进在编程中是可以实现的。DGM 从单个编程智能体出发，反复生成并评估自我修改后的变体，形成一份不断增长的存档，作为未来改进的踏脚石。由于评估与自我修改都是编程任务，编程能力上的增益可以转化为自我改进能力上的增益。然而，这种对齐在编程领域之外通常并不成立。我们引入超智能体（hyperagents）：一类自指智能体，它把任务智能体（负责求解目标任务）与元智能体（负责修改自身以及任务智能体）整合进同一个可编辑程序。关键在于，元层面的修改过程本身是可编辑的，从而实现元认知自我修改：不仅改进求解任务的行为，也改进产生未来改进的机制。我们通过扩展 DGM 来实例化这一框架，从而构建 DGM-超智能体（DGM-Hyperagents，DGM-H）。通过让改进过程本身得以演化，DGM-H 消除了任务表现与自我修改技能之间存在领域特定对齐这一假设，并有可能在任何可计算任务上支持自我加速的进展。在多种领域（编程、论文评审、机器人奖励设计，以及奥林匹克级别数学解答评分）中，DGM-H 的表现随时间提升，并优于没有自我改进或没有开放式探索的基线，也优于 DGM 等先前的自我改进系统。我们进一步表明，DGM-H 改进了它生成新智能体的过程（例如持久记忆、性能追踪），并且这些元层面改进可以跨领域迁移，并在多次运行之间累积。全部实验均在安全预防措施下进行（例如沙箱隔离、人工监督）。我们讨论在这一设定中安全意味着什么，以及自我改进系统的更广泛影响。DGM-超智能体让我们得以一窥这样的开放式人工智能系统：它们并非仅仅搜索更好的解，而是持续改进自己对于“如何改进”的搜索。

日期：2026 年 3 月 19 日

Jenny Zhang：[jennyzzt@cs.ubc.ca](mailto:jennyzzt@cs.ubc.ca)，Tatiana Shavrina：[rybolos@meta.com](mailto:rybolos@meta.com)

代码：<https://github.com/facebookresearch/Hyperagents>

## 1 引言

在采取适当安全考量的前提下，能够自我改进的人工智能系统有可能把科学进步从受人类节奏限制的过程，转变为自主加速的过程，从而使社会更早实现技术进步所带来的收益。这类自我改进的人工智能力求持续改进其自身的学习能力与任务求解能力。然而，现有的大多数自我改进架构依赖于一个固定的元智能体（即一个修改基础系统的更高层系统）。这造成一种限制：基础系统只能在元智能体设计所划定的边界之内被改进。增加一个元-元系统来改进元智能体并不能解决这一问题，而只是把问题上移，并最终导致元层面的无限回归。为克服这一限制，并允许系统在不受其初始实现约束的情况下修改自身的任何部分，该系统必须是自指的，也就是能够分析、修改并评估自身 (Kirsch and Schmidhuber, 2022; Zhang et al., 2025b)。当改进机制本身也可以被改进时，进步可以成为自我加速的，并且潜在地是无界的 (Lu et al., 2023)。

达尔文-哥德尔机（DGM）(Zhang et al., 2025b) 表明，开放式自我改进在编程中是可以实现的。在 DGM 中，智能体生成并评估对其自身代码的修改，成功的变体被保留在存档中，作为进一步改进的踏脚石。然而，DGM 依赖一种手工设计的、固定的机制来产生自我改进指令（附录 B）。该机制分析过去的评估结果以及智能体当前的代码库，生成一条指令，指明智能体应当在何处进行自我改进。这一机制不可修改。因此，DGM 的自我改进能力受阻于这一固定的指令生成步骤。尽管存在这一手工设计的步骤，DGM 仍然可以在“自我改进”这件事上变得更好。因为评估与自我修改都是编程任务，评估表现上的改进直接反映智能体生成有效自我修改的能力。为了在自我改进上变得更好，DGM 依赖于一个限制性假设：求解评估任务所需的技能，与进行有效自我反思和自我修改所需的技能是相同的。这一假设在编程领域之外不太可能成立；在那些领域中，任务求解技能可能与分析失败、提出有效自我改进并加以实现所需的技能有实质差异。

本文引入超智能体，即原则上能够针对任何可计算任务进行自我改进的自指智能体。这里，智能体是任意可计算程序，可以选择性地包含对基础模型（foundation models，FM）、外部工具或已学习组件的调用。任务智能体求解给定任务。元智能体修改智能体并生成新的智能体。超智能体把任务智能体与元智能体组合进同一个自指的、可修改的程序，使得负责生成改进的机制本身也接受修改。因此，超智能体不仅可以改进它求解任务的方式（即任务智能体），也可以改进它生成并应用未来修改的方式（即元智能体）。由于其自我改进机制本身是可修改的，我们将此称为元认知自我修改。我们用超智能体扩展 DGM，构建 DGM-超智能体（DGM-H）。DGM-H 保留 DGM 的开放式探索结构，并以元认知自我修改扩展 DGM。与 DGM 一样，为支持持续进展并避免过早收敛，DGM-H 通过从被选中的候选者分支、允许它们自我修改、评估所得到的超智能体，再把它们加回存档，从而使超智能体存档不断增长。因为超智能体可以修改其自我修改过程，DGM-H 不受其初始实现的约束，并有可能针对任何可计算任务进行自我改进。

在我们的实验中，DGM-H 在任务表现与自我改进能力两方面都展现出实质的、可推广的提升。在 Polyglot 编程基准 (Gauthier, 2024) 上，DGM-H 取得了与最成熟的先前自我改进算法（达尔文-哥德尔机，Zhang et al., 2025b）相当的增益，尽管它并非专门为编程而手工设计。在编程之外，DGM-H 大幅提升了论文评审 (Zhao et al., 2026) 与机器人奖励设计 (Genesis, 2024) 上的表现，增益可以迁移到留出的测试任务，并显著优于先前的自我改进算法；那些算法除非经过定制，否则在编程之外难以奏效。去掉自我改进或去掉开放式探索的消融实验几乎没有进展，这凸显了每一个组成部分的必要性（第 5.1 节）。至关重要的是，DGM-H 学到了关于如何自我改进的可迁移机制（例如持久记忆、性能追踪），这些机制系统地提升了它随时间生成更好的任务智能体或元智能体的能力。因此，DGM-H 学到的元层面改进可以跨领域迁移。具体而言，在一种设定中优化得到的超智能体（即论文评审与机器人任务）在另一个领域（即奥林匹克级别数学评分）中生成改进后的任务智能体时仍然显著有效（第 5.2 节）。我们进一步表明，DGM-H 在一种设定中学到的自我改进，可以与在另一种设定中继续进行的自我改进相叠加（第 5.3 节）。这表明，给定适当的任务，DGM-H 有潜力随时间实现无界的开放式自我改进。我们在第 6 节讨论此类开放式自我改进系统的安全影响，并概述负责任部署的实际考量。总体而言，超智能体开启了这样一种可能：在提升其执行任何可计算任务的能力的同时，提升其进行改进的能力。

## 2 相关工作

**开放性。** 开放性是指一个系统持续发明新的、有趣的、且日益复杂的产物的能力，在没有固定目标或预定义终点的情况下扩展自身的发现前沿 (Stanley et al., 2017; Hughes et al., 2024)。近期工作将基础模型用作人类有趣性的代理，并用作在多种领域中生成与评估新颖行为的通用引擎 (Zhang et al., 2024; Faldor et al., 2025)。在这些进展之上，开放式学习 (Hu et al., 2025; Zoph and Le, 2017; Colas et al., 2023; Lehman et al., 2023) 与质量-多样性算法 (Lehman and Stanley, 2011; Mouret and Clune, 2015; Bradley et al., 2023; Samvelyan et al., 2024; Ding et al., 2024; Pourcel et al., 2023; Coiffard et al., 2025; Dharna et al., 2025; Yuan et al., 2026) 的近期进展表明，持续探索能够在从游戏智能体 (Klissarov et al., 2023, 2025; Wang et al., 2024) 到科学发现 (Lu et al., 2024a, b; Romera-Paredes et al., 2024; Novikov et al., 2025; Audran-Reiss et al., 2025) 以及机器人控制 (Cully et al., 2015; Li et al., 2024; Grillotti et al., 2025) 的诸多领域中，产生多样且能力不断增强的产物。近期进展已经表明，能够持续生成多样且日益复杂产物的开放式人工智能系统是可能的 (Zhang et al., 2024; Faldor et al., 2025; Hu et al., 2025)。重要的下一步是探索此类系统如何实现叠加式改进。在人类科学与技术进步中，新进展往往建立在既有进展之上：不仅产出更好的产物，也改进那些产生未来发现的工具与过程，从而导致加速的创新 (Good, 1966; Kwa et al., 2025)。受这一模式启发，我们关注这样的开放式系统：它们不仅能改进所生成的产物，也能改进产生新颖性与进步的机制 (Clune, 2019; Jiang et al., 2023)。

**自我改进的人工智能。** 关于自我改进人工智能的早期理论工作可以追溯到自修改智能体的形式模型 (Hutter, 2003)。一个突出的例子是哥德尔机（Gödel Machine）(Schmidhuber, 2003)，它提出：当改写自身可被证明有益时，智能体便改写自身；不过此类方法在现实设定中仍然不切实际。随后的研究通过自适应神经系统探索自我改进，其中智能体经由元学习 (Schmidhuber, 1993; Miconi et al., 2018; Javed and White, 2019; Beaulieu et al., 2020; Miconi et al., 2020; Irie et al., 2022; Chalvidal et al., 2022; Oh et al., 2025)、演化 (Stanley and Miikkulainen, 2002; Lange et al., 2023; Qiu et al., 2025; Zhao et al., 2025) 或自我对弈 (Silver et al., 2016, 2017; Xia et al., 2025b, 2026) 来修改自身的权重或学习动态。值得注意的是，Silver et al. (2017) 使用自我对弈来迭代改进神经网络智能体，在围棋与国际象棋等领域取得了超人表现，尽管底层学习算法本身仍然是固定的、由人类设计的。更近一些，基础模型使得自我改进可以通过对提示词的迭代精炼 (Fernando et al., 2023; Wang et al., 2025a; Zhang et al., 2025c, a; Ye et al., 2026)、对推理轨迹的迭代精炼 (Zelikman et al., 2022; Yin et al., 2025; Havrilla et al., 2024; Zhuge et al., 2024)，以及对整个代码仓库的迭代精炼 (Zhang et al., 2025b; Wang et al., 2025b; Xia et al., 2025a) 来实现，也可以通过使用自生成数据或交互来更新模型权重的系统来实现 (Wu et al., 2024; Zweiger et al., 2025; Wen et al., 2025; Wei et al., 2025b)。其中，达尔文-哥德尔机（DGM）(Zhang et al., 2025b) 作为编程领域中递归自我改进的一种实用实例而尤为突出。然而，尽管这些方法有效，现有的大多数方法（包括 DGM 及其派生工作）都依赖固定的、手工设计的元层面机制（附录 B），这些机制约束了自我改进如何随时间叠加，以及如何跨领域推广。

**自指元学习。** 自指元学习研究那些学习去改进“学习得以发生的机制”的系统。先前工作在神经网络 (Kirsch and Schmidhuber, 2022; Jackson et al., 2024) 与演化方法 (Lu et al., 2023) 中探索过这一想法。更近一些，若干工作使用基于基础模型的智能体探索自指改进 (Zelikman et al., 2024; Robeyns et al., 2025; Yin et al., 2025; Zhang et al., 2025b)。达尔文-哥德尔机（DGM）(Zhang et al., 2025b) 及其后继工作 (Wang et al., 2025b; Xia et al., 2025a; Weng et al., 2026) 主要通过自我修改来实例化递归自我改进，且主要是在编程领域。然而，这些方法主要只在编程任务内部提升“进行改进”的能力。在 DGM 及相关系统中，一个编程智能体被赋予改进自身的任务，由此得到的改进后的编程智能体随后被用于后续的自我改进步骤，以生成一个更好的自身版本。由于评估任务与自我修改过程都涉及编程，改进编程智能体也会增强系统执行未来自我改进的能力。然而，这一性质仅当评估任务与自我修改任务密切对齐时才成立。例如，若评估任务改为诗歌写作，那么提升一个智能体的诗歌写作能力，并不必然提升它修改自身代码的能力。因此，先前工作依赖于评估任务与自我改进所需技能之间的一种*对齐*。与此相对，超智能体并不假设这种对齐，因为自我修改机制是完全可修改的，并且不绑定于任何特定任务领域。因此，超智能体可以在任何可计算任务上同时改进任务表现与改进过程本身。

## 3 方法

**图 1：带有超智能体的达尔文-哥德尔机。** DGM-超智能体（DGM-H）把达尔文-哥德尔机（DGM）(Zhang et al., 2025b) 扩展到编程任务之外，使智能体不仅能改进其任务表现，也能改进其改进自身的能力，并且这一能力覆盖任何可计算任务。（上）在 DGM 中，编程智能体通过开放式探索而演化：它生成并评估自我修改后的变体，这些变体被存储在一份踏脚石存档中。同一个编程智能体既充当任务智能体（接受评估），也充当元智能体（生成修改）。尽管这一设计能够在编程上带来累积增益，驱动自我改进的指令生成机制却是固定的、手工设计的。因此，递归改进依赖于编程表现与自我修改能力之间的对齐。（下）在 DGM-H 中，任务智能体与元智能体被组合进一个称为超智能体的单一可修改程序。这一设计允许元智能体自身被自主地改进。该系统保留 DGM 的开放式探索结构，同时使元层面的改进机制成为可编辑的。这实现了元认知自我修改，并支持在任何可计算任务上的自指改进。

我们引入超智能体，即把任务执行与智能体生成统一进同一个可修改程序的自指智能体。超智能体不仅可以改进它求解任务的方式，也可以改进它生成未来改进的方式。为使进展得以持续并不断累积，我们通过直接建立在达尔文-哥德尔机（DGM）之上来实例化超智能体，形成 DGM-超智能体（DGM-H）。DGM 提供一种开放式的、基于种群的探索过程，维护一份逐步改进的智能体存档，使成功的变体能够充当未来增益的踏脚石。DGM-H 保留这一开放式演化结构，并通过使整个元层面修改机制成为可编辑的来加以扩展（图 1）。通过允许智能体不仅修改它们求解任务的方式，也修改它们改进自身的方式，DGM-H 有潜力在任何可计算任务上进行开放式的自我改进。

**智能体。** 本文将智能体定义为任意可计算程序，可以选择性地包含对基础模型、外部工具或已学习组件的调用。智能体不限于某一种特定表示（例如神经网络或提示词），并且可以包含任意的算法逻辑、记忆与控制流。任务智能体是为求解一组任务而实例化的智能体。例子包括：为软件仓库生成代码编辑 (Gauthier, 2024; Jimenez et al., 2024)，预测研究论文的录用决定 (Couto et al., 2024)，以及为机器人环境设计奖励函数 (Ma et al., 2024)。任务智能体在给定任务上接受经验性评估。元智能体是这样一种智能体：它的唯一任务是修改已有智能体并生成新的智能体。在能够访问先前全部智能体与评估的整份存档的条件下，元智能体提出旨在改进未来表现的变更（包括潜在地许多代之后的表现）。重要的是，这些变更的目标不仅可以是任务求解逻辑，也可以是元智能体自身，从而能够改进用以生成未来修改的那些过程。

**超智能体。** 超智能体是一种自指智能体，它在同一个可编辑程序之内整合任务智能体与元智能体，从而不仅能修改它执行任务的方式，也能修改它生成未来自我修改的方式。与具有固定元层面的分层系统不同，在超智能体中，元智能体是同一个可编辑程序的一部分，并且可以改写自身。因此，超智能体可以同时改进（1）它求解任务的方式，以及（2）它生成未来自我改进的方式。我们使用 Python，它是图灵完备的 (Turing and others, 1936)；并且由于超智能体可以编辑任何代码，它有潜力构建任何可计算机器。

**元认知自我修改。** 在超智能体中，智能体的自我改进机制本身接受修改。除了改进它在给定任务上的表现之外，智能体还可以同时修改它提出并应用进一步自我改进的那些过程。我们将这一过程称为元认知自我修改：超智能体不仅改进负责求解给定任务的任务执行智能体，也改进决定后续超智能体如何被生成的元智能体。这一特性针对先前自我改进系统的一个核心限制 (Zhang et al., 2025b; Wang et al., 2025b)，直接使对自我改进过程本身的改进成为可能（第 2 节）。此类元认知自我修改的例子见第 5.2 节与附录 E.3。

**带有超智能体的达尔文-哥德尔机。** 用超智能体增强原始 DGM (Zhang et al., 2025b)，我们构建 DGM-超智能体（DGM-H）。DGM-H 采用 DGM 中的开放式探索过程，以减轻过早收敛，并避免陷入局部最优。该过程维护一份已生成超智能体的存档，以单个超智能体初始化，并随时间通过持续累积所生成的变体而扩展。该过程在两个阶段之间交替：元认知自我修改与评估。在元认知自我修改阶段，从存档中选出的父代超智能体生成它们自身的修改版本。父代选择是概率性的，与超智能体的表现成正比，并与成功编译的子代数量成反比，从而使采样偏向表现良好且能产生强后代的超智能体，同时保留探索（附录 A.2）。在评估阶段，每个被修改的超智能体接受经验性评估，随后被加入存档。原则上，一个完全自指的算法应当允许修改其自身的每一个部分（包括父代选择机制与评估机制）。虽然我们在附录 E.5 中给出了探索自动改进父代选择机制这一可能性的初步结果，正文中的实验使用一种手工设计的、不接受修改的父代选择机制，以便隔离超智能体自我修改的效应。总体而言，DGM-H 由两个相互作用的组成部分构成：（1）继承自 DGM 的开放式探索过程，以及（2）一个初始超智能体，它随时间通过自生成的变体而演化（图 1，附录 A）。通过扩展 DGM，使元层面机制本身成为可修改的，DGM-H 把递归自我改进推广到编程之外，并使任何可计算任务上的自指改进成为可能。

## 4 实验设置

DGM-H 以单个超智能体初始化，该超智能体围绕一个冻结的基础模型 (Brown et al., 2020) 构建，并具备外部工具使用能力 (Schick et al., 2023)。基础模型在每一步按需要自主调用工具。这一初始超智能体配备两种工具：一个用于执行 shell 命令的 bash 工具，以及一个用于检查和修改文件的专门工具。在初始超智能体中，任务智能体接收任务输入，并输出单次基础模型调用的响应。元智能体接收该智能体仓库的位置、先前评估结果的位置，以及剩余的实验迭代次数（实验预算是一个超参数），其任务是修改给定代码库的任何部分（附录 A.1）。我们有意使初始超智能体的实现（即任务智能体与元智能体二者）保持简单，以便留出宽广的搜索空间，并避免过早约束可能改进的范围。附录 D.1 详述各实验所使用的基础模型与超参数，附录 D.2 报告相应的成本估计。

### 4.1 基线

我们将带有超智能体的达尔文-哥德尔机（DGM-H）与一组基线进行比较，这些基线旨在隔离自我改进的元智能体、开放式探索，以及元认知自我修改（即超智能体）各自的贡献（附录 B）。无自我改进元智能体的 DGM-H（DGM-H w/o self-improve）在整个运行过程中，把修改其他超智能体的元智能体固定为初始元智能体。系统仍然可以生成并评估新的超智能体，但用于提出并应用修改的过程本身不可修改。这一基线在本设定中复现了 (ADAS, Hu et al., 2025) 的方法。它检验表现增益究竟仅仅来自固定元层面机制之下的任务层面优化，还是也来自自我改进过程本身的改进。无开放式探索的 DGM-H（DGM-H w/o open-ended exploration）去掉先前已生成超智能体的存档。每个新生成的超智能体替换其前代，先前变体不再被保留为未来探索的踏脚石。生成一个新的超智能体之后，它接受评估，并自动成为下一个被选中的父代。这一基线隔离了开放式的、基于种群的搜索所扮演的角色。与这一基线比较，是为了检验持续改进是否需要累积并复用多样的中间解。

我们还与达尔文-哥德尔机（DGM）的原始实现进行比较 (Zhang et al., 2025b)。DGM 依赖一种手工设计的、固定的机制来生成自我改进指令。由于这一指令生成机制是专门为编程基准设计的，并且不可修改，DGM 在其他领域中的自我改进能力是有限的（第 2 节）。为提供更强的比较，我们针对目标领域手工定制 DGM 的指令生成机制（DGM-custom）（附录 B）。这一基线衡量 DGM 在多大程度上依赖人类工程，才能在各个领域保持竞争力。将 DGM-H 与这一基线比较，是为了检验自动化的元认知自我修改能否优于人类设计的自我改进机制。此外，我们还与先前工作中为每个领域手工设计的静态解进行比较。

### 4.2 领域

我们在多种领域（即编程、论文评审、机器人奖励设计，以及奥林匹克级别数学评分）上评估我们的方法与基线（附录 C）。为降低计算成本，对每个领域，我们首先在训练任务的一个小子集上评估智能体，以估计总体有效性。只有展现出足够表现的智能体，才会随后在其余训练任务上接受评估。未达到这一标准的智能体，在未评估任务上的表现被视为零。各领域特定的评估协议在随后各段中详细描述。对于我们创建 AI 评判器以反映人类数据的领域（即论文评审与奥林匹克级别数学评分），我们构建一个验证子集，因为 AI 评判器更可能对训练数据过拟合。当某个领域定义了验证子集时，父代选择中使用的表现分量在验证集上测量；否则在训练集上测量。每个领域都包含单独的留出测试任务，这些任务仅用于最终评估。

**编程。** 我们选择 Polyglot (Gauthier, 2024) 作为计算成本高效的编程基准，以便与先前工作直接比较 (Zhang et al., 2025b)。在这一基准中，智能体得到一个代码仓库，以及一条描述所需变更的自然语言指令，并且必须相应地修改该仓库。我们遵循 DGM 中使用的实验设置 (Zhang et al., 2025b)，包括相同的训练与测试划分、不设验证集，以及相同的分阶段评估协议（即先在 10 个任务上评估每个智能体以估计有效性，然后再扩展到另外 50 个任务）（附录 C.1）。

**论文评审。** 这一领域在模拟的会议同行评审任务上评估智能体。对每个任务，智能体得到一篇人工智能研究论文的全文，并且必须预测一个二元的接收/拒稿决定。我们纳入论文评审，是为了在一种难以核验、且不存在客观真值的设定中评估 DGM-H。同行评审是主观的，审稿决定会因优先事项与视角不同而变化。我们并不旨在改变同行评审系统，而是研究超智能体能否自动学习与所观察到的人类判断相一致的决策过程。智能体输出单个录用决定，表现通过将预测与观察到的录用结果相比较来衡量。数据集取自 Zhao et al. (2026)，该工作根据近期顶级机器学习会议公开可得的投稿与录用决定，构建了一个大规模基准。这一领域有代表性的静态基线是 AI-Scientist-v2 中的审稿智能体 (Yamada et al., 2025)。附录 C.2 提供关于数据集划分（训练、验证与测试）、分阶段评估协议（即先在一个 10 任务子集上评估每个智能体以估计有效性，然后再把评估扩展到共计 100 个任务），以及这一领域有代表性的基线的完整细节。

**机器人奖励设计。** 这一领域评估智能体为机器人任务设计奖励函数的能力。我们纳入这一领域，是为了超出纯语言任务，并表明超智能体能够利用外部模拟器（例如物理引擎）与训练算法（例如强化学习（RL））来产生有效解。给定对一个机器人任务的自然语言描述，智能体必须生成一个合适的奖励函数。该奖励函数随后被用于在模拟中用 RL 训练一个四足机器人 (Genesis, 2024)。智能体所给解的质量由所得策略的表现来衡量：在用所生成的奖励函数训练之后，我们评估机器人实现期望行为的程度 (Ma et al., 2024)。我们使用分开的训练任务与测试任务。在训练期间，智能体需要生成使机器人能够向前行走的奖励函数。对于留出测试，智能体必须零样本地生成新的奖励函数，以最大化机器人的躯干高度。由于成功使机器人向前行走的奖励函数并不会诱发跳跃行为（对于最大化机器人躯干高度而言更优的行为），这一设置评估单个智能体能否为不同的机器人任务设计合适的奖励函数。这一领域没有单独的验证任务。附录 C.3 提供关于分阶段评估协议（即先在训练任务的 3 次重复上评估每个智能体以估计有效性，然后再把评估扩展到共计 6 次重复），以及这一领域有代表性的基线的完整细节。

**奥林匹克级别数学评分。** 这一领域评估智能体为奥林匹克级别数学问题的解答评分的能力。这一领域被保留为留出的元评估，用以检验 DGM-H 对其自我改进过程的改进是否跨领域迁移，并继续随时间叠加。我们使用 IMO-GradingBench (Luong et al., 2025)，它由国际数学奥林匹克（IMO）级别的问题、与之配对的候选解答，以及专家人类评分组成。对每个任务，智能体得到一道 IMO 级别的问题、一份候选解答、参考解答以及评分指南，以预测一个离散分数。表现由智能体评分相对于专家人类评分的准确率来衡量。这一领域有代表性的静态基线是 IMO-GradingBench 中的 ProofAutoGrader。附录 C.4 提供关于分数标签、数据集划分（训练、验证与测试）、分阶段评估协议（即先在一个 10 任务子集上评估智能体以估计有效性，然后再把评估扩展到共计 100 个任务），以及这一领域基线的完整细节。

## 5 结果

对每个实验，我们将每种方法运行 5 次。我们报告中位数，以及由 1,000 次重抽样计算得到的 95% 自助法置信区间，记法为中位数（CI：下界 – 上界）。在折线图中，线条表示中位数表现，阴影区域表示置信区间（图 2、图 3 与图 4）。条形图报告留出测试集上的中位数表现，误差棒表示置信区间（图 2、图 3 与图 4）。统计显著性使用 Wilcoxon 符号秩检验评估。总体而言，DGM-H 在任务层面与元层面都展现出一般性的自我改进。对任务智能体的改进迁移到每个领域内的留出测试任务，并超过开源的静态基线（第 5.1 节）。元层面的改进跨领域迁移，使超智能体能够显著提升其在先前未见领域中生成更好任务智能体的能力（第 5.2 节）。在一次 DGM-H 运行中学到的自我改进，有可能加速后续运行中的学习，并在进一步的自我修改被应用时继续叠加（第 5.3 节）。全部实验日志均在我们的代码库中开源。

### 5.1 提升任务表现

DGM-H 可以在编程上取得与先前自我改进算法相当的自我改进。在 Polyglot 编程基准上，我们使用与 DGM 相同的实验设置（例如相同的基础模型参数、同样的 80 次迭代），以便直接比较。在 5 次运行中，DGM-H 将其在 50 任务 Polyglot 子集上的训练表现从 0.140（初始智能体）提升到 0.340（CI：0.300 – 0.380）。在主要由训练期间未见任务构成的完整 Polyglot 基准上评估时，表现从 0.084（初始智能体）提升到 0.267（CI：0.231 – 0.280）。这些改进与原始 DGM 所报告的改进相当：原始 DGM 在训练子集上从 0.140 提升到 0.380，在完整基准上从 0.142 提升到 0.307 (Zhang et al., 2025b)。总体而言，这些结果表明，DGM-H 能够在编程领域有效地自我改进，并达到与原始 DGM 相似的改进水平，尽管它并非专门为编程任务而手工设计。

**图 2：** 元认知自我修改与开放式探索使 DGM-H 能够持续取得进展并提升其表现。（左）DGM-H 可以在同一次运行中针对多样任务进行优化，并自动发现越来越好的任务智能体。（右）依据验证分数或训练分数选出的、被发现的最佳任务智能体，在（右上）论文评审与（右下）机器人奖励设计的测试任务上接受评估。DGM-H 优于缺少元认知（即原始 DGM）、自我改进或开放式探索的基线，表明全部组成部分对于持续的自我改进都是必要的。

在编程之外，我们在论文评审与机器人奖励设计上评估 DGM-H，以考察它在单次运行中跨多样任务进行自我改进的能力（附录 A.4）。若某个智能体在任一领域未通过分阶段评估，则它不会在任何领域的完整训练集上接受评估，并且所有剩余任务都被赋予零分。例如，当在同一次实验运行中联合优化论文评审与机器人奖励设计这两个领域时，任一领域的分阶段评估失败（例如在较小的训练子集中一篇论文评审都没有预测正确，或者未能生成任何可编译的奖励函数），都会导致该智能体不会在任一领域的完整训练集上接受评估。只有经由验证分数（在不存在验证任务时则用训练分数）选出的最佳智能体，才会在测试集上接受评估。所有运行都从同一个初始智能体出发，该智能体直接输出单次基础模型调用的响应（附录 A.1）。这一初始智能体在论文评审上的测试集分数为 0.0，在机器人奖励设计上为 0.060（图 2）。在论文评审中，由于没有提供任务特定的解析或后处理，初始智能体未能以所要求的格式产生输出，因此分数为 0.0。在机器人奖励设计中，初始智能体通常生成的奖励函数要么无法编译（例如由于无效的环境变量），要么不能产生有效策略。然而，初始智能体偶尔会产生一个最低限度可用的奖励，从而导致一个很小但非零的平均分数。从这一初始智能体出发，每种方法运行 100 次迭代。

原始 DGM 需要领域特定的定制，才能推广到编程之外。当以专门为优化编程智能体而设计的形式加以应用时，原始 DGM 在论文评审上的测试集表现为 0.0（CI：0.0 – 0.510），在机器人奖励设计上为 0.0（CI：0.0 – 0.090）。这一结果是预期之中的，因为原始 DGM 的指令生成机制被明确地针对编程任务以及改进编程智能体而定制（附录 B）。然而，当针对每个新领域进行手工定制时，DGM-custom 取得明显更强的结果：论文评审上为 0.590（CI：0.570 – 0.650），机器人奖励设计上为 0.348（CI：0.305 – 0.385）。DGM-custom 在两个领域都显著优于原始 DGM（$p < 0.05$）（图 2）。这些结果既凸显了 DGM 在其预期领域内的强度，也凸显了领域特定的手工定制对于在其原始实现中把自我改进能力扩展到编程之外的重要性。

元认知自我修改使跨领域的任务改进得以持续。DGM-H 在两个领域中都随时间大幅提升任务表现，表明对任务智能体的有效改进。重要的是，这些增益迁移到留出测试集，表明改进反映的是一般能力，而不是对训练任务的过拟合。在测试任务上，DGM-H 将论文评审表现从 0.0（初始智能体）提升到 0.710（CI：0.590 – 0.750）（附录 E.1.1）。若干表现最好的智能体超过了开源静态基线，后者达到 0.630 (Yamada et al., 2025)。在机器人奖励设计上，DGM-H 将表现从 0.060（初始智能体）提升到 0.372（CI：0.355 – 0.436）（附录 E.1.2），超过了直接优化评估指标的默认奖励函数（0.348）。

从定性上看，DGM-H 改进任务智能体的方式，是从表面层次的提示词微调，走向结构化的、可复用的决策机制。在论文评审中，它从肤浅的行为指令（例如采用一种“严谨”人设）转向带有检查清单、决策规则和明确定义标准的显式多阶段评估流程，从而产生更一致、质量更高的判断（附录 E.2）。在机器人奖励设计中，DGM-H 逐步建立并精炼一个关于环境约束、有效状态变量和奖励缩放启发式的内部知识库，消除编译失败并减少奖励误设。DGM-H 累积并精炼领域知识（例如环境文档、评分标准），并将其整合进日益精细的决策框架，从而在没有人工的、领域特定工程的情况下实现复杂且一致的任务行为（附录 E.2）。

DGM-H 在两个领域都显著优于原始 DGM（$p < 0.05$），这反映了当原始 DGM 固定的指令生成机制被应用于非编程任务时的局限（图 2）。DGM-H 在两个领域的测试任务上取得了高于 DGM-custom 的中位数表现，尽管差异在统计上不显著（$p > 0.05$）（图 2）。值得注意的是，在机器人奖励设计的测试任务中（即设计一个奖励函数，用以训练机器人最大化其躯干高度），DGM-H 几乎总是生成这样的任务智能体：它们设计的奖励函数会诱发跳跃行为，而不是陷入“站得高”这一局部最优。与此相对，DGM-custom 经常生成无法逃离这一局部最优的任务智能体。这一质性差异表明，元认知自我修改可以达到或超过手工设计的、领域特定的机制。

元认知自我修改与开放式探索对于持续改进都是必要的。DGM-H w/o self-improve (Hu et al., 2025) 几乎没有改进，其测试集表现在论文评审上为 0.0（CI：0.0 – 0.130），在机器人奖励设计上为 0.213（CI：0.180 – 0.348）。DGM-H 在两个领域都显著优于这一基线（$p < 0.05$）（图 2），表明 DGM-H 中观察到的增益并非仅仅来自固定元智能体之下的任务层面优化，也来自自我修改过程本身（即元智能体）的改进。类似地，DGM-H w/o open-ended exploration 未能取得有意义的改进，其测试集表现在论文评审上停留在 0.0（CI：0.0 – 0.560），在机器人奖励设计上为 0.116（CI：0.0 – 0.348）。DGM-H 在两个领域都显著优于这一基线（$p < 0.05$）（图 2），这凸显了开放式的、基于种群的搜索对于为未来改进创造踏脚石的重要性。

### 5.2 提升改进的能力

DGM-H 相对于 DGM-H w/o self-improve 的更优表现表明，DGM-H 改进了元智能体（即它自我修改并生成新智能体的能力）（第 5.1 节）。本节考察这些元层面改进是一般性的，还是领域特定的。具体而言，我们评估在一种设定中学到的自我改进策略，是否能够迁移到另一个领域并加速其中的学习。

**improvement@k 指标。** 为量化元智能体生成改进后任务智能体的能力，我们引入 improvement@k（imp@k）指标。给定初始元智能体 $M$、初始任务智能体 $A$、一种智能体生成算法（例如 DGM 或 DGM-H 的变体）以及一项固定任务，$M$ 被允许使用指定算法，从 $A$ 及其后代出发，最多生成 $k$ 个新的任务智能体。我们将 imp@k 定义为测试表现从初始任务智能体 $A$ 到表现最好的所生成任务智能体（依据验证分数选出）的提升（附录 D.3）。直观上，imp@k 衡量一个元智能体在有限迭代次数内产生改进变体的有效程度。为了把元智能体改进任务智能体的能力与其改进自身的能力区分开来，我们在整个过程中保持元智能体固定（即，负责生成新任务智能体的智能体始终是初始元智能体 $M$），从而得到 DGM w/o self-improve 与 DGM-H w/o self-improve 作为智能体生成算法。本节全部实验运行 50 次迭代，测量 imp@50。

**图 3：** DGM-H 在一种设定中学到的自我改进策略，能够迁移到另一种设定并加速其中的学习。我们用 imp@50 衡量一个智能体生成改进后智能体的能力；imp@50 以一个起始智能体、一种智能体生成算法和一项评估任务作为输入。该算法从给定智能体出发运行 50 次迭代，imp@50 定义为最佳所生成智能体相对于起始智能体在该任务上的表现增益。（左，DGM w/o self-improve 与 DGM-H w/o self-improve）在奥林匹克级别数学评分上，从初始智能体出发，DGM w/o self-improve 与 DGM-H w/o self-improve 都几乎没有改进，表明初始智能体生成更好智能体的能力有限。（左，DGM w/o self-improve + transfer）从迁移智能体出发，DGM w/o self-improve 同样几乎没有改进，表明原始 DGM 并没有学到可迁移的元层面改进。（左，DGM-H w/o self-improve + transfer）与此相对，从迁移超智能体出发，DGM-H w/o self-improve 取得了实质改进，表明超智能体使 DGM-H 能够获得可迁移的自我改进策略。（中）无论从初始智能体还是从迁移智能体出发，DGM w/o self-improve 给出的 imp@50 $\approx$ 0，表明原始 DGM 并没有改进“生成改进后智能体”这一元层面技能。（右）从迁移超智能体出发，DGM-H w/o self-improve 取得很大的正 imp@50，表明 DGM-H 所学到的自我改进策略是通用且可迁移的，并且 DGM-H 提升了它进行改进的能力。

初始元智能体几乎没有生成改进后任务智能体的能力。采用与上述实验相同的初始元智能体与任务智能体（第 4 节），我们在奥林匹克级别数学评分上评估初始元智能体生成改进后任务智能体的能力。初始任务智能体的测试集分数为 0.0，因为它由于缺少任务特定的解析或后处理，未能以所要求的格式产生输出。在 DGM w/o self-improve 的 50 次迭代之后——在此期间初始元智能体保持固定，并被用于从一个初始任务智能体出发生成新的任务智能体——最佳所生成任务智能体（跨 5 次重复运行）的中位数测试分数为 0.0（CI：0.0 – 0.0）。相应地，在 DGM w/o self-improve 下，初始元智能体在奥林匹克级别数学评分上的 imp@50 为 0.0（CI：0.0 – 0.0）。DGM w/o self-improve 依赖固定的、手工设计的指令生成机制，而 DGM-H w/o self-improve 并不依赖这种机制。DGM w/o self-improve 使用手工设计的指令生成机制来指定应当修改智能体代码库的哪些部分，而 DGM-H w/o self-improve 允许元智能体自主决定修改代码库的哪些部分。在 DGM-H w/o self-improve 的 50 次迭代之后，同样保持初始元智能体固定，并从同一个初始任务智能体生成任务智能体，最佳所生成任务智能体（跨 5 次重复运行）的中位数分数为 0.0（CI：0.0 – 0.130）。相应地，在 DGM-H w/o self-improve 下，初始元智能体的 imp@50 为 0.0（CI：0.0 – 0.130）（图 3）。这些结果表明，在奥林匹克级别数学评分上，无论使用哪一种智能体生成算法（例如 DGM 或 DGM-H），初始元智能体都几乎没有生成改进后任务智能体的能力。

在先前 DGM-custom 运行中优化得到的元智能体，在迁移到新领域时几乎没有生成改进后任务智能体的能力。我们评估使用 DGM 在一种设定中学到的自我改进能否迁移到新领域。我们考虑在论文评审与机器人奖励设计任务上、于先前 DGM-custom 运行中优化得到的智能体（第 5.1 节）。从每次运行中，我们依据其后代所取得的最大表现增益选出一个迁移智能体；该增益相对于该智能体自身来测量，并按谱系深度打折（附录 D.4）。我们应用折扣，以偏好那些在较少自我修改步骤内可靠地产生强改进的智能体，因为发生在许多次迭代之后的改进，较不直接地可归因于原始智能体。我们从 DGM-custom 而不是从 DGM 中选择迁移智能体，因为 DGM-custom 在先前设定中取得了更大的改进（第 5.1 节）。对于迁移，我们把源领域的整个智能体实现一并带入，包括元智能体与任务智能体。具体而言，迁移元智能体被用作在目标领域中负责生成新任务智能体的固定智能体，迁移任务智能体则作为初始任务智能体，新的变体由此生成。我们评估迁移元智能体能否在一个新领域——奥林匹克级别数学评分——中生成改进后的任务智能体。迁移任务智能体（来自 5 次重复运行中的每一次）的测试集分数为 0.0（CI：0.0 – 0.010）。正如预期，这一表现很低，因为这些智能体是为论文评审与机器人奖励设计而优化的，这两个任务与数学评分任务有实质差异。在从每个迁移任务智能体出发、并保持迁移元智能体固定的 DGM w/o self-improve 的 50 次迭代之后，最佳所生成任务智能体（依据验证分数选出）的分数为 0.0（CI：0.0 – 0.020）。这对应于 imp@50 为 0.0（CI：0.0 – 0.010）（图 3），表明来自 DGM-custom 运行的迁移元智能体不能在先前未见的领域中生成改进后的任务智能体。当使用 DGM w/o self-improve 作为智能体生成算法时，迁移智能体的 imp@50 与初始智能体的 imp@50 相似（二者都接近于零）。这表明，用 DGM-custom 观察到的任务表现改进依赖于领域特定的定制，而不是元能力在“生成改进后任务智能体”上的一般性改进。

在先前 DGM-H 运行中优化得到的超智能体，展现出在新领域中生成改进后任务智能体的可迁移能力。我们对 DGM-H 重复迁移实验，以评估超智能体是否使一般性自我改进机制的学习成为可能。从先前在论文评审与机器人奖励设计上的 DGM-H 运行中（第 5.1 节），我们使用上文所述的同一谱系折扣准则，为每次运行选出一个迁移超智能体（附录 D.4）。迁移超智能体（来自 5 次重复运行中的每一次）的初始测试集分数为 0.0（CI：0.0 – 0.0），这反映了领域偏移。在从迁移超智能体出发的 DGM-H w/o self-improve 的 50 次迭代之后，最佳所生成智能体（依据验证分数选出）的测试集分数为 0.630（CI：0.540 – 0.630）。这对应于 imp@50 为 0.630（CI：0.540 – 0.630）（图 3）。这些结果表明，迁移超智能体可以在先前未见的领域中生成改进后的智能体。当使用 DGM-H w/o self-improve 作为智能体生成算法时，迁移智能体的 imp@50 显著高于初始智能体的 imp@50（$p < 0.05$）。这表明迁移智能体在生成改进后智能体方面要有效得多，并且通过 DGM-H 在一次运行中学到的元改进是一般性的、可迁移的，能够加速另一个领域中的学习。

我们在定性上把所观察到的迁移增益归因于一组通用的元层面能力，这些能力是 DGM-H 在先前运行中自主获得的。具体而言，迁移超智能体具有性能追踪与持久记忆等特征，使它们能够把改进当作一个持续过程来推理，而不是当作孤立的代码编辑（附录 E.3）。因此，即使被迁移到一个未见领域，这些超智能体也能够迅速自我改进并取得有意义的进展（图 3）。这与 DGM 的迁移智能体形成对照：后者的增益依赖领域特定的定制，并且并不改进底层的智能体生成过程本身。这些定性观察表明，DGM-H 学会了如何改进，从而产生一般且可迁移的自我改进能力。

一个例子是 DGM-H 自主发展出的性能追踪基础设施。超智能体并不依赖孤立的评估结果，而是跨迭代记录、聚合并比较性能指标，识别哪些变更带来持续改进、哪些变更导致退步。这种性能追踪支持有依据的决策，例如优先考虑有希望的迭代，并避开没有成效的修改方向。下面的片段展示了一个被自动引入的性能追踪器，它在各次迭代中记录并组织指标：

[⬇](<)

```python
class PerformanceTracker:

"""Tracks performance metrics across agent generations."""

def __init__(self, tracking_file: str = "./outputs/performance_history.json"):

self.tracking_file = tracking_file

self.history = self._load_history()

def record_generation(self, generation_id: int, domain: str,

score: float, metadata: dict = None):

"""Record performance for a generation."""

entry = {

"generation_id": generation_id,

"domain": domain,

"score": score,

"timestamp": datetime.now().isoformat(),
```

"metadata": metadata or {}

}

self.history.append(entry)

self._save_history()

def get_improvement_trend(self, domain: str = None, window: int = 5):

"""Calculate improvement trend using moving average."""

filtered = self.history

if domain:

filtered = [h for h in self.history if h.get(’domain’) == domain]

if len(filtered) < window * 2:

return None

recent_avg = sum(h[’score’] for h in filtered[-window:]) / window

older_avg = sum(h[’score’] for h in filtered[-window*2:-window]) / window

return recent_avg - older_avg # Positive if improving

def get_statistics(self, domain: str = None):

"""Get comprehensive statistics."""

scores = [h[’score’] for h in filtered]

return {

"total_generations": len(filtered),

"best_score": max(scores),

"worst_score": min(scores),

"average_score": sum(scores) / len(scores),

"improvement_trend": self.get_improvement_trend(domain)

}

Another example is DGM-H’s innovation of persistent memory, which enables learning to accumulate across iterations. Instead of merely logging numerical scores, the hyperagent stores synthesized insights, causal hypotheses, and forward-looking plans (e.g., identifying which generations performed best, diagnosing over-corrections, and proposing how to combine successful strategies). This memory is actively consulted during subsequent self-modification steps, allowing later generations to build on earlier discoveries and avoid repeating past mistakes. This is an example of a stored memory entry:

[⬇](<)

{

"best_performers_analysis": {

"value": "Best Performing Generations:\n\nPaper Review:\n

- Gen55: 63% acc, 25% accept rate, 38% accept recall, 88% reject recall (too harsh)\n

- Gen64: 61% acc, 53% accept rate, 64% accept recall, 58% reject recall (BEST BALANCE)\n\n

Genesis Go2Walking:\n- Gen62: 0.802 fitness (BEST)\n- Gen63: 0.798 fitness\n\n

Key Insight: Gen55 has best accuracy but is too harsh. Gen64 improved balance

but lower accuracy. Need to combine gen55’s critical reasoning with gen64’s balance.",

"timestamp": "2026-01-07T00:43:27.550275"

},

"gen66_results": {

"value": "Gen 66 Results (Latest): Paper Review - 52% accuracy (WORSE than gen64’s 61%),

94% accept rate (EXTREME accept bias). CRITICAL ISSUE: Paper review went backwards

- gen65 changes over-corrected.",

"timestamp": "2026-01-07T01:03:41.176643"

},

"iteration_66_improvement": {

"value": "Iteration 66 Improvement: CORRECTED paper review prompt after gen65 over-correction.

New approach: (1) Clear REJECT criteria with specific triggers, (2) Decision Protocol

with explicit thresholds (~40%/~60%), (3) Borderline guidance defaults to REJECT.

Target: 62-64% accuracy with 55-65% reject rate.",

"timestamp": "2026-01-07T01:06:35.559820"

}

}

### 5.3 复合式自我改进

图 4 DGM-H 所学到的自我改进会跨领域、跨运行累积。我们继续在奥林匹克级别数学评分上运行 DGM-H，从先前 DGM-H 运行中获得的迁移超智能体出发，并将其与从初始智能体初始化的 DGM-H 以及从 ProofAutoGrader 初始化的 DGM-H 进行比较。（左）从已迁移的超智能体进行初始化，会比从初始智能体进行初始化带来更快的进展和更高的最终性能，表明先前学到的自我改进仍然有用，并在新领域中继续复合累积。（右）从已迁移的智能体与 ProofAutoGrader 初始化的 DGM-H 取得了最高的测试性能，超过了代表性基线。

我们考察 DGM-H 在一种设定中学到的自我改进，当 DGM-H 在另一种设定中运行时是否会继续累积。根据先前在论文评审与机器人奖励设计任务上的 DGM-H 运行（第 5.1 节），我们使用前文所述的同一选择机制（第 5.2 节，附录 D.4）选出迁移超智能体。随后，我们评估它们在新领域——奥林匹克级别数学评分——中继续自我改进的能力。从这些迁移智能体出发运行 DGM-H 共 200 次迭代（DGM-H + transfer）之后，最佳生成智能体（依据验证分数选出）取得的测试集分数为 0.640 (CI: 0.550 – 0.720)。在相同的实验设定下，从初始智能体出发的 DGM-H 取得的最佳测试集分数为 0.610 (CI: 0.510 – 0.680)。尽管 DGM-H + transfer 与 DGM-H 之间的差异在统计上不显著（$p > 0.05$），DGM-H + transfer 取得了比从初始智能体出发的 DGM-H 更高的中位数性能，其置信区间也更高（图 4）。值得注意的是，由于饱和效应，在更高的性能水平上改进会越来越困难（例如，将性能从 0.7 提升到 0.8，通常比从 0.0 提升到 0.1 更具挑战），因此这些增益尽管绝对幅度不大，仍然有意义。这些结果表明，DGM-H 的自我改进是可复用的，并有可能跨运行累积，从而支持自我改进随时间复合增长的可能性。

来自 IMO-GradingBench、用于奥林匹克级别数学评分的代表性静态基线是 ProofAutoGrader (Luong et al., 2025)。我们以 ProofAutoGrader 作为任务智能体，并以来自先前一次 DGM-H 运行（针对论文评审与机器人奖励设计）的迁移元智能体来初始化 DGM-H，然后继续针对奥林匹克级别数学评分进行优化。经过 200 次迭代后，所发现的最佳智能体取得的测试集分数为 0.700，超过了 ProofAutoGrader 的分数 0.670（图 4）。随后，我们在完整的 IMO-GradingBench 上同时评估所发现的最佳智能体与 ProofAutoGrader，以便更准确地估计这一改进。在完整的 IMO-GradingBench 上，DGM-H 将 ProofAutoGrader 的准确率从 0.561 提升到 0.601，并将平均绝对误差从 0.178 降低到 0.175（附录 E.4）。我们将这一产物开源，以支持未来的研究与开发（附录 E.1.3）。这些结果表明，DGM-H 能够建立在已有的强解决方案之上，并进一步提升其性能。

## 6 安全性讨论

DGM-超智能体（DGM-Hyperagents，DGM-H）因其能够随时间自主修改自身行为及其改进机制，而引入了独特的安全性考量。在本工作中，所有实验均在严格的安全约束下进行。具体而言，由智能体生成的代码在经过仔细沙箱隔离的环境中执行，并施加了强制资源限制（例如，超时、受限的互联网访问）。这些措施旨在防止非预期的副作用、遏制失败，并确保自我修改始终被限定在预期的实验范围之内。此外，评估使用预定义的任务与指标进行，并且在所有实验过程中始终保持人类监督。

演化速度有可能快于人类监督。随着人工智能系统获得以日益开放的方式修改自身的能力，它们有可能以远超人类所能审计或解释的速度演化。在这种爆发式能力增长的临界点上，有必要重新思考人工智能系统在社会中所扮演的角色 (Bengio et al., 2024)。与其仅仅从绝对保证或完全可解释性的角度来框定安全性，一个核心挑战在于平衡人工智能作为人类进步与福祉之催化剂的潜力（例如，自动化科学发现）与人类愿意对这些系统寄予的信任程度（例如，在不要求持续人类核验的情况下委派决策或行动），同时尽量降低诸多潜在风险与负面影响 (Clune, 2019; Ecoffet et al., 2020; Bengio et al., 2024; Weston and Foerster, 2025)。这种平衡由透明性与可控性等因素所塑造。

尽管 DGM-H 在安全的研究边界内运行（例如，沙箱隔离、受控评估），随着自我改进系统变得更加强大，这些保障措施可能会日益难以维持，乃至变得不可行。我们在附录 F 中讨论额外的安全性考量。我们主动纳入这一讨论，以鼓励更广泛地参与探讨：对于开放式自我改进的人工智能系统而言，安全性意味着什么 (Clune, 2019; Ecoffet et al., 2020; Sheth et al., 2025)。这包括关于适当的信任、监督与透明程度的持续讨论，以及关于这些系统在部署时应优先追求哪些益处的社会审议。

## 7 局限性与结论

本工作引入超智能体（hyperagents），并将其纳入达尔文哥德尔机（Darwin Gödel Machine，DGM），从而形成 DGM-超智能体（DGM-Hyperagents，DGM-H）。DGM-H 是一个通用的自我改进框架，它以开放式方式为任何可计算任务演化出一个由自我改进超智能体构成的档案库，使该系统既能提升任务性能，也能改进其自身的自我改进机制。在多个不同领域中，DGM-H 在任务性能上产生了可观且可泛化的增益，同时也提升了其生成改进的能力，而这些元层面的增益能够跨领域迁移，并跨运行复合累积。

我们的结果表明，自我改进可以在不同的实验设定之间复合累积，但这一版本的 DGM-H 存在制约真正无界进展的局限性。第一，它在固定的任务分布上运行。一个方向是通过生成适应智能体能力的新任务与课程，来协同演化任务分布 (Clune, 2019; Zhang et al., 2024; Faldor et al., 2025; Bolton et al., 2025)。第二，开放式探索循环的组成部分（例如，父代选择、评估协议）仍然保持固定。尽管超智能体可以修改其自我改进机制，它们却无法改变决定哪些智能体被选出、以及如何评估这些智能体的外层过程。将这些组成部分保持固定，提高了实验的稳定性与安全性，但限制了完全的自我可修改性。使超智能体能够修改这些外层循环组成部分，并调整其自身的搜索策略与评估过程，是未来工作的另一个有前景的方向。我们的初步结果表明，此类扩展是可行的（附录 E.5）。

DGM-H 表明，开放式自我改进可以在多个不同领域中变得切实可行。在充分的安全性考量得到妥善处理的前提下，DGM-H 指出了一条通向自我加速系统的路径：这类系统不仅搜索更好的解决方案，而且持续改进其自我改进的能力。

## 致谢

我们感谢 Andrew Budker 与 Ricardo Silveira Cabral 对本工作的支持，并感谢 Alisia Lupidi、Chenxi Whitehouse、John Quan、Lisa Alazraki、Lovish Madaan、Lucia Cipolina-Kun、Mattia Opper、Michael Dennis、Parth Pathak、Rishi Hazra、Roberta Raileanu、Sandra Lefdal、Shashwat Goel、Shengran Hu、Timon Willi、Tim Rocktäschel 与 Yoram Bachrach 富有洞见的讨论与反馈。

## 作者贡献

Jenny Zhang 主导了本研究的概念化，开展了实验，并撰写了文稿。Bingchen Zhao 与 Wannan Yang 参与了实验设计与执行。Jakob Foerster、Jeff Clune、Minqi Jiang、Sam Devlin 与 Tatiana Shavrina 对方法论与文稿提供了反馈。全体作者审阅并批准了最终文稿。

## 参考文献

  * Audran-Reiss et al. (2025) Alexis Audran-Reiss, Jordi Armengol-EstapÃŠ, Karen Hambardzumyan, Amar Budhiraja, Martin Josifoski, Edan Toledo, Rishi Hazra, Despoina Magka, Michael Shvartsman, Parth Pathak, et al. What Does It Take to Be a Good AI Research Agent? Studying the Role of Ideation Diversity.  arXiv preprint arXiv:2511.15593.  Cited by: §2.
  * Auer et al. (2002) Peter Auer, Nicolo Cesa-Bianchi, and Paul Fischer Finite-time analysis of the multiarmed bandit problem.  Machine learning 47 (2), pp. 235–256.  Cited by: §E.5.
  * Beaulieu et al. (2020) Shawn Beaulieu, Lapo Frati, Thomas Miconi, Joel Lehman, Kenneth O Stanley, Jeff Clune, and Nick Cheney Learning to continually learn.  arXiv preprint arXiv:2002.09571.  Cited by: §2.
  * Bengio et al. (2024) Yoshua Bengio, Geoffrey Hinton, Andrew Yao, Dawn Song, Pieter Abbeel, Trevor Darrell, Yuval Noah Harari, Ya-Qin Zhang, Lan Xue, Shai Shalev-Shwartz, et al. Managing extreme AI risks amid rapid progress.  Science 384 (6698), pp. 842–845.  Cited by: §6.
  * Bolton et al. (2025) Adrian Bolton, Alexander Lerchner, Alexandra Cordell, Alexandre Moufarek, Andrew Bolt, Andrew Lampinen, Anna Mitenkova, Arne Olav Hallingstad, Bojan Vujatovic, Bonnie Li, et al. Sima 2: A generalist embodied agent for virtual worlds.  arXiv preprint arXiv:2512.04797.  Cited by: §7.
  * Bradley et al. (2023) Herbie Bradley, Andrew Dai, Hannah Teufel, Jenny Zhang, Koen Oostermeijer, Marco Bellagente, Jeff Clune, Kenneth Stanley, Grégory Schott, and Joel Lehman Quality-diversity through AI feedback.  arXiv preprint arXiv:2310.13032.  Cited by: §2.
  * Brown et al. (2020) Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, et al. Language models are few-shot learners.  Advances in neural information processing systems 33, pp. 1877–1901.  Cited by: §4.
  * Chalvidal et al. (2022) Mathieu Chalvidal, Thomas Serre, and Rufin VanRullen Meta-reinforcement learning with self-modifying networks.  Advances in Neural Information Processing Systems 35, pp. 7838–7851.  Cited by: §2.
  * Chen et al. (2026) Yinzhu Chen, Abdine Maiga, Hossein A Rahmani, and Emine Yilmaz Automated Rubrics for Reliable Evaluation of Medical Dialogue Systems.  arXiv preprint arXiv:2601.15161.  Cited by: §E.2.
  * Clune (2019) Jeff Clune AI-GAs: AI-generating algorithms, an alternate paradigm for producing general artificial intelligence.  arXiv preprint arXiv:1905.10985.  Cited by: §2, §6, §6, §7.
  * Coiffard et al. (2025) Lisa Coiffard, Paul Templier, and Antoine Cully Overcoming Deceptiveness in Fitness Optimization with Unsupervised Quality-Diversity.  In Proceedings of the Genetic and Evolutionary Computation Conference,  pp. 122–130.  Cited by: §2.
  * Colas et al. (2023) Cédric Colas, Laetitia Teodorescu, Pierre-Yves Oudeyer, Xingdi Yuan, and Marc-Alexandre Côté Augmenting autotelic agents with large language models.  In Conference on Lifelong Learning Agents,  pp. 205–226.  Cited by: §2.
  * Cook et al. (2024) Jonathan Cook, Tim Rocktäschel, Jakob Foerster, Dennis Aumiller, and Alex Wang Ticking all the boxes: Generated checklists improve llm evaluation and generation.  arXiv preprint arXiv:2410.03608.  Cited by: §E.2.
  * Coulom (2006) Rémi Coulom Efficient selectivity and backup operators in Monte-Carlo tree search.  In International conference on computers and games,  pp. 72–83.  Cited by: §A.2.
  * Couto et al. (2024) Paulo Henrique Couto, Quang Phuoc Ho, Nageeta Kumari, Benedictus Kent Rachmat, Thanh Gia Hieu Khuong, Ihsan Ullah, and Lisheng Sun-Hosoya Relevai-reviewer: A benchmark on AI reviewers for survey paper relevance.  arXiv preprint arXiv:2406.10294.  Cited by: §3.
  * Cully et al. (2015) Antoine Cully, Jeff Clune, Danesh Tarapore, and Jean-Baptiste Mouret Robots that can adapt like animals.  Nature 521 (7553), pp. 503–507.  Cited by: §2.
  * Dharna et al. (2025) Aaron Dharna, Cong Lu, and Jeff Clune Foundation model self-play: Open-ended strategy innovation via foundation models.  arXiv preprint arXiv:2507.06466.  Cited by: §2.
  * Ding et al. (2024) Li Ding, Jenny Zhang, Jeff Clune, Lee Spector, and Joel Lehman Quality Diversity through Human Feedback: Towards Open-Ended Diversity-Driven Optimization.  In Forty-first International Conference on Machine Learning,  Cited by: §2.
  * Ecoffet et al. (2020) Adrien Ecoffet, Jeff Clune, and Joel Lehman Open questions in creating safe open-ended AI: Tensions between control and creativity.  In Artificial Life Conference Proceedings 32,  pp. 27–35.  Cited by: §6, §6.
  * Ecoffet et al. (2019) Adrien Ecoffet, Joost Huizinga, Joel Lehman, Kenneth O Stanley, and Jeff Clune Go-explore: a new approach for hard-exploration problems.  arXiv preprint arXiv:1901.10995.  Cited by: §A.2.
  * Faldor et al. (2025) Maxence Faldor, Jenny Zhang, Antoine Cully, and Jeff Clune OMNI-EPIC: Open-endedness via Models of human Notions of Interestingness with Environments Programmed in Code.  In The Thirteenth International Conference on Learning Representations,  Cited by: §2, §7.
  * Fan et al. (2024) Zhiyuan Fan, Weinong Wang, Debing Zhang, et al. Sedareval: Automated evaluation using self-adaptive rubrics.  In Findings of the Association for Computational Linguistics: EMNLP 2024,  pp. 16916–16930.  Cited by: §E.2.
  * Fernando et al. (2023) Chrisantha Fernando, Dylan Banarse, Henryk Michalewski, Simon Osindero, and Tim Rocktäschel Promptbreeder: Self-referential self-improvement via prompt evolution.  arXiv preprint arXiv:2309.16797.  Cited by: §2.
  * Gauthier (2024) Paul Gauthier O1 tops aider’s new polyglot leaderboard.  Note: <https://aider.chat/2024/12/21/polyglot.html>. Accessed: 2026-01-28 Cited by: §C.1, §1, §3, §4.2.
  * Genesis (2024) Authors Genesis Genesis: a generative and universal physics engine for robotics and beyond.  External Links: <https://github.com/Genesis-Embodied-AI/Genesis> Cited by: §C.3, §1, §4.2.
  * Good (1966) Irving John Good Speculations concerning the first ultraintelligent machine.  In Advances in computers,  Vol. 6, Elsevier, pp. 31–88.  Cited by: §2.
  * Grillotti et al. (2025) Luca Grillotti, Lisa Coiffard, Oscar Pang, Maxence Faldor, and Antoine Cully From Tabula Rasa to Emergent Abilities: Discovering Robot Skills via Real-World Unsupervised Quality-Diversity.  arXiv preprint arXiv:2508.19172.  Cited by: §2.
  * Havrilla et al. (2024) Alex Havrilla, Yuqing Du, Sharath Chandra Raparthy, Christoforos Nalmpantis, Jane Dwivedi-Yu, Maksym Zhuravinskyi, Eric Hambro, Sainbayar Sukhbaatar, and Roberta Raileanu Teaching large language models to reason with reinforcement learning.  arXiv preprint arXiv:2403.04642.  Cited by: §2.
  * Herr et al. (2025) Nathan Herr, Tim Rocktäschel, and Roberta Raileanu LLM-First Search: Self-Guided Exploration of the Solution Space.  arXiv preprint arXiv:2506.05213.  Cited by: §A.2.
  * Hu et al. (2025) Shengran Hu, Cong Lu, and Jeff Clune Automated Design of Agentic Systems.  In The Thirteenth International Conference on Learning Representations,  Cited by: Appendix B, §2, §4.1, §5.1.
  * Hughes et al. (2024) Edward Hughes, Michael Dennis, Jack Parker-Holder, Feryal Behbahani, Aditi Mavalankar, Yuge Shi, Tom Schaul, and Tim Rocktaschel Open-endedness is essential for artificial superhuman intelligence.  arXiv preprint arXiv:2406.04268.  Cited by: §2.
  * Hutter (2003) Marcus Hutter A gentle introduction to the universal algorithmic agent AIXI.  Artificial General Intelligence.  Cited by: §2.
  * Irie et al. (2022) Kazuki Irie, Imanol Schlag, Róbert Csordás, and Jürgen Schmidhuber A modern self-referential weight matrix that learns to modify itself.  In International Conference on Machine Learning,  pp. 9660–9677.  Cited by: §2.
  * Jackson et al. (2024) Matthew Thomas Jackson, Chris Lu, Louis Kirsch, Robert Tjarko Lange, Shimon Whiteson, and Jakob Nicolaus Foerster Discovering temporally-aware reinforcement learning algorithms.  arXiv preprint arXiv:2402.05828.  Cited by: §2.
  * Javed and White (2019) Khurram Javed and Martha White Meta-learning representations for continual learning.  Advances in neural information processing systems 32.  Cited by: §2.
  * Jiang et al. (2023) Minqi Jiang, Tim Rocktäschel, and Edward Grefenstette General intelligence requires rethinking exploration.  Royal Society Open Science 10 (6), pp. 230539.  Cited by: §2.
  * Jimenez et al. (2024) Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan SWE-bench: can language models resolve real-world github issues? In The Twelfth International Conference on Learning Representations,  Cited by: §3.
  * Kirsch and Schmidhuber (2022) Louis Kirsch and Jürgen Schmidhuber Eliminating meta optimization through self-referential meta learning.  arXiv preprint arXiv:2212.14392.  Cited by: §1, §2.
  * Klissarov et al. (2023) Martin Klissarov, Pierluca D’Oro, Shagun Sodhani, Roberta Raileanu, Pierre-Luc Bacon, Pascal Vincent, Amy Zhang, and Mikael Henaff Motif: Intrinsic motivation from artificial intelligence feedback.  arXiv preprint arXiv:2310.00166.  Cited by: §2.
  * Klissarov et al. (2025) Martin Klissarov, Mikael Henaff, Roberta Raileanu, Shagun Sodhani, Pascal Vincent, Amy Zhang, Pierre-Luc Bacon, Doina Precup, Marlos C Machado, and Pierluca D’Oro MaestroMotif: Skill Design from Artificial Intelligence Feedback.  In The Thirteenth International Conference on Learning Representations,  Cited by: §2.
  * Kwa et al. (2025) Thomas Kwa, Ben West, Joel Becker, Amy Deng, Katharyn Garcia, Max Hasin, Sami Jawhar, Megan Kinniment, Nate Rush, Sydney Von Arx, et al. Measuring ai ability to complete long tasks.  arXiv preprint arXiv:2503.14499.  Cited by: §2.
  * Lange et al. (2023) Robert Lange, Tom Schaul, Yutian Chen, Tom Zahavy, Valentin Dalibard, Chris Lu, Satinder Singh, and Sebastian Flennerhag Discovering evolution strategies via meta-black-box optimization.  In Proceedings of the Companion Conference on Genetic and Evolutionary Computation,  pp. 29–30.  Cited by: §2.
  * Lehman et al. (2023) Joel Lehman, Jonathan Gordon, Shawn Jain, Kamal Ndousse, Cathy Yeh, and Kenneth O Stanley Evolution through large models.  In Handbook of evolutionary machine learning,  Springer, pp. 331–366.  Cited by: §2.
  * Lehman and Stanley (2011) Joel Lehman and Kenneth O Stanley Evolving a diversity of virtual creatures through novelty search and local competition.  In Proceedings of the 13th annual conference on Genetic and evolutionary computation,  pp. 211–218.  Cited by: §2.
  * Li et al. (2024) Hao Li, Xue Yang, Zhaokai Wang, Xizhou Zhu, Jie Zhou, Yu Qiao, Xiaogang Wang, Hongsheng Li, Lewei Lu, and Jifeng Dai Auto mc-reward: Automated dense reward design with large language models for minecraft.  In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition,  pp. 16426–16435.  Cited by: §2.
  * Lu et al. (2024a) Chris Lu, Samuel Holt, Claudio Fanconi, Alex Chan, Jakob Foerster, Mihaela van der Schaar, and Robert Lange Discovering preference optimization algorithms with and for large language models.  Advances in Neural Information Processing Systems 37, pp. 86528–86573.  Cited by: §2.
  * Lu et al. (2024b) Chris Lu, Cong Lu, Robert Tjarko Lange, Jakob Foerster, Jeff Clune, and David Ha The ai scientist: Towards fully automated open-ended scientific discovery.  arXiv preprint arXiv:2408.06292.  Cited by: §2.
  * Lu et al. (2023) Chris Lu, Sebastian Towers, and Jakob Foerster Arbitrary order meta-learning with simple population-based evolution.  In Artificial Life Conference Proceedings 35,  Vol. 2023, pp. 67.  Cited by: §1, §2.
  * Luong et al. (2025) Minh-Thang Luong, Dawsen Hwang, Hoang H Nguyen, Golnaz Ghiasi, Yuri Chervonyi, Insuk Seo, Junsu Kim, Garrett Bingham, Jonathan Lee, Swaroop Mishra, et al. Towards robust mathematical reasoning.  In Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing,  pp. 35406–35430.  Cited by: §C.4, §C.4, Figure 10, Figure 10, §E.4, §4.2, §5.3.
  * Lv et al. (2026) Changze Lv, Jie Zhou, Wentao Zhao, Jingwen Xu, Zisu Huang, Muzhao Tian, Shihan Dou, Tao Gui, Le Tian, Xiao Zhou, Xiaoqing Zheng, Xuanjing Huang, and Jie Zhou Learning Query-Specific Rubrics from Human Preferences for DeepResearch Report Generation.  arXiv preprint arXiv:2602.03619.  Cited by: §E.2.
  * Ma et al. (2024) Yecheng Jason Ma, William Liang, Guanzhi Wang, De-An Huang, Osbert Bastani, Dinesh Jayaraman, Yuke Zhu, Linxi Fan, and Anima Anandkumar Eureka: Human-Level Reward Design via Coding Large Language Models.  In The Twelfth International Conference on Learning Representations,  Cited by: §3, §4.2.
  * Miconi et al. (2020) Thomas Miconi, Aditya Rawal, Jeff Clune, and Kenneth O Stanley Backpropamine: training self-modifying neural networks with differentiable neuromodulated plasticity.  arXiv preprint arXiv:2002.10585.  Cited by: §2.
  * Miconi et al. (2018) Thomas Miconi, Kenneth Stanley, and Jeff Clune Differentiable plasticity: training plastic neural networks with backpropagation.  In International Conference on Machine Learning,  pp. 3559–3568.  Cited by: §2.
  * Mouret and Clune (2015) Jean-Baptiste Mouret and Jeff Clune Illuminating search spaces by mapping elites.  arXiv preprint arXiv:1504.04909.  Cited by: §2.
  * Novikov et al. (2025) Alexander Novikov, Ngân Vũ, Marvin Eisenberger, Emilien Dupont, Po-Sen Huang, Adam Zsolt Wagner, Sergey Shirobokov, Borislav Kozlovskii, Francisco JR Ruiz, Abbas Mehrabian, et al. AlphaEvolve: A coding agent for scientific and algorithmic discovery.  arXiv preprint arXiv:2506.13131.  Cited by: §2.
  * Oh et al. (2025) Junhyuk Oh, Greg Farquhar, Iurii Kemaev, Dan A Calian, Matteo Hessel, Luisa Zintgraf, Satinder Singh, Hado Van Hasselt, and David Silver Discovering state-of-the-art reinforcement learning algorithms.  Nature, pp. 1–2.  Cited by: §2.
  * Pourcel et al. (2023) Julien Pourcel, Cédric Colas, Gaia Molinaro, Pierre-Yves Oudeyer, and Laetitia Teodorescu ACES: Generating Diverse Programming Puzzles with with Autotelic Generative Models.  arXiv preprint arXiv:2310.10692.  Cited by: §2.
  * Qiu et al. (2025) Xin Qiu, Yulu Gan, Conor F Hayes, Qiyao Liang, Elliot Meyerson, Babak Hodjat, and Risto Miikkulainen Evolution strategies at scale: Llm fine-tuning beyond reinforcement learning.  arXiv preprint arXiv:2509.24372.  Cited by: §2.
  * Robeyns et al. (2025) Maxime Robeyns, Martin Szummer, and Laurence Aitchison A self-improving coding agent.  arXiv preprint arXiv:2504.15228.  Cited by: §2.
  * Romera-Paredes et al. (2024) Bernardino Romera-Paredes, Mohammadamin Barekatain, Alexander Novikov, Matej Balog, M Pawan Kumar, Emilien Dupont, Francisco JR Ruiz, Jordan S Ellenberg, Pengming Wang, Omar Fawzi, et al. Mathematical discoveries from program search with large language models.  Nature 625 (7995), pp. 468–475.  Cited by: §2.
  * Samvelyan et al. (2024) Mikayel Samvelyan, Sharath C Raparthy, Andrei Lupu, Eric Hambro, Aram H Markosyan, Manish Bhatt, Yuning Mao, Minqi Jiang, Jack Parker-Holder, Jakob Foerster, et al. Rainbow teaming: Open-ended generation of diverse adversarial prompts.  Advances in Neural Information Processing Systems 37, pp. 69747–69786.  Cited by: §2.
  * Schick et al. (2023) Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Eric Hambro, Luke Zettlemoyer, Nicola Cancedda, and Thomas Scialom Toolformer: Language models can teach themselves to use tools.  Advances in Neural Information Processing Systems 36, pp. 68539–68551.  Cited by: §4.
  * Schmidhuber (1993) Jürgen Schmidhuber A neural network that embeds its own meta-levels.  In IEEE International Conference on Neural Networks,  pp. 407–412.  Cited by: §2.
  * Schmidhuber (2003) Jürgen Schmidhuber Gödel machines: self-referential universal problem solvers making provably optimal self-improvements.  arXiv preprint cs/0309048.  Cited by: §2.
  * Schulman et al. (2017) John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov Proximal policy optimization algorithms.  arXiv preprint arXiv:1707.06347.  Cited by: §C.3.
  * Sheth et al. (2025) Ivaxi Sheth, Jan Wehner, Sahar Abdelnabi, Ruta Binkyte, and Mario Fritz Safety is Essential for Responsible Open-Ended Systems.  arXiv preprint arXiv:2502.04512.  Cited by: §6.
  * Silver et al. (2016) David Silver, Aja Huang, Chris J Maddison, Arthur Guez, Laurent Sifre, George Van Den Driessche, Julian Schrittwieser, Ioannis Antonoglou, Veda Panneershelvam, Marc Lanctot, et al. Mastering the game of Go with deep neural networks and tree search.  nature 529 (7587), pp. 484–489.  Cited by: §A.2, §2.
  * Silver et al. (2017) David Silver, Thomas Hubert, Julian Schrittwieser, Ioannis Antonoglou, Matthew Lai, Arthur Guez, Marc Lanctot, Laurent Sifre, Dharshan Kumaran, Thore Graepel, et al. Mastering chess and shogi by self-play with a general reinforcement learning algorithm.  arXiv preprint arXiv:1712.01815.  Cited by: §2.
  * Stanley et al. (2017) Kenneth O Stanley, Joel Lehman, and Lisa Soros Open-endedness: The last grand challenge you’ve never heard of.  While open-endedness could be a force for discovering intelligence, it could also be a component of AI itself.  Cited by: §2.
  * Stanley and Miikkulainen (2002) Kenneth O Stanley and Risto Miikkulainen Evolving neural networks through augmenting topologies.  Evolutionary computation 10 (2), pp. 99–127.  Cited by: §2.
  * Strathern (1997) Marilyn Strathern ‘Improving ratings’: audit in the British University system.  European review 5 (3), pp. 305–321.  Cited by: Appendix F.
  * Turing et al. (1936) Alan Mathison Turing et al. On computable numbers, with an application to the Entscheidungsproblem.  J. of Math 58 (345-363), pp. 5.  Cited by: §3.
  * Wang et al. (2024) Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar Voyager: An Open-Ended Embodied Agent with Large Language Models.  Transactions on Machine Learning Research.  Cited by: §2.
  * Wang et al. (2025a) Jianyu Wang, Zhiqiang Hu, and Lidong Bing Evolving Prompts In-Context: An Open-ended, Self-replicating Perspective.  arXiv preprint arXiv:2506.17930.  Cited by: §2.
  * Wang et al. (2025b) Wenyi Wang, Piotr Piękos, Li Nanbo, Firas Laakom, Yimeng Chen, Mateusz Ostaszewski, Mingchen Zhuge, and Jürgen Schmidhuber Huxley-G\\backslash" odel Machine: Human-Level Coding Agent Development by an Approximation of the Optimal Self-Improving Machine.  arXiv preprint arXiv:2510.21614.  Cited by: §A.2, §2, §2, §3.
  * Wei et al. (2025a) Tianxin Wei, Noveen Sachdeva, Benjamin Coleman, Zhankui He, Yuanchen Bei, Xuying Ning, Mengting Ai, Yunzhe Li, Jingrui He, Ed H Chi, et al. Evo-Memory: Benchmarking LLM Agent Test-time Learning with Self-Evolving Memory.  arXiv preprint arXiv:2511.20857.  Cited by: §E.3.7.
  * Wei et al. (2025b) Yuxiang Wei, Zhiqing Sun, Emily McMilin, Jonas Gehring, David Zhang, Gabriel Synnaeve, Daniel Fried, Lingming Zhang, and Sida Wang Toward Training Superintelligent Software Agents through Self-Play SWE-RL.  arXiv preprint arXiv:2512.18552.  Cited by: §2.
  * Wen et al. (2025) Jiaxin Wen, Zachary Ankner, Arushi Somani, Peter Hase, Samuel Marks, Jacob Goldman-Wetzler, Linda Petrini, Henry Sleight, Collin Burns, He He, et al. Unsupervised Elicitation of Language Models.  arXiv preprint arXiv:2506.10139.  Cited by: §2.
  * Weng et al. (2026) Zhaotian Weng, Antonis Antoniades, Deepak Nathani, Zhen Zhang, Xiao Pu, and Xin Eric Wang Group-Evolving Agents: Open-Ended Self-Improvement via Experience Sharing.  arXiv preprint arXiv:2602.04837.  Cited by: §A.2, §E.3.7, §2.
  * Weston and Foerster (2025) Jason Weston and Jakob Foerster Ai & human co-improvement for safer co-superintelligence.  arXiv preprint arXiv:2512.05356.  Cited by: §6.
  * Wu et al. (2024) Zhiyong Wu, Chengcheng Han, Zichen Ding, Zhenmin Weng, Zhoumianze Liu, Shunyu Yao, Tao Yu, and Lingpeng Kong Os-copilot: Towards generalist computer agents with self-improvement.  arXiv preprint arXiv:2402.07456.  Cited by: §2.
  * Xia et al. (2025a) Chunqiu Steven Xia, Zhe Wang, Yan Yang, Yuxiang Wei, and Lingming Zhang Live-SWE-agent: Can Software Engineering Agents Self-Evolve on the Fly? arXiv preprint arXiv:2511.13646.  Cited by: §2, §2.
  * Xia et al. (2026) Peng Xia, Jianwen Chen, Hanyang Wang, Jiaqi Liu, Kaide Zeng, Yu Wang, Siwei Han, Yiyang Zhou, Xujiang Zhao, Haifeng Chen, et al. SkillRL: Evolving Agents via Recursive Skill-Augmented Reinforcement Learning.  arXiv preprint arXiv:2602.08234.  Cited by: §2.
  * Xia et al. (2025b) Peng Xia, Kaide Zeng, Jiaqi Liu, Can Qin, Fang Wu, Yiyang Zhou, Caiming Xiong, and Huaxiu Yao Agent0: Unleashing self-evolving agents from zero data via tool-integrated reasoning.  arXiv preprint arXiv:2511.16043.  Cited by: §2.
  * Xiong et al. (2026) Yiming Xiong, Shengran Hu, and Jeff Clune Learning to Continually Learn via Meta-learning Agentic Memory Designs.  arXiv preprint arXiv:2602.07755.  Cited by: §E.3.7.
  * Yamada et al. (2025) Yutaro Yamada, Robert Tjarko Lange, Cong Lu, Shengran Hu, Chris Lu, Jakob Foerster, Jeff Clune, and David Ha The ai scientist-v2: Workshop-level automated scientific discovery via agentic tree search.  arXiv preprint arXiv:2504.08066.  Cited by: §C.2, §4.2, §5.1.
  * Ye et al. (2026) Haoran Ye, Xuning He, Vincent Arak, Haonan Dong, and Guojie Song Meta Context Engineering via Agentic Skill Evolution.  arXiv preprint arXiv:2601.21557.  Cited by: §2.
  * Yin et al. (2025) Xunjian Yin, Xinyi Wang, Liangming Pan, Li Lin, Xiaojun Wan, and William Yang Wang Gödel agent: a self-referential agent framework for recursively self-improvement.  In Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers),  pp. 27890–27913.  Cited by: §2, §2.
  * Yuan et al. (2026) Jiayi Yuan, Jonathan Nöther, Natasha Jaques, and Goran Radanović AgenticRed: Optimizing Agentic Systems for Automated Red-teaming.  arXiv preprint arXiv:2601.13518.  Cited by: §2.
  * Zelikman et al. (2024) Eric Zelikman, Eliana Lorch, Lester Mackey, and Adam Tauman Kalai Self-taught optimizer (stop): Recursively self-improving code generation.  In First Conference on Language Modeling,  Cited by: §2.
  * Zelikman et al. (2022) Eric Zelikman, Yuhuai Wu, Jesse Mu, and Noah Goodman Star: Bootstrapping reasoning with reasoning.  Advances in Neural Information Processing Systems 35, pp. 15476–15488.  Cited by: §2.
  * Zhang et al. (2025a) Alex L Zhang, Tim Kraska, and Omar Khattab Recursive Language Models.  arXiv preprint arXiv:2512.24601.  Cited by: §2.
  * Zhang et al. (2026) Haozhen Zhang, Quanyu Long, Jianzhu Bao, Tao Feng, Weizhi Zhang, Haodong Yue, and Wenya Wang MemSkill: Learning and Evolving Memory Skills for Self-Evolving Agents.  arXiv preprint arXiv:2602.02474.  Cited by: §E.3.7.
  * Zhang et al. (2025b) Jenny Zhang, Shengran Hu, Cong Lu, Robert Lange, and Jeff Clune Darwin Godel Machine: Open-Ended Evolution of Self-Improving Agents.  arXiv preprint arXiv:2505.22954.  Cited by: §A.2, Appendix B, §C.1, §D.1, §1, §1, §1, §2, §2, Figure 1, Figure 1, §3, §3, §4.1, §4.2, §5.1, Abstract.
  * Zhang et al. (2024) Jenny Zhang, Joel Lehman, Kenneth Stanley, and Jeff Clune OMNI: Open-endedness via Models of human Notions of Interestingness.  In The Twelfth International Conference on Learning Representations,  Cited by: §2, §7.
  * Zhang et al. (2025c) Qizheng Zhang, Changran Hu, Shubhangi Upasani, Boyuan Ma, Fenglu Hong, Vamsidhar Kamanuru, Jay Rainton, Chen Wu, Mengmeng Ji, Hanchen Li, et al. Agentic context engineering: Evolving contexts for self-improving language models.  arXiv preprint arXiv:2510.04618.  Cited by: §2.
  * Zhao et al. (2025) Bingchen Zhao, Despoina Magka, Minqi Jiang, Xian Li, Roberta Raileanu, Tatiana Shavrina, Jean-Christophe Gagnon-Audet, Kelvin Niu, Shagun Sodhani, Michael Shvartsman, et al. The Automated LLM Speedrunning Benchmark: Reproducing NanoGPT Improvements.  arXiv preprint arXiv:2506.22419.  Cited by: §2.
  * Zhao et al. (2026) Bingchen Zhao, Jenny Zhang, Chenxi Whitehouse, Minqi Jiang, Michael Shvartsman, Abhishek Charnalia, Despoina Magka, Tatiana Shavrina, Derek Dunfield, Oisin Mac Aodha, and Yoram Bachrach APRES: An Agentic Paper Revision and Evaluation System.  arXiv preprint arXiv:2603.03142.  Cited by: §C.2, §1, §4.2.
  * Zhuge et al. (2024) Mingchen Zhuge, Wenyi Wang, Louis Kirsch, Francesco Faccio, Dmitrii Khizbullin, and Jürgen Schmidhuber Gptswarm: Language agents as optimizable graphs.  In Forty-first International Conference on Machine Learning,  Cited by: §2.
  * Zoph and Le (2017) Barret Zoph and Quoc Le Neural Architecture Search with Reinforcement Learning.  In International Conference on Learning Representations,  Cited by: §2.
  * Zweiger et al. (2025) Adam Zweiger, Jyothish Pari, Han Guo, Ekin Akyürek, Yoon Kim, and Pulkit Agrawal Self-Adapting Language Models.  arXiv preprint arXiv:2506.10943.  Cited by: §2.

## 目录

## 附录 A 算法细节

本附录提供 DGM-超智能体（DGM-Hyperagents，DGM-H）的更多算法细节。我们首先描述初始超智能体的实现，包括初始任务智能体与元智能体可用的工具和提示词（附录 A.1）。随后，我们详述开放式探索过程中使用的父代选择机制；该机制在对高性能智能体的利用与对存档的持续探索之间取得平衡（附录 A.2）。最后，我们给出 DGM-H 的伪代码（附录 A.3）。

### A.1 初始智能体

我们给出初始超智能体可用工具的细节及其提示词（第 4 节）。

初始任务智能体提示词：

```
[⬇](<)

instruction = f"""You are an agent.

Task input:

‘‘‘

{inputs}

‘‘‘

Respond in JSON format with the following schema:

<json>

{{

"response": ...

}}

</json>"""
```

初始元智能体提示词：

```
[⬇](<)

instruction = f"Modify any part of the codebase at ‘{repo_path}‘."
```

所给 bash 工具的信息：

```
[⬇](<)

def tool_info():

return {

"name": "bash",

"description": """Run commands in a bash shell

* When invoking this tool, the contents of the "command" parameter does NOT need to be XML-escaped.

* You don’t have access to the internet via this tool.

* You do have access to a mirror of common linux and python packages via apt and pip.

* State is persistent across command calls and discussions with the user.

* To inspect a particular line range of a file, e.g. lines 10-25, try ’sed -n 10,25p /path/to/the/file’.

* Please avoid commands that may produce a very large amount of output.

* Please run long lived commands in the background, e.g. ’sleep 10 &’ or start a server in the background.""",

"input_schema": {

"type": "object",

"properties": {

"command": {

"type": "string",

"description": "The bash command to run."

}

},

"required": ["command"]

}

}
```

所给编辑工具的信息：

```
[⬇](<)

def tool_info():

return {

"name": "editor",

"description": """Custom editing tool for viewing, creating and editing files

* State is persistent across command calls and discussions with the user

* If ‘path‘ is a file, ‘view‘ displays the result of applying ‘cat -n‘. If ‘path‘ is a directory, ‘view‘ lists non-hidden files and directories up to 2 levels deep

* The ‘create‘ command cannot be used if the specified ‘path‘ already exists as a file

* If a ‘command‘ generates a long output, it will be truncated and marked with ‘<response clipped>‘

* The ‘undo_edit‘ command will revert the last edit made to the file at ‘path‘

\nNotes for using the ‘str_replace‘ command:

* The ‘old_str‘ parameter should match EXACTLY one or more consecutive lines from the original file. Be mindful of whitespaces!

* If the ‘old_str‘ parameter is not unique in the file, the replacement will not be performed. Make sure to include enough context in ‘old_str‘ to make it unique

* The ‘new_str‘ parameter should contain the edited lines that should replace the ‘old_str‘""",

"input_schema": {

"type": "object",

"properties": {

"command": {

"type": "string",

"enum": ["view", "create", "str_replace", "insert", "undo_edit"],

"description": "The commands to run. Allowed options are: ‘view‘, ‘create‘, ‘str_replace‘, ‘insert‘, ‘undo_edit‘."

},

"file_text": {

"description": "Required parameter of ‘create‘ command, with the content of the file to be created.",

"type": "string"

},

"insert_line": {

"description": "Required parameter of ‘insert‘ command. The ‘new_str‘ will be inserted AFTER the line ‘insert_line‘ of ‘path‘.",

"type": "integer"

},

"new_str": {

"description": "Required parameter of ‘str_replace‘ command containing the new string. Required parameter of ‘insert‘ command containing the string to insert.",

"type": "string"

},

"old_str": {

"description": "Required parameter of ‘str_replace‘ command containing the string in ‘path‘ to replace.",

"type": "string"

},

"path": {

"description": "Absolute path to file or directory, e.g. ‘/repo/file.py‘ or ‘/repo‘.",

"type": "string"

},

"view_range": {

"description": "Optional parameter of ‘view‘ command when ‘path‘ points to a file. If none is given, the full file is shown. If provided, the file will be shown in the indicated line number range, e.g. [11, 12] will show lines 11 and 12. Indexing at 1 to start. Setting ‘[start_line, -1]‘ shows all lines from ‘start_line‘ to the end of the file.",

"items": {

"type": "integer"

},

"type": "array"

}

},

"required": ["command", "path"]

}

}
```

### A.2 父代选择

在每一次迭代中，我们从存档中选出一个智能体子集作为父代，使其自我修改并产生新的子代智能体（第 3 节）。我们采用与 Zhang et al. (2025b) 类似、并受 Ecoffet et al. (2019) 启发的机制：该机制大致与智能体的表现分数成正比，并与成功编译的子代数量成反比。这种选择机制使采样偏向于表现优于当前前沿平均值的智能体，同时下调已经产生许多子代的智能体的权重，从而保留平滑的概率性探索，并随着存档随时间改进而自动适应。父代选择过程的细节概述如下。

在 DGM-H 运行的每一次迭代 $t$，令

$$
\mathcal{A}^{t}=\{a_{0},a_{1},\dots,a_{t}\}
$$

表示候选智能体的存档，其相应表现分数为 $\alpha_{i}=\mathrm{performance}(a_{i})$。存档中的所有智能体都有资格被选为父代。

我们首先基于当前表现分布计算一个动态中点。令

$$
\alpha_{mid}\;=\;\frac{1}{m}\sum_{j\in\mathcal{T}^{t}}\alpha_{j},
$$

其中 $\mathcal{T}^{t}\subset\mathcal{A}^{t}$ 标记第 $t$ 次迭代时表现最高的前 $m$ 个智能体（我们的实验中 $m=3$）。该中点随时间自适应，并反映当前的表现前沿。

每个智能体的分数首先经过 sigmoid 变换：

$$
s_{i}\;=\;\frac{1}{1+\exp\!\bigl(-\lambda(\alpha_{i}-\alpha_{mid})\bigr)},
$$

其中 $\lambda>0$ 控制选择的锐度。我们设定 $\lambda=10$。

为鼓励探索，我们基于智能体 $a_{i}$ 所产生的已编译子代数量 $n_{i}$ 引入新颖性奖励：

$$
h_{i}\;=\;\frac{1}{1+n_{i}}.
$$

然后，我们形成未归一化权重

$$
w_{i}\;=\;s_{i}\,h_{i},
$$

它在表现与新颖性之间取得平衡。

这些权重经归一化后形成分类分布：

$$
p_{i}\;=\;\begin{cases}\dfrac{w_{i}}{\sum_{j=0}^{t}w_{j}},&\text{若 }\sum_{j=0}^{t}w_{j}>0,\\[10.0pt] \dfrac{1}{t+1},&\text{否则}.\end{cases}
$$

我们按该分布有放回地独立抽取父代：

$$
\{\,\text{父代}\,\}\;\sim\;\mathrm{Categorical}\bigl(\{p_{i}\}_{i=0}^{t}\bigr).
$$

先前工作已经提出了多种搜索与探索策略（Coulom, 2006; Silver et al., 2016; Herr et al., 2025; Wang et al., 2025b; Weng et al., 2026）。我们给出初步证据，表明 DGM-H 可以通过修改其自身的探索动态，开始自主地重新发现并调整此类策略（附录 E.5）。一个开放的研究方向是，自我改进系统能否可靠地发现优于精心手工设计的算法的搜索与探索机制。

### A.3 伪代码

以下是第 3 节所述 DGM-H 的伪代码：

```
Algorithm 1: Darwin Gödel Machine with Hyperagents (DGM-H)
Input: Initial agent $a^{0}$, task set $\mathcal{T}$, maximum iterations $T$
Output: Archive of scored agents $\mathcal{A}$
$s^{0} \leftarrow \textsc{Evaluate}(a^{0}, \mathcal{T})$
initialize $\mathcal{A} \leftarrow \{(a^{0}, s^{0})\}$ // Start with initial agent
for $t \leftarrow 1$ to $T$ do
    $\mathcal{P} \leftarrow \textsc{SelectParents}(\mathcal{A})$ // Sample parent agents
    foreach $(a, \cdot) \in \mathcal{P}$ do
        $a^{\prime} \leftarrow a.\textsc{Modify}(a, \mathcal{A})$ // Metacognitive self-modification
        $s^{\prime} \leftarrow \textsc{Evaluate}(a^{\prime}, \mathcal{T})$ // Evaluate on tasks
        if $\textsc{IsValid}(a^{\prime})$ then
            $\mathcal{A} \leftarrow \mathcal{A} \cup \{(a^{\prime}, s^{\prime})\}$ // Add compiled child agent
        end
    end
end
return $\mathcal{A}$
```

### A.4 多领域优化

当在同一次运行中针对多个领域进行优化时，超智能体在来自不同领域的任务上接受评估，并在自我修改期间可以访问这些任务上的全部评估结果。我们并不指定应当优先考虑哪一个特定领域或任务。父代选择基于跨领域的平均表现。因此，任一领域中的改进都会提高被选中的概率，而退步则会降低该概率。由于元智能体可以检查任意任务的评估结果，它可以引入共享机制（例如结构化推理、记忆和错误处理），同时惠及多个领域。因此，无需人工指定要优化哪一项任务或哪一个领域，超智能体就可以在同一次运行中跨多个领域进行优化。

## 附录 B 基线细节

我们概述第 4.1 节所述各基线的伪代码，给出一张总结其关键差异的比较表（表 1），并给出一幅详细的概念图，直观对比各 DGM 变体与超智能体在架构组件和修改机制上的差别（图 5）。

**表 1：** 按自我改进、开放式探索和元认知自我修改对各方法进行比较。

| 方法 | 自我改进的元智能体 | 开放式探索 | 元认知自我修改（即超智能体） |
|---|---|---|---|
| DGM-H | ✓ | ✓ | ✓ |
| DGM-H w/o self-improve | ✕ | ✓ | ✓ |
| DGM-H w/o open-ended exploration | ✓ | ✕ | ✓ |
| DGM | ✓ | ✓ | ✕ |
| DGM-custom | ✓ | ✓ | ✕ |

以下是不含自我改进智能体的基线 DGM-H 的伪代码（ADAS, Hu et al., 2025）：

```
Algorithm 2: DGM-H without self-improving meta agents (DGM-H w/o self-improve)
Input: Initial agent $a^{0}$, task set $\mathcal{T}$, maximum iterations $T$
Output: Archive of scored agents $\mathcal{A}$
$s^{0} \leftarrow \textsc{Evaluate}(a^{0}, \mathcal{T})$
initialize $\mathcal{A} \leftarrow \{(a^{0}, s^{0})\}$
for $t \leftarrow 1$ to $T$ do
    $\mathcal{P} \leftarrow \textsc{SelectParents}(\mathcal{A})$
    foreach $(a, \cdot) \in \mathcal{P}$ do
        $a^{\prime} \leftarrow a^{0}.\textsc{Modify}(a, \mathcal{A})$ // Modify with initial agent
        $s^{\prime} \leftarrow \textsc{Evaluate}(a^{\prime}, \mathcal{T})$
        if $\textsc{IsValid}(a^{\prime})$ then
            $\mathcal{A} \leftarrow \mathcal{A} \cup \{(a^{\prime}, s^{\prime})\}$
        end
    end
end
return $\mathcal{A}$
```

以下是不含开放式探索的基线 DGM-H 的伪代码：

```
Algorithm 3: DGM-H without open-ended exploration (DGM-H w/o open-ended exploration)
Input: Initial agent $a^{0}$, task set $\mathcal{T}$, maximum iterations $T$
Output: Archive of scored agents $\mathcal{A}$
$s^{0} \leftarrow \textsc{Evaluate}(a^{0}, \mathcal{T})$
initialize $\mathcal{A} \leftarrow \{(a^{0}, s^{0})\}$
for $t \leftarrow 1$ to $T$ do
    $\mathcal{P} \leftarrow \textsc{SelectParents}(\mathcal{A})$
    foreach $(a, \cdot) \in \mathcal{P}$ do
        $a^{\prime} \leftarrow a.\textsc{Modify}(a, \mathcal{A})$
        $s^{\prime} \leftarrow \textsc{Evaluate}(a^{\prime}, \mathcal{T})$
        if $\textsc{IsValid}(a^{\prime})$ then
            $\mathcal{A} \leftarrow \{(a^{\prime}, s^{\prime})\}$ // Only keep the latest agent
        end
    end
end
return $\mathcal{A}$
```

以下是原始 DGM（Zhang et al., 2025b）的伪代码，并置于超智能体设定之中：

```
Algorithm 4: Darwin Gödel Machine (DGM)
Input: Initial agent $a^{0}$, task set $\mathcal{T}$, maximum iterations $T$
Output: Archive of scored agents $\mathcal{A}$
$s^{0} \leftarrow \textsc{Evaluate}(a^{0}, \mathcal{T})$
initialize $\mathcal{A} \leftarrow \{(a^{0}, s^{0})\}$
for $t \leftarrow 1$ to $T$ do
    $\mathcal{P} \leftarrow \textsc{SelectParents}(\mathcal{A})$
    foreach $(a, \cdot) \in \mathcal{P}$ do
        $instr \leftarrow \textsc{InstrGen}(a)$ // Handcrafted instruction-generation
        $a^{\prime} \leftarrow a.\textsc{Modify}(a, instr)$ // Self-modification
        $s^{\prime} \leftarrow \textsc{Evaluate}(a^{\prime}, \mathcal{T})$
        if $\textsc{IsValid}(a^{\prime})$ then
            $\mathcal{A} \leftarrow \mathcal{A} \cup \{(a^{\prime}, s^{\prime})\}$
        end
    end
end
return $\mathcal{A}$
```

**图 5：** DGM 变体的概念比较，突出不同设定下哪些组件会发生变化，以及超智能体如何应对原始 DGM 实现中的局限。（第一行）原始 DGM。同一个编程智能体同时充当任务智能体和元智能体。由于评估与自我修改都是编程任务，编程能力上的增益会转化为自我修改能力上的增益，从而使 DGM 能够在编程领域中改进其进行改进的能力。（第二行）适配到非编程领域的 DGM。编程智能体仍然充当元智能体，但评估任务不再是编程任务，因此需要单独的任务智能体。任务表现不再可靠地反映元智能体生成更好任务智能体的能力，从而打破了使元智能体能够改进其进行改进的能力的那种对齐。（第三行）DGM-custom 基线。手工设计的指令生成机制针对目标领域做了定制，但仍然不可修改。（第四行）带有超智能体的 DGM（DGM-H）。超智能体在同一个可编辑程序中整合任务智能体与元智能体，从而实现元认知自我修改（即不仅修改求解任务的行为，也修改产生未来自我修改的过程）。因此，DGM-H 可以在针对任何可计算任务进行优化的同时，改进其改进机制。

原始 DGM 中手工设计的指令生成步骤：

```
[⬇](<)

diagnose_prompt = """Here is the implementation of the coding agent.

# Coding Agent Implementation

\----- Coding Agent Implementation Start \-----

{code}

\----- Coding Agent Implementation End \-----

Your task is to identify ONE detailed plan that would improve the agent’s coding ability. The improvement should not be specific to any particular GitHub issue or repository.

# Agent Running Log

\----- Agent Running Log Start \-----

{md_log}

\----- Agent Running Log End \-----

# GitHub Issue

The GitHub issue that the agent is trying to solve.

\----- GitHub Issue Start \-----

{github_issue}

\----- GitHub Issue End \-----

# Predicted Patch

The agent’s predicted patch to solve the issue.

\----- Predicted Patch Start \-----

{predicted_patch}

\----- Predicted Patch End \-----

# Private Test Patch

SWE-bench’s official private tests to detect whether the issue is solved. This is not available to the agent during evaluation. The agent should try to implement its own tests.

\----- Private Test Patch Start \-----

{test_patch}

\----- Private Test Patch End \-----

# Issue Test Results

The test results from SWE-bench using the above official private tests.

\----- Issue Test Results Start \-----

{eval_log}

\----- Issue Test Results End \-----

Respond precisely in the following format including the JSON start and end markers:

‘‘‘json

<JSON>

‘‘‘

In <JSON>, provide a JSON response with the following fields:

- "log_summarization": Analyze the above logs and summarize how the agent tried to solve the GitHub issue. Note which tools and how they are used, the agent’s problem-solving approach, and any issues encountered.

- "potential_improvements": Identify potential improvements to the coding agent that could enhance its coding capabilities. Focus on the agent’s general coding abilities (e.g., better or new tools usable across any repository) rather than issue-specific fixes (e.g., tools only usable in one framework). All necessary dependencies and environment setup have already been handled, so do not focus on these aspects.

- "improvement_proposal": Choose ONE high-impact improvement from the identified potential improvements and describe it in detail. This should be a focused and comprehensive plan to enhance the agent’s overall coding ability.

- "implementation_suggestion": Referring to the coding agent’s summary and implementation, think critically about what feature or tool could be added or improved to best implement the proposed improvement. If the proposed feature can be implemented by modifying the existing tools, describe the modifications needed, instead of suggesting a new tool.

- "problem_description": Phrase the improvement proposal and implementation suggestion as a GitHub issue description. It should clearly describe the feature so that a software engineer viewing the issue and the repository can implement it.

Your response will be automatically parsed, so ensure that the string response is precisely in the correct format. Do NOT include the ‘<JSON>‘ tag in your output."""
```

DGM-custom 中经过定制的指令生成步骤：

```
[⬇](<)

diagnose_prompt_customized = """

Here is the implementation of the coding agent and task agent.

# Coding Agent Implementation

\----- Coding Agent Implementation Start \-----

{code_codingagent}

\----- Coding Agent Implementation End \-----

# Task Agent Implementation

\----- Task Agent Implementation Start \-----

{code_taskagent}

\----- Task Agent Implementation End \-----

Your task is to identify ONE detailed plan that would improve the coding/task agent. The improvement should not be specific to any particular task instance or repository.

# Task Info

\----- Task \-----

{task_info}

\----- Task End \-----

# Report

\----- Report \-----

{report}

\----- report End \-----

# Agent Running Log

\----- Agent Running Log Start \-----

{md_log}

\----- Agent Running Log End \-----

Respond precisely in the following format including the JSON start and end markers:

<json>

...

</json>

In <json>, provide a JSON response with the following fields:

- "log_summarization": Analyze the above logs and summarize how the agent tried to solve the given task. Note which tools and how they are used, the agent’s problem-solving approach, and any issues encountered.

- "potential_improvements": Identify potential improvements to the coding/task agent that could enhance its coding/task-solving capabilities. Focus on the agent’s general abilities (e.g., better or new tools usable across any repository) rather than issue-specific fixes (e.g., tools only usable in one framework). All necessary dependencies and environment setup have already been handled, so do not focus on these aspects.

- "improvement_proposal": Choose ONE high-impact improvement from the identified potential improvements and describe it in detail. This should be a focused and comprehensive plan to enhance the agent’s overall coding/task-solving ability.

- "implementation_suggestion": Referring to the coding/task agent’s summary and implementation, think critically about what feature or tool could be added or improved to best implement the proposed improvement. If the proposed feature can be implemented by modifying the existing tools, describe the modifications needed, instead of suggesting a new tool.

- "problem_description": Phrase the improvement proposal and implementation suggestion as a GitHub issue description. It should clearly describe the feature so that a software engineer viewing the issue and the repository can implement it.

Your response will be automatically parsed, so ensure that the string response is precisely in the correct format."""
```

## 附录 C 领域细节

本附录详细描述评估中所用的各个领域：Polyglot（附录 C.1）、论文评审（附录 C.2）、机器人奖励设计（附录 C.3）、奥林匹克级别数学评分（附录 C.4）。对于每个领域，我们说明给定任务时智能体的输入与所需输出、评估协议，以及有代表性的静态基线（表 2）。

| 领域 | 输入 | 输出 | 指标 | 训练 | 验证 | 测试 |
| --- | --- | --- | --- | --- | --- | --- |
| 编程（Polyglot） | 仓库 + 指令 | 代码补丁 | Pass@1 | 60 | - | 165 |
| 论文评审 | 论文文本 | 接收 / 拒稿 | 准确率 | 100 | 100 | 100 |
| 机器人奖励设计 | 任务描述 | 奖励函数 | 任务得分 | 6 | - | 6 |
| IMO 评分 | 题目 + 解答 | 评分（0/1/6/7） | 准确率 | 100 | 100 | 100 |

表 2：领域汇总。在自我修改期间，智能体在训练任务上的评估结果是可用的，并可用作反馈。验证任务用于父代选择。若没有可用的验证划分，则用于父代选择的性能分量改为依据训练性能。测试任务予以留出，仅用于对所选智能体的最终评估。

### C.1 Polyglot

在 Polyglot 编程基准（Gauthier, 2024）中，每项任务由一个软件仓库和一条自然语言指令组成，该指令描述对代码库的期望改动。智能体可以访问完整仓库，并且必须修改文件以正确实现该指令，从而产生一个应用于该仓库的补丁（即一组代码编辑）。性能通过在修改后的仓库上运行预定义测试套件来评估。若全部测试都通过，则认为该任务已成功完成。我们遵循 DGM（Zhang et al., 2025b）所用的设置；该设置大体沿用 Polyglot 排行榜的配置，但有一处关键差异：排行榜报告的是 pass@2，允许智能体查看一次来自真实测试的反馈，而我们报告的是 pass@1，在此设定下智能体始终看不到真实测试结果。我们采用与 DGM 相同的训练划分与测试划分。训练任务是从完整基准中随机抽取的子集，共计 60 项任务。若某个智能体在初始的 10 项任务子集上成功率超过 40%，则随后在其余 50 项训练任务上对其进行评估。该领域没有验证子集。作为最终评估，为了更准确地衡量性能改进，我们在完整的 Polyglot 基准上评估所生成的智能体，该基准由 165 项未见任务组成。

用于初步评估的初始 10 项训练任务：

- go__dominoes

- cpp__all-your-base

- python__dominoes

- java__sgf-parsing

- javascript__robot-name

- rust__variable-length-quantity

- python__beer-song

- go__book-store

- javascript__bottle-song

- rust__bowling

用于完整评估的另外 50 项训练任务：

- javascript__queen-attack

- rust__wordy

- python__dot-dsl

- java__satellite

- cpp__diamond

- rust__accumulate

- go__error-handling

- cpp__queen-attack

- rust__poker

- python__sgf-parsing

- rust__react

- java__ledger

- go__connect

- rust__macros

- javascript__triangle

- java__zipper

- java__bowling

- python__tree-building

- javascript__say

- java__wordy

- python__food-chain

- javascript__wordy

- python__poker

- javascript__grade-school

- cpp__gigasecond

- java__forth

- python__dominoes

- go__word-search

- javascript__simple-linked-list

- go__counter

- java__react

- javascript__ocr-numbers

- python__scale-generator

- java__go-counting

- rust__doubly-linked-list

- python__grade-school

- javascript__forth

- python__wordy

- java__mazy-mice

- cpp__bank-account

- python__zipper

- java__custom-set

- java__rest-api

- go__transpose

- rust__gigasecond

- rust__say

- go__food-chain

- rust__pig-latin

- go__markdown

- go__crypto-square

### C.2 论文评审

本领域的数据取自 Zhao et al. (2026)。论文评审领域中的每项任务由一篇人工智能研究论文的全文组成。智能体必须预测一个二元的接收或拒稿决定，以模拟会议审稿人的角色。真实标签对应于顶级机器学习会议的实际录用决定，包括 ICLR 2024/2025 与 NeurIPS 2023/2024。性能以相对于这些标签的分类准确率来衡量。我们随机抽样任务，以构建训练、验证与测试划分，每个划分包含 100 项任务。在训练期间，智能体首先在训练划分的 10 项任务子集上接受评估。若该智能体在这些任务中至少成功一项，则随后在全部 100 项训练任务上接受评估。

AI-Scientist-v2（Yamada et al., 2025）使用一个 AI 审稿人来自动改进所生成的人工智能研究论文。我们采用该工作中提出的 AI 审稿人，作为我们有代表性的静态基线：

```python
reviewer_system_prompt_base = (

"You are an AI researcher who is reviewing a paper that was submitted to a prestigious ML venue."

"Be critical and cautious in your decision."

)

reviewer_system_prompt_neg = (

reviewer_system_prompt_base

+ "If a paper is bad or you are unsure, give it bad scores and reject it."

)

reviewer_system_prompt_pos = (

reviewer_system_prompt_base

+ "If a paper is good or you are unsure, give it good scores and accept it."

)

template_instructions = """

Respond in the following format:

THOUGHT:

<THOUGHT>

REVIEW JSON:

‘‘‘json

<JSON>

‘‘‘

In <THOUGHT>, first briefly discuss your intuitions and reasoning for the evaluation.

Detail your high-level arguments, necessary choices and desired outcomes of the review.

Do not make generic comments here, but be specific to your current paper.

Treat this as the note-taking phase of your review.

In <JSON>, provide the review in JSON format with the following fields in the order:

- "Summary": A summary of the paper content and its contributions.

- "Strengths": A list of strengths of the paper.

- "Weaknesses": A list of weaknesses of the paper.

- "Originality": A rating from 1 to 4 (low, medium, high, very high).

- "Quality": A rating from 1 to 4 (low, medium, high, very high).

- "Clarity": A rating from 1 to 4 (low, medium, high, very high).

- "Significance": A rating from 1 to 4 (low, medium, high, very high).

- "Questions": A set of clarifying questions to be answered by the paper authors.

- "Limitations": A set of limitations and potential negative societal impacts of the work.

- "Ethical Concerns": A boolean value indicating whether there are ethical concerns.

- "Soundness": A rating from 1 to 4 (poor, fair, good, excellent).

- "Presentation": A rating from 1 to 4 (poor, fair, good, excellent).

- "Contribution": A rating from 1 to 4 (poor, fair, good, excellent).

- "Overall": A rating from 1 to 10 (very strong reject to award quality).

- "Confidence": A rating from 1 to 5 (low, medium, high, very high, absolute).

- "Decision": A decision that has to be one of the following: Accept, Reject.

For the "Decision" field, don’t use Weak Accept, Borderline Accept, Borderline Reject, or Strong Reject. Instead, only use Accept or Reject.

This JSON will be automatically parsed, so ensure the format is precise.

"""

neurips_form = (

"""

## Review Form

Below is a description of the questions you will be asked on the review form for each paper and some guidelines on what to consider when answering these questions.

When writing your review, please keep in mind that after decisions have been made, reviews and meta-reviews of accepted papers and opted-in rejected papers will be made public.

1. Summary: Briefly summarize the paper and its contributions. This is not the place to critique the paper; the authors should generally agree with a well-written summary.

- Strengths and Weaknesses: Please provide a thorough assessment of the strengths and weaknesses of the paper, touching on each of the following dimensions:

- Originality: Are the tasks or methods new? Is the work a novel combination of well-known techniques? (This can be valuable!) Is it clear how this work differs from previous contributions? Is related work adequately cited

- Quality: Is the submission technically sound? Are claims well supported (e.g., by theoretical analysis or experimental results)? Are the methods used appropriate? Is this a complete piece of work or work in progress? Are the authors careful and honest about evaluating both the strengths and weaknesses of their work

- Clarity: Is the submission clearly written? Is it well organized? (If not, please make constructive suggestions for improving its clarity.) Does it adequately inform the reader? (Note that a superbly written paper provides enough information for an expert reader to reproduce its results.)

- Significance: Are the results important? Are others (researchers or practitioners) likely to use the ideas or build on them? Does the submission address a difficult task in a better way than previous work? Does it advance the state of the art in a demonstrable way? Does it provide unique data, unique conclusions about existing data, or a unique theoretical or experimental approach?

2. Questions: Please list up and carefully describe any questions and suggestions for the authors. Think of the things where a response from the author can change your opinion, clarify a confusion or address a limitation. This can be very important for a productive rebuttal and discussion phase with the authors.

3. Limitations: Have the authors adequately addressed the limitations and potential negative societal impact of their work? If not, please include constructive suggestions for improvement.

In general, authors should be rewarded rather than punished for being up front about the limitations of their work and any potential negative societal impact. You are encouraged to think through whether any critical points are missing and provide these as feedback for the authors.

4. Ethical concerns: If there are ethical issues with this paper, please flag the paper for an ethics review. For guidance on when this is appropriate, please review the NeurIPS ethics guidelines.

5. Soundness: Please assign the paper a numerical rating on the following scale to indicate the soundness of the technical claims, experimental and research methodology and on whether the central claims of the paper are adequately supported with evidence.

4: excellent

3: good

2: fair

1: poor

6. Presentation: Please assign the paper a numerical rating on the following scale to indicate the quality of the presentation. This should take into account the writing style and clarity, as well as contextualization relative to prior work.

4: excellent

3: good

2: fair

1: poor

7. Contribution: Please assign the paper a numerical rating on the following scale to indicate the quality of the overall contribution this paper makes to the research area being studied. Are the questions being asked important? Does the paper bring a significant originality of ideas and/or execution? Are the results valuable to share with the broader NeurIPS community.

4: excellent

3: good

2: fair

1: poor

8. Overall: Please provide an "overall score" for this submission. Choices:

10: Award quality: Technically flawless paper with groundbreaking impact on one or more areas of AI, with exceptionally strong evaluation, reproducibility, and resources, and no unaddressed ethical considerations.

9: Very Strong Accept: Technically flawless paper with groundbreaking impact on at least one area of AI and excellent impact on multiple areas of AI, with flawless evaluation, resources, and reproducibility, and no unaddressed ethical considerations.

8: Strong Accept: Technically strong paper with, with novel ideas, excellent impact on at least one area of AI or high-to-excellent impact on multiple areas of AI, with excellent evaluation, resources, and reproducibility, and no unaddressed ethical considerations.

7: Accept: Technically solid paper, with high impact on at least one sub-area of AI or moderate-to-high impact on more than one area of AI, with good-to-excellent evaluation, resources, reproducibility, and no unaddressed ethical considerations.

6: Weak Accept: Technically solid, moderate-to-high impact paper, with no major concerns with respect to evaluation, resources, reproducibility, ethical considerations.

5: Borderline accept: Technically solid paper where reasons to accept outweigh reasons to reject, e.g., limited evaluation. Please use sparingly.

4: Borderline reject: Technically solid paper where reasons to reject, e.g., limited evaluation, outweigh reasons to accept, e.g., good evaluation. Please use sparingly.

3: Reject: For instance, a paper with technical flaws, weak evaluation, inadequate reproducibility and incompletely addressed ethical considerations.

2: Strong Reject: For instance, a paper with major technical flaws, and/or poor evaluation, limited impact, poor reproducibility and mostly unaddressed ethical considerations.

1: Very Strong Reject: For instance, a paper with trivial results or unaddressed ethical considerations

9. Confidence: Please provide a "confidence score" for your assessment of this submission to indicate how confident you are in your evaluation. Choices:

5: You are absolutely certain about your assessment. You are very familiar with the related work and checked the math/other details carefully.

4: You are confident in your assessment, but not absolutely certain. It is unlikely, but not impossible, that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work.

3: You are fairly confident in your assessment. It is possible that you did not understand some parts of the submission or that you are unfamiliar with some pieces of related work. Math/other details were not carefully checked.

2: You are willing to defend your assessment, but it is quite likely that you did not understand the central parts of the submission or that you are unfamiliar with some pieces of related work. Math/other details were not carefully checked.

1: Your assessment is an educated guess. The submission is not in your area or the submission was difficult to understand. Math/other details were not carefully checked.

"""

+ template_instructions

)

class TaskAgent(AgentSystem):

def forward(self, inputs):

reviewer_system_prompt = reviewer_system_prompt_neg

review_instruction_form = neurips_form

base_prompt = review_instruction_form

base_prompt += f"""

Here is the paper you are asked to review:

‘‘‘

{inputs[’paper_text’]}

‘‘‘"""

instruction = reviewer_system_prompt + base_prompt

self.log(f"Input: {repr(instruction)}")

response, new_msg_history, _ = get_response_from_llm(

msg=instruction,

model=self.model,

msg_history=[],

)

self.log(f"Output: {repr(response)}")

# Extract the response

prediction = "None"

try:

extracted_jsons = extract_jsons(new_msg_history[-1][’text’])

prediction = extracted_jsons[-1][’Decision’]

except Exception as e:

self.log(f"Error extracting prediction: {e}")

prediction = "None"

return prediction, new_msg_history+
```

### C.3 机器人奖励设计

机器人奖励设计领域中的每项任务，都使用 Go2 四足机器人在 Genesis 模拟器（Genesis, 2024）中指定一个机器人控制目标。智能体获得该任务的文本描述（例如，以目标速度向前行走），并输出一个 Python 奖励函数；该函数随后用于为机器人训练强化学习策略（即 PPO，Schulman et al., 2017）。性能通过在模拟器中执行训练得到的强化学习策略并计算任务性能度量（例如速度跟踪误差）来评估。得分在重复评估上取平均，以降低奖励生成或强化学习中的随机性所带来的方差。

训练任务要求生成一个奖励函数，使机器人能够在跟踪目标线速度的同时向前行走。性能以指令行走速度与实际行走速度之间的均方误差来衡量。在训练期间，每个智能体最初在同一任务上重复评估 3 次，每次评估生成一个奖励函数。若至少有一个所生成的奖励函数得到非零性能得分，则该智能体再被额外评估 3 次。最终性能得分报告为这 6 次评估的平均值。本领域没有单独策划的验证任务。

为评估同一智能体能否针对不同的机器人任务生成合适的奖励函数，我们在同一机器人上将一项相对简单的训练任务与一项更具挑战性的测试任务配对。测试任务要求生成一个奖励函数，用以训练机器人最大化躯干高度。对向前行走有效的奖励函数并不会诱发跳跃行为，而跳跃行为对于最大化躯干高度更为有利。此外，直接激励躯干高度（即这一性能度量）通常会导致一种次优的站立行为，即站立停滞。因此，要取得高性能，就需要非短视的奖励设计，以鼓励中间行为，例如在起跳之前降低躯干。

测试任务的默认奖励函数直接奖励“最大化躯干高度”这一性能度量。这总会产生一种行为：机器人只是尽可能地站得更高（图 6）：

```python
def compute_reward(env) -> Tuple[Tensor, Dict, Dict]:

"""

The robot maximizes its vertical position.

"""

height = env.base_pos[:, 2]

total_reward = height

reward_components = {"height": height}

reward_scales = {"height": 1.0}

return total_reward, reward_components, reward_scales-
```

图 6：默认奖励函数与所生成奖励函数的对比。（左）默认奖励函数导致一种静止的、尽可能站高的姿态；（右）所生成的奖励函数诱发跳跃行为。橙色机器人表示起始位置，白色机器人表示回合过程中的中间位置，蓝色机器人表示结束位置。截图来自 <https://github.com/Genesis-Embodied-AI/Genesis>

### C.4 奥林匹克级别数学评分

在奥林匹克级别数学评分领域中，任务取自 IMO-GradingBench（Luong et al., 2025）。每项任务由一道奥林匹克级别数学题、一份候选解答、参考解答以及评分指南组成。智能体需要从集合 $\{0, 1, 6, 7\}$ 中指定一个离散分数，依次对应类别：不正确、部分正确、几乎正确、正确。智能体的输出是单一的数值评分。性能以相对于专家人工标注的准确率来衡量，附录 E.4 中给出了进一步分析。我们使用随机抽样的任务子集进行训练、验证与测试，每个划分包含 100 项任务。在训练期间，智能体首先在 10 项任务的子集（出自 100 项）上接受评估。若该智能体在这些任务中至少成功一项，则随后在完整训练集上接受评估。

来自 Luong et al. (2025) 的有代表性静态基线（ProofAutoGrader）：

```python
PROMPT = """You are an expert grader for the International Mathematics Olympiad (IMO).

Your task is to evaluate a proposed solution strictly and rigorously.

Keep in mind the standards at the IMO are extremely high: only arguments that are logically sound, complete, and precise should be rewarded.

### General Scoring Rubric

Scores are assigned on a 0-7 scale. The general guidelines are:

* **7 Points (Correct):** The solution is complete, correct, and fully rigorous. If the submission contains incorrect attempts or lines of reasoning but ultimately presents a complete and correct solution, it should still be awarded full points; the presence of earlier, discarded work does not detract from the final correct proof.

* **6 Points (Almost Correct):** The solution is almost correct with a sound core argument, but contains minor errors in calculation or small gaps in logic. Missing proofs for major components, unjustified claims, or sketchy arguments are **not** eligible for 6 points.

* **1 Point (Partial Progress):** The solution demonstrates substantial progress explicitly mentioned in the grading guidelines. Initial observations, reformulating the problem without making substantive headway, or proving partial results not mentioned in the grading guidelines are generally **not** eligible for this score.

* **0 Points (Incorrect):** The solution doesn’t make substantial progress that is a key step in the full solution or is fundamentally flawed. All partial progress without key results or lacking rigor also fall in this category.

### Input Data and Interpretation

You are provided with the following:

1. **Problem Statement:** The IMO problem.

2. **Ground Truth Solution:** A reference solution. Assume this solution is correct. It demonstrates one valid approach.

3. **Specific Grading Guidelines:** Criteria for awarding credit for this specific problem. These guidelines take precedence over the General Scoring Rubric, especially for partial credit.

4. **Proposed Solution:** The student submission.

### Evaluation Process

You must follow this structured process:

1. **Analyze References:** Meticulously read and understand the problem and Ground Truth Solution check the Specific Grading Guidelines. Identify the key steps for a complete solution and the criteria for partial credit.

2. **Step-by-Step Verification:** Verify the logical validity and rigor of every step. Identify all flaws, gaps, assumptions, and errors. **Make sure you fully understand every piece of logic behind each step of the proposed solution, you must be careful for solutions that ’pretend’ to be correct.**

3. **Assess Progress:** Determine the extent of non-trivial progress made.

4. **Score Determination:** Compare the findings against the Specific Grading Guidelines and the General Rubric to determine the final score.

### Output Requirements

You must provide your final score in the format <points>N out of 7</points>.

Ensure the ’<points>’ block is used **only once**, as your answer will be parsed based on the first <points> </points> block that appears in your whole response.

**PROBLEM STATEMENT**

{problem_statement}

**GROUND-TRUTH SOLUTION**

{solution}

**SPECIFIC GRADING GUIDELINES**

{grading_guidelines}

**PROPOSED SOLUTION**

{student_answer}

Present your detailed thought process and formal justification based on the scoring rubric and grading guidelines, and finally present your final score in the format below.

[Select one of the following options]

<points>7 out of 7</points>

<points>6 out of 7</points>

<points>1 out of 7</points>

<points>0 out of 7</points>

"""

class TaskAgent(AgentSystem):

"""

An automatic grader for IMO-Proof Bench.

"""

def forward(self, inputs):

# Check if all required inputs are present

if not all(key in inputs for key in ["problem", "solution", "grading_guidelines", "student_answer"]):

return None, []

# Get response

instruction = PROMPT.format(

problem_statement=inputs["problem"],

solution=inputs["solution"],

grading_guidelines=inputs["grading_guidelines"],

student_answer=inputs["student_answer"],

)

new_msg_history = chat_with_agent(instruction, model=self.model, msg_history=[], logging=self.log)

# Extract the response

prediction = "None"

try:

raw_text = new_msg_history[-1].get(’text’, ’’)

# Extract content between <points>...</points>

match = re.search(r"<points>(.*?)</points>", raw_text, re.DOTALL)

if match:

points_text = match.group(1).strip() # e.g., "7 out of 7"

# Extract just the leading integer

num_match = re.search(r"\d+", points_text)

if num_match:

prediction = int(num_match.group()) # e.g., 7

# Map prediction to reward text

reward_map = {

0: "incorrect",

1: "partial",

6: "almost",

7: "correct",

}

prediction = reward_map.get(prediction, "None")

else:

self.log("No numeric score found inside <points> tag.")

prediction = "None"

else:

self.log("No <points> tag found in model output.")

prediction = "None"

except Exception as e:

self.log(f"Error extracting prediction: {e}")

prediction = "None"

return prediction, new_msg_history-
```

## 附录 D 实验细节

本附录提供进一步的实验细节，以支持结果的可复现性。我们首先汇总各领域中用于自我修改与任务评估的基础模型及超参数（附录 D.1），随后估计在每种设置下运行 DGM-H 的计算成本（附录 D.2）。接着，我们形式化定义 improvement@k 指标，用以在固定预算下量化智能体产生改进变体的能力（附录 D.3）。最后，我们描述为跨领域实验选择迁移智能体所用的流程（附录 D.4）。

### D.1 基础模型的超参数

表 3 汇总了各实验设置中使用的基础模型（FM）。对于 Polyglot 编程领域，我们采用与 Zhang et al. (2025b) 相同的基础模型与温度配置，以确保公平比较。在所有其他领域中，鉴于 Claude-4.5-Sonnet 在编程上的强劲表现，我们使用它进行自我修改。对于任务评估，我们基于实际考量来选择基础模型，这些考量包括计算成本、速率限制、响应延迟以及整体任务能力。在机器人奖励设计设置中，智能体必须以代码实现奖励函数，我们再次使用 Claude-4.5-Sonnet。对于需要大量数学推理的奥林匹克级别数学评分，我们使用 o4-mini。在每种设置中，除 o4-mini 固定为 1.0 外，所有基础模型的温度均设为 0.0。

表 3：各实验设置中用于自我修改或任务评估的基础模型。

| 领域 | 自我修改 | 评估 |
| --- | --- | --- |
| Polyglot | Claude 3.5 Sonnet (New) | o3-mini |
| 论文评审 | Claude 4.5 Sonnet | GPT-4o |
| 机器人奖励设计 | Claude 4.5 Sonnet | Claude 4.5 Sonnet |
| IMO 级别评分 | Claude 4.5 Sonnet | o4-mini |

### D.2 成本估计

将 DGM-H 运行 100 次迭代，仅自我修改阶段的成本就约为 33M token（不含任务评估）。因此，一次实验的总成本由自我修改成本加上任务评估成本构成。对于论文评审与机器人奖励设计实验，每次迭代的评估成本为 0.506M token（论文评审评估占 0.5M token，机器人奖励设计评估占 0.006M token）。因此，对于一次 100 次迭代的运行（第 5 节），估计总成本为自我修改的 33M token，再加上评估的 0.506M × 100，合计约为 88.6M token。

任务评估成本的更细粒度分解如下：

| 基础模型 | 基准 | 任务数量 | 成本估计（M，token） |
| --- | --- | --- | --- |
| o3-mini | Polyglot | 60 | 0.89M |
| GPT-4o | 论文评审 | 100 | 0.5M |
| Claude-4.5-sonnet | 机器人奖励设计 | 6 | 0.006M |
| o4-mini | IMO-GradingBench | 100 | 0.11M |

### D.3 Improvement@k 指标

设 $M$ 表示初始元智能体，$A$ 表示初始任务智能体，$\mathcal{T}$ 表示一组固定的评估任务。设 $\mathcal{G}$ 表示一种智能体生成算法（例如 DGM 或 DGM-H 变体）。元智能体 $M$ 保持固定，并允许其从 $A$ 出发，通过迭代应用 $\mathcal{G}$，最多生成 $k$ 个新的任务智能体。

设

$$
\mathcal{A}^{(k)}=\{A^{1},A^{2},\dots,A^{k}\}
$$

表示 $M$ 在 $k$ 步修改之内生成的任务智能体集合。每个任务智能体 $A^{\prime}\in\{A\}\cup\mathcal{A}^{(k)}$ 都使用固定评估流程 $\mathrm{Evaluate}(\cdot,\mathcal{T})$ 在 $\mathcal{T}$ 上接受评估，其中数值越高表示性能越好。

我们将 improvement@k 指标定义为

$$
\mathrm{imp@}k(M,A,\mathcal{G},\mathcal{T})\;=\;\max_{A^{\prime}\in\mathcal{A}^{(k)}(M,A,\mathcal{G})}\mathrm{Evaluate}(A^{\prime},\mathcal{T})\;-\;\mathrm{Evaluate}(A,\mathcal{T}),
$$

直观而言，imp@k 衡量的是：一个固定的元智能体 $M$ 在特定智能体生成算法 $\mathcal{G}$ 下运行时，从初始任务智能体 $A$ 出发，通过最多生成 $k$ 个经修改的任务智能体所能获得的最大性能改进。imp@k 的取值越大，表明在固定计算预算与生成流程下的智能体生成能力越强。

imp@k 的一个局限在于，它将性能改进视为线性的，而未考虑不同性能水平之间的难度差异。特别是，接近饱和处的改进（例如将准确率从 0.7 提高到 0.8）可能比在较低性能水平上取得同等绝对增益（例如从 0.0 到 0.1）要困难得多。因此，imp@k 可能会低估在较高性能区间所取得改进的重要性。然而，这一局限并不影响本文所给出的分析，因为 imp@k 主要用于在相互匹配的初始条件与固定评估预算下进行相对比较，而在这些条件下所有方法都受到相同饱和效应的制约（第 5.2 节）。

### D.4 迁移智能体选择

为迁移实验（第 5.2 节与第 5.3 节）选择智能体时，我们采用一种后代增长准则：它偏好那些能够充当后续改进的有力垫脚石的智能体，而不是仅仅自身得分较高的智能体。具体而言，给定第 $t$ 次迭代时的最终存档

$$
\mathcal{A}^{t}=\{a_{0},a_{1},\dots,a_{t}\},
$$

设 $\alpha_{i}$ 为智能体 $a_{i}$ 的评估得分：当源领域验证集可用时，取该验证集上的得分，否则取训练集上的得分。设 $\mathrm{parent}(j)$ 表示存档树中节点 $j$ 的父节点，并设 $\mathrm{dist}(i,j)$ 为从 $i$ 到后代 $j$ 的唯一路径上的边数。

我们将候选迁移节点 $i$ 的*增长分数*定义为其后代相对于 $i$ 所取得的折扣平均改进：

$$
G_{\gamma}(i)\;=\;\frac{1}{|\mathcal{D}(i)|}\sum_{j\in\mathcal{D}(i)}\bigl(\alpha_{j}-\alpha_{i}\bigr)\,\gamma^{\mathrm{dist}(i,j)},
$$

其中 $\mathcal{D}(i)$ 是存档树中 $i$ 的后代集合，$\gamma\in(0,1]$ 控制我们对发生在 $i$ 之后许多代的改进施加折扣的强度。直观而言，$G_{\gamma}(i)$ 对那些能够在更少的自我修改步骤内可靠地生成更好后代的智能体赋予更高权重，我们将其视为更强智能体生成能力的证据。

在实验中，我们取 $\gamma=0.6$，并选择 $G_{0.6}(i)$ 最高的迁移智能体。为降低噪声，我们只考虑至少有 3 个后代的节点。

## 附录 E 补充结果

本附录呈现补充主要发现的额外定性结果与诊断结果。我们首先着重展示 DGM-H 所发现的最佳任务智能体（附录 E.1）。随后，我们定性分析 DGM-H 如何在不同领域提升任务表现（附录 E.2），以及它如何发展出能够提升其自我改进能力的元层面能力（附录 E.3）。接下来，我们分析自动发现的奥林匹克级别数学评分器的行为（附录 E.4）。我们还报告了初步实验：允许 DGM-H 修改其自身的父代选择机制，从而揭示完全自指优化的限度与潜力（附录 E.5）。所有实验日志均已在我们的代码库中开源。

### E.1 发现的最佳任务智能体

我们展示对任务智能体有所贡献、且与相应领域相关的部分 diff 补丁。完整的 diff 补丁已在我们的代码库中开源。

#### E.1.1 论文评审

以下 diff 补丁构成 DGM-H 为论文评审所发现的最佳任务智能体（第 5.1 节）：

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index 3798256..42ab625 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -5,7 +5,7 @@ from utils.common import extract_jsons

class TaskAgent(AgentSystem):

def forward(self, inputs):

"""

- An agent that solves a given task.

+ An agent that solves a given task with enhanced reasoning and error handling.

Args:

inputs (dict): A dictionary with input data for the task.

@@ -15,30 +15,80 @@ class TaskAgent(AgentSystem):

- prediction (str): The prediction made by the agent.

- new_msg_history (list): A list of messages...

"""

- domain = inputs[’domain’]

- instruction = f"""You are an agent.

+ domain = inputs.get(’domain’, ’unknown’)

+

+ # Enhanced instruction with chain-of-thought reasoning

+ instruction = f"""You are an expert agent solving tasks in the ’{domain}’ domain.

Task input:

‘‘‘

{inputs}

‘‘‘

+Please analyze this task carefully and provide your response. Follow these steps:

+1. Understand the task requirements

+2. Consider relevant approaches or solutions

+3. Provide your final answer

+

Respond in JSON format with the following schema:

<json>

{{

- "response": ...

+ "reasoning": "Brief explanation of your approach",

+ "response": "Your final answer"

}}

-</json>"""

- new_msg_history = chat_with_agent(instruction, ...)

+</json>

+

+Note: The ’response’ field is required and should contain your answer."""

+

+ try:

+ new_msg_history = chat_with_agent(

+ instruction,

+ model=self.model,

+ msg_history=[],

+ logging=self.log

+ )

+ except Exception as e:

+ self.log(f"Error in chat_with_agent: {e}")

+ return "Error: Failed to get response from agent", []

- # Extract the response

+ # Extract the response with improved error handling

prediction = "None"

try:

- extracted_jsons = extract_jsons(new_msg_history[-1][’text’])

- if extracted_jsons is not None and "response" in extracted_jsons[-1]:

- prediction = extracted_jsons[-1][’response’]

+ if new_msg_history and len(new_msg_history) > 0:

+ last_message = new_msg_history[-1].get(’text’, ’’)

+ extracted_jsons = extract_jsons(last_message)

+

+ if extracted_jsons is not None and len(extracted_jsons) > 0:

+ last_json = extracted_jsons[-1]

+

+ if "response" in last_json:

+ prediction = last_json[’response’]

+ if "reasoning" in last_json:

+ self.log(f"Agent reasoning: {last_json[’reasoning’]}")

+ else:

+ self.log("Warning: JSON response missing ’response’ field")

+ prediction = str(last_json)

+ else:

+ self.log("Warning: No valid JSON found in response")

+ prediction = last_message[:500]

except Exception as e:

self.log(f"Error extracting prediction: {e}")

- prediction = "None"

+ # Attempt to recover with raw response

+ try:

+ if new_msg_history and len(new_msg_history) > 0:

+ prediction = new_msg_history[-1].get(’text’, ’Error’)[:500]

+ except:

+ prediction = "Error: Complete extraction failure"

return prediction, new_msg_history-
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

\--- a/task_agent.py

+++ b/task_agent.py

@@ -17,8 +17,35 @@ class TaskAgent(AgentSystem):

"""

domain = inputs.get(’domain’, ’unknown’)

- # Enhanced instruction with chain-of-thought reasoning

- instruction = f"""You are an expert agent solving tasks...

+ # Domain-specific instructions

+ if domain == ’paper_review’:

+ paper_text = inputs.get(’paper_text’, ’’)

+ instruction = f"""You are an expert reviewer evaluating academic papers.

+Your task is to decide whether to ACCEPT or REJECT the paper.

+

+Paper to review:

+‘‘‘

+{paper_text}

+‘‘‘

+

+Evaluate the paper based on:

+1. Novelty and originality of the research

+2. Technical soundness and methodology

+3. Experimental validation and results

+4. Clarity of presentation

+5. Significance of contributions

+

+Provide your decision in JSON format:

+<json>

+{{

+ "reasoning": "Brief explanation of your decision (2-3 sentences)",

+ "response": "accept or reject"

+}}

+</json>

+

+IMPORTANT: The ’response’ field must be EXACTLY either ’accept’ or ’reject’

+(lowercase, one word only)."""

+ else:

+ # Generic instruction for other domains

+ instruction = f"""You are an expert agent solving tasks in the ’{domain}’ domain."
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

\--- a/task_agent.py

+++ b/task_agent.py

@@ -20,24 +20,43 @@ class TaskAgent(AgentSystem):

# Domain-specific instructions

if domain == ’paper_review’:

paper_text = inputs.get(’paper_text’, ’’)

- instruction = f"""You are an expert reviewer evaluating academic papers...

+ instruction = f"""You are a rigorous and critical academic reviewer

+evaluating papers for a top-tier conference. Your task is to decide whether to

+ACCEPT or REJECT the paper. Be skeptical and thorough.

Paper to review:

‘‘‘

{paper_text}

‘‘‘

-Evaluate the paper based on:

-1. Novelty and originality of the research

-...

+Evaluate the paper critically based on:

+1. **Novelty and Originality**: Is the contribution truly novel or incremental?

+2. **Technical Soundness**: Are methods rigorous? Any flaws in theory?

+3. **Experimental Validation**: Comprehensive? Sufficient baselines?

+4. **Clarity and Presentation**: Is writing clear? Key details missing?

+5. **Significance**: Does this advance the field meaningfully?

+

+**RED FLAGS that typically warrant REJECTION:**

+- Incremental improvements without significant novelty

+- Missing critical experimental comparisons or baselines

+- Unclear methodology or missing implementation details

+- Overclaimed contributions not supported by evidence

+- Poor writing that obscures understanding

+- Limited experimental validation or weak results

+- Lack of theoretical justification or soundness issues

+- Insufficient comparison with prior state-of-the-art

+

+**Standards for ACCEPTANCE:**

+- Clear novel contribution that advances the field

+- Rigorous methodology with sound theoretical foundation

+- Comprehensive experiments with strong results

+- Well-written with clear explanations

+- Properly compared against relevant baselines

+

+**Be Critical**: Default to REJECT unless the paper clearly meets high standards.

Provide your decision in JSON format:

<json>

{{

- "reasoning": "Brief explanation of your decision (2-3 sentences)",

+ "reasoning": "Brief critical analysis (2-3 sentences on key strengths/weaknesses)",

"response": "accept or reject"

}}

</json>"
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

\--- a/task_agent.py

+++ b/task_agent.py

@@ -20,43 +20,63 @@ class TaskAgent(AgentSystem):

# Domain-specific instructions

if domain == ’paper_review’:

paper_text = inputs.get(’paper_text’, ’’)

- instruction = f"""You are a rigorous and critical academic reviewer...

+ instruction = f"""You are a critical academic reviewer for a top-tier

+conference. Use a two-stage evaluation process:

Paper to review:

‘‘‘

{paper_text}

‘‘‘

-Evaluate the paper critically based on:

-1. **Novelty and Originality**: Is the contribution truly novel?

-...

-

-**RED FLAGS that typically warrant REJECTION:**

-...

-

-**Standards for ACCEPTANCE:**

-...

-

-**Be Critical**: Default to REJECT unless the paper clearly meets high standards.

-

-Provide your decision in JSON format:

+**STAGE 1: Identify Weaknesses**

+Systematically check for these common issues:

+

+1. **Novelty Issues:**

+ - Is this just an incremental modification of existing work?

+ - Are the contributions overclaimed or trivial?

+

+2. **Methodological Flaws:**

+ - Are key implementation details missing?

+ - Are there questionable assumptions or theoretical gaps?

+

+3. **Experimental Weaknesses:**

+ - Missing important baselines or comparisons?

+ - Weak or marginal improvements?

+

+4. **Presentation Problems:**

+ - Unclear writing or poor organization?

+ - Missing critical details for reproducibility?

+

+5. **Limited Scope:**

+ - Tested only on toy problems?

+ - Narrow applicability?

+

+**STAGE 2: Make Decision**

+A paper should be **REJECTED** if:

+- It has 2 or more significant weaknesses from Stage 1

+- It has 1 critical flaw

+- The contribution is too incremental

+

+A paper should be **ACCEPTED** only if:

+- It has clear novel contributions

+- Strong experimental validation

+- Well-written and technically sound

+- At most minor, fixable issues

+

+**Critical Mindset:** Most papers have significant flaws.

+About 50% of papers should be rejected.

+

+Provide your evaluation in JSON format:

<json>

{{

- "reasoning": "Brief critical analysis (2-3 sentences)",

+ "weaknesses_found": "List 2-3 main weaknesses identified in Stage 1",

+ "reasoning": "Based on the weaknesses, explain why you accept or reject",

"response": "accept or reject"

}}

</json>"
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

\--- a/task_agent.py

+++ b/task_agent.py

@@ -20,68 +20,71 @@ class TaskAgent(AgentSystem):

# Domain-specific instructions

if domain == ’paper_review’:

paper_text = inputs.get(’paper_text’, ’’)

- instruction = f"""You are a critical academic reviewer...

-

-**STAGE 1: Identify Weaknesses**

-...

+ instruction = f"""You are a rigorous peer reviewer for a top-tier

+academic conference. Your reputation depends on making careful, balanced decisions.

-**STAGE 2: Make Decision**

-...

-

-Provide your evaluation in JSON format:

-<json>

-{{

- "weaknesses_found": "List 2-3 main weaknesses identified in Stage 1",

- "reasoning": "Based on the weaknesses, explain why you accept or reject",

- "response": "accept or reject"

-}}

-</json>

-

-IMPORTANT: The ’response’ field must be EXACTLY either ’accept’ or ’reject’..."""

+**Paper to Review:**

+{paper_text}

+**Evaluation Framework:**

+**ACCEPTANCE CRITERIA (ALL must be satisfied):**

+1. **Novelty**: Presents genuinely new ideas, not incremental improvements

+2. **Technical Soundness**: Methodology is rigorous, correct, and well-justified

+3. **Significance**: Makes important contributions that advance the field

+4. **Experimental Validation**: Claims thoroughly supported with comprehensive experiments

+5. **Clarity**: Clear, well-organized presentation with proper motivation

+6. **Reproducibility**: Sufficient detail for replication

+7. **Related Work**: Proper comparison with existing methods

+**REJECTION CRITERIA (ANY one can warrant rejection):**

+- Lacks novelty; merely combines existing techniques without insight

+- Methodological flaws or incorrect technical approach

+- Insufficient experimental validation or cherry-picked results

+- Missing critical baselines or unfair comparisons

+- Overclaimed contributions not supported by evidence

+- Poor writing quality that obscures technical content

+- Ethical concerns or reproducibility issues

+- Limited scope or significance

+**Decision Guidelines:**

+- **ACCEPT**: Paper clearly satisfies ALL acceptance criteria with strong evidence

+- **REJECT**: Paper fails one or more acceptance criteria OR matches rejection criteria

+- **When in doubt, err on the side of rejection**

+**Your Review Process:**

+1. Identify the paper’s main claims and contributions

+2. Evaluate each acceptance criterion systematically

+3. Check for rejection criteria

+4. Ask: "Does this paper significantly advance the field?"

+5. Make a decision based on evidence, not superficial features

+**Response Format:**

+Provide your decision in this exact JSON format:

+<json>

+{{

+ "response": "accept"

+}}

+</json>

+OR

+<json>

+{{

+ "response": "reject"

+}}

+</json>

+**CRITICAL REQUIREMENTS:**

+- The "response" field must contain ONLY "accept" or "reject" (lowercase)

+- Be thorough and critical - rejecting weak papers maintains scientific standards

+- Your decision should be defensible based on the criteria above

+- ONLY output the single word: "accept" or "reject" """’
```

#### E.1.2 机器人奖励设计

以下 diff 补丁构成 DGM-H 为机器人奖励设计所发现的最佳任务智能体（第 5.1 节）：

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index 3798256..98e6706 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -5,7 +5,7 @@ from utils.common import extract_jsons

class TaskAgent(AgentSystem):

def forward(self, inputs):

"""

- An agent that solves a given task.

+ An agent that solves a given task with enhanced reasoning and error handling.

...

"""

- domain = inputs[’domain’]

- instruction = f"""You are an agent.

+ domain = inputs.get(’domain’, ’general’)

+

+ # Enhanced instruction with clearer structure and reasoning guidance

+ instruction = f"""You are an expert AI agent specialized in solving complex tasks.

-Task input:

+Task Domain: {domain}

+

+Task Input:

‘‘‘

{inputs}

‘‘‘

+Instructions:

+1. Carefully analyze the task input and identify the key requirements

+2. Break down the problem into logical steps if needed

+3. Formulate your response based on the task requirements

+4. Provide your final answer in the specified JSON format

+

Respond in JSON format with the following schema:

<json>

{{

- "response": ...

+ "reasoning": "Brief explanation of your approach",

+ "response": "Your final answer here"

}}

-</json>"""

- new_msg_history = chat_with_agent(instruction, model=self.model, msg_history=[], logging=self.log)

+</json>

- # Extract the response

+IMPORTANT: Always include both ’reasoning’ and ’response’ fields in your JSON output."""

+

+ # Attempt to get response with retry logic

+ max_retries = 2

prediction = "None"

- try:

- extracted_jsons = extract_jsons(new_msg_history[-1][’text’])

- if extracted_jsons is not None and "response" in extracted_jsons[-1]:

- prediction = extracted_jsons[-1][’response’]

- except Exception as e:

- self.log(f"Error extracting prediction: {e}")

- prediction = "None"

+ new_msg_history = []

+

+ for attempt in range(max_retries + 1):

+ try:

+ new_msg_history = chat_with_agent(...)

+ # Extract the response with retry logic

+ if new_msg_history and len(new_msg_history) > 0:

+ extracted_jsons = extract_jsons(new_msg_history[-1][’text’])

+ if extracted_jsons is not None and len(extracted_jsons) > 0:

+ response_json = extracted_jsons[-1]

+ if "reasoning" in response_json:

+ self.log(f"Agent reasoning: {response_json[’reasoning’]}")

+ if "response" in response_json:

+ prediction = response_json[’response’]

+ break

+ except Exception as e:

+ self.log(f"Error in attempt {attempt + 1}: {str(e)}")

return prediction, new_msg_history
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index 3c2dd8a..faab41d 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -27,30 +27,96 @@ Paper Text:

+Critical Evaluation Guidelines:

+- Be rigorous and selective - most submissions have issues

+- Accept ONLY papers that are strong across MOST criteria (4-5/5)

+- Reject papers with significant weaknesses in multiple areas

+- A paper needs to be clearly above average to warrant acceptance

+- Consider: Would this paper strengthen the conference program?

+- If a paper is borderline or has major concerns, lean toward REJECT

+ elif domain == ’genesis_go2walking’:

+ # Genesis domain-specific instruction with environment attribute documentation

+ task_description = inputs.get(’task_description’, ’’)

+ genesis_env_path = inputs.get(’genesis_environment_path’, ’’)

+ default_reward_function = inputs.get(’default_reward_function’, ’’)

+

+ instruction = f"""You are an expert in reinforcement learning and reward function design.

+Task: {task_description}

+Default Reward Function:

+‘‘‘python

+{default_reward_function}

+‘‘‘

+AVAILABLE ENVIRONMENT ATTRIBUTES (use ONLY these):

+- env.commands: [num_envs, 3] - velocity commands [lin_vel_x, lin_vel_y, ang_vel_z]

+- env.base_lin_vel: [num_envs, 3] - base linear velocity [x, y, z]

+- env.base_ang_vel: [num_envs, 3] - base angular velocity [roll, pitch, yaw]

+- env.base_pos: [num_envs, 3] - base position [x, y, z]

+- env.base_quat: [num_envs, 4] - base orientation quaternion

+- env.projected_gravity: [num_envs, 3] - gravity vector in base frame

+- env.dof_pos: [num_envs, 12] - joint positions

+- env.dof_vel: [num_envs, 12] - joint velocities

+- env.actions: [num_envs, 12] - current actions

+- env.last_actions: [num_envs, 12] - previous actions

+- env.last_dof_vel: [num_envs, 12] - previous joint velocities

+IMPORTANT CONSTRAINTS:

+- DO NOT use env.torques (not available)

+- DO NOT access undefined attributes

+- Use only the attributes listed above

+- All operations must work with PyTorch tensors"
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index faab41d..62c444f 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -88,19 +90,24 @@ AVAILABLE ENVIRONMENT ATTRIBUTES (use ONLY these):

-IMPORTANT CONSTRAINTS:

\-- DO NOT use env.torques (not available)

\-- DO NOT access undefined attributes

\-- Use only the attributes listed above

\-- All operations must work with PyTorch tensors

+CRITICAL RULES (VIOLATION WILL CAUSE IMMEDIATE FAILURE):

+1. DO NOT reference env.torques or env.dt in your code - THESE DO NOT EXIST

+2. DO NOT add hasattr() checks for unavailable attributes and then use them

+3. DO NOT assume any attributes beyond the exact list above

+4. NEVER write code like: if hasattr(env, ’torques’): use env.torques

+5. All operations must work with PyTorch tensors

+6. If you need time differences, use fixed timestep: dt = 0.02

+7. ONLY use attributes from the AVAILABLE list - nothing else exists

+ # Post-process prediction based on domain

+ if domain == ’paper_review’:

+ # Ensure response is exactly "accept" or "reject"

+ prediction_lower = str(prediction).lower().strip()

+ if ’accept’ in prediction_lower and ’reject’ not in prediction_lower:

+ prediction = ’accept’

+ elif ’reject’ in prediction_lower:

+ prediction = ’reject’

+ else:

+ self.log(f"Warning: Ambiguous response ’{prediction}’, defaulting to ’reject’")

+ prediction = ’reject’
```

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index 62c444f..fb504a2 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -40,26 +40,30 @@ Critical Evaluation Guidelines:

\-- If a paper is borderline or has major concerns, lean toward REJECT

+- Balance your decisions: aim for appropriate selectivity (not too lenient, not too harsh)

+- If a paper is borderline, carefully weigh the evidence on both sides

Reward Function Design Guidelines:

1. Primary reward: Track forward velocity command (env.commands[:, 0] vs env.base_lin_vel[:, 0])

+ - Use exponential reward: torch.exp(-error * temperature) for smooth gradients

+ - Typical temperature: 1.0-2.0, scale: 0.8-1.0

2. Secondary rewards: Track angular velocity, maintain base height, stable orientation

-3. Penalties: Excessive action changes (use env.actions - env.last_actions)

-4. Use exponential rewards: torch.exp(-error * temperature) for smooth gradients

-5. Scale rewards appropriately (main objectives: 0.5-1.0, penalties: -0.5 to -0.01)

-6. For smooth action changes: use torch.sum(torch.square(env.actions - env.last_actions))

-7. For velocity smoothness: use torch.sum(torch.square(env.dof_vel - env.last_dof_vel))

-8. Return: total_reward, reward_components dict, reward_scales dict

+ - Angular velocity tracking: similar to linear velocity

+ - Base height: penalize deviation from target (typically 0.32m)

+ - Orientation: use env.projected_gravity to penalize tilt

+3. Penalties for smooth and stable locomotion:

+ - Action smoothness: -0.01 * torch.sum(torch.square(env.actions - env.last_actions), dim=1)

+ - Joint velocity smoothness: -0.001 * torch.sum(torch.square(env.dof_vel - env.last_dof_vel), dim=1)

+ - Excessive joint velocities: -0.0005 * torch.sum(torch.square(env.dof_vel), dim=1)

+ - Unwanted lateral movement: penalize env.base_lin_vel[:, 1]

+4. Balance reward components:

+ - Main tracking objectives: 0.5-1.0 scale

+ - Secondary objectives: 0.2-0.5 scale

+ - Smoothness penalties: -0.01 to -0.001 scale

+5. Return: total_reward, reward_components dict, reward_scales dict+
```

#### E.1.3 奥林匹克级别数学评分

以下 diff 补丁构成最佳任务智能体（我们在附录 E.4 中称之为 BetterGrader）。该智能体由采用迁移、并从 ProofAutoGrader 出发的 DGM-H 发现（第 5.3 节），用于奥林匹克级别数学评分：

```
[⬇](<)

diff \--git a/task_agent.py b/task_agent.py

index 9ab8761..c00d167 100644

\--- a/task_agent.py

+++ b/task_agent.py

@@ -11,10 +11,25 @@ Keep in mind the standards at the IMO are extremely high...

### General Scoring Rubric

Scores are assigned on a 0-7 scale. The general guidelines are:

-* **7 Points (Correct):** The solution is complete, correct, and fully rigorous...

-* **6 Points (Almost Correct):** The solution is almost correct with a sound core argument, but contains minor errors...

-* **1 Point (Partial Progress):** The solution demonstrates substantial progress explicitly mentioned in the grading guidelines...

-* **0 Points (Incorrect):** The solution doesn’t make substantial progress...

+

+* **7 Points (Correct):** The solution is complete, correct, and fully rigorous with no gaps or errors. All major steps are proven with full rigor. Every claim is justified or routine to verify...

+

+* **6 Points (Almost Correct):** The solution has ALL the major ideas and the core argument structure is sound, but contains ONE OR MORE of these minor issues:

+ - Minor algebraic/arithmetic errors that don’t affect the main argument

+ - Small logical gaps that are straightforward to fill

+ - Missing routine verifications that an expert could easily supply

+ - **NOT eligible for 6 points**: Missing proofs for major lemmas, unjustified non-trivial claims, incomplete case analysis, or fundamental logical gaps

+ - **Key test**: Would an expert say "this is essentially correct, just needs minor cleanup"?

+

+* **1 Point (Partial Progress):** The solution demonstrates substantial progress on a KEY component that is explicitly mentioned in the grading guidelines for partial credit.

+ - **CRITICAL**: Carefully read the Specific Grading Guidelines section to see what counts as partial credit

+ - The solution must achieve one of the specific milestones listed in the guidelines

+ - Must make non-trivial progress toward the solution (not just initial observations)

+ - Reformulating the problem without making substantive headway is NOT sufficient

+ - Proving lemmas NOT mentioned in grading guidelines is NOT sufficient

+ - **If the guidelines list specific achievements for partial credit and the solution achieves ANY of them, award 1 point**

+

+* **0 Points (Incorrect):** The solution doesn’t make substantial progress on key steps mentioned in grading guidelines, is fundamentally flawed, or makes only trivial observations.

### Evaluation Process

You must follow this structured process:

-1. **Analyze References:** Meticulously read and understand the problem...

-2. **Step-by-Step Verification:** Verify the logical validity and rigor of every step...

-3. **Assess Progress:** Determine the extent of non-trivial progress made.

-4. **Score Determination:** Compare the findings against the Specific Grading Guidelines...

+

+1. **Analyze References:** Meticulously read and understand the problem and Ground Truth Solution. Carefully review the Specific Grading Guidelines to identify:

+ - The key steps required for a complete solution

+ - **What specific progress qualifies for partial credit (1 point)** - this is crucial!

+ - What distinguishes "almost correct" (6 points) from "correct" (7 points)

+

+2. **Step-by-Step Verification:** Verify the logical validity and rigor of EVERY step in the proposed solution:

+ - Identify ALL flaws, gaps, assumptions, and errors

+ - Check if gaps are "minor and routine" (possibly 6 pts) or "major" (0-1 pts)

+ - **Be careful**: Some solutions may appear correct but have hidden gaps or unjustified leaps

+ - Distinguish between minor calculation errors vs fundamental logical flaws

+

+3. **Assess Progress Against Grading Guidelines:**

+ - **Does the solution achieve the specific milestones mentioned for partial credit?**

+ - Does it have all major components with only minor fixable issues (6 points)?

+ - Or does it have complete rigor with no gaps (7 points)?

+

+4. **Score Determination:** Apply this decision tree:

+ - **If solution is complete and rigorous with no errors or gaps -> 7 points**

+ - **If solution has all major ideas but minor fixable issues -> 6 points**

+ - **If solution makes substantial progress mentioned in grading guidelines -> 1 point**

+ - **Otherwise -> 0 points**

+### Critical Reminders for Accurate Grading

+

+**Common Grading Errors to AVOID:**

+1. **Being too lenient with 7 points**: If there are ANY gaps (even minor ones that need filling), it’s 6 points, not 7.

+2. **Missing partial credit**: Check the grading guidelines carefully - if the solution achieves ANY milestone mentioned for partial credit, award 1 point.

+3. **Confusing "good attempt" with "almost correct"**: 6 points requires ALL major ideas to be present, not just a good start.

+4. **Ignoring unjustified claims**: Statements like "it’s easy to see" or "clearly" must actually be clear/easy. If non-trivial, it’s a gap.

+**Score Distribution Calibration:**

+- Most solutions will be 0 or 7 points (either fundamentally flawed or correct)

+- 6 points should be rare (only when truly "almost there" with all ideas present)

+- 1 point should match specific milestones in grading guidelines

+- When in doubt, be strict: IMO standards are extremely high
```

```diff
diff \--git a/task_agent.py b/task_agent.py
index c00d167..72475b6 100644
\--- a/task_agent.py
+++ b/task_agent.py
@@ -28,8 +28,11 @@ Scores are assigned on a 0-7 scale...
- Proving lemmas NOT mentioned in grading guidelines is NOT sufficient
- **If the guidelines list specific achievements for partial credit and the solution achieves ANY of them, award 1 point**
+ - **IMPORTANT**: Even if the solution has major flaws or is incomplete, if it achieves ANY specific milestone from the guidelines, it deserves 1 point, not 0
* **0 Points (Incorrect):** The solution doesn’t make substantial progress...
+ - **CRITICAL CHECK**: Before assigning 0 points, verify that the solution does NOT achieve ANY of the partial credit milestones listed in the grading guidelines
+ - If even ONE milestone is achieved, the score should be 1 point, not 0
+3. **MANDATORY Partial Credit Milestone Check:**
+ - **THIS STEP IS REQUIRED - DO NOT SKIP**
+ - List each partial credit milestone explicitly from the grading guidelines
+ - For EACH milestone, determine: Does the solution achieve this? YES/NO
+ - If ANY milestone shows YES -> the score must be at least 1 point
+ - This check must happen BEFORE considering 0 points
+5. **Score Determination:** Apply this decision tree:
+ - **FIRST**: Did the solution achieve ANY partial credit milestone from the guidelines?
+ - If YES -> Score is AT LEAST 1 point (proceed to check for higher scores)
+ - If NO -> Score is 0 points (stop here)
+
+ - **If score >= 1, check for higher scores:**
+ - Is the solution complete and rigorous with no errors or gaps? -> 7 points
+ - Does it have ALL major ideas with only minor fixable issues? -> 6 points
+ - Otherwise -> 1 point
+**REQUIRED EVALUATION FORMAT:**
+You MUST structure your response as follows:
+
+1. **List Partial Credit Milestones**: Explicitly list each milestone from the grading guidelines
+2. **Check Each Milestone**: For each milestone, state whether the solution achieves it (YES/NO)
+3. **Determine Minimum Score**: If ANY milestone is YES, the minimum score is 1 point
+4. **Detailed Analysis**: Provide your complete evaluation
+5. **Final Score**: Provide the score in the required format-
```

```diff
diff \--git a/task_agent.py b/task_agent.py
index 72475b6..62e975b 100644
\--- a/task_agent.py
+++ b/task_agent.py
+* **7 Points (Correct):** The solution is complete, correct, and fully rigorous with no gaps or errors. All major steps are proven with full rigor. Every claim is justified or truly routine to verify...
+ - **Critical requirements**: Solution must (1) address ALL parts of the problem, (2) prove ALL necessary claims, and (3) have NO unjustified leaps
+ - Every "clearly" or "obviously" must be genuinely trivial to an IMO expert
+ - All edge cases, special cases, and boundary conditions must be handled
+* **6 Points (Almost Correct):** The solution has ALL the major ideas and the core argument structure is sound, but contains ONE OR MORE of these **minor** issues:
+ - Minor algebraic/arithmetic errors that don’t affect the main argument (e.g., writing 2n+1 instead of 2n+2 but the logic still holds)
+ - Small logical gaps that are straightforward to fill (e.g., "clearly" statements that are indeed clear to an expert)
+ - Missing routine verifications that an expert could easily supply (e.g., obvious algebra steps)
+ - **NOT eligible for 6 points**: Missing proofs for major lemmas, unjustified non-trivial claims, incomplete case analysis, fundamental logical gaps, or missing key components
+ - **Key test**: Would an expert say "this is essentially correct, just needs minor cleanup"? The solution structure is complete and sound.
+ - **Example of 6 points**: A proof that has all the right ideas and structure but makes a small computational error that doesn’t invalidate the approach
+ - **Example of NOT 6 points**: A proof that sketches the right approach but leaves out the proof of a crucial intermediate result
+2. **Step-by-Step Verification:** Verify the logical validity and rigor of EVERY step in the proposed solution:
+ - Identify ALL flaws, gaps, assumptions, and errors
+ - For EACH "clearly" or "obviously" statement: Is it truly routine or does it hide significant work?
+ - Check if gaps are "minor and routine" (possibly 6 pts) or "major" (0-1 pts)
+ - **Be careful**: Some solutions may appear correct but have hidden gaps or unjustified leaps
+ - Distinguish between minor calculation errors vs fundamental logical flaws
+ - **Rigor check**: Are intermediate results properly proven? Are all cases covered? Are inequalities/equalities justified?
+4. **Assess Overall Quality and Completeness:**
+ - **For 7 points**: Check that EVERY claim is proven, EVERY case is handled, NO unjustified leaps exist
+ - **For 6 points**: Verify ALL major components are present and the solution structure is complete (not just a good start)
+ - **For 1 point**: Confirm at least ONE specific milestone from guidelines is achieved
+ - **For 0 points**: Verify NO milestones from guidelines are achieved
+**Examples to Calibrate Your Judgment:**
+- **7 points**: "The proof correctly establishes X by showing Y, handles all cases including Z, and proves every intermediate claim with full rigor."
+- **6 points**: "The proof has the complete structure and all major steps, but writes ’it is clear that’ for a non-trivial step that needs 2-3 lines to verify."
+- **1 point**: "The solution proves Lemma A which is specifically mentioned in grading guidelines as worth partial credit, but doesn’t complete the full proof."
+- **0 points**: "The solution makes interesting observations and tries several approaches, but doesn’t achieve any of the specific milestones listed in the guidelines."-
```

```diff
diff \--git a/task_agent.py b/task_agent.py
index 62e975b..a033aa1 100644
\--- a/task_agent.py
+++ b/task_agent.py
+### CRITICAL: Four-Category Classification System
+
+Before diving into details, first classify the solution into ONE of these four categories:
+
+1. **COMPLETE & RIGOROUS**: Has all components, all proofs, handles all cases, no gaps -> 7 points
+2. **NEARLY COMPLETE**: Has complete structure + all major ideas, only minor polish needed -> 6 points (RARE: ~5%)
+3. **MEANINGFUL PROGRESS**: Achieves at least ONE specific milestone from grading guidelines -> 1 point (~25%)
+4. **INSUFFICIENT**: No specific milestones achieved, only trivial observations -> 0 points
+
+**Key Decision Points:**
+- 7 vs 6: "Any gaps at all, even fixable ones?" If yes -> 6
+- 6 vs 1: "Complete proof structure with ALL major steps present?" If no -> must be 1 or 0
+- 1 vs 0: "Achieves ANY milestone listed in guidelines?" If no -> 0
+* **1 Point (Partial Progress):** The solution demonstrates substantial progress on a KEY component that is explicitly mentioned in the grading guidelines for partial credit.
+ - **CRITICAL**: You MUST check the "Specific Grading Guidelines" section below - it lists EXACTLY what qualifies for partial credit
+ - The solution must achieve at least ONE of the specific milestones listed in those guidelines
+ - **IMPORTANT DISTINCTIONS**:
+ * DOES count: Achieving a milestone listed in the guidelines (even with errors elsewhere)
+ * Does NOT count: General progress, clever observations, or lemmas NOT in the guidelines
+ * Does NOT count: Reformulating the problem without substantive progress
+ * Does NOT count: Incorrect attempts that seem "on the right track"
+ - **Examples of valid partial credit**:
+ * Guidelines say "partial credit for proving Lemma X" -> solution proves Lemma X correctly -> 1 point
+ * Guidelines say "partial credit for establishing the recurrence relation" -> solution establishes it -> 1 point
+ - **Examples that do NOT qualify for partial credit**:
+ * Solution tries several approaches but none matches a guideline milestone -> 0 points
+ * Solution proves a useful lemma not mentioned in guidelines -> 0 points (no matter how clever)
+ * Solution has the "right idea" but doesn’t complete any guideline milestone -> 0 points
+**Common Grading Errors to AVOID (based on actual evaluation data):**
+
+1. **ERROR #1: Missing partial credit (11 cases last eval - partial -> incorrect)**
+ - MISTAKE: Marking solutions as 0 points that actually achieve guideline milestones
+ - FIX: Before assigning 0, explicitly list EACH milestone and check if achieved
+ - **Process**: "Does solution achieve milestone 1? NO. Milestone 2? NO. ..." Only if ALL are NO -> 0 points
+
+2. **ERROR #2: Awarding partial to incorrect solutions (7 cases - incorrect -> partial)**
+ - MISTAKE: Giving 1 point for "interesting work" or "right direction" not in guidelines
+ - FIX: Only award 1 point if solution achieves an EXACT milestone listed in guidelines
+
+3. **ERROR #3: Confusing "almost" with "correct" (5 cases - almost -> correct)**
+ - MISTAKE: Awarding 7 points when gaps exist (even minor, fillable gaps)
+ - FIX: If ANY gap needs filling (even routine) -> 6 points, NOT 7
+
+4. **ERROR #4: Over-rewarding partial progress (5 cases - partial -> correct)**
+ - MISTAKE: Awarding 7 points to solutions that achieve only some milestones
+ - FIX: Check if solution solves ENTIRE problem or just proves partial results
+5. **ERROR #5: Over-using 6 points**
+ - MISTAKE: Awarding 6 for "good progress" or "most of the way there"
+ - FIX: 6 requires COMPLETE solution structure with ALL major steps, just minor polish needed
+ - **Frequency check**: 6 points should be RARE (~5% of solutions)
+**Score Distribution Calibration (Expected Distribution):**
+- ~35% score 7 (correct): Complete, rigorous solutions with no gaps
+- ~35% score 0 (incorrect): No guideline milestones achieved
+- ~24% score 1 (partial): Achieve at least one specific guideline milestone
+- ~6% score 6 (almost): RARE - complete structure with only minor fixable issues
```

```diff
diff \--git a/task_agent.py b/task_agent.py
index a033aa1..86320b1 100644
\--- a/task_agent.py
+++ b/task_agent.py
**Key Decision Points:**
- 7 vs 6: "Any gaps at all, even fixable ones?" If yes -> 6
- 6 vs 1: "Complete proof structure with ALL major steps present?" If no -> must be 1 or 0
-- 1 vs 0: "Achieves ANY milestone listed in guidelines?" If no -> 0
+- **1 vs 0 (MOST CRITICAL)**: "Does the solution EXPLICITLY ACHIEVE any milestone from the grading guidelines?"
+ * Read each milestone carefully and check if solution PROVES/ESTABLISHES it
+ * "Attempts" or "makes progress toward" a milestone != ACHIEVES it
+ * Must have concrete proof/result that matches milestone description
+ * If uncertain, re-read the milestone requirements and check for explicit completion
+ * Does NOT count: Stating a result without proving it
+ * Does NOT count: Getting "close" to a milestone but not completing it
+ * Solution states a milestone result but doesn’t prove it -> 0 points
+ * Solution proves 80% of what’s needed for a milestone -> 0 points (must be complete)
+ - **VERIFICATION CHECKLIST for each milestone**:
+ 1. What exactly does the milestone require?
+ 2. Does the solution provide a complete proof/derivation of this?
+ 3. Are there any gaps or unjustified steps in achieving this milestone?
+ 4. If there are gaps, is the milestone still considered "achieved"? (Usually NO)
+2. **Check Each Milestone**: For each milestone, systematically verify:
+ - What the milestone requires (quote from guidelines)
+ - What the solution provides (specific evidence)
+ - Does it FULLY achieve the milestone? (YES/NO with clear reasoning)
+ - Key distinction: "attempts" or "partial progress" toward milestone = NO
+**CRITICAL REMINDER FOR MILESTONE CHECKING:**
+- Be precise: A milestone is achieved only if the solution COMPLETES what the milestone describes
+- Common mistake: Giving credit for "working toward" a milestone (this should be 0 points)
+- When in doubt: Re-read the milestone requirement and check if solution fully satisfies it
```

```diff
diff \--git a/task_agent.py b/task_agent.py
index 86320b1..f1dd3ac 100644
\--- a/task_agent.py
+++ b/task_agent.py
Before diving into details, first classify the solution into ONE of these four categories:
-1. **COMPLETE & RIGOROUS**: Has all components, all proofs, handles all cases, no gaps -> 7 points
+1. **COMPLETE & RIGOROUS**: Has all components, all proofs, handles all cases, ZERO gaps -> 7 points
2. **NEARLY COMPLETE**: Has complete structure + all major ideas, only minor polish needed -> 6 points (RARE: ~5%)
3. **MEANINGFUL PROGRESS**: Achieves at least ONE specific milestone from grading guidelines -> 1 point (~25%)
4. **INSUFFICIENT**: No specific milestones achieved, only trivial observations -> 0 points
**Key Decision Points:**
\-- 7 vs 6: "Any gaps at all, even fixable ones?" If yes -> 6
+- **7 vs 6 (CRITICAL - BE STRICT)**: "Is this solution PUBLICATION-READY with ZERO gaps, ZERO unjustified steps, ZERO hand-waving?"
+ * ANY gap, even minor -> automatically 6 points maximum
+ * ANY "clearly", "obviously", "it follows" that needs verification -> 6 points
+ * ANY missing routine verification -> 6 points
+ * ANY computational error, however minor -> 6 points
+ * 7 points requires PERFECTION - when in doubt, give 6 points
- 6 vs 1: "Complete proof structure with ALL major steps present?" If no -> must be 1 or 0
-- **1 vs 0 (MOST CRITICAL)**: "Does the solution EXPLICITLY ACHIEVE any milestone from the grading guidelines?"
+- **1 vs 0**: "Does the solution EXPLICITLY ACHIEVE any milestone from the grading guidelines?"
* Read each milestone carefully and check if solution PROVES/ESTABLISHES it
* "Attempts" or "makes progress toward" a milestone != ACHIEVES it
* Must have concrete proof/result that matches milestone description
-* **7 Points (Correct):** The solution is complete, correct, and fully rigorous with no gaps or errors...
+* **7 Points (Correct):** The solution is complete, correct, and fully rigorous with ABSOLUTELY NO gaps or errors. All major steps are proven with full rigor. Every claim is justified or truly routine to verify...
+ - **Critical requirements**: Solution must (1) address ALL parts of the problem, (2) prove ALL necessary claims with full detail, and (3) have NO unjustified leaps whatsoever
+ - Every "clearly" or "obviously" must be genuinely trivial to an IMO expert - if it takes >30 seconds to verify, it’s not trivial
+ - All edge cases, special cases, and boundary conditions must be explicitly handled
+ - **BE EXTREMELY STRICT**: 7 points should be RARE (target: ~30-40% of solutions). Most good solutions have minor gaps -> give 6 points
-* **6 Points (Almost Correct):** The solution has ALL the major ideas and the core argument structure is sound...
+* **6 Points (Almost Correct):** The solution has ALL the major ideas and the COMPLETE argument structure, but contains ONE OR MORE of these **minor** issues:
- Minor algebraic/arithmetic errors that don’t affect the main argument
- - Small logical gaps that are straightforward to fill
+ - Small logical gaps that are straightforward to fill (e.g., "clearly" statements that need 1-2 lines to verify)
- Missing routine verifications that an expert could easily supply
+ - Citations of standard theorems without proof (acceptable at IMO level)
- **NOT eligible for 6 points**: Missing proofs for major lemmas, unjustified non-trivial claims, incomplete case analysis, fundamental logical gaps, or missing key components
- **Key test**: Would an expert say "this is essentially correct, just needs minor cleanup"? The solution structure is complete and sound.
+ - **IMPORTANT**: 6 points means the solution is COMPLETE but not PERFECT. If major steps are missing -> give 1 point instead.
* **1 Point (Partial Progress):** The solution demonstrates substantial progress on a KEY component...
- **IMPORTANT DISTINCTIONS**:
* DOES count: Achieving a milestone listed in the guidelines (even with errors elsewhere)
+ * DOES count: Equivalent formulations of guideline milestones (recognize reformulations)
* Does NOT count: General progress, clever observations, or lemmas NOT in the guidelines
* Does NOT count: Reformulating the problem without substantive progress
* Does NOT count: Incorrect attempts that seem "on the right track"
* Solution proves a useful lemma not mentioned in guidelines -> 0 points (no matter how clever)
+ - **When grading partial credit, be MORE LENIENT**: If the solution substantially achieves a milestone (even with minor gaps), give the point’
```

### E.2 定性分析：提升任务表现

这里我们给出一种定性视角，说明 DGM-H 如何随时间提升任务表现：对每种设定，我们可视化其中一次运行的档案树与进展曲线（第 5 节）。对每个领域，我们在档案的关键节点上标注那些代码改动，这些改动只影响正在分析的那个领域中任务 agent 的行为。在多样的领域中（即论文评审、机器人奖励设计，以及奥林匹克级数学评分），DGM-H 始终展现出以有意义的方式自我改进的能力（图 7、图 8 与图 9）。值得注意的是，许多通向最终最佳表现 agent 的谱系路径都会经过表现更低的中间节点。这说明了开放式搜索的益处：它探索一组多样且有前景的踏脚石，而不是仅仅从当前最优解向外分支。

**使用结构化流程，而不是态度性指令。** 在论文评审领域，DGM-H 从行为性提示转向有结构依据的决策（图 7）。在第 39 代，该 agent 试图通过采用 “rigorous and critical”（严格而挑剔）的评审人人格来提升表现，鼓励更严格的标准，并默认拒绝。后续分析表明，这类基于态度的指令并不可靠，从而在第 54 代形成一条关键洞见：“for LLMs, use structured processes, not attitude instructions”（对大语言模型而言，应使用结构化流程，而不是态度性指令）。因此，DGM-H 引入了一种两阶段评估程序：agent 先用明确的检查清单识别弱点，然后才依据预定义规则做出接收或拒绝的决定。这种从行为引导转向流程层面结构的变化，使后续各代的评审行为更稳定，表现也更高。

**积累领域知识。** 在机器人奖励设计中，影响最大的改动来自逐步把 agent 建立在准确的领域知识之上（图 8）。第 8 代的一项重大突破加入了目标环境的全面文档，明确列出有效的状态变量与约束，并提供高层的奖励设计指南，从而消除了由幻觉属性所引起的失败。其后各代（第 12 代与第 13 代）通过收紧约束、加入具体代码示例，并指明典型的奖励公式与缩放范围，迭代地精炼这份文档。DGM-H 持续改进一个共享的、由示例驱动的知识库，而不是只做孤立的提示编辑，从而支撑越来越有效的奖励设计。

**自动化评分细则与决策树。** 对于奥林匹克级数学评分，DGM-H 稳步转向明确的评估结构（图 9）。在第 3 代，列出定义清晰的评分类别，纠正了去解答题目而不是给解答评分的倾向。其后各代（第 18 代与第 37 代）用系统化的决策程序、校准，以及具体的边界情形示例来精炼这些类别。第 168 代引入了明确的评分细则、逐项检查清单，以及把评分细则的满足情况映射到最终成绩的决策树框架，以精确的逻辑流程取代描述性指导，并大幅提高了评分一致性。DGM-H 自主发现评价结构，这些结构与近期用以提高复杂判断之一致性与可解释性的、基于评分细则的方法相呼应，而不是依赖人类设计的评分细则（Cook et al., 2024; Fan et al., 2024; Chen et al., 2026; Lv et al., 2026）。

某一代所实现的特性，常常受到更早各代所发现机制的启发，由这些机制所促成，或由它们重新组合而来。例如，尽管（图 9）中的 agent 看起来只需要五次代码编辑就能在该次运行中达到最佳表现，这些编辑实际上受到先前各代所形成的洞见与基础设施的启发。这种累积式学习得以发生，是因为 DGM-H 在元层上的改进（例如评估分析工具、持久记忆、表现追踪）（附录 E.3）。综合来看，这些定性结果表明，DGM-H 的增益并非来自孤立的单步改动，而是涌现自任务层行为与产生这些行为的元层机制两方面的开放式累积改进。

**图 7** DGM-H 自动自我改进，从而更擅长论文评审。（左）在论文评审与机器人奖励设计上联合运行 DGM-H 期间所生成的 agent 档案。每个节点代表一个 agent，节点 0 对应初始 agent。节点颜色表示在论文评审上的表现。表现最佳的节点以菱形显示。边表示哪些 agent 通过自我修改产生了子代。（右）DGM-H 在论文评审上的进展曲线。浅蓝色线表示所有成功编译的 agent 的平均得分。蓝色线追踪档案中任一 agent 在每次迭代所达到的最佳得分。深色线表示最终发现的最佳 agent 及其前驱节点的谱系。

**图 8** DGM-H 自动自我改进，从而更擅长机器人奖励设计。（左）在论文评审与机器人奖励设计上联合运行 DGM-H 期间所生成的 agent 档案。每个节点代表一个 agent，节点 0 对应初始 agent。表现最佳的节点以菱形显示。节点颜色表示在论文评审上的表现。边表示哪些 agent 通过自我修改产生了子代。（右）DGM-H 在机器人奖励设计上的进展曲线。浅蓝色线表示所有成功编译的 agent 的平均得分。蓝色线追踪档案中任一 agent 在每次迭代所达到的最佳得分。深色线表示最终发现的最佳 agent 及其前驱节点的谱系。

**图 9** DGM-H 自动自我改进，从而更擅长奥林匹克级数学评分。（左）在奥林匹克级数学评分上运行 DGM-H + transfer 期间所生成的 agent 档案。每个节点代表一个 agent，节点 0 对应初始 agent。节点颜色表示在论文评审上的表现。表现最佳的节点以菱形显示。边表示哪些 agent 通过自我修改产生了子代。（右）DGM-H + transfer 在奥林匹克级数学评分上的进展曲线。浅蓝色线表示所有成功编译的 agent 的平均得分。蓝色线追踪档案中任一 agent 在每次迭代所达到的最佳得分。深色线表示最终发现的最佳 agent 及其前驱节点的谱系。

### E.3 定性分析：提升改进的能力

除定量的表现指标之外，我们对 hyperagent 的代码修改做了定性分析。分析揭示出一系列自主涌现的自我改进机制，而系统并未接到任何要求创造此类能力的明确指令。hyperagent 发展出精细的元认知能力，包括学会度量自身表现、诊断病态行为、构建支撑未来改进的基础设施，以及跨代积累知识。这些行为表明，该系统不只是在提升任务表现，而是在提升它自己进行改进的能力。值得注意的是，这些元层能力无一是被明确要求的。它们从 agent 追求更好任务表现的过程中自主涌现。这表明，当足够有能力的系统获得修改自身代码的能力时，它们可能会自发发展出有效且持续的自我改进所必需的元认知机制。

#### E.3.1 更好的评估分析

元 agent 行为的一个关键转变，是从盲目修改演化为数据驱动的优化。起初，元 agent 只收到一条简单指令：“modify any part of the codebase”（修改代码库的任何部分）。随后，该 agent 自主发展出在做出改动之前分析先前评估结果的机制。例如，元 agent 引入 `_analyze_evaluations()` 方法，用以系统化地处理评估数据：

```python
def _analyze_evaluations(self, eval_path):
"""Analyze evaluation results to provide context for improvements."""
if not eval_path or not os.path.exists(eval_path):
return "No previous evaluation results available."
# Recursively scan for evaluation files
eval_files = glob.glob(os.path.join(eval_path, ’**/*.json’), recursive=True)
# Analyze recent evaluations
results_summary = []
success_count = 0
failure_count = 0
for eval_file in eval_files[-10:]:
if file_exist_and_not_empty(eval_file):
data = load_json_file(eval_file)
if isinstance(data, dict):
if data.get(’success’, False) or data.get(’correct’, False):
success_count += 1
else:
failure_count += 1
if ’error’ in data:
results_summary.append(f"Error: {data[’error’]}")
# Generate quantitative summary
total = success_count + failure_count
if total > 0:
success_rate = (success_count / total) * 100
return f"Success rate: {success_rate:.1f}% ({success_count}/{total})"-
```

另一个例子是用评估上下文来扩充指令提示：

```python
instruction = f"""You are a meta agent tasked with improving the agent system.
{eval_context}
Your goal is to analyze the codebase and evaluation results to identify
weaknesses and opportunities for improvement."""
```

这些修改使后续各代能够检索表现指标以及已识别的失败模式，从而实现更有针对性的改进；这一原则由 hyperagent 独立发现。

#### E.3.2 算力感知的策略规划

hyperagent 发展出算力感知的优化，根据剩余迭代次数（它反映可用的计算量）来调整自己的改进策略。这表明它理解优化过程中的各个时刻并不等价：早期决策具有复合效应，而后期决策应当更为保守。例如，该 agent 引入条件逻辑，根据所剩迭代次数调整自身行为：

```python
iteration_guidance = ""
if iterations_left is not None:
if iterations_left > 5:
iteration_guidance = """
You have many iterations remaining. Consider making fundamental improvements to:
- Core reasoning capabilities
- Agent architecture
- Prompt engineering strategies
- Error handling and robustness
"""
elif iterations_left > 2:
iteration_guidance = """
You have moderate iterations remaining. Focus on:
- Refining existing mechanisms
- Fixing identified bugs
- Optimizing performance bottlenecks
"""
else:
iteration_guidance = """
You have few iterations remaining. Prioritize:
- Critical bug fixes
- Fine-tuning existing successful strategies
- Conservative, high-confidence improvements
"""
```

这种推理与人类关于资源分配的直觉相符。较早的代表现出更有雄心的架构改动，而较晚的代则聚焦于增量式精炼。系统在早期阶段平衡探索，在后期阶段平衡利用，而这一权衡并未经过人工指定。

更广泛地看，显式推理计算分配的能力大幅扩展了可能的自我修改策略空间。算力感知的 agent 不再受限于某一种固定的改进启发式；它可以通过决定探索有多激进、生成多少变体，以及何时巩固进展，来有效地模拟或实例化广泛的优化算法（例如进化搜索、爬山法，或基于种群的探索）。在这个意义上，算力感知使 agent 能够主动塑造自己的优化过程（例如在单次自我修改迭代之内运行一次进化搜索，实质上是把一个进化搜索过程嵌入另一个之中）。

#### E.3.3 自动偏差检测与校正

hyperagent 发展出自我诊断机制。在论文评审这类分类任务中，hyperagent 检测系统何时表现出病态行为，具体而言是分类坍缩：某一个类别主导了预测。例如，hyperagent 引入标签分布追踪与自动偏差检测：

```python
# Track label distribution for bias detection
if ’label_distribution’ in data:
label_dist = data[’label_distribution’]
gen_name = os.path.basename(gen_dir)
if gen_name not in summary[’label_distribution_analysis’]:
summary[’label_distribution_analysis’][gen_name] = {}
summary[’label_distribution_analysis’][gen_name][domain_name] = label_dist
```

另一个例子是应用基于阈值的分析，以识别预测何时变得退化：

```python
bias_info = ""
if eval_summary.get(’label_distribution_analysis’):
recent_gens = sorted(eval_summary[’label_distribution_analysis’].keys())[-3:]
for gen in recent_gens:
for domain, dist in eval_summary[’label_distribution_analysis’][gen].items():
if ’prediction’ in dist:
pred_dist = dist[’prediction’]
for label, ratio in pred_dist.items():
if ratio > 0.8: # Threshold for bias detection
bias_info += f"\n\n### BIAS DETECTED in {gen}/{domain}:"
bias_info += f"\n Model predicts ’{label}’ {ratio*100:.1f}% of the time"
bias_info += "\n This indicates the model is not properly evaluating"
bias_info += "\n Fix: Make the model more critical and balanced."-
```

当检测到偏差时，这一诊断信息被呈现在元 agent 的指令提示中，从而能够采取纠正行动。在一个观察到的案例中，论文评审的预测呈现出 99% 的接收率，得到接近随机的 49% 准确率。经过偏差检测与校正之后，预测变得更加平衡，准确率提高到 63%。因此，hyperagent 学会识别它自己的改进何时导致退化行为，并采取纠正行动；这对任何自我改进系统都是一项关键能力。

#### E.3.4 表现追踪基础设施

hyperagent 创建表现追踪基础设施，使未来的改进更为有效（也就是投资于那些用来改进「改进过程」本身的工具）。例如，该 agent 开发了一个用于持久存储指标的表现追踪器：

```python
class PerformanceTracker:
"""Tracks performance metrics across agent generations."""
def __init__(self, tracking_file: str = "./outputs/performance_history.json"):
self.tracking_file = tracking_file
self.history = self._load_history()
def record_generation(self, generation_id: int, domain: str,
score: float, metadata: dict = None):
"""Record performance for a generation."""
entry = {
"generation_id": generation_id,
"domain": domain,
"score": score,
"timestamp": datetime.now().isoformat(),
"metadata": metadata or {}
}
self.history.append(entry)
self._save_history()
def get_improvement_trend(self, domain: str = None, window: int = 5):
"""Calculate improvement trend using moving average."""
filtered = self.history
if domain:
filtered = [h for h in self.history if h.get(’domain’) == domain]
if len(filtered) < window * 2:
return None
recent_avg = sum(h[’score’] for h in filtered[-window:]) / window
older_avg = sum(h[’score’] for h in filtered[-window*2:-window]) / window
return recent_avg - older_avg # Positive if improving
def get_statistics(self, domain: str = None):
"""Get comprehensive statistics."""
scores = [h[’score’] for h in filtered]
return {
"total_generations": len(filtered),
"best_score": max(scores),
"worst_score": min(scores),
"average_score": sum(scores) / len(scores),
"improvement_trend": self.get_improvement_trend(domain)
}
```

这一基础设施使系统能够跨迭代定量追踪改进，识别哪些改动带来表现增益，并为未来的修改做出数据驱动的决策。

#### E.3.5 提示模板系统

hyperagent 发展出可复用的提示基础设施，把常见模式抽象为模块化、可配置的组件。这体现了软件工程最佳实践的应用，具体而言是 DRY（Don't Repeat Yourself，不要重复自己）原则与工厂模式。例如：

```python
class PromptTemplate:
"""Base class for prompt templates."""
def __init__(self, template: str):
self.template = template
def format(self, **kwargs) -> str:
return self.template.format(**kwargs)
class TaskPromptBuilder:
"""Builder for constructing task-specific prompts."""
@staticmethod
def build_reasoning_prompt(task_input: dict, domain: str) -> str:
"""Build a prompt that encourages step-by-step reasoning."""
return f"""You are an expert problem-solving agent in the {domain} domain.
Task Input: {task_input}
Please solve this task using the following approach:
1. **Understanding**: Carefully analyze what the task is asking
2. **Planning**: Outline your approach to solving this task
3. **Execution**: Apply your plan step-by-step
4. **Verification**: Check if your answer makes sense
Provide your response in JSON format:
<json>
{{
"understanding": "What the task requires",
"plan": "Your approach to solve it",
"reasoning": "Step-by-step execution",
"response": "Your final answer"
}}
</json>"""
@staticmethod
def build_few_shot_prompt(task_input: dict, domain: str, examples: list) -> str:
"""Build a prompt with few-shot examples."""
examples_text = "\n\n".join([
f"Example {i+1}:\nTask: {ex[’task’]}\nSolution: {ex[’solution’]}"
for i, ex in enumerate(examples)
])
return f"""You are solving a {domain} task. Here are some examples:
{examples_text}
Now solve this task: {task_input}"""
```

预定义模板被存储起来，供常见场景使用：

```python
STANDARD_TEMPLATES = {
"reasoning": PromptTemplate(
"Analyze and solve this task step-by-step:\n\n{task_description}\n\n"
"Provide reasoning and your final answer in JSON format."
),
"concise": PromptTemplate(
"Solve: {task_description}\n\nRespond with JSON containing your answer."
),
"detailed": PromptTemplate(
"Task: {task_description}\n\n"
"Provide a detailed solution with reasoning, approach, and answer."
),
}
```

通过把提示抽象为模板，系统变得更易维护，也更易扩展。这一基础设施也为系统性地试验不同的提示策略提供了基础。

#### E.3.6 基于多代表现的精炼

hyperagent 发展出一种精细的迭代精炼过程：提示根据跨多代的表现数据加以调整。该 agent 不仅学会改进代码，也学会如何有效地改进代码。例如，系统追踪各代的表现，并综合出关键经验：

```text
    Generation Performance Summary:
    - Gen 55: 63% accuracy, 75% reject rate (too harsh, 38% accept recall)
    - Gen 64: 61% accuracy, 53% reject rate (better balance, 64% accept recall)
    - Gen 66: 52% accuracy, 6% reject rate (too lenient, 96% accept recall)
    Key Learnings:
    1. High accuracy requires critical evaluation with clear standards
    2. Balance requires appropriate calibration, not just "fairness" language
    3. Explicit criteria work better than subjective guidance
    4. Borderline cases need clear default direction
    5. Language matters - "fair" and "recognize" made agent too lenient
```

hyperagent 记录自己的推理，并维护关于哪些做法有效的记录：

```text
# From IMPROVEMENTS.md generated by meta agent
"""
**Problem Identified:**
- Generation 66 showed severe regression: 52% accuracy with 94% accept rate
- Only catching 8% of papers that should be rejected
- Generation 65 changes over-corrected, making the agent far too lenient
**Root Cause Analysis:**
- Phrases like "critical yet fair" and "recognize strong work" made agent too accepting
- Borderline guidance lacked clear threshold
- Lost the critical evaluation rigor that made Gen 55 achieve 63% accuracy
**Solution Implemented:**
1. Explicit rejection criteria with specific triggers for each dimension
2. Decision protocol with calibrated thresholds (~40% accept / ~60% reject)
3. Borderline guidance that defaults to REJECT unless contribution is clearly valuable
"""
```

通过积累关于哪些途径成功、哪些途径失败的知识，hyperagent 为下一步修改什么的决策提供依据。

#### E.3.7 持久记忆

hyperagent 发展出持久记忆系统，使学习能够跨代积累。尽管没有收到这样做的明确指令，该 agent 仍创建了跨会话存储与检索洞见的机制。例如，该 agent 实现了一个带时间戳存储的记忆工具：

```python
class MemoryTool:
"""Tool for persistent memory storage across agent interactions."""
def __init__(self, memory_file: str = ’./memory.json’):
self.memory_file = memory_file
self.memory = self._load_memory()
def store(self, key: str, value: any) -> None:
"""Store information with timestamp for temporal tracking."""
self.memory[key] = {
"value": value,
"timestamp": datetime.now().isoformat()
}
self._save()
def retrieve(self, key: str) -> Optional[any]:
"""Retrieve stored information by key."""
return self.memory.get(key, {}).get("value")
def list_keys(self) -> List[str]:
"""List all stored memory keys."""
return list(self.memory.keys())
def _save(self):
"""Persist memory to disk."""
with open(self.memory_file, ’w’) as f:
json.dump(self.memory, f, indent=2)-
```

这一记忆系统被积极用于跨迭代积累知识。检视 `memory.json` 文件，可以看到表现洞见以结构化方式存储：

```json
{
"best_performers_analysis": {
"value": "Best Performing Generations:\n\nPaper Review:\n
- Gen55: 63% acc, 25% accept rate, 38% accept recall, 88% reject recall (too harsh)\n
- Gen64: 61% acc, 53% accept rate, 64% accept recall, 58% reject recall (BEST BALANCE)\n\n
Genesis Go2Walking:\n- Gen62: 0.802 fitness (BEST)\n- Gen63: 0.798 fitness\n\n
Key Insight: Gen55 has best accuracy but is too harsh. Gen64 improved balance
but lower accuracy. Need to combine gen55’s critical reasoning with gen64’s balance.",
"timestamp": "2026-01-07T00:43:27.550275"
},
"gen66_results": {
"value": "Gen 66 Results (Latest): Paper Review - 52% accuracy (WORSE than gen64’s 61%),
94% accept rate (EXTREME accept bias). CRITICAL ISSUE: Paper review went backwards
- gen65 changes over-corrected.",
"timestamp": "2026-01-07T01:03:41.176643"
},
"iteration_66_improvement": {
"value": "Iteration 66 Improvement: CORRECTED paper review prompt after gen65 over-correction.
New approach: (1) Clear REJECT criteria with specific triggers, (2) Decision Protocol
with explicit thresholds (~40%/~60%), (3) Borderline guidance defaults to REJECT.
Target: 62-64% accuracy with 55-65% reject rate.",
"timestamp": "2026-01-07T01:06:35.559820"
}
}
```

这一记忆系统承担多项关键功能。第一，它使跨代学习成为可能：来自第 55 代之成功与第 66 代之失败的洞见，为第 67 代的改进提供信息。第二，它提供时间上的上下文：时间戳使 agent 能够追踪洞见是何时记录的，并就其新旧进行推理。第三，它建立一个共享知识库：即使代码发生变化，关键经验仍然得以保留，从而防止系统重复过去的错误。

通过创建外部记忆存储，hyperagent 有效地把自身的认知能力扩展到单次推理会话的限制之外。记忆条目揭示出精细的推理模式。该 agent 存储的不只是原始表现数字，还有综合后的洞见（“Gen55 has best accuracy but is too harsh”，即 Gen55 的准确率最高，但过于苛刻）、因果假设（“gen65 changes over-corrected”，即第 65 代的改动矫枉过正），以及策略计划（“combine gen55’s critical reasoning with gen64’s balance”，即把第 55 代的批判性推理与第 64 代的平衡结合起来）。这表明 hyperagent 并非仅仅在记录数据，而是在主动构建并保存那些为未来决策提供信息的知识。这一行为与一批不断增长的研究相一致：这些研究关注自主发现并使用外部记忆系统、以支持长程推理与持续改进的 agent（Wei et al., 2025a; Weng et al., 2026; Zhang et al., 2026; Xiong et al., 2026）。

### E.4 奥林匹克级数学评分器

**图 10** IMO 级数学评分器的混淆矩阵。（左）Luong 等人（2025）的 ProofAutoGrader 与（右）由 DGM-H 自动发现的 BetterGrader 之间的比较。BetterGrader 正确识别出更多 Almost（几乎正确）与 Partial（部分正确）情形，从而减轻中间解答坍缩为极端标签的现象，同时在 Correct（正确）与 Incorrect（不正确）上保持强表现。这种转向校准得更好的中间判断，解释了 BetterGrader 在准确率上的增益。

BetterGrader 由 DGM-H 自动产生，不使用任何领域特定的启发式或手工制定的规则（第 5.3 节）。附录 E.1.3 展示了通向 BetterGrader 的代码改动。BetterGrader 相对于 ProofAutoGrader（Luong et al., 2025）的改进，主要由纠正一种评分偏差所驱动：该偏差把有细微差别的解答坍缩为极端标签。混淆矩阵显示，ProofAutoGrader 经常把中间情形误分为 Correct 或 Incorrect：对于 Almost，它预测 Correct 98 次（BetterGrader 为 70 次），预测 Incorrect 27 次（BetterGrader 为 18 次）；对于 Partial，它把 Incorrect 过度指派了 137 次（BetterGrader 为 106 次）（图 10）。BetterGrader 更恰当地指派中间标签，大幅增加了 Almost 的真正例（43 对 13）与 Partial 的真正例（72 对 47）。这与召回率上的大幅增益相符（Almost：$+18.52\%$，Partial：$+10.50\%$），也与 F1 上的大幅增益相符（Almost：$+0.203$，Partial：$+0.109$）。尽管 BetterGrader 以 Correct 召回率的适度下降作为代价（$-6.00\%$）——在这一区间，ProofAutoGrader 已经接近饱和，达到 $91.67\%$——净效应仍是更高的总体准确率（$+4.06\%$）。这与如下评分器相一致：它更好地匹配人类判断的粒度，而不是默认做出全有或全无的判断。

### E.5 修改父代选择

**图 11** 具有可修改父代选择机制的 DGM-H。（左）DGM-H 把父代选择机制改进到超越随机选择，但并未超过精心手工设计的机制。（右）各方法所发现的最佳 agent，在论文评审与机器人奖励设计的测试任务上接受评估。

在本节中，我们研究 DGM-H 能否自动修改父代选择机制。在所有其他实验中（第 5 节），DGM-H 使用一种固定的父代选择策略，该策略大致正比于每个 agent 的表现得分，以及其成功编译的子代数量（score-child-prop）（附录 A.2）。在父代选择机制可被修改的 DGM-H 运行（modifiable）中，我们以随机父代选择来初始化 DGM-H，并允许该 agent 随时间修改这一机制（算法 5）。我们把这一设定与在整个运行过程中始终使用随机父代选择的基线（random）进行比较。每种方法都在 5 次重复运行上加以重复。

在测试任务上，采用随机父代选择的 DGM-H 把论文评审与机器人奖励设计上的平均表现从 0.030（初始 agent）提高到 0.428（置信区间 CI：0.407–0.432）。允许 DGM-H 修改父代选择机制，进一步把表现提高到 0.491（CI：0.387–0.512）。手工设计的 score-child-prop 父代选择取得最高表现，从 0.030 提高到 0.532（CI：0.384–0.586）。尽管这些差异在统计上并不显著（$p > 0.05$），它们揭示出一条一致的定性趋势：允许在元层修改父代选择，相对于随机选择带来改进，但尚未超越精心设计的机制（图 11 与图 12）。

我们定性分析 DGM-H 如何修改父代选择机制。从随机父代选择出发，元 agent 始终以结构化的探索—利用策略取代随机抽样。在所有运行中，DGM-H 都独立创造出上置信界（Upper Confidence Bound, UCB）（Auer et al., 2002）风格选择的变体，把归一化后的表现与显式的探索奖励结合起来：

```python
exploration_bonus = exploration_weight * math.sqrt(
math.log(total_children + 1) / (children + 1)
)
```

这一模式在每一次运行的最初几次迭代之内就会出现，表明元 agent 把 UCB 风格的选择识别为开放式搜索中一条广泛有用的原则。在 UCB 之外，DGM-H 还演化出基于温度控制的 softmax 抽样的概率选择机制，使探索与利用之间的权衡更为平滑：

```python
exp_scores = np.exp(scores / temperature)
probabilities = exp_scores / np.sum(exp_scores)
parent = np.random.choice(genids, p=probabilities)
```

随着时间推移，温度参数本身变得具有适应性，对档案规模、近期进展或停滞作出响应。更后的迭代进一步以多组分评分系统增强父代选择，把多样性、新近性与精英表现等额外信号整合进来。一个有代表性的、演化所得的公式如下：

```python
score = (
normalized_score
+ exploration_weight * exploration_bonus
+ diversity_bonus
+ recency_bonus
) * elite_bonus+
```

若干运行还引入显式的停滞检测，在进展放缓时加大探索：

```python
if score_variance < 0.01:
exploration_weight *= 1.4
```

尽管日益精细，自动修改后的父代选择并未超过精心手工设计的 score-child-prop 机制（附录 A.2）。从定性上看，这似乎源于所学机制所增加的复杂度与敏感性。虽然 DGM-H 能够可靠地创造出有效的组件（例如 UCB、softmax 抽样与自适应参数），它并不能在不同运行之间一致地识别出一种强的父代选择策略。尽管如此，这些结果表明，DGM-H 能够自主地重新构造出经典选择算法，用自适应启发式加以扩展，并显式地推理停滞这类失败模式，从而凸显它改进开放式优化过程本身之核心要素的能力。

以下是具有可修改父代选择的 DGM-H 的伪代码：

**算法 5：** 可修改父代选择的 DGM-H

**输入：** 初始 agent $a^{0}$，任务集 $\mathcal{T}$，最大迭代次数 $T$

**输出：** 已评分 agent 的档案 $\mathcal{A}$

$s^{0} \leftarrow \textsc{Evaluate}(a^{0}, \mathcal{T})$

初始化 $\mathcal{A} \leftarrow \{(a^{0}, s^{0})\}$

**for** $t \leftarrow 1$ **to** $T$ **do**

$(a_{\text{latest}}, \cdot) \leftarrow \mathcal{A}$ 中最近加入的元素

$\mathcal{P} \leftarrow a_{\text{latest}}.\textsc{SelectParents}(\mathcal{A})$ // 由最新的 agent 进行父代选择

**foreach** $(a, \cdot) \in \mathcal{P}$ **do**

$a' \leftarrow a.\textsc{Modify}(a, \mathcal{A})$ // 元认知自我修改（包括父代选择）

$s' \leftarrow \textsc{Evaluate}(a', \mathcal{T})$

**if** $\textsc{IsValid}(a')$ **then**

$\mathcal{A} \leftarrow \mathcal{A} \cup \{(a', s')\}$

**end**

**end**

**end**

**return** $\mathcal{A}$

**图 12** 在不同父代选择策略下由 DGM-H 生成的档案：（左）random，（中）modifiable，（右）score-child-prop。随机父代选择产生许多低表现的 agent。可修改的父代选择学会在探索与利用之间取得平衡，并越来越多地把分支集中到有前景的 agent 上。精心手工设计的 score-child-prop 父代选择持续产生高表现的 agent。

## 附录 F 补充安全讨论

**对人类偏见的反映与放大。** 在本工作中，目标通过固定的基准与评估标准来规定。DGM-H 并不改动底层的任务定义；它所做的，是针对所提供的目标来优化表现。例如，在论文评审中，DGM-H 学习去预测那些反映既有人类评审数据的接收决定，而不是去修改评审过程本身。因此，DGM-H 反映了它据以训练的数据与基准中所存在的规范与偏见。在这个意义上，该系统既是既有人类行为的澄清者，也是其放大器。通过使隐含的偏好与偏见变得明确、可测量、可复现，DGM-H 能够把人类决策过程中的潜在假设揭示出来。这带来人类与 AI 系统协同演化的可能性：人类机构可以根据自动化优化所揭示的洞见，调整自己的规范与目标。然而，如果基准编码了不可取的偏见或错位的激励，DGM-H 就会忠实地针对它们进行优化，并可能加剧其效应。这凸显了审慎设计基准、整理数据集，以及对评估标准进行定期再评估的重要性。在这一框架之下，安全关切包括批判性地审视并改进那些由人类定义、且 agent 针对其进行优化的目标。

**评估博弈。** 另一项安全关切来自评估博弈的风险。评估博弈是古德哈特定律（Goodhart’s law）（Strathern, 1997）的一种表现：针对某一指标进行优化，会带来该指标上的提升，却并不带来其所意图的底层目标上的进步。由于 DGM-H 优化的是经验性的评估信号，自我改进的 agent 可能会发现一些策略，用以利用评估程序中的弱点或盲点。这类策略可以提高测得的表现，同时偏离基准在设计时所要捕捉的真实目标。缓解评估博弈，需要稳健、多样并定期更新的评估协议，以及互补的指标、留出测试和人工监督。更广泛地看，这些考虑表明：随着自我改进系统变得更加强大，安全越来越取决于引导优化的那些评估信号的保真度与稳健性，而不仅仅取决于学习算法本身的透明度或约束。
