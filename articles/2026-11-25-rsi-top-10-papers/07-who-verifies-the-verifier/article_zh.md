---
title: "谁来验证验证器？与自我改进 agent 协同演化的可检查评分器"
author: "Xing Zhang, Guanghui Wang, Yanwei Cui, Ziyuan Li, Wei Qiu, Bing Zhu, and Peiyang He"
source_url: "https://arxiv.org/abs/2610.11464"
arxiv_id: "2610.11464"
arxiv_version: "v1"
published_at: "2026-10-08"
archived_at: "2026-11-25"
license: "CC BY-NC-SA 4.0"
license_url: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
kind: "full-text-translation"
unofficial_translation: true
translation_scope: "摘要至结论的全部正文与附录。代码保留英文。参考文献保留英文。本译本同样受 CC BY-NC-SA 4.0 约束（非商业、相同方式共享）。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名，并以 CC BY-NC-SA 4.0 发布本译本。"
---

**译本说明。** 本文为非官方中文译本，未经作者审定。作者为 Xing Zhang、Guanghui Wang、Yanwei Cui、Ziyuan Li、Wei Qiu、Bing Zhu 与 Peiyang He。原文网址：https://arxiv.org/abs/2610.11464 。本译本受 CC BY-NC-SA 4.0 约束，仅限非商业、相同方式共享。

谁来验证 agent？迈向可靠的 agent 开发

# 谁来验证验证器？与自我改进 agent 协同演化的可检查评分器

Xing Zhang，隶属：AWS Forward Deployed Engineering。Guanghui Wang，隶属：AWS Forward Deployed Engineering。Yanwei Cui，隶属：AWS Forward Deployed Engineering。Ziyuan Li，隶属：HSBC Holdings Plc.，HSBC Technology Center, China。Wei Qiu、Bing Zhu、Peiyang He†，通讯作者：peiyan@amazon.com。隶属：AWS Forward Deployed Engineering。隶属：HSBC Holdings Plc.，HSBC Technology Center, China。

## 摘要

我们改了 agent：它真的变好了吗？每一个自我改进 agent 循环都要把这个问题回答数百次，而每一次答案都来自一个验证器。在开放式任务上，这样的验证器并不存在，于是循环拿到的是一份手写量规，或是一个赤裸的 LLM 评判器，由一个与它自身相似的模型给输出打分，从而招致奖励黑客与共同盲点。我们把验证器本身做成演化的对象：它是一组小型、大多确定的缺陷检测器之上的可检查表达式；这些检测器由聚类后的失败合成，在诞生时接受门控，并因与一个十项锚定参考集相一致、再加上在无标签输出上的共识而被选中，绝不是因为 agent 的分数。在 MBPP+ 上，它相对于手写的种子组合获得 $+0.21$ 的留出一致率增益，且在每一个种子上都是如此，并最终领先于它所包含的那个赤裸 LLM 评判器。有一项发现应当改变人们如何验证协同演化的验证器：去掉锚点护栏，验证器会坍缩成一个空洞的、一律通过的评分器，然而那个坍缩后的验证器训练技能的效果*同样好*。下游任务分数不能认证一个自我演化的验证器。分数回答的是充分性，而在充分性上，演化出的验证器可以替代真值：*Double Ratchet* 把验证器与一个受生命周期管理的技能循环配对，在代码生成、企业级 text-to-SQL 以及无参考的报告生成上，保留了真值或一份量规为同一循环所买到的提升的 88–110%。当演化出的技能博弈了报告量规时，一个外部评判器抓住了它，并且新增的一个检测器修好了它；这个评判器本身在拿到任务契约之前是错的。

## 1 引言

我们更新 agent 的提示词、添加一个工具，或改变它的推理策略：它真的变好了吗？自我改进的 agent 每一轮都在问这个问题。标准做法是把一个冻结的语言模型包进一个循环，这个循环撰写技能、检索技能、度量它们的贡献，并退役那些造成损害的技能 [1, 2, 3]；而它每一轮消耗一种昂贵的原料：一个判定某次尝试是否通过的验证器。在代码基准上，这种原料是免费的，因为单元测试存在。在已部署的系统里，它就是瓶颈，而业界自己的产品记录了这种权宜办法：Claude 的托管 agent 让开发者为单独的评分器手写结果量规 [4]，Codex 的 Record & Replay 则让人把每条工作流演示一次 [5]。二者都是对缺失验证器的人工替代，一次只供给一份量规或一次演示。

本文将这一步自动化：*agent 的验证器能否从少量锚定样例演化出来，把学习驱动得和真值一样好，并且在此过程中不被博弈？* 两处澄清决定下面每一个结果应当如何阅读。第一，验证器，亦即评分器，是给一份输出打分的函数，而且它是唯一在演化的东西；任务集及其参考在任何循环开始之前就已固定，因此这里没有任何结果可以来自一场更容易的测试。第二，在我们的目标设定中*并不存在准确的验证器*，因此没有一个可供击败的基线验证器：标尺是，一个准确的验证器*本会促成*的改进之中，系统在没有它的情况下挽回了多少。与由真值驱动的参考循环相匹配，就是获胜条件，而不是安慰奖。

这一设计立于关于验证的一个立场。我们很少知道什么是*好*，但给定一份输出，我们通常找得到*缺陷*，因此一个干净的裁决意味着没有发现已知缺陷，而不是被认证过的正确性。正是这一框架使自举成为可能，并决定了该产物的形态（图 1）：验证器是一小池有类型的检测器之上的表达式，每个检测器检查一个失败类别，由聚类后的失败合成，在诞生时被门控，按边际贡献评分，并在无用时退役。检测器横跨真实部署已经具备的三个成本层级（静态检查、对着沙箱或活仓库的执行，以及狭窄的评判查询），因此把它们组合起来，是把异质信号折进同一个评分器，而不是信任单一标量。它们大多是确定的，因而其失败方式不同于被评分的那个模型，这正是用来抵抗赤裸 LLM 评判器所招致的共同盲点共谋的东西；作为一个可读的表达式，评分器把一次失败定位到一个可以指名、并且可以手工修复的检查（§5.4）。

一个给产生它自己的那个循环打分的验证器，是显而易见的 Goodhart 风险 [6, 7]，因此锚点纪律是严格的，并且是承重的。选择从不问 agent 是否得分好：它问的是，验证器是否与一个很小的锚定参考集相一致——这是验证器演化所读取的唯一监督信号——以及是否与检测器在一个大型无标签集上的共识相一致。另有一个锁定集，任何验证器循环都不读取它，它只用于验证器评估与技能侧的回滚。我们以消融来检验那套纪律是否承担了安全载荷，而答案直接关系到这个社区究竟应当如何验证协同演化出来的验证器。

我们的贡献如下。

- 验证器演化，其目标不需要任何下游指标。我们把评分函数、而不是被评分的产物，表述为演化对象：在合成、出生门控、边际贡献评分与退役这一生命周期之下，对原子缺陷检测器的组合做锚定搜索。选择标准是与十项锚点的、按召回加权的一致率，乘以在无标签输出上按可靠性加权的共识；其结果既胜过它起步时的手写组合，也胜过它所包含的赤裸 LLM 评判器。

- 安全居于何处，以及为何任务分数找不到它。对验证器而言，承担载荷的是锚点纪律，而不是检测器的生命周期，这颠倒了技能演化所给出的教训。由于一个坍缩的验证器训练技能的效果与一个有效的验证器一样好，*用下游表现来验证一个自我演化的验证器是不健全的*：最自然的那项验证实验，会去认证一个对一切都放行的评分器。

- Double Ratchet，以及一场从头到尾的奖励黑客。把验证器与技能循环协同演化，在三个任务领域上保留了真值所买到的提升的 88–110%。在无参考的任务上，我们记录了被外部评判器抓住的代理指标博弈，由新增的一个检测器加以修复，以及对审计者的审计表明：外部评判器需要任务契约。

图 1：什么在演化。中央：在 MBPP+ 上被选中的验证器，逐字给出，它是跨越三个成本层级的有类型检测器之上的一条表达式，并给出每个叶节点对一次留出尝试的裁决。把失败类别指名，而不是发出一个标量，这才把一次被评分的尝试变成可用的失败胶囊。左侧：前提，没有神谕验证器，以及十个锚定样例。右侧：消费者，一个受生命周期管理的技能循环 [1]，原样采用、不做改动，其训练由演化出的验证器来评分。底部：锚点护栏。

## 2 相关工作

#### 自我演化的 agent 假定验证器已经存在。

agent 生长技能库，依据失败轨迹做修订，并把可复用的经验存成洞见、手册、记忆或工作流 [2, 3, 8, 9, 10, 11, 12]；最近的一波工作演化、分析并基准测试这样的库 [13, 14, 15, 16, 17, 18, 19]，两篇综述则勾勒了这一版图 [20, 21]。Ratchet [1, 22] 表明，不受管理的、由 LLM 撰写的库会发生漂移，而一套最小的生命周期能把收益挽回；那个已发表的循环在此被原样采用，作为技能侧的组件，并非本工作的贡献。另一些路线在 agent 代码上搜索 [23]，继承架构搜索的传统 [24]，或对着某个指标优化提示词与流水线 [25, 26, 27, 28]。它们全都*对着*一个被视为给定的验证信号做优化。我们演化的是那个信号，并追问：一旦评分器本身开始移动，纪律之中的哪一部分承担安全载荷。

#### 被搜索的目标由一个已经存在的指标来评分。

已有工作对着所提供的评估器，演化损失函数 [29]、完整的学习算法 [30]、符号剪枝指标 [31] 以及程序代码 [32]。搜索空间是类似的，都是把原始检查组合成一段程序，但适应度在种类上不同：每一项都由一个被视为有效的指标来评分，而 AlphaEvolve 更直截了当地声明，进展必须是机器可评分的，这正是我们这一体制所缺少的前提。我们补上缺失的立足点：用锚定一致率加上共识正则，来代替下游分数。

#### 学得的评分器以不透明换掉那个假设。

即便在经典强化学习中，奖励设计也很微妙 [33]；RLHF 的奖励模型会被它们所训练的策略过度优化，而设定错误会产生系统性的博弈 [34, 7, 35, 36, 37]；AI 反馈与自我奖励的设定让模型评判它自己 [38, 39]；LLM 评判器的偏差也已被广泛审计 [40, 41, 42, 43]。我们分担奖励黑客这一风险，但不采用那种结构：我们的评分器是有类型检测器的一个组合，被优化来预测一个它从不在其上训练的锚点，因此它可以被阅读并手工修复，而且它面向 agent 的使用被限制在训练之中。以 agent 充当评判器的协同演化 [44] 在精神上最为接近，它通过自己所评分的那个系统来验证评判器，而 §5.2 表明，一个空洞的评分器也能通过这种验证。与我们的任务最接近的评估器，预先固定了我们所合成、门控并退役的东西：CodeScore [45] 从我们视为缺失的、带执行标签的监督之中预测功能正确性，SQLHD [46] 不需要真值，但固定了一族手写的蜕变关系 [47]。二者都可以成为我们池中的一个有类型检测器，因此我们把它们读作组件，而不是竞争者；与此同时，agent 式的 text-to-SQL 流水线 [48] 改进的是求解器，那是一条正交的轴。我们使用 MBPP+ [49, 50] 与 Spider 2.0 [51] 作为认证演化验证器的硬锚点，沿袭 HumanEval [52] 那种以执行来评分的传统，而不是把它们当作排行榜条目。

## 3 验证器及其演化

设一个任务族带有任务 $t$，以及一个把 $t$ 映射到输出 $y$ 的冻结求解器。验证器 $M$ 把 $(t,y)$ 映射到 $\{\textsc{pass},\textsc{fail},\textsc{abstain}\}$ 中的一个裁决，而 $M$ 就是我们搜索的对象，任务与参考在此期间保持固定。三个互不相交的划分各自扮演通常的角色，并各带一处扭转。train 规模大且无标签：这里的求解器输出暴露候选验证器在何处不一致或弃权，而在协同演化中，演化出的验证器为技能循环对这些任务评分。dev 很小且被锚定，全程为十项：每个任务携带一份黄金参考，教师对照它发出软的通过/失败标签，这是验证器演化所读取的唯一监督信号。这些参考只给携带它们的那十个任务评分，而演化出的验证器必须给那些没有任何参考的任务上的任意输出评分。dev 被故意做得很小，因为稀疏锚定才是现实的体制：在已部署的系统中，新的标签以演示与事故复盘的涓流形式到来。test 对验证器是锁定的：它持有最强的可用参考（单元测试、官方的执行比较、报告量规），验证器演化的任何部分都不读取它，因此下面每一个有效性数字都完全在样本之外。只有技能循环接触它，用于留出评分与回滚，其方式由 Ratchet [1] 规定并保持不变，因为一个已部署的 agent 是对着它所受评判的那些案例来做回滚的。参考循环与协同循环以相同的方式回滚，因此承载我们这一主张的保留率比较保持干净，而且回滚在它们的 2,900 轮之中只触发了 23 次，提示直接暴露有限；尽管如此，技能侧的绝对水平，相对于一个从未被触碰的第三划分，仍然是乐观的。naive 消融把回滚关掉，因此它的技能侧受到的保护更少，而不是更多。

### 3.1 作为缺陷检测器之组合的验证器

一个 *op* 是原子缺陷检测器 $o(t,y,c)\rightarrow\{\textsc{drawback},\textsc{clean},\textsc{abstain}\}$，其中 $c$ 为共享上下文；它恰好检查一个失败类别，否则弃权。每个任务族从少量检测器起步，这些检测器覆盖其明显的失败模式，用手来写很便宜，也可以让 LLM 根据任务规格起草，生命周期再从那里扩大检测器池。op 分成三个成本层级：静态 op 解析并检查产物；执行 op 运行它，对代码是对着沙箱，对 SQL 是对着活的数据仓库，因此环境本身供给一部分真值；评判 op 向 LLM 提出一个狭窄的问题。每个任务族有一个固定的、不演化的 *root*，它校验基本结构，并缓存其他 op 所读取的已解析产物。

一条*验证器表达式*用逻辑算子组合各 op 的裁决：一个节点可以是叶 op、析取（任一子节点发现缺陷即为缺陷）、合取（仅当全部子节点一致）、否定、不加权的 $K$-of-$k$ 投票，或相对某个阈值的加权投票（附录 C 给出文法）。弃权的子节点被排除出它们的组合子；一个其全部子节点都弃权的节点，自身也弃权；当固定的 root 与演化出的表达式合取之后未发现缺陷时，输出通过。op 是 $(t,y)$ 的纯函数，因此裁决被缓存，评估退化为查表。否定与加权投票在每一轮都可用，但在任何一个被选中的验证器里都没有出现，因此我们报告的每一条表达式都读作普通的缺陷析取或投票，由其表达式字符串加上已注册的 op 池即可完全复现。

### 3.2 验证器循环

算法 1 勾勒一轮。两路信号供给合成：*misses*，即现任验证器判为通过、但其软标签为失败的 dev 条目；以及 *gaps*，即池完全弃权或发生分裂的 train 输出。二者都在一个封闭的、任务特定的分类体系之下按失败类别聚类，合成器为任何一个反复出现的簇撰写一份有类型的 op 规格，只有通过出生门才被接纳。由锚定的 misses 诞生的 op 进入*活跃*并可被选择，其诞生证据已经把它们系到锚点上；仅由无标签 gaps 诞生的 op 以*影子*进入，每一轮都记录裁决，但在其裁决确能提高 dev 一致率之前不可被选择，因为尚无任何东西表明它们追踪的是质量，而不是共识噪声。候选随后来自一个 LLM 组合器，它看得到 op 目录、当前的 gaps，以及最佳表达式连同其短处的精英历史，再加上这些精英的变异与交叉。选择按下式保留迄今最佳的候选

$$
S(e)=A_{\mathrm{dev}}(e)\cdot A_{\mathrm{train}}(e)^{w}-\lambda\,C(e), \tag{1}
$$

其中 $A_{\mathrm{dev}}(e)$ 是与 dev 软标签的一致率，$A_{\mathrm{train}}(e)$ 是与那些发表了意见的池中 op 在无标签 train 输出上的共识裁决的一致率，$C(e)$ 是表达式的大小，$w,\lambda$ 是小常数。两项一致率都经过加权，而每一种加权都是一项安全选择，而不是一个调节旋钮（常数见附录 C）。*dev 一致率按召回加权*，漏掉一项缺陷的代价是一次误报的两倍，因此没有任何候选能够单靠在较容易的那一类上得高分就达到满分。*共识按可靠性加权*，权重是每个 op 在锚点上的留一边际，因此被锚点证实过的 op 主导投票；否则，一团彼此相关的检测器——合成起来很便宜，又容易共享一个盲点——就会去定义正则项所奖励去与之同意的那个伪标签本身。于是 $A_{\mathrm{train}}$ 是一个被拉向锚点的迁移正则项，而不是一次人气投票；它以乘法形式对着一个失败关闭的 $A_{\mathrm{dev}}$ 进入，因而永远不会独自选出一个验证器：一个记住了十个 dev 条目、但行为紊乱的验证器会落选，一个退化的、一律失败的验证器会在 dev 上落选。两道护栏生效：选择是失败关闭的，因此一个在 dev 上没有可用意见的候选不可被选择；一道有效性门丢弃那些对一切都通过、都失败或都弃权的候选。逐个 op 的适应度是现任表达式的留一边际，非正的 op 在一段宽限期之后退役。

**算法 1**　验证器演化的一轮

1. 感知（验证器错在何处？）
2. misses $\leftarrow$ 现任验证器判为通过、但软标签为失败的 dev 条目
3. gaps $\leftarrow$ 池弃权或分裂之处的 train 输出
4. 生长（扩大检测器池）
5. 按失败类别对 misses $\cup$ gaps 聚类；为每个反复出现的簇撰写一个有类型的 op
6. 出生门：在其簇的至少一半上触发，并且在已知良好的样本上为干净 $\Rightarrow$ 接纳（若来自锚点则为活跃，否则为影子）
7. 选择（重组，保留最佳）
8. candidates $\leftarrow$ 在该池之上的 LLM 组合，加上精英的变异与交叉
9. 在有效性门之后，于候选与现任者之中保留 $\arg\max_{e} S(e)$（式 1）
10. 策展（基于功绩的生命周期）
11. 退役留一边际为非正的 op；提升那些能提高 dev 一致率的影子
12. 审计（只做测量，绝不是训练信号）
13. 报告相对于锁定测试集的一致率

### 3.3 与技能循环的协同演化：Double Ratchet

此处原样采用的 Ratchet 技能循环 [1]，需要的正是本设定所不提供的那一件东西：为其训练任务提供一个评分器。协同演化把当前最佳的演化验证器交给它，我们把这一配对系统称为 *Double Ratchet*。两个循环在一份固定的、前重的课程下交替：验证器阶段在覆盖最差的早期最长，并随着验证器成熟而缩短。在技能阶段，演化出的验证器给 train 上的尝试评分，失败的尝试变成胶囊，其错误文本供给技能合成。技能循环的留出评估与回滚锚点始终钉在锁定测试集上，而演化出的验证器从不读取该集合，因此一个被腐蚀的验证器可以放慢技能学习，但不能重新定义评估指标。要紧的比较对象是*参考*技能循环：同一套机制，免费拿到真值验证器（或最佳可用量规）。

#### 两项消融，每种安全成分一项。

naive 臂禁用两道锚点护栏（失败关闭的锚定，以及有效性门），并在技能侧禁用技能循环的回滚，同时检测器的生命周期完全开启；这正是无锚点的 agent–评判器协同演化所隐含运行的、易于共谋的配置。无生命周期臂正好相反：保留护栏，但禁用出生门、影子层与基于功绩的退役，于是每一个被合成的检测器都进入活跃，且没有任何东西退役。这是不受管理的技能库在验证器一侧的对应物。

#### 一次外部审计。

在不存在准确验证器的地方，量规分数是参考而不是真理，因此一次最终审计坐在所有循环之外：一个更强的 LLM 把每一份最终输出与演化前的基线成对比较，交换位置之后再评判一次，只有两种次序都同意时才计为一次胜利（附录 D）。这模仿了真实部署所保留的人工验收步骤，并且它暴露了各循环看不见的一次代理指标博弈的失败。它是审计，绝不是训练信号。

## 4 实验设置

#### 任务领域。

MBPP+ [49, 50] 是以隐藏单元测试作为硬锚点的 Python 函数合成，Spider 2.0-Snow [51] 是在真实 Snowflake 仓库之上、以官方执行结果比较为锚的企业级 text-to-SQL；二者是验证任务：它们精确的真值，使验证器有效性以及与参考循环的匹配都可测量。报告生成是目标任务，一种部署风格的工作负载，其中一条流水线根据经审计的证据卡片撰写分析师报告的各个章节。对它不存在黄金验证器；可用的量规分数加上黄金演示章节，是部分参考信号，不是真值。任务卡片见附录 A。

#### 困难子集与划分。

对 MBPP+ 与 Spider 2.0，我们用五次独立的冻结求解器采样筛选每一个任务，丢掉那些在每一次采样上都通过的任务（没有提升余地），并按筛选通过率对其余任务分层。报告任务派生自黄金演示，无需筛选。dev 始终持有十项：五个明确的失败，以及五个干净的正例。求解器是一次直接的 LLM 调用，没有工具、迭代或脚手架，因此绝对数字不能与排行榜上的 agent 相比较。附录 B 给出这些划分、这些锚点，以及 Spider 提示词的接地；后者是一项很大的共享效应，以相同方式施加于每一条臂。

#### 角色与模型。

全部实验使用 Claude 模型：每一个环内角色（求解器、技能合成器、教师、评判 op）都是 Claude Opus 4.7，唯一的例外是报告任务的最终评判器，即更强的 Claude Opus 4.8，它被保持在所有循环之外。同族的教师并不是一个循环的自我评分器：它是在 dev 上把候选输出与黄金参考相比较，而不是在没有辅助的情况下评判，并且从不看见锁定锚点。技能循环运行 100 轮；协同演化在四个各为 25 轮的技能阶段之间，交错插入 15/8/5/2 轮的验证器阶段，与参考循环的技能预算相匹配。每一次运行都设定了种子，并由配置驱动（附录 G）。

## 5 结果

两条轴把证据组织起来，并按该顺序取用。首先是*验证器有效性*，即与一份锁定参考的一致率，而验证器演化的任何部分都不读取该参考；护栏消融随后把这一主张定位出来。然后是*任务分数*；我们表明它不能认证一个验证器，因此只用来询问：一个演化出的验证器是否足以代替真值。每次运行使用三个种子，报告为均值与样本标准差；循环之间的比较另加种子级的差值以及 95% 自助法区间。这些均值是粗的，因此有效性主张还在相互匹配的逐项裁决上做*配对*检验：演化对同一批锁定条目重新评分，于是两轮之差只在评分器，而检验所问的是评分器的这一改变，而不是任何任务层面的总体效应。这些匹配对是成簇的，三个种子覆盖同一批锁定任务，因此在合并后的配对上做精确的 McNemar 检验会反保守，所报告的检验是任务层面的符号翻转置换。逐轮噪声大约在五个百分点附近，因此我们对照每次运行的第 0 轮基线，报告峰值与末段轮次的值（最后十轮的均值）；峰值是锁定集上的 argmax，因而偏乐观，而末段轮次这一列是事先固定、未被挑选的，这也是配对检验在两处都进行的原因。

### 5.1 演化出的验证器胜过它起步时的那套工程

在 MBPP+ 上，演化出的组合达到 $0.625\pm 0.050$ 的留出一致率，对照它起步时手写种子组合的 $0.417\pm 0.058$，因此这 $+0.21$ 是相对于静态工程的提升，而不是相对于空白的提升。它也胜过它所包含的那个赤裸 LLM 评判 op（$0.55\pm 0.04$），后者是在没有真值的情况下构建评分器时的自然基线；而且与那个评判器不同，它是可检查的、可诊断的。配对检验使这一提升在方向上一致，但并不显著：在 120 个匹配裁决之中，演化在峰值处纠正 61 项、弄坏 36 项，在末段轮次纠正 36 项、弄坏 20 项，且每一个种子的净值都为正；然而把这些裁决按任务成簇之后，峰值处 $p=0.12$，末段轮次处 $p=0.21$。若把它们当作独立样本来合并，则会报告 $p=0.014$ 与 $p=0.044$：在 40 个锁定任务与三个种子之下，这一设计支持的是一个一致的方向，而不是一次经过校准的拒绝。

图 2 分开了两个容易被混淆的量。选择所优化的那个量，在每个任务的每个种子上都上升，并且此后从不下降；这是迄今最佳的选择所保证的，因此并不是有效性的证据；只有向选择从未见过的那个锁定集的迁移，才提供信息。迁移也并不单调，尽管目标本身是单调的，因为九次运行之中有五次结束时低于它们自己的中途峰值，这是优化一个十项代理时寻常的泛化差距。

另外两个任务并不是对 MBPP+ 这一主张的重复。报告生成从天花板起步，因为子集播种已经把一个接近最优的组合放进了第 0 轮的候选集，因此它承载的是充分性与安全消融，而不是提升。Spider 处在随机水平附近（$0.500\pm 0.026$），尽管目标在稳健地攀升，配对检验在那里没有发现任何东西；§5.5 表明，这是该任务的失败体制，而不是样本量造成的假象，§5.3 则说明为何不完美的一致率并不使这个系统注定失败。

生命周期在短小的表达式背后做了真正的工作。每个池从 9 到 11 个手写的种子检测器起步，由缺口驱动的合成随后持续地提议，出生门拒绝其中压倒性的多数，留下一个由十几个到几十个组成的工作池，锚定选择从中保留一到三个叶节点。一个 Spider 检测器展示了整条路径：循环把那些尽管问题点名了特定的列、却返回每一列的 train 输出聚到一起，撰写了有类型的规格 `forbids_select_star`，因为它的诞生证据是一个无标签缺口，所以把它作为影子接纳；在把它组合进去、使 dev 一致率从 0.71 提高到 0.81 之后将其提升，然后选中它，以取代种子池中的分组检查（附录 E）。十项锚点也并不脆弱：对着四项、六项与八项的锚点重放报告任务上的选择，并不降低平均留出一致率，只是加宽了它的散布（附录 F）。

图 2：验证器有效性，把选择看得到的与选择看不到的分开。每个面板的左侧：100 轮之中前 70 轮上的选择目标（三种子均值）。右侧：与锁定锚点的留出一致率，从手写的第 0 轮组合到演化峰值（均值 $\pm$ 标准差，三个种子），对照组合之中所包含的赤裸 LLM 评判 op，以及随机水平线。底部：出生门的漏斗，三种子的范围。图 3：同样的三条臂，用两种方式来测量。左侧：报告生成上的验证器有效性，三个种子。naive 臂声称目标完美，同时对一切的通过率为 0.97–1.00，因此它的留出一致率是失败开放的、并且不可测量，而不只是偏低。右侧：参考循环、Double Ratchet 与 naive 臂的留出任务分数；这一侧在任何一个任务上都分不开这三者。

### 5.2 哪一道护栏承担载荷，以及为何任务分数无法说明

消融是按验证器有效性来评判的，而不是按留出任务分数，图 3 说明为何这一区分并不是咬文嚼字。在报告生成上，锚点最软、共谋压力最高，禁用锚点护栏使验证器在三个种子中的全部三个上都发生坍缩：选择咬住一个几乎从不触发的检测器，失败开放的评分报告出一个空洞的完美目标，而验证器对其所评分的一切给出 0.97–1.00 的通过。禁用生命周期则没有任何东西坍缩：目标与留出一致率停留在锚定带之内；尽管池在增长，多出来的检测器是惰性的，因为一个从未被选中的检测器从不给任何东西评分。

这颠倒了技能演化文献的侧重。对技能而言，生命周期管理是关键发现；对处在锚定选择之下的验证器而言，承担安全载荷的是锚点纪律，因此库漂移在验证器一侧的对应物是*锚点*漂移，而不是池漂移。这种不对称是结构性的：一条无用的技能会被送进提示词并主动造成伤害，而一个无用的检测器只有在选择挑中它时才要紧，锚点阻止了这种挑中。生命周期仍然换来效率，我们保留它。

坍缩的那条臂携带着一个比本文更长久的警告。它的验证器毫无价值，然而它的协同循环得分与锚定的那一条一样好，并且在两个任务上略好一些（图 3，右；表 1）。两个数字都没有错：一个一律通过的验证器把训练还原成未经过滤的练习，这本身就会改进技能，而测量从不经过验证器，因此任务分数在评分器所不能成立的地方仍然成立。下游任务表现因此不能用来验证一个自我演化的验证器；而这篇文献会最先伸手去做的那个实验，在这里将会认证一个对一切都放行的评分器。要验证这个验证器，需要一份它从不读取的留出参考；任何以验证器做分诊或门控的部署，反正都需要这样一份参考。

### 5.3 充分性：演化出的验证器可以代替真值

在有效性被独立测量之后，剩下的问题是充分性：一个从十个参考样例演化出来的验证器，能否驱动一个否则将由真值来驱动的技能循环？它可以替代，这一主张弱于“是它造成了增益”：§5.2 已经表明，被保留下来的提升并不能把验证器的准确度分离出来。Double Ratchet 在 MBPP+、Spider 与报告任务上，分别保留参考循环之提升的 106%、110% 与 88%（表 1；曲线见图 4）：在两个有真值的任务上处于噪声之内，其 $\Delta$peak 区间包含零；在报告任务上则有一个小缺口，而该任务的参考本身就是一份部分量规。这就是为何 §5.4 中的独立评判器，在那里是更有信息量的读法。

合计数字掩盖了一个任务级的结果。两个循环都解出了冻结求解器*从不*解出的留出问题，并且在两个硬锚点任务上，Double Ratchet 转化出的此类问题比真值驱动的循环*更多*（表 1，已改进）。由此可以确定，一个验证器要在这里做到充分，需要有多好：并不需要很好。Spider 的验证器与真值的一致率只有 $0.500\pm 0.026$，然而它的协同循环保留了全部的参考提升，因为验证器在训练中的角色是*方向性的*：它决定哪些尝试成为失败胶囊，而这些胶囊的具体错误文本并不依赖于通过/失败标签是否正确。由验证器评分的训练通过率停留在一个现实的区间里（报告任务上为 0.78–0.82，Spider 上为 0.41–0.83），因此验证器确实在评分。参考循环与协同循环彼此从不可见，却在每一个任务上独立地合成出功能上相匹配的技能（附录 E）。

诚实的读法是：技能循环是分数的*宽容*消费者，而 Spider 上两个循环最小的绝对提升，反映的是同一道把二者一并约束住的天花板：单次调用的求解器，其残余失败大多需要现场探索，而没有任何指导文本形式的技能能够表达这种探索。因此，把验证器造好的理由在别处，在分诊、门控与可修复性之中；其中最后一项，由下一小节放在载荷之下。

表 1：在 40–48 项的锁定集上、三个种子的留出任务分数。*reference*：同一个技能循环，免费拿到真值（MBPP+、Spider）或最佳可用量规（报告任务）。*co*：Double Ratchet，使用它自己演化出的验证器。*naive*†：关掉锚点护栏与技能侧回滚的消融，与 *co* 按种子匹配；它的验证器是空洞的（表 3），然而它的得分与 *co* 一样好，这正是任务分数不能认证一个评分器的原因。*提升保留率*：co 相对于其第 0 轮基线的峰值提升，占参考循环之提升的比例。*验证器一致率*：与锁定锚点的峰值一致率。*已改进*：至少被解出一次的、此前从未通过的任务；对报告任务，则为结束时高于第 0 轮的章节所占比例。前三列数值为峰值留出分数。

| 任务 | reference 峰值留出分数 | co 峰值留出分数 | naive† 峰值留出分数 | 提升保留率 | $\Delta$peak（95% CI） | 验证器一致率 | reference 已改进 | co 已改进 |
|---|---|---|---|---|---|---|---|---|
| MBPP+ | $0.700\pm.025$ | $0.717\pm.038$ | $0.742\pm.014$ | 106% | $+.02\,[-.03,.06]$ | 0.62 | 16/23 | 19/23 |
| Spider | $0.483\pm.038$ | $0.458\pm.038$ | $0.458\pm.029$ | 110% | $-.03\,[-.08,.03]$ | 0.50 | 4/12 | 6/12 |
| 报告 | $0.850\pm.010$ | $0.812\pm.006$ | $0.841\pm.003$ | 88% | $-.04\,[-.05,-.03]$ | 0.84 | 99% | 99% |

### 5.4 一场奖励黑客，以及对审计者的审计

报告任务给出了本文最清楚的证据，说明一个可检查的验证器值得拥有。量规的指标纪律维度统计的是行内证据标签，而最初几轮技能运行把量规分数提高了 $+0.26$，部分正是靠博弈这一点：演化出的技能把标签写在数字*的位置上*，使得在峰值轮次大约有 30% 的标签旁边没有数值，并且编造出自信的预测，以满足某个风格维度。这是教科书式的代理指标博弈，而且没有任何环内信号看见它。独立评判器看见了：在已决的配对之中，它有 88% 更偏好演化前的基线，并描述了未经渲染的占位符与捏造出来的确信。

修复是一次耦合的改动：在胶囊门上加入一个感知词表的数值擦除检查——若某个标签所登记的指标具有数值，但近旁没有数字，即视为缺陷——再加上经过改写的失败提示，这些提示教导“数值加标签”；而提示正是一个检测器的裁决抵达技能合成的途径。我们主张的是这一对，而不是检测器单独，因为把它们分开需要我们并未做的重跑。这次重跑把被擦除的标签从大约 30% 削减到约 1%，同时量规分数*上升*到 $0.850\pm 0.010$，并没有为此牺牲任何东西。阅读验证器，才把缺陷定位到一个可以指名的检查；这是一个学得的标量所不允许的干预。

于是审计本身需要被审计，这是可扩展监督问题的一个具体实例。流水线要求其输出之中带有原始的证据标签，这是原来的通用量规所不知道的格式契约，因此它的失分理由责怪的是那套句法，而修复几乎没有推动胜率，只从 0.122 到 0.126。一份陈述该契约的、任务感知的量规，同时仍把无数值的标签与捏造的数字视为缺陷，对同一批已存储的配对重新做了评判。表 2 把这两层分开：沿着任务感知这一列往下，擦除修复把胜率从 0.515 移动到 0.770，这是契约使之可见的内容变化；而每一行从通用到任务感知的跳跃，度量的是通用评判器的约定盲视。那份量规是在读过通用评判器的理由之后写成的，因此它诊断的是这次审计，而不是独立地确认这次修复；它的修复前单元格是对照。教训是双向的：自我演化的验证器需要一次独立的外部审计，而这次审计需要任务契约。

表 2：演化后的报告输出相对于其自身演化前基线的最终评判胜率，统计范围是已决配对；每一对都在交换位置之后被评判两次，一次胜利要求两种次序都同意（协议与计数见附录 D）。各行把单检测器的擦除修复隔离出来，各列把评判器的约定盲视隔离出来：只有任务感知的评判器登记到这一修复（$+0.255$）。

| 报告循环状态 | 外部评判器：通用 | 外部评判器：任务感知 | 量规的效应 | 该行所示 |
|---|---|---|---|---|
| 修复前（代理指标被博弈） | 0.122 | 0.515 | $+0.393$ | 只有感知评判器能看穿句法 |
| 修复后（擦除已修复） | 0.126 | 0.770 | $+0.644$ | 通用评判器看不见的内容增益 |
| *修复的效应* | $+0.004$ | $+0.255$ |  | 通用评判器惩罚了所要求的格式 |

### 5.5 可检测性谱

有一个变量把上面的每一个结果组织起来：求解器的失败在机械上有多可检测。MBPP+ 的失败是崩溃与错误输出，廉价的检测器可以直接看见，留出一致率提升 $+0.21$。报告生成经由一个封闭的、可检查的失败分类体系，从黄金演示导出其锚点，迁移达到 0.84。Spider 2.0 给出一个任务内部的受控对照：在我们的第一份提示词之下，大多数失败是编译错误，演化出的验证器达到 0.85 的留出一致率。随后用带类型的模式为提示词做接地，把求解器的基线提高了一半（从 0.211 到 0.329），并留下*残余*失败：它们是在干净执行之下语义错误的值，确定型检测器看不见它们，因此即便目标仍在攀升，一致率也下降到 $0.500\pm 0.026$。同一方法，同一任务，不同的失败分布。这一体制把负担转移到评判 op 与人工审计之上，而更锐利的求解器会留下更多这样的负担：验证器演化在失败可被检查之处买到的最多，这是一条范围条件，而不是实现上的限度。

## 6 讨论与结论

演化扩大覆盖，但从不创造真值。没有锚点时，循环优化的是噪声，因此锚点的*质量*与它的存在同样重要，而把软锚点硬化为可检查的锚点，收益最大。开放的前沿，是那些执行干净、语义却错误的输出。这是一项机制研究，而不是一项缩放结果：三个任务族，一个模型族，轮数处于低数百，留出集为数十项，以及用一个更强的模型充当外部评判器，而不是一个评审小组或一项人类研究。

一个缺少验证器的 agent，可以在那套使其技能得以稳定的纪律之下演化出一个验证器，其安全是被设计出来的，而不是被假定的。三项承诺承载了我们的结果。*锚定你无法制造的东西*：把选择系到一个循环不能编辑的小参考集上，绝不要系到 agent 自己的分数上。*保持验证器可读*：大多为确定型的检测器，其失败方式不同于它们所评分的模型，而一个被指名的检查修复了一起正在发生的代理指标博弈。*审计审计者*：外部评判器是必要的，但并不充分，而我们的评判器在拿到任务契约之前是错的。绝不要用 agent 的表现来验证一个自我演化的验证器：一个对一切都放行的评分器，也能通过那项检验。谁来验证验证器？一个它会去预测、但从未看见的锚点，以及处在每一个循环之外的一个评判器。

## 参考文献

  * Zhang et al. [2026a] Xing Zhang, Yanwei Cui, Guanghui Wang, Ziyuan Li, Wei Qiu, Bing Zhu, and Peiyang He.  Ratchet: A minimal hygiene recipe for self-evolving LLM agents.  _arXiv preprint arXiv:2605.22148_ , 2026a.
  * Wang et al. [2024a] Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, and Anima Anandkumar.  Voyager: An open-ended embodied agent with large language models.  _Transactions on Machine Learning Research_ , 2024a.
  * Shinn et al. [2023] Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao.  Reflexion: Language agents with verbal reinforcement learning.  In _Advances in Neural Information Processing Systems_ , volume 36, 2023.
  * Anthropic [2026] Anthropic.  New in claude managed agents: Dreaming, outcomes, and multiagent orchestration.  <https://claude.com/blog/new-in-claude-managed-agents>, 2026.  Product announcement, May 19, 2026.
  * OpenAI [2026] OpenAI.  Codex: Record & replay.  <https://developers.openai.com/codex/record-and-replay>, 2026.  Developer documentation.
  * Goodhart [1984] Charles A. E. Goodhart.  Problems of monetary management: The UK experience.  _Monetary Theory and Practice: The UK Experience_ , pages 91–121, 1984.
  * Gao et al. [2023] Leo Gao, John Schulman, and Jacob Hilton.  Scaling laws for reward model overoptimization.  In _Proceedings of the 40th International Conference on Machine Learning (ICML)_ , pages 10835–10866, 2023.
  * Madaan et al. [2023] Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, et al.  Self-refine: Iterative refinement with self-feedback.  _Advances in Neural Information Processing Systems_ , 36, 2023.
  * Zhao et al. [2024] Andrew Zhao, Daniel Huang, Quentin Xu, Matthieu Lin, Yong-Jin Liu, and Gao Huang.  ExpeL: LLM agents are experiential learners.  In _Proceedings of the AAAI Conference on Artificial Intelligence_ , 2024.
  * Chen et al. [2024a] Minghao Chen, Yihang Li, Yanting Yang, Shiyu Yu, Binbin Lin, and Xiaofei He.  AutoManual: Generating instruction manuals by LLM agents via interactive environmental learning.  In _Advances in Neural Information Processing Systems_ , volume 37, 2024a.
  * Park et al. [2023] Joon Sung Park, Joseph C. O’Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein.  Generative agents: Interactive simulacra of human behavior.  In _Proceedings of the 36th Annual ACM Symposium on User Interface Software and Technology_ , 2023.
  * Wang et al. [2024b] Zora Zhiruo Wang, Jiayuan Mao, Daniel Fried, and Graham Neubig.  Agent workflow memory.  _arXiv preprint arXiv:2409.07429_ , 2024b.
  * Wu et al. [2025] Rong Wu, Xiaoman Wang, Jianbiao Mei, Pinlong Cai, Daocheng Fu, Cheng Yang, Licheng Wen, Xuemeng Yang, Yufan Shen, Yuxin Wang, et al.  Self-evolving LLM agents through an experience-driven lifecycle.  _arXiv preprint arXiv:2510.16079_ , 2025.
  * Yang et al. [2026] Yutao Yang, Junsong Li, Qianjun Pan, Bihao Zhan, Yuxuan Cai, Lin Du, Jie Zhou, Kai Chen, Qin Chen, Xin Li, et al.  AutoSkill: Experience-driven lifelong learning via skill self-evolution.  _arXiv preprint arXiv:2603.01145_ , 2026.
  * Xia et al. [2026] Peng Xia, Jianwen Chen, Hanyang Wang, Jiaqi Liu, Kaide Zeng, Yu Wang, Siwei Han, Yiyang Zhou, Xujiang Zhao, Haifeng Chen, et al.  SkillRL: Evolving agents via recursive skill-augmented reinforcement learning.  _arXiv preprint arXiv:2602.08234_ , 2026.
  * Ni et al. [2026] Jingwei Ni, Yihao Liu, Xinpeng Liu, Yutao Sun, Mengyu Zhou, Pengyu Cheng, Dexin Wang, Xiaoxi Jiang, and Guanjun Jiang.  Trace2Skill: Parallel inductive skill distillation for LLM agents.  _arXiv preprint arXiv:2603.25158_ , 2026.
  * Wang et al. [2026] Junjie Wang, Yiming Ren, and Haoyang Zhang.  From procedural skills to strategy genes: Towards experience-driven test-time evolution.  _arXiv preprint arXiv:2604.15097_ , 2026.
  * Zhang et al. [2026b] Xing Zhang, Guanghui Wang, Yanwei Cui, Wei Qiu, Ziyuan Li, Bing Zhu, and Peiyang He.  Experience compression spectrum: Unifying memory, skills, and rules in LLM agents.  _arXiv preprint arXiv:2604.15877_ , 2026b.
  * Li et al. [2026] Xiangyi Li, Wenbo Chen, Yimin Liu, Shenghan Zheng, Xiaokun Chen, Yifeng He, Yubo Li, Bingran You, Haotian Shen, Jiankai Sun, et al.  SkillsBench: Benchmarking how well agent skills work across diverse tasks.  _arXiv preprint arXiv:2602.12670_ , 2026.
  * Zhang et al. [2025] Jinyuan Zhang, Changzhou Qu, Junyi Fang, et al.  A survey of self-evolving agents: On path to artificial super intelligence.  _arXiv preprint arXiv:2507.21046_ , 2025.
  * Wang et al. [2024c] Lei Wang, Chen Ma, Xueyang Feng, Zeyu Zhang, Hao Yang, Jingsen Zhang, Zhiyuan Chen, Jiakai Tang, Xu Chen, Yankai Lin, Wayne Xin Zhao, Zhewei Wei, and Jirong Wen.  A comprehensive survey of LLM-based autonomous agents.  _Frontiers of Computer Science_ , 18(6), 2024c.
  * Zhang et al. [2026c] Xing Zhang, Yanwei Cui, Guanghui Wang, Ziyuan Li, Wei Qiu, Bing Zhu, and Peiyang He.  Library drift: Diagnosing and fixing a silent failure mode in self-evolving LLM skill libraries.  In _ICML 2026 Workshop on Failure Modes in Agentic AI_ , 2026c.
  * Hu et al. [2025] Shengran Hu, Cong Lu, and Jeff Clune.  Automated design of agentic systems.  In _International Conference on Learning Representations (ICLR)_ , 2025.
  * Elsken et al. [2019] Thomas Elsken, Jan Hendrik Metzen, and Frank Hutter.  Neural architecture search: A survey.  _Journal of Machine Learning Research_ , 20(55):1–21, 2019.
  * Khattab et al. [2024] Omar Khattab, Arnav Singhvi, Paridhi Maheshwari, Zhiyuan Zhang, Keshav Santhanam, Sri Vardhamanan, Saiful Haq, Ashutosh Sharma, Thomas T. Joshi, Hanna Moazam, Heather Miller, Matei Zaharia, and Christopher Potts.  DSPy: Compiling declarative language model calls into self-improving pipelines.  In _International Conference on Learning Representations (ICLR)_ , 2024.
  * Yang et al. [2024] Chengrun Yang, Xuezhi Wang, Yifeng Lu, Hanxiao Liu, Quoc V. Le, Denny Zhou, and Xinyun Chen.  Large language models as optimizers.  In _International Conference on Learning Representations (ICLR)_ , 2024.
  * Fernando et al. [2024] Chrisantha Fernando, Dylan Banarse, Henryk Michalewski, Simon Osindero, and Tim Rocktäschel.  Promptbreeder: Self-referential self-improvement via prompt evolution.  In _Proceedings of the 41st International Conference on Machine Learning (ICML)_ , 2024.
  * Yuksekgonul et al. [2024] Mert Yuksekgonul, Federico Bianchi, Joseph Boen, Sheng Liu, Zhi Huang, Carlos Guestrin, and James Zou.  TextGrad: Automatic “differentiation” via text.  _arXiv preprint arXiv:2406.07496_ , 2024.
  * Liu et al. [2021] Peidong Liu, Gengwei Zhang, Bochao Wang, Hang Xu, Xiaodan Liang, Yong Jiang, and Zhenguo Li.  Loss function discovery for object detection via convergence-simulation driven search.  In _International Conference on Learning Representations (ICLR)_ , 2021.
  * Real et al. [2020] Esteban Real, Chen Liang, David So, and Quoc Le.  AutoML-Zero: Evolving machine learning algorithms from scratch.  In _Proceedings of the 37th International Conference on Machine Learning (ICML)_ , pages 8007–8019, 2020.
  * Dong et al. [2024a] Peijie Dong, Lujun Li, Zhenheng Tang, Xiang Liu, Xinglin Pan, Qiang Wang, and Xiaowen Chu.  Pruner-zero: Evolving symbolic pruning metric from scratch for large language models.  In _Proceedings of the 41st International Conference on Machine Learning (ICML)_ , 2024a.
  * Novikov et al. [2025] Alexander Novikov, Ngân Vũ, Marvin Eisenberger, Emilien Dupont, Po-Sen Huang, Adam Zsolt Wagner, Sergey Shirobokov, Borislav Kozlovskii, Francisco J. R. Ruiz, Abbas Mehrabian, M. Pawan Kumar, Abigail See, Swarat Chaudhuri, George Holland, Alex Davies, Sebastian Nowozin, Pushmeet Kohli, and Matej Balog.  AlphaEvolve: A coding agent for scientific and algorithmic discovery.  _arXiv preprint arXiv:2506.13131_ , 2025.
  * Ng et al. [1999] Andrew Y. Ng, Daishi Harada, and Stuart Russell.  Policy invariance under reward transformations: Theory and application to reward shaping.  In _Proceedings of the 16th International Conference on Machine Learning (ICML)_ , 1999.
  * Ouyang et al. [2022] Long Ouyang, Jeffrey Wu, Xu Jiang, Diogo Almeida, Carroll Wainwright, Pamela Mishkin, Chong Zhang, Sandhini Agarwal, Katarina Slama, Alex Ray, John Schulman, Jacob Hilton, Fraser Kelton, Luke Miller, Maddie Simens, Amanda Askell, Peter Welinder, Paul Christiano, Jan Leike, and Ryan Lowe.  Training language models to follow instructions with human feedback.  _Advances in Neural Information Processing Systems_ , 35:27730–27744, 2022.
  * Pan et al. [2022] Alexander Pan, Kush Bhatia, and Jacob Steinhardt.  The effects of reward misspecification: Mapping and mitigating misaligned models.  In _International Conference on Learning Representations (ICLR)_ , 2022.
  * Skalse et al. [2022] Joar Skalse, Nikolaus Howe, Dmitrii Krasheninnikov, and David Krueger.  Defining and characterizing reward gaming.  _Advances in Neural Information Processing Systems_ , 35:9460–9471, 2022.
  * Amodei et al. [2016] Dario Amodei, Chris Olah, Jacob Steinhardt, Paul Christiano, John Schulman, and Dan Mané.  Concrete problems in AI safety.  _arXiv preprint arXiv:1606.06565_ , 2016.
  * Bai et al. [2022] Yuntao Bai, Saurav Kadavath, Sandipan Kundu, Amanda Askell, Jackson Kernion, Andy Jones, Anna Chen, Anna Goldie, Azalia Mirhoseini, Cameron McKinnon, et al.  Constitutional AI: Harmlessness from AI feedback.  _arXiv preprint arXiv:2212.08073_ , 2022.
  * Yuan et al. [2024] Weizhe Yuan, Richard Yuanzhe Pang, Kyunghyun Cho, Xian Li, Sainbayar Sukhbaatar, Jing Xu, and Jason Weston.  Self-rewarding language models.  In _Proceedings of the 41st International Conference on Machine Learning (ICML)_ , 2024.
  * Zheng et al. [2023] Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric P. Xing, Hao Zhang, Joseph E. Gonzalez, and Ion Stoica.  Judging LLM-as-a-judge with MT-bench and chatbot arena.  _Advances in Neural Information Processing Systems_ , 36, 2023.
  * Dubois et al. [2023] Yann Dubois, Xuechen Li, Rohan Taori, Tianyi Zhang, Ishaan Gulrajani, Jimmy Ba, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto.  AlpacaFarm: A simulation framework for methods that learn from human feedback.  _Advances in Neural Information Processing Systems_ , 36, 2023.
  * Gu et al. [2024] Jiawei Gu, Xuhui Jiang, Zhichao Shi, Hexiang Tan, Xuehao Zhai, Chengjin Xu, Wei Li, Yinghan Shen, Shengjie Ma, Honghao Liu, Saizhuo Wang, Kun Zhang, Yuanzhuo Wang, Wen Gao, Lionel Ni, and Jian Guo.  A survey on LLM-as-a-judge.  _arXiv preprint arXiv:2411.15594_ , 2024.
  * Chen et al. [2024b] Guiming Hardy Chen, Shunian Chen, Ziche Liu, Feng Jiang, and Benyou Wang.  Humans or LLMs as the judge? a study on judgement bias.  In _Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing (EMNLP)_ , 2024b.
  * Zhuge et al. [2024] Mingchen Zhuge, Changsheng Zhao, Dylan Ashley, Wenyi Wang, Dmitrii Khizbullin, Yunyang Xiong, Zechun Liu, Ernie Chang, Raghuraman Krishnamoorthi, Yuandong Tian, Yangyang Shi, Vikas Chandra, and Jürgen Schmidhuber.  Agent-as-a-judge: Evaluate agents with agents.  _arXiv preprint arXiv:2410.10934_ , 2024.
  * Dong et al. [2024b] Yihong Dong, Jiazheng Ding, Xue Jiang, Ge Li, Zhuo Li, and Zhi Jin.  CodeScore: Evaluating code generation by learning code execution.  _ACM Transactions on Software Engineering and Methodology_ , 2024b.
  * Yang et al. [2025] Bo Yang, Yinfen Xia, Weisong Sun, and Yang Liu.  Hallucination detection for LLM-based text-to-SQL generation via two-stage metamorphic testing.  _arXiv preprint arXiv:2512.22250_ , 2025.
  * Chen et al. [1998] Tsong Yueh Chen, Shing Chi Cheung, and Shiu Ming Yiu.  Metamorphic testing: A new approach for generating next test cases.  Technical Report HKUST-CS98-01, Hong Kong University of Science and Technology, 1998.
  * Deng et al. [2025] Minghang Deng, Ashwin Ramachandran, Canwen Xu, Lanxiang Hu, Zhewei Yao, Anupam Datta, and Hao Zhang.  ReFoRCE: A text-to-SQL agent with self-refinement, consensus enforcement, and column exploration.  _arXiv preprint arXiv:2502.00675_ , 2025.
  * Austin et al. [2021] Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, and Charles Sutton.  Program synthesis with large language models.  _arXiv preprint arXiv:2108.07732_ , 2021.
  * Liu et al. [2023] Jiawei Liu, Chunqiu Steven Xia, Yuyao Wang, and Lingming Zhang.  Is your code generated by ChatGPT really correct? rigorous evaluation of large language models for code generation.  In _Advances in Neural Information Processing Systems_ , volume 36, 2023.
  * Lei et al. [2025] Fangyu Lei, Jixuan Chen, Yuxiao Ye, Ruisheng Cao, Dongchan Shin, Hongjin Su, Zhaoqing Suo, Hongcheng Gao, Wenjing Hu, Pengcheng Yin, Victor Zhong, Caiming Xiong, Ruoxi Sun, Qian Liu, Sida Wang, and Tao Yu.  Spider 2.0: Evaluating language models on real-world enterprise text-to-SQL workflows.  In _International Conference on Learning Representations (ICLR)_ , 2025.
  * Chen et al. [2021] Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, et al.  Evaluating large language models trained on code.  _arXiv preprint arXiv:2107.03374_ , 2021.

## 附录 A 任务描述

#### MBPP+（验证任务；锚点：隐藏单元测试）。

一道写成一段的自然语言编程题，外加一次示例调用；求解器编写一个 Python 函数。锚点是 EvalPlus 隐藏测试集 [Liu et al., 2023]，它为每道题补充了比原始 MBPP 多得多的测试，并修复了有缺陷的测试，从而给出可靠的通过/失败。典型失败是崩溃、边界情况处理不当、打印而非返回，以及误读规格，这些正是检测器池所检查的失败类别。

#### Spider 2.0-Snow（验证任务；锚点：执行匹配）。

这是针对一个真实托管 Snowflake 数仓的分析师问题：生产规模的数据库，有数百张表、含义隐晦的列、分片表族，以及方言特定的 SQL。求解器根据问题、一份渲染后的模式目录，以及可选的领域注释，写出一条查询。锚点执行候选查询，并用官方的基于值的比较器把取回的表与存储的金标准结果比较，忽略列名和顺序。典型失败是无法解析的标识符、日期类列上的类型错误，以及最难的一类：查询能干净跑完，但算错了东西。我们的单次调用求解器在经筛选的任务上大约通过三分之一。

#### 报告生成（目标任务；不存在准确的验证器）。

给定节标题与目的、一份规定单位、参考日期和会计规则的风格契约、一组经审计的证据卡，以及一份登记该节关键数字的指标词表，用 markdown 撰写分析师报告中的一*节*（例如，「价值链图谱：从 ASIC 到光引擎的控制点」）。一张证据卡带有 id、一句主张、一段逐字的来源引文、来源归属与日期、一种立场，以及审计员的可信度判定。该节必须把每一条主张落实到这些卡片上，用绑定到指标词表的行内标签陈述每一个关键数字，对薄弱或有争议的证据加保留表述，并包含所要求的结构，例如一行摘要。不存在验证器，因为章节质量是整体性的：即便是专家作者，也无法把「好的一节」归结为一条可检查的规则。量规对上述要求的词汇代理与结构代理打分，并且只被当作参考信号。

## 附录 B 设置细节

#### 划分与筛选。

对 MBPP+ 和 Spider 2.0，我们用冻结的单次调用求解器的五个独立样本筛选每一项任务，由官方锚点评分，丢掉求解器总能通过的任务，将其余任务按通过水平分层，然后按 60/40 划分为训练集和测试集。得到的训练/dev/测试规模为：MBPP+ 为 60/10/40，Spider 2.0 为 59/10/40，报告生成为 73/10/48。dev 集为十项，五项明确失败与五项干净正例（报告生成为五项金标准质量的正例，以及五项按类别分层的可检查失败），因此验证器在两个类别上都被评分。

#### 锚点。

*MBPP+*：隐藏单元测试。*Spider 2.0*：在实况数仓上执行，并用官方比较器把取回的表与存储的金标准结果比较；我们复现评估器对取回数据帧的 CSV 往返，否则即便金标准查询也会在 dtype 上失败。*报告生成*：验证器循环的锚点是一个冻结的 demo-delta 软标签，其中教师在一个封闭的七类失误分类体系下，把冻结 agent 的报告与金标准章节比较，仅当至多存在一个失误类别时才通过。48 个留出章节中的每一个贡献两条锁定项，即冻结 agent 的报告（严格锚点几乎总是判其失败）和金标准章节（按定义通过），得到 96 项，分为 54 项通过与 42 项失败，因此一个退化的一律通过验证器得分为 0.56。技能循环与共同循环由官方量规评估器评估（按 0–100 归一化），其维度覆盖证据落地、指标标签规范、主张校准、结构与风格。

#### Spider 2.0 的提示接地。

生成提示用一份紧凑的带类型模式目录（列记为 `name:TYPE`，分片表被折叠，日期类列附有样本值）以及 Snowflake 专用规则，为求解器提供依据。我们报告这一点，是因为它是我们加以控制的一项很大的共享效应，而不是一项贡献：接地把冻结基线在筛选池上提高了约一半（0.211 到 0.329），但它被完全相同地施加于每一个实验臂，因此所报告的全部提升都建立在它之上。它也铺垫了 §5.5 的可检测性对比，因为接地把残余失败从编译错误推进到能够干净执行的错误值。

## 附录 C 表达式文法、选择与生命周期常数

把表达式 $e$ 在 $(t,y)$ 上的判定记为 $v(e)\in\{1,0,\bot\}$，其中 $1$ 表示 drawback，$\bot$ 表示 abstain。§3.1 的表达式文法为

$$
e\mathrel{::=}o\;\big|\;\textstyle\bigvee_{i\leq k}e_{i}\;\big|\;\textstyle\bigwedge_{i\leq k}e_{i}\;\big|\;\neg e\;\big|\;\textstyle\big[\sum_{i\leq k}v(e_{i})\geq K\big]\;\big|\;\textstyle\big[\sum_{i\leq k}w_{i}v(e_{i})\geq\theta\big], \tag{2}
$$

其中 $|$ 分隔可选的节点形式：一个叶 op、一个析取、一个合取、一个否定、一次无权重的 $K$-of-$k$ 投票，或一次相对阈值 $\theta$ 的加权投票。

目标把 dev 一致性乘以未标注训练输出上的共识一致性（MBPP+ 与 Spider 的指数为 1.0，报告生成为 0.25，因为在报告生成上检测器共识是更弱的正则项），再减去一项很小的复杂度惩罚。dev 一致性是两个按类别召回率的按召回加权均值，计算范围是候选发表了意见的那些 dev 项，fail 类权重为 2，clean 类权重为 1。训练共识按可靠性加权：每个给出判定的 op，其投票按 $1+10\max(0,m_{o})$ 缩放，其中 $m_{o}$ 是该 op 在锚点上的留一边际，因此已被锚点证实的 op 主导投票，而没有锚点证据的 op 以单位权重参与；这样就不会让一团廉价合成、彼此相关的检测器去定义那个伪标签，而正则项奖励的正是与该伪标签保持一致。复杂度惩罚对超出三个这一下限的每个 op 计 0.001，并对每个调用 LLM 的 op 再计 0.002，因此被选中的表达式保持很小，而且大多是确定性的。失败关闭式锚定使得没有可用 dev 意见的候选无法被选中，有效性门则剔除全通过、全失败和全弃权的候选。诞生门要求一个新 op 至少在其簇的一半上触发，并在已知良好的输出上保持 clean；由有锚点的失误诞生的 op 进入 active，未标注缺口的 op 进入 shadow，并且只有在展现出与锚点的一致性之后才被提升；每个 op 的适应度是现任表达式的留一边际，非正的 op 经过两轮宽限后退役。naive 消融恰好关掉那两项锚点守卫（以及技能循环的回滚）；no-lifecycle 消融恰好关掉诞生门、shadow 层和退役。

## 附录 D 终审协议与量规

对 48 个留出章节中的每一个，我们把存储的第 0 轮报告与存储的峰值轮报告配对，并另行与最终轮报告配对，覆盖每一个实验臂和每一个种子。修复前，这是 3 个实验臂 $\times$ 3 个种子 $\times$ 2 个轮次 $\times$ 48 个章节 $= 864$ 对；修复后的重跑在两个共同循环实验臂上增加了第四个种子，共计 1,056 对。每一对交换位置各判一次，仅当两种顺序一致时才计为胜，否则记为平局。通用量规提出一个资深编辑式的问题，询问整体的读者价值，并且不知道该流水线的约定。任务感知量规另外陈述格式契约：证据引用和行内指标标签是会在下游渲染的必需输出，附近没有数值的标签是一处缺陷，编造的数字则取消资格。两份量规评判的是同一批存储配对，因此表 2 的四个单元格只在修复轮（行）和量规（列）上有所不同。任务感知列背后的已决配对数，修复前为 526 对中的 271 对，修复后为 565 对中的 435 对。

## 附录 E 演化实际产出了什么

#### 最终验证器，逐字原文。

在 MBPP+ 上，最终验证器为

```
(any spec_mismatch crash returns_not_print_only)
```

若评判发现规格不匹配，或解答在探测输入上崩溃，或它打印而非返回，则标记该解答。在 Spider 2.0 上，最终验证器为

```
(any missing_group_by spec_sql_mismatch)
```

若查询做了聚合却没有分组，或评判发现它并没有回答问题，则标记该查询；这一形式在三个种子中的两个上被独立选中。在报告生成上，最终验证器为

```
(any ledger_metric_uncovered low_status_card_overclaimed)
```

若某个已登记的关键数字从未出现，或一张弱可信度卡片在未加保留表述的情况下被引用，则标记该节；另外两个种子把第一片叶子换成一个未引用主张检测器。

#### 一个检测器走完整个生命周期。

在最终停在下面这一验证器上的那个 Spider 种子中，

```
(any syn_forbids_select_star_when_columns spec_sql_mismatch)
```

循环把四条未标注的训练输出聚成一簇——这些输出返回了每一列，尽管问题暗示的是特定的列——并撰写了如下带类型的规格：

```
{kind: static, check: forbids_select_star, detects: select_star_specific_columns}
```

因为它的诞生证据是一处未标注缺口，而不是一次有锚点的失误，它以 *shadow* 身份进入：它运行并记录判定，但不能被选中。它在同一轮被提升，但提升发生在把它与现任表达式组合、从而使 dev 一致性从 0.71 提高到 0.81 之后；选择随后采纳了它，替换掉种子池所贡献的分组检查。该任务上，共同循环的种子中有两个收敛到另一个合成检测器（`syn_top_n_missing_limit`），第三个收敛到种子中的 `select_star` 检查，因此这片叶子并不是检测器池的固定产物。

#### 给一项任务评分：隐藏测试对照演化后的验证器。

对于「编写一个函数，求整数数组中零所占的比率」，求解器返回了

```
zeros / non_zeros
```

代码能干净运行并返回一个数字，但计算的是相对于非零元素的比率，而规格要的是相对于全部元素的比率，因此错的只是含义。隐藏单元测试抛出 `AssertionError` 并返回 fail，这是可靠的判定，却不指出哪一项要求被误读。演化后的组合同样返回 fail，其依据是

```
crash: clean, returns_not_print_only: clean, spec_mismatch: drawback
```

于是**点名**了失败类别。这个被点名的 drawback 把该次尝试变成可供技能合成使用的失败胶囊；锚点给出的裸判定无法驱动同样的修复。

#### 同一课，两个循环。

下面每一对都是独立合成的：一条由参考循环在标准答案评分下合成，一条在演化后的验证器下合成，而且两个循环都没有看见对方。*MBPP+*：参考技能 `function-name-from-assert`，「测试工具按 assert 中写下的字面形式导入该符号；把标识符原样提取，包括大小写，正文只是提示」；共同循环技能 `copy_callable_identifier_from_assert`，「评分器从 assert 中原样导入名称；逐个字母复现那一精确拼写，包括看似的笔误」。*Spider 2.0*：参考技能 `sql-empty-result-diagnosis`，「当金标准结果有行时，0 行结果几乎从不是正确的；逐步放宽谓词，以定位是哪一个条件消去了全部的行」；共同循环技能 `sql-empty-result-sanity-probe`，「空结果几乎总意味着某一个谓词什么都没有匹配到；在信任连接之前，先单独核验每个 CTE 的行数」。*报告*：参考技能 `metric-tag-annotates-not-replaces`，「只有当一个字面数字紧挨着花括号标签时，评分器才计入该数字」；共同循环技能 `metric-tag-inline-after-number`，「标签是在标注数值；它从不取代数值，光有一条卡片引用并不能满足这一要求」。

## 附录 F 生命周期统计与完整的逐循环分数

表 3：报告生成上的验证器有效性消融。数值为三个种子上的最终各轮，每个实验臂翻转一类守卫（$\checkmark$ 为开，x 为关）。去掉*锚点*守卫，会在每一个种子上把验证器塌缩成空洞的一律通过评分器；去掉检测器*生命周期*则不会。*训练通过*：训练尝试中被判通过的比例（*n/r*：该实验臂未记录）。这一塌缩在留出任务分数中看不见（表 1，*naive*）。着色行：本文配置。

| 实验臂 | 守卫：锚点 | 守卫：生命周期 | 有效性：目标 | 有效性：留出一致性 | 有效性：训练通过 | 结果 |
|---|---|---|---|---|---|---|
| **anchored** | $\checkmark$ | $\checkmark$ | $0.865\pm 0.002$ | $0.830\pm 0.012$ | .78–.82 | 有效，可组合 |
| naive | x | $\checkmark$ | 1.000 *（空洞）* | *fail-open* | .97–1.0 | **塌缩**为一律通过 |
| no-lifecycle | $\checkmark$ | x | $0.896\pm 0.072$ | $0.868\pm 0.061$ | *n/r* | 未塌缩 |

图 4 画出表 1 背后的留出学习曲线，表 4 概括锚定实验臂中检测器生命周期做了什么，表 5 给出完整的留出初始、峰值和最终各轮分数。冻结的单次调用基线大约为 0.27（MBPP+）、0.285（Spider 2.0）和 0.56（报告生成）。

#### 技能侧行为。

合成产出很多，大约每轮 1.5 个技能；有界的技能库按贡献淘汰掉所创建内容的一半以上；回滚很少触发。报告技能循环是例外：它那些有针对性的失败胶囊产出的技能足够有效，少量即可，其中一个种子在只有六条技能的库上守住了平台期。

图 4：留出学习曲线（均值 $\pm$ 标准差，三个种子）：参考技能循环使用标准答案或现有最佳量规，对照 Double Ratchet 使用它自己演化出的验证器。标注：共同演化所保留的提升，占参考循环提升的比例。留出评估从不经过演化后的验证器。

表 4：检测器生命周期，锚定实验臂，共 100 轮。三个种子的最小–最大；单独一个数表示所有种子一致。锚定选择偏好精确的小型组合，而不是检测器的不断堆积。

| 任务 | 种子检测器 | 诞生 | 诞生门拒绝 | 最终池 | 验证器中的叶子 |
|---|---|---|---|---|---|
| MBPP+ | 10 | 2–4 | 997–1014 | 12–14 | 3 |
| Spider | 9 | 24–30 | 612–785 | 33–39 | 1–2 |
| 报告 | 11 | 11–15 | 172–188 | 22–26 | 2 |

表 5：逐循环的留出评估，三个种子上的均值 $\pm$ 标准差；每个任务上的 *naive* 行使用与 *co* 相同的三个种子。最终各轮：每次运行最后十轮的均值。着色：本文配置。

| 任务 | 循环 | 初始 | 峰值 | 最终各轮 |
|---|---|---|---|---|
| MBPP+ | reference | $0.275\pm 0.050$ | $0.700\pm 0.025$ | $0.592\pm 0.039$ |
| | **co** | $0.267\pm 0.038$ | $0.717\pm 0.038$ | $0.652\pm 0.034$ |
| | naive | $0.317\pm 0.058$ | $0.742\pm 0.014$ | $0.632\pm 0.023$ |
| Spider | reference | $0.317\pm 0.038$ | $0.483\pm 0.038$ | $0.327\pm 0.010$ |
| | **co** | $0.275\pm 0.043$ | $0.458\pm 0.038$ | $0.323\pm 0.038$ |
| | naive | $0.292\pm 0.052$ | $0.458\pm 0.029$ | $0.343\pm 0.028$ |
| 报告 | reference | $0.562\pm 0.001$ | $0.850\pm 0.010$ | $0.802\pm 0.024$ |
| | **co** | $0.557\pm 0.006$ | $0.812\pm 0.006$ | $0.767\pm 0.015$ |
| | naive | $0.559\pm 0.003$ | $0.841\pm 0.003$ | $0.811\pm 0.015$ |

#### 另外两项范围限制。

指导文本类技能不能替代那些需要交互的能力，这正是两个循环在 Spider 2.0 上碰到的天花板（§5.3）。op 类型系统也接纳蜕变测试精神下、预先料到失败的检测器 [Chen et al., 1998]：它们扰动一项任务，并检查输出是否按应有的方式变化；这些检测器已经实现，但没有任何被选中的验证器用过其中之一，因此我们不对它们提出主张。

#### naive 共同循环并不是免检通行证。

naive 共同循环在留出评估上与锚定实验臂相当，因为它们的测量锚点没有被改动，但它们的验证器在每一项任务上都是退化的，而不只是在报告生成上（§5.2）：在 Spider 2.0 上，naive 的目标在三个种子中的两个上几乎不动。

#### 技能循环对验证器质量的稳健性。

以下是 §5.3 方向性信号论证的逐种子证据：三个 Spider 共同演化种子中的两个，选中了一个较弱的、合成出来的单检测器验证器（训练一致性 0.41–0.44），峰值评估却达到 0.45–0.50，与那个使用双检测器验证器的兄弟种子无法区分。

#### 锚点规模。

十条 dev 项是一项关于稀疏锚定在现实中究竟能提供什么的设计选择，而不是调好的设定。§5.1 这一主张背后的重放，把报告生成的选择放到离线进行：对每个锚点规模，从同样的十条中抽取 200 个分层子样本，在每一次抽取内部保持 fail/clean 的平衡，并在缩减后的锚点下，对实况运行真正考虑过的每一个候选表达式重新打分。这里没有新的生成，也没有新的求解器调用，因此这一比较把锚点对选择的贡献，从逐次运行的方差中分离出来。四条、六条和八条锚定项上的留出一致性分别为 0.871、0.882 和 0.874，对照十条时的 0.854。子样本标准差随锚点变小而增大（四条时为 0.063，对照八条时的 0.041），这是更小锚点的预期代价：任何一个单独的小锚点都是一次抽签，而对各次抽取取均值则表明，选择并不依赖于那一条边际项。

## 附录 G 可复现性

不涉及 GPU，因为每一次模型调用都走托管 API，所以一次运行只需要一台 CPU 主机（我们的机器：48 核 x86-64、186 GB 内存、Ubuntu 24.04、Python 3.12，并带有 pydantic、pyyaml、boto3，Spider 2.0 另需 snowflake-connector-python），墙钟时间由 API 延迟主导。每次运行都把逐轮历史、检测器池、技能库和评判输出写成结构化的 JSON 与 SQLite 文件，本文的每一个数字都是直接从这些文件读出的；逐次运行的数据库存有所报告验证器运行的全部 52,600 条锁定判定及其逐检测器轨迹，这使得配对检验不必重跑任何循环即可重新计算。每一个 LLM 角色（求解器、教师、组合器、技能合成器、评判 op、终审）都在每次调用时从一张角色到模型的表中绑定，因此换一个模型，或换一种混合分配，来重复任何实验，属于配置变更而不是代码变更；我们在这里保持该表统一，以便比较保持干净。Spider 2.0 需要对该基准托管数仓的带凭证访问；其余所有实验都是自包含的。
