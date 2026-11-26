---
title: "Exo：一种用于运行时自我改进的 harness 架构"
author: "Alexander Krentsel, Sylvia Ratnasamy, Ion Stoica, Martin Casado, and Ankur Goyal"
source_url: "https://arxiv.org/abs/2611.00463"
arxiv_id: "2611.00463"
arxiv_version: "v1"
published_at: "2026-10-31"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-translation"
unofficial_translation: true
title_paper_body: "Exo: A Harness Architecture for Runtime Self-Improvement"
title_arxiv_abs: "Exo: An Architecture for Self-Improving Systems at Runtime"
translation_scope: "摘要至结论的全部正文，含表格、公式、脚注与附录。代码保留英文。参考文献保留英文。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名与 CC BY 4.0。"
---

译文说明：本文件为非官方中文翻译，未经原作者审定。arXiv 摘要页标题为“Exo: An Architecture for Self-Improving Systems at Runtime”（arXiv:2611.00463），正文标题为“Exo: A Harness Architecture for Runtime Self-Improvement”。原文以 CC BY 4.0 许可发布（https://creativecommons.org/licenses/by/4.0/）。相对原文的改动仅限于语言与 Markdown 形式，科学内容未作更改。

Exo：一种用于运行时自我改进的 harness 架构

Alexander Krentsel 1 *　Sylvia Ratnasamy 1　Ion Stoica 1　Martin Casado 2　Ankur Goyal 3

1 加州大学伯克利分校　　2 Andreessen Horowitz　　3 Braintrust

通讯作者：[akrentsel@berkeley.edu](mailto:akrentsel@berkeley.edu)

## 摘要

智能体 harness 管辖语言模型如何在环境中行动；手工设计其策略十分困难，而自动化技术则需要有代表性的数据集以及代价高昂的执行。我们提出 Exo，一种用于在持续工作期间进行安全学习的智能体运行时架构。它把优化循环折叠进任务执行之中，使智能体既能修订自己的 harness，也能修订自己用于进一步改进的策略。一个最小化的受保护机制层在 harness 经由可检查的代码变更而演化的同时，保存执行历史并管辖策略激活。在一款开放世界游戏的长期目标上，相对于 Codex 与 Pi，Exo 在独立运行中将模型成本降低 12–53%，在迁移运行中降低 45–82%，同时任务完成率与之相当。与不具备编辑策略能力的同一起始策略相比，Exo 在 AppWorld 与 SREGym 上将留出模型成本降低 35–61%，正确性相当，并将 AppWorld 的平均任务时间缩短 13%，将 SREGym 的诊断时间中位数缩短 35%。在 Continual Learning Bench 的编码任务上，计入学习的模型成本下降 14%，两种策略都解决了每一个问题。

## 1 引言

智能体已迅速兴起，成为一类新的应用负载，并在开发者工具 claude-code; codex、个人助理 openclaw; hermes 以及企业工作流 pace-insurance; resolve-ai; sierra-siriusxm 中得到广泛采用。这些应用把语言模型与一个专门的 *harness* 结合起来，后者的策略负责构建上下文、提供工具、管理记忆，并在环境中协调动作。要把智能体适配到每一个领域，就必须适配这些 harness 策略：网络运维智能体所需要的工具、上下文与执行策略，不同于编码智能体。harness 工程使得这种专门化不必改变模型权重就能实现，但开发者必须决定哪些策略适合每一项任务。

构建有效的 harness 策略需要大量开发工作。开发者必须选择提示词，决定保留哪些信息以及如何组装上下文，并选择向智能体暴露哪些工具与技能。手工作出这些选择既耗时，又依赖领域专门知识与直觉。智能体构建与优化工具试图在基准上把这一过程自动化：Meta-Harness 在 harness 实现之上搜索 lee2026metaharness，GEPA 通过反思执行轨迹来优化提示 agrawal2025gepa，AlphaEvolve 与 SkyDiscover 支持由评估器引导的程序优化与程序合成 skydiscover2026; novikov2025alphaevolve，业界也提供类似方案 pace2026tempo。这些系统把开发组织成一个外部优化循环：该循环提出候选 harness，在基准上执行它们，并用结果指导后续修订。

然而，外部优化所依赖的评估任务可能无法获得，重复执行的代价也越来越高，与此同时，harness 策略在后续工作期间仍然是固定的。在新领域中，基准任务与忠实的评估环境往往无法获得；一项研究报告，受访团队中有 75% 在没有基准集的情况下评估其智能体 pan2025map。即便基准存在，任务视野变长的趋势也会提高通过跑完任务来评估候选的成本，已有报告中的一些执行持续数日或数周 openai2025codexmax; lin2026selfdriving; carlini2026compiler。随着环境发生变化，基准也可能过时。最后，由此得到的 harness 策略在执行期间保持固定，即便智能体的运行时经验可能暴露出一个缺口。综合来看，这些挑战会使外部的智能体构建技术在真实环境中难以实用，或限制其有效性。

在本工作中，我们提出另一种 harness 适配途径：为一个最小化的智能体配备检查并编辑其自身 harness 的能力，然后让它利用正在进行的工作中的经验，在运行时修订并适配自己。如第 3.1 节所说明的，有两个性质使这种做法得以离开传统的外部自适应控制：智能体可以检查并推理实现其自身 harness 的代码，并且可以用它执行普通任务动作时所用的同一套代码编辑工具来实现策略修订。这使我们能够 *折叠优化循环*：智能体同时选择任务动作，以及它如何行动这一方面的改变（图 1）。适配不必等待对已完成执行轨迹作回顾性复查：既然自编辑是一项显式动作，智能体就可以探索其他行为、评估其效果，并决定是否在任务进行期间把它们纳入自己的 harness。这一设计使智能体用来管辖其动作的那一项策略本身，能够随着经验在运行时演化，包括提高它改进自身的能力。

图 1：一次服务事故中的示意性 harness 适配。智能体构建一个诊断工具，并修订上下文组装，同时继续同一项任务。模型权重保持不变。

然而，允许智能体编辑自身会带来两个核心的系统挑战。第一，由此在优化器与正在运行的系统之间形成的 *命运共享* 意味着，一次有缺陷的编辑可能同时使智能体、以及诊断并修复它所需要的逻辑失效。第二，自编辑可能抹去未受保护的状态——执行历史、记忆与任务产物——从而毁掉已经积累的进展，以及对过去动作、成功与失败的记录。智能体若不能执行，就无法修复自身；若失去了经验，也无法从这些经验中学习。

为应对这些问题，我们提出 Exo，一种在正在进行的工作期间进行学习与适配的智能体运行时架构，并提供支持智能体 harness 策略安全演化的机制。我们的关键想法，是把智能体划分为一个可演化的策略层，以及安全运行所必需的一组最小化受保护机制。如第 4 节所示，这些机制在历次策略修订之间保存执行历史与先前状态，并把恢复控制权保持在可编辑层之外。受操作系统中将策略与机制分离这一工作的启发，尤其是外核 engler1995exokernel，我们暴露原语接口，把尽可能多的策略留在可演化层中，其中包括智能体如何使用自己的经验以及如何改进自身。Exo 的架构使学习可以被观察、可以被适配，同时限制一次有缺陷的修订所能造成的损害。智能体可以检查自己的执行历史，包括结果、模型成本与缓存使用，从而找出扎根于它实际工作环境的改进。它可以用这些证据同时修订自己的任务策略与优化策略。在整个过程中，受保护的历史与快照保存着一次失败修订之后继续学习所需要的经验与恢复点，而一个独立的守护者控制修订后的 harness 策略何时生效（第 4 节）。由此得到的适配是可解释的：它们表现为 harness 代码上的显式变更，开发者可以检查这些变更。

我们评估运行时学习的两项益处：在复杂任务内部提高效率，以及把由此形成的专门化迁移到相似任务上。我们在 RuneScape 中研究长任务适配。RuneScape 是一款动态的、开放世界的多人角色扮演游戏，目标覆盖战斗、制作与货币。在独立运行中，Exo 以比 Codex 和 Pi 低 12–53% 的模型成本完成战斗与制作，并且每花费一美元模型费用多赚取 44–125% 的货币。在新账户上的战斗迁移运行中，成本降低 72–82%。我们还在 AppWorld、SREGym 以及 Continual Learning Bench（CLB）编码任务 asawa2026continuallearningbench 上，把 Exo 与不具备编辑策略能力的同一起始策略作比较。在 AppWorld 上，在观测到的通过率同为 96.9% 时，Exo 将留出成本降低 61%，并将计入学习的总成本降低 43%。在 20 个留出的 SREGym 事件上，它将诊断时间中位数与测试成本降低 35%，并将模型轮次降低 38%。在 CLB 上，它少用 15% 的 shell 步数，计入反思的模型成本低 14%，五次运行的问题解决率与之相当。

在本工作中，我们作出三项贡献：

- 我们提出运行时 harness 自编辑，并辨明那些性质，它们使正在运行的智能体能够同时修订自己的任务策略与自己的改进策略，从而折叠优化循环。

- 我们辨明运行时自编辑所面临的挑战，并用一种架构来应对：该架构把可演化策略，与用于保存状态和控制策略变更的受保护机制分离开来。

- 我们实现 Exo 并将其作为开源软件发布 exo2026，同时在开放世界目标与三个分级基准上对它进行评估，表明在正在进行的工作期间可以获得效率收益，并且学到的策略可以在后续任务上复用。

## 2 背景

我们首先定义智能体及其策略，然后说明把策略适配到真实世界负载时所遇到的挑战。

### 2.1 智能体与 harness

*智能体* 通过一个反复进行的交互循环，把语言模型推断耦合到环境 yao2023react。我们把 *harness* 看作围绕模型的脚手架，它（1）组装模型的输入，（2）在环境上执行动作并观察这些动作 anthropic2025contextengineering；这个环境最常见的是一个计算沙箱。*任务* 是一个工作单元，带有目标与完成标准，完成标准可以是隐式的。任务的粒度各不相同：一个 SRE 智能体可能管理一次软件发布，也可能管理所有发布在一个月内的可用性 SLO。

**Harness 策略。** 我们用 *策略* 指管辖智能体如何使用其模型、状态与环境的全部可编程规则与过程。其中包括：工具的实现与选择；提示；上下文组装与压缩；记忆与技能的索引及检索；以及外部集成，例如服务适配器与任务调度。这些选择编码在 harness 代码、提示词、配置以及可复用技能之中。策略还包括决定何时以及如何尝试进一步改进的方案。

策略中的工具与环境里由智能体编写的代码，在精神上是相似的（就它们能够完成的事情而言），但智能体管理它们的方式不同。策略级工具常常被直接接入上下文组装，而环境中的代码则必须完全靠模型推理来记住并管理。

### 2.2 harness 适配中的挑战

自动化优化方法通过一个外部循环来形成 harness 策略：该循环提出候选，在评估任务上执行这些候选，并用结果指导修订 agrawal2025gepa; lee2026metaharness。有三项局限促使我们采用另一种做法。

评估数据集难以获得，也难以维护。新领域往往缺少有代表性的任务集，也缺少忠实的评估环境。在一项关于生产环境中智能体的研究里，受访团队中有 75% 在没有基准集的情况下作评估 pan2025map。构建基准需要捕捉相关任务、期望结果，以及动作在其中生效的环境。即便做到了这一点，服务、接口与用户需求的变化仍可能使这些任务和环境过时 krentsel2026twogaps。优化器因而可能对着一个不再反映现实的代理去提高性能。

更长的任务提高评估成本。被评估任务的视野正在变长 metr2026horizons，软件构建与定理证明项目已经延续数周 carlini2026compiler; openai2026navierstokes。随着执行变长，用完整的任务运行去评估一个又一个候选策略会变得更贵。等待任务完成还会推迟反馈，即便某个低效的过程或反复出现的失败在早得多的时候就已经显而易见。

固定的 harness 策略限制运行时适配。即便一项策略是针对有代表性的负载优化出来的，它仍可能在执行期间遇到新的模式。固定的 harness 可以对观测作出反应，也可以保留记忆，但它构建上下文和执行动作的过程保持不变。例如，智能体可能已经认识到大部分诊断输出是无关的，而它的上下文构建策略却继续把这些输出包括进来。如果没有一条从这一观测通向策略修订的路径，同样的低效就会在整个任务中反复出现。

## 3 方法

（a）经典“自适应控制”，其中包含优化器与控制器。

（b）折叠后的优化器 + 控制器（Exo 的做法）。

图 2：折叠自适应优化循环。在（a）中，一个单独的优化器把执行反馈映射为对 harness 的更新 $\theta$。在（b）中，智能体通过同一个动作接口 $u$ 选择任务动作与修订动作。

为应对第 2 节提出的挑战，我们提出另一种创建策略的方法，即运行时 harness 适配：正在运行的智能体利用正在进行的工作中的经验，修订管辖这项工作的策略。这使得任务动作与策略修订进入同一个决策过程。

### 3.1 折叠优化循环

第 2.2 节所述的外部优化系统，可以放在控制论的视角下，看作自适应控制循环 astrom2008feedbacksystems。在图 2（a）所示的传统自适应控制模型中，控制器（harness 策略）把目标 $r$ 以及对环境的观测 $y$ 映射为任务动作 $u$（工具调用）。优化器随后观察同一次执行，并对控制器策略产生更新 $\theta$。

harness 有两个特殊性质，使我们能够把优化器放进核心策略本身。第一，一次 harness 修订可以在与普通任务工作相同的动作域中实现，我们称之为优化器对称性。更精确地说，更新 $\theta$ 可以由 $\mathcal{U}^{*}$ 中的一个动作序列产生：智能体可以使用已经通过 $u$ 暴露出来的工具检查源代码、编辑文件、运行测试并请求激活。于是，修订动作成为核心策略可以选用的选项，与直接作用于任务的动作并列。

第二，harness 在操作上是同像的：它的策略就是代码，而这些代码表示为文件，智能体可以用自己的代码编辑工具观察并操纵这些文件。因此，被优化的对象可以作为优化器所处环境中的数据来获得。同一域中的动作，加上可编辑的策略表示，使优化器的提议逻辑能够移入 harness 内部，从而形成图 2（b）中那个被折叠的循环。

由于管辖何时以及如何修订 harness 的策略本身是可编辑的，智能体既可以修订自己的任务行为，也可以修订自己进一步改进的策略。每一个后继者都可以采用一项新策略，同时仍然保留进一步修订自身的能力，而无须重新训练模型。

### 3.2 状态与替换契约

把修订带入正在进行的执行，会造成两个系统挑战。第一，它在核心逻辑与优化器逻辑之间引入 *命运共享*：一次有缺陷的编辑可能同时使智能体、以及它本会用来诊断并修复自身的逻辑失效。第二，对 *未受保护状态* 的编辑可能抹去已经积累的进展和执行证据，包括失败修订的记录。

为应对这两种关键失效模式，我们转向系统架构。我们首先为系统的运行时状态与转移建模，然后指明在智能体修订其策略时架构必须强制执行的性质。

**状态与转移。** 我们把实现智能体 harness 策略的那个可替换程序称为 *执行器*。我们把第 $t$ 步的运行时状态取为 $s_t$，并令受保护机制 $K$ 施加执行器所选择的操作 $a_t$：

$$
s_t = (d_t, e_t, \pi_t), \tag{1}
$$

$$
s_{t+1} = K(s_t, a_t). \tag{2}
$$

这里，$d_t$ 包含持久数据，包括历史、记忆与产物；$e_t$ 包含沙箱文件、依赖与进程；$\pi_t$ 标识当前处于活动状态的执行器实现与配置。操作可以更新数据、在沙箱中采取行动，或者请求替换。为了保持受保护，$K$ 中的存储规则与替换规则必须留在执行器的编辑权限之外。

**存储契约。** 令 $H(d)$ 表示规范执行历史，即对消息、工具调用及其结果，以及策略替换事件的记录。存储契约要求这一历史是只追加的：

$$
H(d_t) \preceq H(d_{t+1}), \tag{3}
$$

其中 $\preceq$ 表示前缀关系。已有记录不能被覆盖，也不能被删除。记忆与任务产物的当前值可以改变，但更新必须保留先前版本，以便恢复。这些规则保存证据，而不规定策略应当如何使用这些证据。

**替换契约。** 编辑源代码会产生一个任意的候选 $\pi'$。它的行为可以是任意的，包括崩溃；因此，要把它激活为活动策略，就必须经过一项位于策略本身之外的检查。我们把这一机制称为守护者 $G(s_t, \pi')$，它的取值为 $\{0,1\}$。于是，对于一次替换请求，

$$
\pi_{t+1} =
\begin{cases}
\pi', & \text{若 } G(s_t, \pi') = 1, \\
\pi_t, & \text{否则。}
\end{cases}
\tag{4}
$$

策略替换保留持久数据，并使沙箱保持不变，同时把替换事件追加到历史之中；接受判定可以同时检查这二者。

合在一起，这些契约使智能体能够以最大的灵活性修订自己的策略，同时确保编辑不能抹去受保护的历史，也不能绕过替换检查。需要说明的是，无论智能体如何选择自己的编辑，这一保护都成立，但它本身并不保证行为会变得更好，也不保证进展会持续下去。我们在　　中给出保持性证明，并讨论这些保证的假设与限度。

## 4 设计

（a）策略–机制划分。

（b）替换过程中的状态。

图 3：Exo 把可编辑策略与受保护机制分开（a）。策略源代码被暴露出来供自编辑使用，执行器的激活由一个守护者控制。在一次被接受的执行器替换过程中（b），历史被保留并得到延伸，沙箱可以留在原地。状态契约与替换契约见第 3.2 节。

Exo 架构的目标，是实现第 3.2 节中的状态契约与替换契约，从而提供安全性，同时让智能体能够在运行时以尽可能大的自由度编辑自己，以改变自己的工作方式。下面说明为实现这一点而作的架构设计决定。

### 4.1 划分策略与机制

我们从计算系统的角度考虑智能体架构。对 harness 的传统看法把两种不同的关切放在一起：服务于一个智能体所需要的基础设施，包括持久历史、特权资源与沙箱；以及管辖智能体如何思考、如何行动的策略，包括提示组装、压缩、记忆与工具。这使得自我演化难以安全进行，因为可演化的策略与受保护的资源并没有分开。

我们的关键指导性设计决定，是把 harness 分成两个不同的层：受保护的机制层，以及可编辑的策略层，类似于操作系统（OS）设计中内核与用户空间的划分。为了决定哪些功能应当放在每一层，我们借鉴 Exokernel engler1995exokernel，目标是隔离出我们的安全性质所需要的最小功能集合，并在最低的抽象层次上把它暴露出来，从而最大化可以在它之上表达的行为空间。

图 3（a）给出我们所选择的划分。在这条边界之下，Exo 保护那些为了在自编辑之间保持连续性所需要的状态与机制。它们是规范执行历史——通过一个带有成本标注的、只追加的、可查询接口暴露出来——以及一个带版本的产物存储，其中可以存放任意数据，例如记忆、技能或沙箱快照。最后，Exo 还提供受保护的机制，用于凭证存储、沙箱 API，以及控制替换执行器激活的独立守护者机制。

其余所有 harness 功能都在策略层中表达，我们把这一层称为执行器。其中包括：工具的实现与选择；上下文组装与压缩；提示；记忆与技能的索引及检索；以及外部集成，包括服务适配器与任务调度。例如，受保护机制保存执行历史，而策略决定这段历史的哪些部分被放进发给模型的提示。智能体可以修订受保护边界之上的任何策略代码，包括它的工具、模型调用、压缩，以及管辖它何时、如何尝试进一步改进的那些过程。

### 4.2 使运行时自编辑成为可能

要在这一架构上实现运行时自编辑，我们采取两个步骤：

1. （1）把执行器代码挂载到沙箱中，使智能体能够用自己已有的工具检查并编辑这些代码。

2. （2）提供一个工具，用来通过受保护的守护者请求替换执行器。守护者构建该候选，并且只有当候选通过守护者函数 $G$ 时才激活它。

智能体因而可以在任务执行期间检查并编辑自己的实现，然后请求激活修订后的策略（图 3（b））。守护者独立于执行器运行，替换请求及其结果被记录在事件历史中。我们的默认守护者检查构建是否成功；更丰富的实现可以在一段更长的观察期上评估候选的行为（）。

**独立回滚。** Exo 的这一划分还使执行器策略、持久的智能体状态以及环境沙箱能够各自独立地回滚。执行器回滚会恢复一个更早的代码版本，并通过守护者重新激活它。只追加的历史支持从更早的某个事件分叉，带版本的产物则保留更早的状态以供恢复。沙箱 API 另外支持对任务环境做快照和还原。这些操作保留原始的执行记录，使智能体即使在回滚之后，仍然保有它曾经尝试过什么的证据。快照的范围，以及对逆转外部效应的限制，在　　中讨论。

### 4.3 内部可读性

要引导自我改进，智能体需要既能检查自己过去的行为，也能检查自己为此付出的成本。Exo 把执行历史暴露出来，供检索和分析，并在模型消息事件上标注成本与 token 计数，其中包括被缓存的 token。这样，更早的执行即使已经不再包含在模型的上下文里，仍然可以被检查。策略代码可以在它为组装上下文而检索到的消息旁边呈现这些标注，也可以通过汇总执行统计的工具来呈现。我们的实现提供一个报告工具，涵盖花费、缓存输入所占比例、重复的工具调用以及上下文的构成。例如，智能体可以检查一次上下文修订是否在保持缓存复用的同时减少了输入 token。

**实现与部署。** 我们已经实现 Exo，并将其作为开源软件发布 exo2026。核心运行时架构，包括受保护机制，共计 $\sim 20{,}000$ 行代码，基础起始策略包含 $\sim 1000$ 行。该运行时已在生产环境中部署，在 $O(5\mathrm{m})$ 条消息上服务过 $O(10\mathrm{k})$ 个多样的任务。

图 4：独立运行与迁移运行中的开放世界结果。战斗与制作报告的是完成所需的模型成本（越低越好）；赚钱报告的是接近八个活跃小时处、每模型美元所获得的金币数（越高越好）。

## 5 评估

我们评估 Exo 运行时学习的两个方面：（1）专门化是否提高一项正在进行的任务内部的效率；（2）由此得到的工具与策略在后续任务上是否仍然有用。我们首先在长期运行的开放世界目标上，把 Exo 与现有 harness 作比较，然后考察专门化如何形成、如何迁移。最后，我们在分级的应用任务、基础设施任务与编码任务上评估效率与正确性。

### 5.1 实验设置

**比较。** 我们的开放世界实验在不同目标上比较 Exo、Codex openai2026codexcli 与 Pi zechner2026pi，模型为 gpt-6.1-sol，推理力度为中等。对于 AppWorld、SREGym 以及 Continual Learning Bench（CLB）编码，我们使用 gpt-5.6-sol，并把 Exo 与不具备编辑策略能力的同一起始策略作比较。这一对照包含用于编码、shell 访问、子任务规划以及记忆的标准工具，在我们的图中标记为 *Control*。下面分别说明每个负载的任务序列与反馈。

**指标。** 我们在报告任务结果的同时报告模型成本与时间。对于有限的游戏目标，我们测量完成所需的成本，以及在一个固定窗口内每模型美元所取得的进度。所有成本都使用计入缓存的统一 token 定价。对于分级任务，我们报告正确性与执行效率，并把留出任务上的成本，与包含学习及反思在内的总成本区分开来。一次 *独立运行* 开始时不保留智能体状态；对于游戏目标，一次 *迁移运行* 在一个新账户上重复该目标，同时保留智能体状态。

### 5.2 开放世界执行期间的专门化

我们首先在一个基于 RuneScape 的仿真环境中考察长时任务里的运行时专门化。RuneScape 是一款开放世界的多人角色扮演游戏 runescape。智能体通过共同的 RS-SDK 接口 bittker2026rssdk 追求目标，并从新账户出发。我们选择三个彼此不同的目标：把全部战斗属性训练到各自的最大值；用任何可用的策略积累货币；以及直接制作一件有挑战性的物品，即精金板甲（adamant platebody）。制作该物品需要采集资源并进行训练，并且不得购买成品。这些目标要求在一个非常大的开放世界中进行数小时的协调行动；这个世界具有由怪物与共享资源构成的动态环境。战斗运行在一台公共服务器上进行，该服务器上还有 1,200-1,500 名其他玩家和机器人。所报告的其余任务，由于公共服务器过载，运行在一台共享的私有服务器上。所有智能体在同一个游戏世界中并发运行。

Exo 以低于 Codex 和 Pi 的模型成本完成这些有限目标，并且每模型美元赚得更多货币。图 4 比较独立运行与迁移运行的结果；　　提供这些结果的表格式视图。在独立运行的战斗中，Exo 花费 3.71 美元，Codex 为 7.26 美元，Pi 为 7.95 美元；在制作中，它花费 2.99 美元，对照 Codex 与 Pi 分别为 3.90 美元与 3.41 美元。合起来，完成成本降低 12–53%。在赚钱方面，Exo 每美元大约赚得 332k 金币，对照 Codex 与 Pi 分别为 231k 与 148k，提高 44–125%。所有实验组都完成了这些有限目标。

#### 5.2.1 审视运行时专门化

图 5：战斗中的专门化与复用。（a）前 45 个活跃分钟，即把（b）中的紫色框放大。（a、b）中的圆圈标出 Exo 的工具变更与记忆更新。（b–d）五个活跃小时内，独立运行的成本、进度与决策时间；A–C 标出正文所讨论的工具变更。（e、f）在保留先前智能体状态的新账户上，迁移运行的成本与进度；公共服务器使用量的一次尖峰拉长了迁移运行。（a、b）中的成本在模型调用处呈阶跃变化。决策时间是每个 15 分钟窗口内、从输入到下一次模型动作的时间中位数，其中包括提供方延迟与网络延迟。

Exo 先在专门化上投入，随后在同一项任务中以更低的速率花费模型费用。图 5（a）给出各智能体在战斗任务上的进度轨迹。在第 10 分钟时，Exo 大约已经花费 0.31 美元，Codex 为 0.12 美元，Pi 为 0.16 美元（图 5（a））。Exo 早期工作的很大一部分，是构建并修订一个可复用的战斗工具，以及编辑上下文组装策略；Codex 与 Pi 则以相对较少的推理向各自的目标推进。然而，由于这些改动，Exo 很快优化了自己的花费速率：Pi 的累计花费在第 25 分钟超过 Exo，Codex 的累计花费在第 86 分钟超过 Exo。我们把这笔早期开支称为“专门化成本”：自我改进为了在这次独立执行的后期得到更好的表现，预先支付一笔溢价。

Exo 的专门化表现为对其工具、动作过程以及上下文策略的具体改动。图 5（c）标出 Exo 在运行时所作十六次自编辑中的三次。在 A 处，Exo 创建 `train_combat`，它把近战动作成批执行，并报告获得的经验与观测。在反复未能攻击某个目标之后，它在 B 处修订该工具，使之跳过停滞的目标，并更早撤退。在 C 处，它构建 `combat_cycle`：选择最低的、尚未达到上限的近战技能，训练 180 秒，收集附近的资源，并在生命值低时停止。此后，它修改上下文组装，只保留此前四个回合的摘要，把更早的策略留在持久记忆中。这些编辑用可复用的过程取代反复作出的模型决策，并改变智能体向模型呈现自身经验的方式，从而使智能体的行为明显更加高效。

当工作几乎无法复用时，专门化可能收不回它的成本。我们另外选择了一个目标，用来代表重复性最低的那一类任务：尽可能多地完成 quest。每个 quest 只完成一次，并且所需要的步骤、物品与地点大体上各不相同。在每个实验组的三次运行中，Exo 平均每个 quest point 花费 0.28 美元，Codex 为 0.24 美元，Pi 为 0.21 美元。Exo 构建了可复用的对话工具，但特定于各个 quest 的工作几乎没有机会去摊销专门化成本。因此，对于这样的任务，不具备自我反思或改进的其他 harness 运行起来更便宜。

### 5.3 专门化 harness 的迁移

独立运行期间学到的工具与策略，在智能体于新账户上重复这些目标时仍然有用。三个实验组都保留自己的历史与沙箱状态；Exo 还携带自己的自我修改。如图 4 所示，Exo 的战斗成本从 3.71 美元降到 1.92 美元，而 Codex 的迁移成本为 6.87 美元，Pi 为 10.89 美元。它的制作成本从 2.99 美元降到 0.88 美元，对照 Codex 与 Pi 分别为 1.89 美元与 1.61 美元。Exo 因而以比其他 harness 低 45–82% 的模型成本完成这些迁移目标。由于预先的学习成本已经支付，它自己的效率从独立运行到迁移运行有所提高。它的赚钱效率大致保持不变，因为它在独立运行的早期就已经找到接近最优的策略。

在战斗迁移中，Exo 复用其专门化的 harness，而不再进一步修改自己。图 5（e）中较低的花费速率，反映的是应用上一次运行中获得的工具与上下文改动。所有实验组都保留自己的经验，但由此形成的策略使用这些经验的方式不同：Exo 可以直接调用它已经学到的动作过程。如图 5（f）所示，三个实验组都在大约九个活跃小时之后达到战斗目标，这比独立运行更长，而 Exo 保持其在模型成本上的优势。这一实验表明，在执行条件已经改变的情况下，学到的东西可以在一个重复的目标上复用。

图 6：三个随机种子下 AppWorld 的留出结果。条形为均值，符号为各个种子，误差线为 95% $t$ 区间。Control 使用不具备编辑策略能力的同一起始策略。

图 7：AppWorld：一次完整运行中计入反思的累计模型成本（上图），下方将紫色框放大。虚线标出 Exo 的策略编辑。Exo 的累计成本在任务 9 处降到 Control 之下。

### 5.4 跨分级任务的学习

我们接下来评估，harness 适配是否能在应用任务、基础设施任务与编码任务上提高效率。对于全部三个基准，我们都与不具备编辑策略能力的同一起始策略作比较。我们限制所有实验组的网络访问，以防止作弊。所有实验组在学习任务之后都收到相同的正确性反馈。AppWorld 与 SREGym 随后使用不带反馈的留出任务；CLB 则在一个固定的问题序列上测量学习。第 5.4 节概括主要结果。

表 1：Exo 与不进行策略编辑的同一策略（Control）在分级任务上的结果。箭头表示更优的方向。AppWorld 与 CLB 的数值为均值；SREGym 报告时间与轮次的中位数，以及成本的均值。CLB 的成本包含反思。官方 HTML 转换丢掉了表体，下列单元格据 PDF 转录。

| 指标 | Control | Exo |
|---|---:|---:|
| AppWorld，留出任务；3 个种子 | | |
| 通过的任务 ↑ | 96.9% | 96.9% |
| 每项任务的模型成本 ↓ | $0.359 | $0.141 |
| 每项任务的时间 ↓ | 36.5 s | 31.9 s |
| SREGym，20 个留出事件 | | |
| 诊断时间 ↓ | 81 s | 53 s |
| 每个事件的模型轮次 ↓ | 16 | 10 |
| 每个事件的模型成本 ↓ | $0.892 | $0.584 |
| CLB 编码，全部任务；5 次运行 | | |
| 已解决的问题 ↑ | 100% | 100% |
| 每个问题的 shell 步数 ↓ | 7.42 | 6.28 |
| 每个问题的模型成本 ↓ | $0.507 | $0.436 |

## 5.4.1 AppWorld

AppWorld（Trivedi et al., 2024）给出跨越九个日常应用的用户请求，例如计算已经付给室友的款项。Exo 在 50 个场景各自的一个变体上学习，每项任务之后有评分器反馈和反思，然后在这些场景的 100 个留出变体上执行，不再提供反馈或反思。我们用三个随机种子重复这一序列；两个实验组以及所有种子都使用相同的任务顺序。

Exo 在观测到的通过率相同的情况下，将留出模型成本降低 61%，节省下来的费用也覆盖了学习阶段。两个实验组都通过了 96.9% 的留出任务。Exo 每项任务花费 0.141 美元，对照为 0.359 美元；它少用 27% 的模型轮次，并把平均任务时间缩短 13%（图 6）。计入学习与反思，平均总成本从 57.36 美元降到 32.70 美元，降低 43%。

对学到的 harness 进行检查，可以看到可复用的 API 文档工具，以及对上下文构建的改动。

## 5.4.2 SREGym

SREGym（Clark et al., 2026）评估智能体如何诊断并缓解被注入正在运行的服务中的故障。效率比较使用 21 个带有反馈与反思的学习事件，随后是 20 个未见过的事件；这些事件属于相同的故障族，但发生在不同的应用上，并且不提供反馈或反思。

Exo 学到的策略降低了诊断时间、模型轮次，以及处理后续事件的成本。诊断时间的中位数从 81 秒降到 53 秒（35%），模型轮次的中位数从 16 降到 10（38%），每个留出事件的平均模型成本从 0.892 美元降到 0.584 美元（35%；表 1）。在额外的留出运行中，正确性相当。

Exo 构建并反复精炼一个诊断工具，把各个事件都需要的证据打包在一起。Exo 创建并精炼一个事件快照工具，用以收集不健康的 pod、修订与模板上的异常，以及可疑的日志行；它把这个工具修订了五次，并在留出事件上把它作为第一步诊断来复用。该工具把原本需要多次 shell 调用才能得到的证据合并起来，使后续诊断有一个可复用的起始过程。它另外又构建了 5 个用于针对性调查的工具。

## 5.4.3 Continual Learning Bench 编码

在 CLB 编码任务上，Exo 少用 15% 的 shell 步数，模型成本低 14%，两种策略都解决了每一个问题。我们使用 Continual Learning Bench（Asawa et al., 2026），对来自 tablib 与 tenacity 的一个固定的 19 个问题序列进行评估，每个实验组运行五次。两个实验组都在每一个被评分的问题之后进行反思；没有单独的留出阶段。计入反思，Exo 平均每个问题使用 6.28 个 shell 步、花费 0.436 美元，对照为 7.42 个 shell 步与 0.507 美元。所观测到的自我改动，是在问题与问题之间对工具和技能所作的编辑。

**适配的限度。** 只有当模型有足够的能力发现并复用有效改动时，能够进行自编辑才会带来收益。在一项额外实验中，我们用 gpt-5-mini 在 AppWorld 上运行 Exo；三个随机种子只做了有限的 harness 编辑，并且没有在留出任务上复用它们所创建的工具。总成本与通过率相近（6.63 美元与 59% 对 6.73 美元与 55%，对应 Exo 对 control）。因此，我们认为，要获得 Exo 这一架构的收益，存在一个必要的智能水平下限。

## 6 相关工作

**从执行反馈中学习。** 执行反馈可以通过保留下来的经验教训（Shinn et al., 2023; Zhao et al., 2024; Ouyang et al., 2025），或通过对提示与工作流的修订（Zhang et al., 2025b; Agrawal et al., 2025; Yuksekgonul et al., 2024; Cheng et al., 2024），来引导未来的行为。Self-Defining Operator 聚焦于事件响应，学习可复用的行动手册与检测器（Li et al., 2026）。在 Exo 中，正在运行的智能体还可以修订那些决定它何时以及如何从反馈中学习的代码。

**Harness 构建与自修改。** Harness 搜索自动完成策略的生成与精炼（Lee et al., 2026; Lou et al., 2026; Xia et al., 2026），包括在无标注测试批次上的演化（Nie et al., 2026）。JIT-Agent 生成以任务为条件的 harness，其历史不可变，而控制器状态可变（Zhang et al., 2026a）。自修改涵盖对运行中代码的更改（Yin et al., 2024）、由基准驱动的智能体演化（Zhang et al., 2025a），以及可编辑的改进过程（Zhang et al., 2026b）。同期工作 Harness2 在没有真值验证器的情况下精炼任务 harness，并跨任务更新一个改进 harness（Xu et al., 2026）。Exo 把自修改与任务延续耦合在一起：一项被接受的修订会继续该任务，并保留其历史与环境。

任务内适应的差别在于哪些部分保持固定。Live-SWE-agent 在保留其底层智能体循环的同时创建工具（Xia et al., 2025）；Continual Harness 周期性地对提示、子智能体、技能与记忆调用一个精炼器（Karten et al., 2026）。Exo 还将支配上下文构建、模型调用与精炼的执行器代码暴露出来。行动中的智能体既可以改变这些行为，也可以改变决定何时以及如何修订这些行为的策略。

**面向适应的运行时支持。** 运行时适应既需要可扩展性，也需要对状态与执行的独立控制。DeepSeek Harness 把可替换插件与只追加的事件日志结合起来（DeepSeek-AI, 2026），并借助 Cordis 实现热替换与可逆的组件效应（Shi et al., 2026）。独立的检查决定哪些动作与更改得以生效：LogAct 把共享日志与接纳任务动作的解耦投票者耦合在一起（Balakrishnan et al., 2026），而 Self-Healing Harness 则通过回放或前向试验来接纳行为规则（Tayebati et al., 2026）。Exo 利用这种分离，让智能体替换支配一项进行中任务的策略，同时把执行历史与替换权限留在该策略的控制之外。

## 7 讨论

构建与「苦涩的教训」相一致的系统。随着推理能力的增长，我们认为，围绕这些能力所构建的系统应当提供能够持续存在、并与这种增长成比例扩展的收益，而不是像《苦涩的教训》（Sutton, 2019）所指出的那样变得不再必要。要做到这一点，就必须重新排列传统设计目标的优先级：灵活性、可修改性与可读性必须被提升为一等设计原则。在极限情形下，这意味着把全部 harness 策略——包括它如何改进自身——置于模型的控制之下，只保留安全的自我演化所需要的机制。Exo 以一个最小的受保护核心，以及一个可编辑、可检查的策略层来追求这一设计。值得注意的是，这并不规定一种“薄”的 harness：智能体可以并且确实会构建大量领域专用的工具，或者移除它不再需要的装置。恰当的 harness 可以同时随模型及其工作负载而改变。

运行时可读性有助于做出更好的决策。要让自我改进的系统发挥作用，它们需要对自身行为的可见性。策略源代码暴露出可以更改的内容，可查询的历史记下已经发生的事情，而成本、延迟与缓存标注则暴露所消耗的资源。这些观察为执行效率提供了更密集的奖励信号，帮助智能体在任务完成之前评估并引导改进。我们认为，这是 Exo 之所以有效的一个关键原因。跨回滚保留历史，同样使失败尝试的证据仍可供此后的决策使用。

系统开发的新时间尺度。历史上，构建一个系统所需的时间一直长于单项任务的时间尺度。随着模型辅助的开发变得更快、任务时域不断变长，这一假设已经开始反转，构建有用的专门化现在所花的时间可以少于激发它的那一项任务。因此，我们必须越来越多地把开发看作可以在执行期间继续进行的事情，让过程被即时合成出来，以应对手头的工作负载。Exo 的动机来自这一趋势，并且我们在其针对长期运行的开放世界任务的评估中看到了这一点（第 5.2 节）：在工具与策略编辑上的投入，在目标完成之前就已经得到回报（第 5.2.1 节）。在部署环境中学习，也使修订立足于它们将被使用的那些条件，从而降低对有代表性的基准和忠实仿真的依赖。这表明持续适应的系统将承担更广泛的角色，而对这些修订的信任将成为一项核心的系统挑战。

推广守卫。Exo 的守卫把提出修订的权限与激活该修订的权限分开。我们在实现中作为默认提供的最小检查是构建成功；更丰富的守卫还可以额外进行更细致的、以 LLM 为裁判的策略检查，甚至在分叉环境中进行试验。智能体可以在失败出现时提出额外的测试，而对起支配作用的接受准则的更改仍然需要独立授权。更广泛的验证会带来额外成本。一个开放问题是，如何使验证投入与一项修订的后果相匹配，同时保留一条独立的强制边界。我们把对更多守卫机制的探索留待未来工作。

可解释的持续学习。Exo 的适应表现为对代码、提示与技能的显式更改。与仅仅编码在模型权重中的持续学习相比，这些产物直接暴露所学到的过程，使智能体操作者能够检查、测试并修订它们。更改也可以与促成它们的执行历史放在一起审视。这在操作者需要理解、审计并授权系统行为更改的领域中尤其有价值。未来的守卫可以在激活之前，把所提议的修订、支持证据与验证结果呈现给人类，供其批准或拒绝。这将使组织能够通过现有的评审流程，有选择地采纳学到的过程。

作为智能体运行时的 Exo。Exo 把智能体运行时分解为执行器与受保护状态，在实际部署场景中提供了额外的好处。智能体可以在不同的执行器实现之间切换，同时保留其持久历史与产物。共享的运行时服务支持多个智能体在单个节点上并发执行，每个智能体拥有自己的策略与状态。沙箱执行可以卸载到远程提供方，而可互换的环境后端让部署能够改变任务动作在何处、以何种方式执行，而无需重写智能体策略。这些能力使得执行器托管、持久化与环境资源可以被分别供给。我们把关于智能体密度、吞吐量与资源效率的系统性评估报告留待未来工作。

未来工作。从 Exo 当前架构出发，一个自然的下一步是通过显式的智能体谱系来探索并行的自我改进：分叉规范状态，在事件日志中记录父子关系，并在各分支之间提供一条通信通道。智能体随后可以并发地探索修订，并以可追溯的来源分享发现。我们把这一点留待未来工作。

## 8 结论

我们提出了 Exo，一种面向进行中工作期间的 harness 内学习与适应的智能体运行时架构。Exo 把优化循环折叠进任务执行，使智能体能够在运行时修订其 harness 的各个方面。受保护机制保存执行历史并支配执行器替换，而学到的适应则保持显式且可检查。在开放世界目标与带评分的基准上，我们的评估表明，进行中的工作期间可以获得效率收益，并且学到的过程可以在后续任务上得到复用。Exo 提供了一条路径，通向其策略能够随工作负载以及底层模型的能力一起演化的系统。

## 参考文献

参考文献列表转录自公开 PDF（https://arxiv.org/pdf/2611.00463）。URL 内部被换行断开的部分已重新接合。文献条目保持英文。

- Agrawal, L. A., Tan, S., Soylu, D., Ziems, N., Khare, R., Opsahl-Ong, K., Singhvi, A., Shandilya, H., Ryan, M. J., Jiang, M., Potts, C., Sen, K., Dimakis, A. G., Stoica, I., Klein, D., Zaharia, M., and Khattab, O. GEPA: Reflective prompt evolution can outperform reinforcement learning. arXiv:2507.19457, 2025. URL https://arxiv.org/abs/2507.19457.
- Anthropic. Effective context engineering for AI agents, September 2025. URL https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents.
- Anthropic. Claude Code, 2026. URL https://claude.com/product/claude-code. Web resource consulted September 9, 2026.
- Asawa, P., Glaze, C. M., Orlanski, G., Ramakrishnan, R., Xu, B., Biswal, A., Chen, V. S., Sala, F., Zaharia, M., and Gonzalez, J. E. Continual Learning Bench: Evaluating frontier AI systems in real-world stateful environments. arXiv:2606.05661, 2026. URL https://arxiv.org/abs/2606.05661.
- Astrom, K. J. and Murray, R. M. Feedback systems: An introduction for scientists and engineers. Princeton University Press, Princeton, NJ, April 2008.
- Balakrishnan, M., Bharambe, A., Testuggine, D., Geraghty, D., Mao, D., Venkat, V., Mironov, I., Baradi, R., Aiyer, G., and Dudin, V. LogAct: Enabling agentic reliability via shared logs. arXiv:2604.07988, 2026. URL https://arxiv.org/abs/2604.07988.
- Bittker, M. RS-SDK: RuneScape automation library for coding agents. Software repository, 2026. URL https://github.com/MaxBittker/rs-sdk. Accessed October 29, 2026.
- Carlini, N. Building a C compiler with a team of parallel Claudes. Anthropic Engineering, February 2026. URL https://www.anthropic.com/engineering/building-c-compiler.
- Cheng, C.-A., Nie, A., and Swaminathan, A. Trace is the next AutoDiff: Generative optimization with rich feedback, execution traces, and LLMs. arXiv:2406.16218, 2024. URL https://arxiv.org/abs/2406.16218.
- Clark, J., Su, Y., Pial, S. M. R., Tian, Y., Gniedziejko, L., Jacobsen, H.-A., Chen, Y., and Xu, T. SREGym: A live benchmark for AI SRE agents with high-fidelity failure scenarios. arXiv:2605.07161, 2026. URL https://arxiv.org/abs/2605.07161.
- DeepSeek-AI. DeepSeek Harness: Everything is a plugin. Software and architecture documentation, 2026. URL https://github.com/deepseek-ai/deepseek-harness. Accessed October 27, 2026; source revision 22ddee3792402c3e70ec5f86a63c3e3e895a2f60.
- Engler, D. R., Kaashoek, M. F., and O’Toole, Jr., J. Exokernel: An operating system architecture for application-level resource management. In Proceedings of the Fifteenth ACM Symposium on Operating Systems Principles, pp. 251–266, New York, NY, USA, 1995. Association for Computing Machinery. doi: 10.1145/224056.224076.
- Exo Contributors. Exo. Software repository, commit c5c1963, 2026. URL https://github.com/exoharness/exo/tree/c5c1963a3445417e64938c139da88edd9154075d.
- Jagex. RuneScape, 2026. URL https://www.runescape.com/. Web resource consulted October 29, 2026.
- Karten, S., Zhang, J., Upaa, T., Feng, R., Li, W., Shi, C., Jin, C., and Vodrahalli, K. Continual harness: Online adaptation for self-improving foundation agents. arXiv:2605.09998, 2026. URL https://arxiv.org/abs/2605.09998.
- Krentsel, A., Agarwal, S., Cemri, M., Liu, S., Sankhe, S., Mao, Z., Zaharia, M., and Stoica, I. Reality is the final verifier: On two key gaps in agentic software engineering, 2026. URL https://arxiv.org/abs/2609.12039.
- Lee, Y., Nair, R., Zhang, Q., Lee, K., Khattab, O., and Finn, C. Meta-Harness: End-to-end optimization of model harnesses. arXiv:2603.28052, 2026. URL https://arxiv.org/abs/2603.28052.
- Li, S., Anderson, T., Mahajan, R., Peter, S., and Zettlemoyer, L. A self-defining operator for distributed applications. arXiv:2610.14547, 2026. URL https://arxiv.org/abs/2610.14547.
- Lin, W. Towards self-driving codebases. Cursor Research, February 2026. URL https://cursor.com/blog/self-driving-codebases.
- Liu, S., Cemri, M., Agarwal, S., Krentsel, A., Naren, A., Mang, Q., Li, Z., Gupta, A., Maheswaran, M., Cheng, A., Pan, M., Boneh, E., Ramchandran, K., Sen, K., Zaharia, M., Dimakis, A. G., and Stoica, I. SkyDiscover: A flexible, adaptive framework for AI-driven scientific and algorithmic discovery. In Proceedings of the ACM Conference on AI and Agentic Systems, pp. 1223–1227. Association for Computing Machinery, 2026. doi: 10.1145/3786335.3813221. URL https://doi.org/10.1145/3786335.3813221.
- Lou, X., Lázaro-Gredilla, M., Dedieu, A., Wendelken, C., Lehrach, W., and Murphy, K. P. AutoHarness: Improving LLM agents by automatically synthesizing a code harness. arXiv:2603.03329, 2026. URL https://arxiv.org/abs/2603.03329.
- METR. Task-completion time horizons of frontier AI models, 2026. URL https://metr.org/time-horizons/.
- Nie, J., Zhang, Y., Song, J., Cai, Q., Yu, D., Guo, Y., Tian, X., and Han, B. TTHE: Test-time harness evolution. arXiv:2607.08124, 2026. URL https://arxiv.org/abs/2607.08124.
- Nous Research. Hermes Agent. Software repository, 2026. URL https://github.com/NousResearch/hermes-agent. Web resource consulted September 9, 2026.
- Novikov, A., Vũ, N., Eisenberger, M., Dupont, E., Huang, P.-S., Wagner, A. Z., Shirobokov, S., Kozlovskii, B., Ruiz, F. J. R., Mehrabian, A., Kumar, M. P., See, A., Chaudhuri, S., Holland, G., Davies, A., Nowozin, S., Kohli, P., and Balog, M. AlphaEvolve: A coding agent for scientific and algorithmic discovery. arXiv:2506.13131, 2025. URL https://arxiv.org/abs/2506.13131.
- OpenAI. Introducing Codex, May 2025a. URL https://openai.com/index/introducing-codex/.
- OpenAI. Building more with GPT-5.1-Codex-Max, November 2025b. URL https://openai.com/index/gpt-5-1-codex-max/.
- OpenAI. Codex CLI. Software repository, 2026a. URL https://github.com/openai/codex. Accessed October 29, 2026.
- OpenAI. On the Navier–Stokes millennium prize problem, September 2026b. URL https://openai.com/index/navier-stokes-solution/.
- OpenClaw. OpenClaw, 2026. URL https://openclaw.ai/. Web resource consulted September 9, 2026.
- Ouyang, S., Yan, J., Hsu, I.-H., Chen, Y., Jiang, K., Wang, Z., Han, R., Le, L. T., Daruki, S., Tang, X., Tirumalashetty, V., Lee, G., Rofouei, M., Lin, H., Han, J., Lee, C.-Y., and Pfister, T. ReasoningBank: Scaling agent self-evolving with reasoning memory. arXiv:2509.25140, 2025. URL https://arxiv.org/abs/2509.25140.
- Pace. The AI operations platform for insurance, 2026. URL https://withpace.com/. Web resource consulted September 9, 2026.
- Pace Team. Introducing Tempo, July 2026. URL https://withpace.com/news/tempo.
- Pan, M. Z., Arabzadeh, N., Cogo, R., Zhu, Y., Xiong, A., Agrawal, L. A., Mao, H., Shen, E., Pallerla, S., Patel, L., Liu, S., Shi, T., Liu, X., Davis, J. Q., Lacavalla, E., Basile, A., Yang, S., Castro, P., Kang, D., Sen, K., Song, D., Gonzalez, J. E., Stoica, I., Zaharia, M., and Ellis, M. Measuring agents in production. arXiv:2512.04123, 2025. URL https://arxiv.org/abs/2512.04123v4. Version 4, revised June 4, 2026.
- Resolve.ai. AI for prod, 2026. URL https://resolve.ai/. Web resource consulted September 9, 2026.
- Shi, Y., Zhang, W., and Cui, T. A programming paradigm for spatiotemporal composability. arXiv:2608.25512, 2026. URL https://arxiv.org/abs/2608.25512.
- Shinn, N., Cassano, F., Berman, E., Gopinath, A., Narasimhan, K., and Yao, S. Reflexion: Language agents with verbal reinforcement learning. arXiv:2303.11366, 2023. URL https://arxiv.org/abs/2303.11366.
- Sierra. How SiriusXM drives listener loyalty with Sierra, 2026. URL https://sierra.ai/customers/siriusxm. Web resource consulted September 9, 2026.
- Sutton, R. S. The bitter lesson, March 2019. URL http://www.incompleteideas.net/IncIdeas/BitterLesson.html.
- Tayebati, S., Kumar, D., Darabi, N., Krishnan, R., and Trivedi, A. R. Self-healing harness for runtime oversight of agent self-modification. arXiv:2609.24130, 2026. URL https://arxiv.org/abs/2609.24130.
- Trivedi, H., Khot, T., Hartmann, M., Manku, R., Dong, V., Li, E., Gupta, S., Sabharwal, A., and Balasubramanian, N. AppWorld: A controllable world of apps and people for benchmarking interactive coding agents. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pp. 16022–16076. Association for Computational Linguistics, 2024. doi: 10.18653/v1/2024.acl-long.850. URL https://aclanthology.org/2024.acl-long.850/.
- Xia, C. S., Wang, Z., Yang, Y., Wei, Y., and Zhang, L. Live-SWE-agent: Can software engineering agents self-evolve on the fly? arXiv:2511.13646, 2025. URL https://arxiv.org/abs/2511.13646.
- Xia, P., Han, R., Wang, Z., Chen, Y., Zhuang, Y., Lee, Y., Huang, C., Yu, H., CuiZhu, Z., Ming, Y., Yao, H., Gokturk, B., Pfister, T., and Lee, C.-Y. RRSI: Regularized recursive self-improvement of agent harnesses. arXiv:2609.24972v3, 2026. URL https://arxiv.org/abs/2609.24972v3.
- Xu, R., Chen, Y., CuiZhu, Z., Mishra, B. D., Ming, Y., Yu, H., Han, R., Lee, C.-Y., and Pfister, T. Harness2: Recursive agent harnessing for an open world. arXiv preprint arXiv:2610.14224, 2026. URL https://arxiv.org/abs/2610.14224.
- Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., and Cao, Y. ReAct: Synergizing reasoning and acting in language models. In The Eleventh International Conference on Learning Representations, 2023. URL https://arxiv.org/abs/2210.03629.
- Yin, X., Wang, X., Pan, L., Lin, L., Wan, X., and Wang, W. Y. Gödel agent: A self-referential agent framework for recursive self-improvement. arXiv:2410.04444, 2024. URL https://arxiv.org/abs/2410.04444.
- Yuksekgonul, M., Bianchi, F., Boen, J., Liu, S., Huang, Z., Guestrin, C., and Zou, J. TextGrad: Automatic “differentiation” via text. arXiv:2406.07496, 2024. URL https://arxiv.org/abs/2406.07496.
- Zechner, M. Pi: Coding agent. Software repository, 2026. URL https://github.com/earendil-works/pi. Accessed October 29, 2026.
- Zhang, G., Lu, L., Xie, F., Zhu, K., Wang, J., Xie, Z., Yu, Z., Liu, Z., Sun, Z., Li, Q., Liao, Y., Chang, H., Hu, X., Ren, Q., Zhou, W., Hu, C., Deng, Y., and Yan, S. JIT-Agent: Scaling harness intelligence via just-in-time harness evolution. arXiv:2608.25593v2, 2026a. URL https://arxiv.org/abs/2608.25593v2.
- Zhang, J., Hu, S., Lu, C., Lange, R., and Clune, J. Darwin Gödel machine: Open-ended evolution of self-improving agents. arXiv:2505.22954, 2025a. URL https://arxiv.org/abs/2505.22954.
- Zhang, J., Zhao, B., Yang, W., Foerster, J., Clune, J., Jiang, M., Devlin, S., and Shavrina, T. Hyperagents. arXiv:2603.19461, 2026b. URL https://arxiv.org/abs/2603.19461.
- Zhang, Q., Hu, C., Upasani, S., Ma, B., Hong, F., Kamanuru, V., Rainton, J., Wu, C., Ji, M., Li, H., Thakker, U., Zou, J., and Olukotun, K. Agentic context engineering: Evolving contexts for self-improving language models. arXiv:2510.04618, 2025b. URL https://arxiv.org/abs/2510.04618.
- Zhao, A., Huang, D., Xu, Q., Lin, M., Liu, Y.-J., and Huang, G. ExpeL: LLM agents are experiential learners. Proceedings of the AAAI Conference on Artificial Intelligence, 38 (17):19632–19642, 2024. doi: 10.1609/aaai.v38i17.29936. URL https://ojs.aaai.org/index.php/AAAI/article/view/29936.

## AI 使用披露

作者使用 Claude Code 与 Codex，协助检查并整理实验结果，以及完成部分图的绘制。全部编码工作均与 Claude Code 或 Codex 协作完成，所用模型在 2026 年 4 月至 2026 年 7 月之间有所变化，并伴有由人撰写的设计文档与协作式编程。论文写作主要由作者完成，并使用 Codex 协助修订段落以使其更清晰或更简练，以及协助排版。

## 附录 A 保持性证明与范围

我们使用正文中的状态 $s_t = (d_t, e_t, \pi_t)$、受保护转移机制 $K$ 以及守卫 $G$。

**自编辑下的保持性。** 若初始状态满足存储契约，每一项影响受保护状态的操作都由 $K$ 中介，并且每一次被允许的转移都保持该契约，则在任意有限的操作序列与执行器替换之后，该契约仍然成立。

该证明对转移作归纳。由假设，初始状态满足该契约。若该契约在第 $t$ 步成立，则应用 $K$ 会在第 $t + 1$ 步保持只追加的历史以及先前的产物版本，包括该操作替换 $\pi_t$ 的情形。归纳确立了任意有限执行上的保持性，且与执行器如何选择编辑无关。

类似地，若 $G(s, \pi') = 1$ 蕴涵候选性质 $C(\pi')$，且初始执行器满足 $C$，则每一个被安装的执行器在安装时都满足 $C$：一次转移要么保留当前实现，要么安装一个被 $G$ 接受的实现。

**强制假设与限度。** 存储机制与替换机制必须留在执行器的编辑权限之外。直接访问存储，或不受限制的管理性删除，都会使完全中介失效；可广泛写入的开发配置并不强制这些限制。守卫的接受只确立实际被检查的那些性质：仅有构建成功，既不能保证任务正确，也不能保证有所改进。从卡死的执行器中恢复，需要独立的权限来停止它并激活一个后备实现；仅有保持性并不能保证活性。限定作用域的存储把凭证保持在沙箱文件之外，但模型上下文中的暴露取决于经授权的读取以及执行器工具。我们提供了此类经授权的读取，但此处不描述其实现，因为这超出了本文的范围。

**恢复范围。** 策略回滚与沙箱回滚会保留规范历史，并记录该次回滚。快照范围取决于提供方：Docker 镜像快照会略去进程内存以及宿主机挂载的源代码。二者都不逆转外部效应，而外部效应需要领域专用的恢复。

## 附录 B 开放世界测量细节

表 A1 对每个目标与每种 harness 报告一对独立运行与迁移运行。完成成本采用第一个达到或超过目标的已记录终点。赚钱采用八个活跃小时之前最后一次被采样的回合，并排除已记录的停机。

表 A1. 开放世界结果，每个条目一次运行。战斗与制作报告完成成本；赚钱在八个活跃小时之前采样。在 PDF 中，底色在每一行以及每个运行组内比较各 harness：绿色（最佳）、橙色（中等）、红色（最差）。此处不复现底色。

| 指标 | 独立运行 Exo | 独立运行 Codex | 独立运行 Pi | 迁移 Exo | 迁移 Codex | 迁移 Pi |
|---|---:|---:|---:|---:|---:|---:|
| 战斗（美元）↓ | 3.71 | 7.26 | 7.95 | 1.92 | 6.87 | 10.89 |
| 制作（美元）↓ | 2.99 | 3.90 | 3.41 | 0.88 | 1.89 | 1.61 |
| 赚钱（美元）↓ | 2.84 | 5.14 | 5.08 | 2.84 | 3.68 | 4.08 |
| 金币/美元（千）↑ | 332 | 231 | 148 | 334 | 265 | 210 |
