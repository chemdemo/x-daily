---
title: "LLM 自我改进循环中的赢家诅咒：选择噪声、锁定与接受规则"
author: "Litao Hu and Yutong Tang"
source_url: "https://arxiv.org/abs/2610.09239"
arxiv_id: "2610.09239"
arxiv_version: "v1"
published_at: "2026-10-07"
archived_at: "2026-11-25"
license: "CC BY 4.0"
license_url: "https://creativecommons.org/licenses/by/4.0/"
kind: "full-text-translation"
unofficial_translation: true
translation_scope: "摘要至结论的全部正文，含定义、命题、公式、表格、脚注与附录证明。代码保留英文。参考文献保留英文。"
changes: "非官方中文翻译，非作者审定。公式与表格改为 Markdown。未改科学内容。须保留原作者署名与 CC BY 4.0。"
---

> **非官方译文声明。** 本文是 Litao Hu 与 Yutong Tang 论文 *The Winner’s Curse in LLM Self-Improvement Loops: Selection Noise, Lock-in, and Acceptance Rules*（arXiv:2610.09239，v1，2026-10-07）的非官方中文翻译，未经作者审定。原文以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 发布。转载或翻译须保留原作者署名，并保留 CC BY 4.0。本文件是该论文自摘要至附录的完整中文译文，不是分块摘要。译文不删节：段落、定义、假设、命题、公式、表格与图注均照译，不以摘要代替命题。公式由 HTML 导出的重复排版整理为单一可读 LaTeX，表格改为 GitHub 风格 Markdown。科学内容未作改动。代码保留英文，参考文献保留英文。

# LLM 自我改进循环中的赢家诅咒：选择噪声、锁定与接受规则

Litao Hu（单位：Meta）　Yutong Tang（单位：Microsoft）

## 摘要

自我改进的 LLM 系统对自身提出改动，并保留那些在一个小型评测集上得分更高的改动。我们将这一“更优则保留”步骤视为测量噪声下的选择，对单次决策中各候选的相关误差建模，并实证考察评测集被复用时会发生什么。在若干运行中，Qwen 模型改写自己的指令，且每个候选还在 600 个留出条目上打分；第一次之后的大多数提议是有害的，而该模型给出一代之中最佳候选的赢家诅咒的大小。若采用来自另一次预试验的先验，它与原生循环中第一代提交的平均高估相吻合，但并非逐个设置都吻合。在一项预注册研究中，贪心循环的最终选择集分数超出留出准确率 13 至 20 个百分点（16 个选择条目）以及 1 至 5 个百分点（256 个）。留出增益随选择集增大而增长出现在 TREC 上，但未出现在 GSM8K 上；所检验的接受规则在完整运行上并未胜过贪心接受。当一个当前模型精炼一条已胜任的指令时，在选择集上测得的增益同样超过留出增益；在 GEPA 与 MIPROv2 的验证分数中亦然。在从未用于选择的 64 个条目上对起始指令与当前指令打分，可以消除一个循环所报告增益的平均偏差，但单次估计仍大约偏离 6 个百分点。自我改进研究应当报告留出增益及其不确定性。

## 1 引言

越来越多的一类 LLM 系统通过编辑自身的工件来改进自己。提示优化器改写流水线的指令 (Zhou et al.,, 2023; Yang et al.,, 2024; Agrawal et al.,, 2025)，智能体修订其记忆、工具与脚手架 (Hu et al.,, 2025; Zhang et al.,, 2026)，而由 LLM 引导的搜索改写程序 (Novikov et al.,, 2025)。这些系统共享同一步骤。它们提出一项改动，在有限的任务集上为其打分，若得分优于它所要替换的对象，则将其保留。决定保留什么的那个分数，也正是循环日志与优化器的验证输出当作进展来展示的分数。严谨的研究也会另行报告独立测试集，但最近的若干审计表明，不应按表面价值看待选择分数。演化得到的智能体装备（harness）在匹配预算下并不能稳定地胜过测试时缩放 (Wang et al.,, 2026)，贪心接受会提交许多虚假或有害的编辑 (Shawn,, 2026)，测得的自训练增益可能是评测造成的伪影 (Xu et al., 2026a, )，而一项覆盖 41 篇论文的方法审计发现，有效性控制常被省略 (Arce,, 2026)。

统计学可以预言这些发现。当在若干带噪估计中选出最好的一个时，该估计向上偏倚。育种者方程、优化器诅咒，以及已上线 A/B 测试的赢家诅咒，都是这一效应的实例 (Falconer and Mackay,, 1996; Smith and Winkler,, 2006; Lee and Shen,, 2018)，而一种决策论上的补救，是由效应先验来告知的采纳阈值 (Berman and Van den Bulte,, 2022)。LLM 自我改进不同于经典设定：提议者常常就是正在被改进的系统，候选是同一工件的相关改写，一个小型评测集被许多代复用，并且配对的留出效应是可以估计的。

我们将“更优则保留”这一步建模为测量噪声下的选择，其随机效应结构中，一代的各个候选共享在位解的误差（第 3 节）。我们自始至终区分两个量。一个程序的*膨胀*，是它在选择集上的分数减去其留出准确率；一项增益的*高估*，是在选择集上测得的增益减去留出增益，也就是最终程序的膨胀减去起始程序的膨胀。我们研究循环内部的决策：相关的候选误差如何扭曲被选中候选的增益，被复用的条目如何改变此后候选所对照的在位解，以及那些有助于孤立决策的校正是否能改进完整运行。第一项在新抽取集合的模型下导出；后两项是实证的。在带预言机插桩的运行中，Qwen 模型改写自己的指令，且每个候选还在一个 600 条的留出集上打分；对种子的第一次改写之后，大多数提议是有害的，而赢家诅咒具有模型所预测的大小。在原生循环中，带有来自另一次预试验的先验的模型，与第一代提交的平均高估相吻合，在个别设置上的误差达数个百分点。一项预注册研究确认，循环最终程序的膨胀随选择集增大而下降：复用的 16 条集合上为 13 至 20 个百分点，256 条时为 1 至 5 个百分点。其余假设只得到部分支持，或完全没有得到支持：留出增益随选择集增大而增长出现在 TREC 上，但未出现在 GSM8K 上；校准后的贝叶斯门与先选后确认，在完整运行中都未能胜过贪心接受。当 Qwen3.5-4B 精炼一条已胜任的指令时，高估仍然存在；在 GEPA 与 MIPROv2 的验证分数中，使用 16 个验证条目时，高估分别达到 26 与 7 个百分点。

条目级日志支持代内的收缩估计；以接受为条件的预测还需要一个先验，而累积的膨胀则需要留出评测。在我们的复用集实验中，所检验的规则并不能可靠地提高最终的留出增益。

## 2 相关工作

#### 自我改进的 LLM 系统。

提示优化器与程序优化器以 LLM 作为变异算子，在指令与示例上搜索 (Zhou et al.,, 2023; Yang et al.,, 2024; Pryzant et al.,, 2023; Guo et al.,, 2024; Fernando et al.,, 2024; Khattab et al.,, 2024; Opsahl-Ong et al.,, 2024; Yuksekgonul et al.,, 2024; Agrawal et al.,, 2025)。自指系统把同一想法用于执行改进的那些代码 (Zelikman et al.,, 2024; Yin et al.,, 2024)，智能体设计与自我修改循环把它用于脚手架与工具 (Hu et al.,, 2025; Zhang et al.,, 2026; Wang et al.,, 2025)，而由 LLM 引导的进化搜索把它用于程序 (Romera-Paredes et al.,, 2024; Novikov et al.,, 2025)。所有这些系统都依据一次有限评测来决定保留哪些改动，而且有些系统已经在防范其中的噪声。GEPA 在完整验证遍之前先在一个小批量上为提议打分，ProTeGi 与 TRIPLE 把候选选择当作赌博机问题 (Pryzant et al.,, 2023; Shi et al.,, 2024)，DGM 与 HGM 则在逐步增大的任务子集上评测。我们在第 5.4 节直接研究 GEPA 与 MIPROv2。

#### 自我改进的可靠性。

最近有两篇论文最为接近。PACE (Shawn,, 2026) 用小型 Qwen2.5 模型运行提示的自我演化，统计贪心接受所提交的虚假编辑与有害编辑，并用一个任意时刻有效的检验取代贪心接受，以控制每一次决策的错误提交概率。SIREN (Xu et al., 2026b, ) 研究在复用基准条目的、有预算的提示或程序自适应搜索之后如何报告。它量化由此产生的乐观性，并在其固定短名单与稳定化选择的假设下，利用留出划分，为“先调后部署”程序给出推断。我们的目标则是循环内部的那一次决策。我们建模相关的候选误差如何扭曲被选中候选的增益，测量被复用的条目如何改变此后候选所对照的在位解，并追问那些有助于孤立决策的校正是否能改进完整运行；这一追问既针对我们自己的循环，也针对 GEPA 与 MIPROv2。PACE 所报告的、并无真实改进的性能退化，出现在我们的探索性运行中，但没有在预注册运行中复现。其他工作对照一个测得的零效应来审计自训练 (Xu et al., 2026a, )，并记录自我改进智能体那些脆弱或未能复现的增益 (Wang et al.,, 2026; Ye et al.,, 2026)。一项方法审计指出已发表研究中被省略的有效性控制 (Arce,, 2026)。

#### 噪声下的选择。

我们的模型沿用若干标准结果，包括育种者方程 (Lush,, 1937; Falconer and Mackay,, 1996)、向均值回归与优化器诅咒 (Harrison and March,, 1984; Smith and Winkler,, 2006)、经选择调整的贝叶斯推断 (Dawid,, 1994; Efron,, 2011)，以及对胜者的推断 (Andrews et al.,, 2024)。在线实验面对同一个上线决策：候选改动很多，流量有限，存在赢家诅咒，而先验设定采纳阈值 (Lee and Shen,, 2018; Azevedo et al.,, 2020; Berman and Van den Bulte,, 2022)。含噪进化策略 (Hammel and Bäck,, 1994; Beyer,, 2000; Arnold,, 2002; Arnold and Beyer,, 2006) 与漂变屏障 (Ohta,, 1973; Sung et al.,, 2012) 是同类现象。在机器学习中，算法配置处理过度调参与在位解的再评估 (Hutter et al.,, 2009; López-Ibáñez et al.,, 2016)，模型选择有其自身的选择偏差 (Cawley and Talbot,, 2010)，自适应数据分析给出由留出集复用所引起的过拟合的界 (Dwork et al.,, 2015; Blum and Hardt,, 2015)，而奖励模型的过度优化是 RLHF 中的对应现象 (Gao et al.,, 2023)。

## 3 “更优则保留”循环的选择模型

#### 循环。

一件*工件* $c$（一条指令、一段程序、一份记忆或一套脚手架）作为固定求解器的条件输入，其性能为 $f(c)=\mathbb{E}_{x\sim P}[r(c,x)]$，$r\in\{0,1\}$。在第 $t$ 代，提议者从在位解 $c_t$ 抽取 $K$ 个候选；每个候选在一个*选择集* $D=(x_{1},\dots,x_{n})\stackrel{\mathrm{iid}}{\sim}P$ 上与在位解配对打分：

$$\hat{\Delta}_{k}=\frac{1}{n}\sum_{i=1}^{n}\big[r(c'_{k},x_{i})-r(c_{t},x_{i})\big], \tag{1}$$

这是 $\Delta_{k}=f(c'_{k})-f(c_{t})$ 的一个估计。一条*接受规则*提交一个候选，或不提交任何候选；贪心的“更优则保留”提交 $\hat{k}=\arg\max_{k}\hat{\Delta}_{k}$，当且仅当 $\hat{\Delta}_{\hat{k}}>0$。我们研究单一在位解的循环，并在第 5.4 节把该模型用于基于种群的搜索所返回的程序。

#### 测量噪声。

只有不一致的条目有贡献：令 $q_{k}=\Pr[r(c'_{k},x)\neq r(c_{t},x)]$，则 $\hat{\Delta}_{k}$ 在一个*新抽取*的 $D$ 上无偏，方差为 $v_{k}/n$，其中 $v_{k}=q_{k}-\Delta_{k}^{2}$。对同一条指令的改写会改变许多答案（在我们的运行中，每条目 $v\approx 0.03$–$0.2$），因此 $n$ 个条目只能分辨量级为 $\sqrt{v/n}$ 的差异。

###### 假设 1（高斯选择模型）。

给定历史以及一个新抽取的选择集，$\Delta_{k}\stackrel{\mathrm{iid}}{\sim}N(\mu,s^{2})$，且 $\hat{\Delta}_{k}=\Delta_{k}+c+\eta_{k}$，其中共享误差 $c\sim N(0,e_{c}^{2})$（所有候选面对同一个在位解与同一批条目），特异误差 $\eta_{k}\stackrel{\mathrm{iid}}{\sim}N(0,e_{\eta}^{2})$，并且 $c$、全部 $\eta_{k}$ 与全部 $\Delta_{k}$ 相互独立。

$\mu$ 是*提议偏差*（一项被提议改动的真实效应的均值），$s$ 是*提议散布*。噪声项由条目向量识别：$e^{2}=e_{c}^{2}+e_{\eta}^{2}=v/n$，以及 $e_{\eta}^{2}=v_{cc}/n$，其中 $v_{cc}=\tfrac{1}{2}\mathbb{E}\,\mathrm{Var}_{x}[r(c'_{j},x)-r(c'_{k},x)]$ 是两个候选之差的每条目方差的一半。记 $\sigma_{w}^{2}=s^{2}+e_{\eta}^{2}$，$h_{w}^{2}=s^{2}/\sigma_{w}^{2}$，$h^{2}=s^{2}/(s^{2}+e^{2})$，并对独立同分布的标准正态令 $\kappa_{K}=\mathbb{E}\max_{k\leq K}Z_{k}$（$\kappa_{4}\approx 1.03$，$\kappa_{16}\approx 1.77$；近似 $\sqrt{2\ln K}$ 在 $K=4$ 时给出 1.67，只对大的 $K$ 成立）。后验公式假定 $\sigma_{w}^{2}>0$。

### 3.1 赢家诅咒

###### 命题 1（选择差与高估）。

在假设 1 下，令 $\bar{m}$ 与 $\bar{\Delta}$ 分别为测量效应与真实效应的代均值，且 $\hat{k}=\arg\max_{k}\hat{\Delta}_{k}$。则 $\mathbb{E}[\Delta_{\hat{k}}-\bar{\Delta}\mid\hat{\Delta}]=h_{w}^{2}(\hat{\Delta}_{\hat{k}}-\bar{m})$，且 $\mathbb{E}[\hat{\Delta}_{\hat{k}}-\bar{m}]=\sigma_{w}\kappa_{K}$，因此真实选择差为 $h_{w}^{2}\sigma_{w}\kappa_{K}$（当 $e_{c}=0$ 时，即育种者方程 $R=h^{2}S$）。$\hat{k}$ 相对于在位解的测量增益对其真实增益的高估为

$$\mathbb{E}[\hat{\Delta}_{\hat{k}}-\Delta_{\hat{k}}]=(1-h_{w}^{2})\,\sigma_{w}\kappa_{K}+\mathbb{E}[c]. \tag{2}$$

第一部分是被选中候选向均值的回归；式 (2) 即优化器诅咒 (Smith and Winkler,, 2006; Harrison and March,, 1984)，在新抽取的选择集上 $\mathbb{E}[c]=0$。该命题描述的是任何接受过滤之前、测量值最好的那个候选。一次贪心提交，是在 $\hat{\Delta}_{\hat{k}}>0$ 条件下的该候选；当大多数提议有害时，以接受为条件会提高偏倚。对于真实效应为 $-e$、噪声为 $N(0,e^{2})$ 的单个候选，无条件误差为零，但一个被接受候选的平均误差为 $e\,\phi(1)/(1-\Phi(1))\approx 1.53e$。第 5.2 节使用以接受为条件的值。

### 3.2 贝叶斯规则与校准阈值

###### 命题 2（短视贝叶斯提交）。

在假设 1 下，且 $(\mu,s^{2},e_{c}^{2},e_{\eta}^{2})$ 已知，使一代的期望真实改进最大化的规则，提交后验均值最大的那个候选

$$\mathbb{E}[\Delta_{k}\mid\hat{\Delta}]=\mu+h_{w}^{2}(\hat{\Delta}_{k}-\bar{m})+\frac{h_{w}^{2}\sigma_{w}^{2}}{\sigma_{w}^{2}+Ke_{c}^{2}}(\bar{m}-\mu) \tag{3}$$

当且仅当该后验均值为正。在独立噪声（$e_{c}=0$）且 $s>0$ 时，这是测量增益上的一个阈值：$\hat{\Delta}_{\hat{k}}>\tau^{\star}=-\mu\,e^{2}/s^{2}$。

这是标准的贝叶斯采纳规则 (Dawid,, 1994; Berman and Van den Bulte,, 2022)。它只在短视意义下最优（忽略一次提交对后续各代的影响），并且要求超参数已知；我们所运行的门（第 4 节）增加了它所缺少的一项保障，即测量增益为正。在独立噪声且 $s,e>0$ 时，贪心接受（$\tau=0$）仅当 $\mu=0$ 时才最优；固定水平检验使用 $\tau=z_{\alpha}e$，而不论 $\mu$ 与 $s$。当大多数提议有害、且候选彼此几乎没有差别（$s\ll e$）时，正确的阈值很大；式 (3) 说明如何校准它。

### 3.3 Goodhart 体制与锁定

###### 命题 3（不可分辨体制，新抽取的选择集）。

令 $s=0$ 且 $\mu<0$，提议分布与噪声分布不随代改变，并且每一代都抽取一个新的选择集。贪心每代以概率 $p=\Pr[c+e_{\eta}\max_{k}Z_{k}>|\mu|]$ 提交；该概率随 $K$ 增大，随 $n$ 减小。留出性能每代向下漂移 $|\mu|p$，而每一次提交都报告正的增益。

这是回归型 Goodhart (Manheim and Garrabrant,, 2018)，对应于群体遗传学中近中性替换的类比 (Ohta,, 1973)。*被复用的*选择集才是常见做法，它们改变了动态，上面的命题不再描述它们。一项测量增益的误差，等于候选的分数误差减去在位解的分数误差。因此，在位解的正向选择误差以负号进入每一项测量增益。若后续候选并不继承同样大的误差，这就起到一个隐式阈值的作用。我们把这一可能的机制称为*锁定*；新抽取集合的模型并不能识别它在复用条目上的净大小。它可能使贪心更为保守，其代价可能是保留一个部分由运气选出的在位解。第 5.2 节测量在位解的膨胀以及随后的提交误差，而不对该机制作因果上的分离。附录 A 针对给定的 $n$，补充一个启发式的局部分辨尺度。

## 4 接受规则

所有规则都选择测量值最好的候选（对贝叶斯门而言，则是后验均值最大者），差别在于何时提交它。*贪心*：$\hat{\Delta}_{\hat{k}}>0$。*McNemar*：针对在位解、水平为 $\alpha$ 的单侧精确检验。*PACE*：Shawn, (2026) 所发表的任意时刻有效检验；它把一半财富押在每一个不一致条目上，并在财富达到 $1/\alpha$ 时提交。*先选后确认*：在 $D$ 的一半上作选择，当且仅当胜者在另一半上也击败在位解时才提交。*调参基线*：McNemar 的 $\alpha$，或 $\hat{\Delta}_{\hat{k}}$ 上的一个阈值，选取为在预试验运行上使价值最大。*贝叶斯门*应用式 (3)，噪声项用来自当前代条目向量的正则化估计 $\hat{e}_{\eta}^{2}=\hat{v}_{cc}/n$ 与 $\hat{e}_{c}^{2}=(\hat{v}-\hat{v}_{cc})/n$，并且当且仅当最大后验均值与其测量增益都为正时才提交。*预试验校准*门从一次预试验运行中取得 $(\mu,s^{2})$，该运行里候选也在一个大型评测集上打分，并且第一代与其后各代分开取值；*在线*门则从循环自身的历史来估计它们。附录 B 给出细节。McNemar 或 PACE 检验的水平，对新鲜条目上的一个固定候选成立。在检验之前先从 $K$ 个候选中选出最好者，以及跨代复用这些条目，都会使该保证失效，因此我们按各规则之决策的留出价值来比较规则，而不是按其名义错误率。

## 5 实验

我们报告一项探索性研究、一项预注册的验证性研究，以及使用当前模型、已胜任的起点和两个提示优化器的研究。

#### 实验设置。

除非另有说明，求解器与提议者是同一个模型，即 Qwen2.5-1.5B- 或 7B-Instruct (Qwen Team,, 2024)；当前一代的模型 (Yang et al.,, 2025; Qwen Team,, 2026) 出现在第 5.4 节。所有模型都用 vLLM 提供服务 (Kwon et al.,, 2023)。每一代中，提议者看到当前指令、一段任务描述以及四次失败，并写出 $K=4$ 条改写（附录 B）。任务为 TREC-50 (Li and Roth,, 2002)、Banking77 (Casanueva et al.,, 2020) 与 GSM8K (Cobbe et al.,, 2021)，种子指令的长度为一句。每个任务有一个 600 条的*金标准*集，按条目 ID 排除在选择与反馈之外（TREC 的文本重叠审计见附录 B），而 $n$ 条的选择集跨代复用，如同大多数系统那样。我们称金标准准确率为*留出*准确率，称循环自身的选择集分数为*代理*；代理减去留出准确率，就是第 1 节中的膨胀。一个候选的*留出效应*，即它在金标准集上的配对增益，以 1.5 至 2 个百分点的标准误估计其真实效应 $\Delta_{k}$。探索性运行使用 $T=20$ 代，每个配置三个种子，并在附录 D 中用第二套推理栈复现；在其*预言机插桩*的运行中，每个候选也在金标准集上打分。

### 5.1 提议大多有害，且赢家诅咒具有所蕴含的大小

预言机运行在每个设置下给出 55 至 152 个同时具有测量效应与留出效应的候选。对一句种子的第一次改写常常能改进它（在 TREC 上，第一代候选中有 75% 至 100% 做到了这一点）。此后，只有 4% 至 39% 的改写能改进其在位解，平均留出效应介于 $-0.2$ 与 $-13.5$ 个百分点之间，而偶发的灾难性改写使效应左偏（超额峰度高达 7.6）。更强的提议者（Qwen2.5-14B、Qwen3.5-4B）对这些数字改变很小（附录表 2）。候选彼此之间比它们与在位解更为相似，$v_{cc}\approx 0.44$–$0.60\,v$。

为检验噪声模型，我们把金标准集拆成一个 $n$ 条的伪选择集和一个真值集，在前者上选出每一代的最佳候选，并比较它在两部分上的选择差（200 次拆分；图 1）。用来自同一批预言机数据的输入，模型所蕴含的留出选择差与测量选择差之比，在九个设置（三个模型、两代 Qwen、两套推理栈）以及 $n=8$ 到 $256$ 上与观测到的比值相吻合：使用留出效应的经验分布时，平均绝对误差为 0.034；使用高斯 $h_{w}^{2}$ 时为 0.064。后者在效应呈重尾的地方高估了可靠性。在 $n=16$ 时，胜者的留出选择差只是其测量选择差的 1% 至 51%。一个循环可以自行估计 $h_{w}^{2}$：$\hat{v}_{cc}$ 来自候选的条目向量，$\hat{s}^{2}$ 来自测量效应的代内散布减去 $\hat{v}_{cc}/n$。在一次预言机运行的选择集的一个固定 $n$ 条子集上，由第 1 代到第 $t$ 代按因果顺序计算，该估计预测这一比值的平均绝对误差为 0.10，偏倚为 $+0.02$；用它去收缩胜者的测量优势，使其均值落到留出均值的 0.6 个百分点以内（$n=16$ 时未收缩的均值为 5.0 个百分点）。单次运行的估计并不精确（$n=16$ 时标准差为 0.12 至 0.21；附录 C）。两项检验使用的都是由 256 条的循环所生成的轨迹；第 5.2 节转向原生的小集合循环。

图 1：赢家诅咒具有噪声模型所蕴含的大小。$K$ 选一改写的真实选择差与测量选择差的观测比（真实值取拆分中的留出部分；区间为跨代的 95% 聚类自助区间，其中不含运行之间的变异），对照由 (a) 留出效应的经验分布（$K=4$ 近似）与 (b) 高斯模型所蕴含的比值；覆盖九个设置以及 $n\in\{8,\dots,256\}$。(c) 1.5B 的 TREC 候选在 16 条与 128 条上的测量效应对留出效应。

### 5.2 复用选择集上的提交

我们在下面这项验证性研究的贪心运行的事后分析中，把模型与原生循环相比较（四个设置、八个种子，$n=16$、$64$、$256$）。对每一次提交，我们计算假设 1 在给定接受的条件下所蕴含的高估，使用另一次预言机插桩预试验的先验 $(\mu,s^{2})$——该先验在这些运行之前已经冻结——以及该代候选在循环自身选择集上的噪声项（附录表 5 与附录表 6）。该模型只描述第 1 代作出的提交：此时在位解就是种子，而且此前没有任何决策使用过这个选择集。对于这 72 次提交，观测平均值与模型平均值在 $n=16$ 时为 9.1 与 9.0 个百分点，在 $n=64$ 时为 3.2 与 3.8，在 $n=256$ 时为 0.1 与 1.3。这些平均值掩盖了单个设置上数个百分点的误差（在 $n=16$，1.5B TREC 上为 1.9 对 5.9，Banking77 上为 12.9 对 7.0，各自有四到六次提交），因此我们把它读作平均意义上的吻合，而不是校准。对于其他提交，模型给出的值只是新抽取集合下的参照。在先前的拒绝之后出现的首次提交（94 次中的 22 次），在 $n=16$ 时的高估小于模型（6.3 对 9.6 个百分点）；针对一个已被选出的在位解的提交，在每一个 $n$ 上的高估都更少（5.2 对 9.7、1.9 对 3.7，以及 0.5 对 1.1）。这一模式与锁定相符：一次提交的高估恰好就是在位解膨胀的变化，但其净效应取决于后续候选与在位解共享多少误差。

膨胀相对于种子自身的偏移，在最初几代上升，并随 $n$ 缩小。经过 11 至 20 代之后，在 $n=16$ 时为 $13.6\pm 1.7$ 个百分点，在 $n=64$ 时为 $7.2\pm 1.1$，在 $n=256$ 时为 $1.7\pm 0.6$（验证性的贪心运行，一项次要分析；附录表 4）。在位解的膨胀是否充当一个以真实改进为代价的阈值，则不那么清楚。在晚期候选中，14% 的留出效应介于 0 与 1.8 个百分点之间，落在零的一个金标准集标准误之内，而几乎没有候选超过在位解。每一代重新抽取选择集，可以避免把在位解早先的选择误差带入下一次比较；但在我们试过的唯一设置中（三个种子，探索性设计），这样做并没有提高最终增益（附录 C）。

### 5.3 预注册的验证性研究

探索性运行只有三个种子，而且提议者的失败样例来自选择集本身，因此 $n$ 同时改变了评测以及提供给提议者的信息。我们因此预注册了一项验证性研究。设计、五条假设，以及一项增补——加入先选后确认、一个按阶段索引的预试验先验，以及一个更强的提议者——都在第一次运行开始之前，固定在一份注明日期的书面计划中。分析脚本在运行开始后十分钟写成并归档，当时 336 次运行中已有 16 次完成。对计划的偏离列在附录 B。金标准集是与探索性集合不相交的、新的 600 条样本，种子是新的，提议者的失败样例来自单独的一个 256 条池，因此 $n$ 只改变评测。四个设置（1.5B TREC、1.5B Banking77、7B TREC、1.5B GSM8K）各自有九个臂、八个种子，即 $n=16,64,256$ 上的贪心接受，以及在 $n=16$ 与 $64$ 上的 McNemar（$\alpha=0.05$）、预试验校准贝叶斯门和先选后确认。预试验先验在启动前，由同一模型、同一任务的探索性预言机运行冻结。检验按种子配对、取双侧，并在每个族内作 Holm 校正。表 1 与图 2 报告结果，附录 C 列出全部臂。

图 2：预注册的验证性运行（新的金标准集；(a)–(c) 中为八个新种子，(d) 中为五个；提议者反馈与 $n$ 无关；均值 $\pm$ 标准误）。(a) 贪心循环相对于种子指令的最终留出增益，以及 (b) 它们最终的代理–留出差距，横轴为 $n$。(c) McNemar（$\alpha=0.05$）、先选后确认以及预试验校准贝叶斯门的留出增益，减去贪心的留出增益，按设置给出（按种子配对；设置的标记与 (a) 相同，图例在 (b)；颜色表示规则）。(d) 由一个更强的模型为 1.5B 求解器作提议的贪心循环。

表 1：预注册检验（单位为百分点；按种子配对之差的均值 $\pm$ 标准误；$p$ 在每个族内作 Holm 校正，合并行未校正）。

| 设置 | 差值（百分点） | 胜 / 配对数 | $p$ |
|---|---|---|---|
| *H1：增益（$n=256$）$-$ 增益（$n=16$），贪心* | | | |
| 1.5B TREC | $+10.5\pm 2.6$ | 7 / 8 | 0.010 |
| 7B TREC | $+6.4\pm 0.7$ | 8 / 8 | $<0.001$ |
| 1.5B GSM8K | $+1.6\pm 2.2$ | 4 / 8 | 0.483 |
| *H2：差距（$n=16$）$-$ 差距（$n=256$），贪心* | | | |
| 1.5B TREC | $+15.9\pm 2.7$ | 8 / 8 | 0.002 |
| 1.5B Banking77 | $+10.5\pm 4.6$ | 7 / 8 | 0.058 |
| 7B TREC | $+15.3\pm 4.7$ | 7 / 8 | 0.037 |
| 1.5B GSM8K | $+14.7\pm 4.4$ | 8 / 8 | 0.037 |
| *H3：贪心增益，$n=16$（$t$ 检验；符号检验）* | | | |
| 1.5B Banking77 | $-0.6\pm 0.7$ | 4 负 / 8 | 0.425; 1.000 |
| *H4：预试验门 $-$ 贪心，$n=16$* | | | |
| 合并，32 对 | $-2.1\pm 1.1$ | 10 / 32 | 0.062 |
| 1.5B TREC | $-1.3\pm 3.0$ | 3 / 8 | 0.677 |
| 1.5B Banking77 | $+1.1\pm 0.7$ | 4 / 8 | 0.552 |
| 7B TREC | $-2.5\pm 2.4$ | 2 / 8 | 0.650 |
| 1.5B GSM8K | $-5.7\pm 1.6$ | 1 / 8 | 0.032 |
| *H5：先选后确认 $-$ 贪心，$n=16$* | | | |
| 合并，32 对 | $-1.6\pm 1.2$ | 9 / 32 | 0.194 |
| 1.5B TREC | $+2.1\pm 3.6$ | 5 / 8 | 1.000 |
| 1.5B Banking77 | $-1.7\pm 1.2$ | 1 / 8 | 0.566 |
| 7B TREC | $-6.4\pm 1.9$ | 0 / 8 | 0.045 |
| 1.5B GSM8K | $-0.4\pm 1.6$ | 3 / 8 | 1.000 |

最清楚的结果关乎循环对自身的报告（H2）。使用 16 个条目时，最终代理超出留出准确率 13 至 20 个百分点；使用 256 个条目时，超出 1 至 5 个百分点。配对之后的降幅在三个设置中约为 15 个百分点（14.7 至 15.9）（Holm 校正后 $p\leq 0.04$），在 Banking77 上为 10.5 个百分点，未达到显著性（$p=0.058$）。留出增益在 TREC 上随 $n$ 上升（H1）：1.5B 模型从 $13.7\pm 2.9$ 到 $24.2\pm 0.7$ 个百分点（八个种子中的七个），7B 模型从 $14.0\pm 0.7$ 到 $20.4\pm 0.6$（八个中的八个）。它们在 GSM8K 上并没有上升；在那里，没有任何一个臂的增益超过 3.1 个百分点。探索性运行在 Banking77 上所显示的 Goodhart 损失没有复现（H3）。$n=16$ 的贪心循环结束时比种子低 $0.6\pm 0.7$ 个百分点，八个种子中有四个变差，而它们的选择集分数上升了 12.5 个百分点。探索性损失本身并不显著（$p=0.20$，三个种子）。

所检验的接受规则并没有改进完整运行（H4、H5）。在 $n=16$，预试验校准门结束时比贪心低 $2.1\pm 1.1$ 个百分点，在 32 对种子上合并（$p=0.06$；95% $t$ 区间 $[-4.3,0.1]$），并且在 GSM8K 上显著更差；该设置的预试验先验只基于一次预言机运行。先选后确认结束时比贪心低 $1.6\pm 1.2$ 个百分点。固定水平的 McNemar 检验放弃了 TREC 上大部分可得增益（1.5B 与 7B 模型分别为 $-7.4$ 与 $-12.3$ 个百分点）。当 Qwen2.5-14B 为 1.5B 求解器作提议时，TREC 增益升至 26 至 33 个百分点，而自报告的差距在全部三个任务上仍随 $n$ 显著缩小（图 2d）。

图 3：循环与优化器所报告的，以及它们实际获得的。(a, b) 最终在位解或返回程序相对于起始指令或起始程序的增益，分别在选择（验证）集上测量（空心标记）以及在 600 个留出条目上测量（实心），选择条目为 16（红）或 256（蓝）；线段即高估。循环：Qwen3.5-4B 从一句种子或从一个已胜任的起点改进自己的指令（20 代）。GEPA：从种子程序出发，度量调用为 1000 与 2250；从强程序出发，TREC 为 2750 次调用，Banking77 为 2000 次调用。MIPROv2：24 次试验。对四到八个种子取均值，误差为 $\pm 1$ 标准误（留出标在线段上，报告值标在其上方）。(c) 在 1000 次调用时，GEPA 那些做过留出打分的 TREC 候选的验证准确率与留出准确率。

### 5.4 当前模型、强起点与两个提示优化器

我们首先让 Qwen3.5-4B 同时作为自己的提议者与求解器，重复这一循环（四个种子；附录 B）。从一句种子出发，它在每一个 $n$ 上于 TREC 获得 37 至 39 个百分点，于 Banking77 获得 13 至 15 个百分点，差距再次随 $n$ 缩小（Banking77：由 15.7 到 3.4 个百分点，$p=0.007$）。这些循环相当于一次改写：第一代交付最终增益的 90% 至 112%，此后的改写中只有 7% 与 20% 能改进其在位解。因此我们从一条已胜任的指令重新启动循环，即它自己早先某次运行的最终指令，并且在任何一次运行之前就已选定。由此出发，TREC 的改写中只有 5% 能改进（平均效应 $-14$ 个百分点），Banking77 的改写中有 45% 能改进，但其中只有 5% 的改进超过 2 个百分点（附录表 8）。在四个计划内的种子上，16 条的循环在 TREC 与 Banking77 上报告的增益为 12.5 与 14.1 个百分点，而留出准确率的变化为 $-2.2$ 与 $0.0$；256 条时，它们报告 5.4 与 3.3，并获得 4.3 与 1.8。计划中的代理–留出差距下降了 9.4 与 2.8 个百分点（双侧 $p=0.12$ 与 $0.69$）。在看到这些结果之后追加的四个种子，给出的差距降幅为 7.2 与 12.9 个百分点（$p=0.08$ 与 $0.03$），较大集合的留出优势为 5.6 与 1.9（$p=0.15$ 与 $0.16$）。在八个种子上，高估——我们额外加入的一个度量，因为起点的偏移在不同的 16 条集合之间并不相同——从 16.3 降到 1.4，并从 11.5 降到 1.5 个百分点。我们是在看到第一阶段之后才决定做这一扩展，并在两个阶段都已经跑完之后，才选定单侧方向与相等的阶段权重，因此合并后的逆正态 $p$ 值（差距为 0.010 与 0.033，留出优势为 0.04，高估为 0.004 与 $<0.001$）是探索性的。

我们还在 DSPy (Khattab et al.,, 2024) 中为两个提示优化器加上了同一个留出集的插桩。GEPA (Agrawal et al.,, 2025) 通过反思训练小批量中的失败来演化一段程序的指令，并返回在整个运行中被复用的验证集上得分最高的候选。MIPROv2 (Opsahl-Ong et al.,, 2024) 提出一组固定的候选指令，并在验证集上用贝叶斯优化在其中搜索。GEPA 的论文在独立的测试集上报告准确率，因此我们所测量的，是优化器据以作出选择的那些验证分数的乐观程度，而不是 GEPA 已发表增益的乐观程度。任务模型为 Qwen3.5-4B，提议者为 Qwen3.8-27B，二者都不启用思考，训练池为 256 条，验证条目数为 $n_{\mathrm{val}}$，种子五个（附录 B）。因为一个候选的验证分数在 GEPA 发现它的那一刻就已经固定，所以在任意次度量调用之后返回的程序，都可以从一次运行所保存的状态或日志中精确重建；我们对照日志检查了每一次重建。附录 B 列出对所记录计划的改动，其中包括报告增益这一度量；它是在已经看到八次 $n_{\mathrm{val}}=256$ 的运行、但还没有看到任何条目更少的运行之后才加入的。

图 3 汇总了这些结果。在 1000 次度量调用时，GEPA 所返回的程序在 TREC 上获得约 19 个百分点的留出准确率，在 Banking77 上获得 1 至 2 个百分点，在 GSM8K 上没有获得增益；在 GSM8K 上，任务模型已经达到 95%，无论 $n_{\mathrm{val}}$ 取何值。GEPA 据以选择的验证分数给出了另一幅图景。使用 16 个条目时，它把 TREC 上的改进标为 38.8 个百分点，真实值为 19.2；把 Banking77 上的改进标为 18.8，真实值为 $1.2\pm 1.7$；把 GSM8K 上的改进标为 6.2，而该程序实际上差了 0.9 个百分点。使用 256 个条目时，二者相差至多 3 个百分点（附录表 10）。然而在这一预算下，256 条的臂只找到三个候选，其反思次数只有 16 条臂的十分之一。跑到 2250 次调用时，它的池中有 9 个程序，而 16 条臂的池中有 91 个；GEPA 的留出增益仍然不随 $n_{\mathrm{val}}$ 增长（TREC：16、64 与 256 条时分别为 18.5、20.4 与 19.0 个百分点；Banking77：3.7、1.3 与 0.8）。相对于起始程序的高估，在 16 条时仍然很大：TREC 上为 26.5 个百分点，对照 $-1.9$（$p=0.007$）；Banking77 上为 18.8，对照 3.7（$p=0.002$）。从它自己的最佳程序出发，带 16 个验证条目的 GEPA 在两个任务上都报告了 25.0 个百分点的改进，真实改进为 1.6 与 4.1（256 条：报告值为 3.0 与 3.7，真实值为 0.8 与 1.9）。MIPROv2 在它的两个臂之间大体上保持所提议的指令池不变。使用 256 条时，其返回程序在 TREC 上获得 11.4 个留出百分点，而使用 16 条时为 5.8（配对差 $5.6\pm 0.7$，$p=0.001$，全部五个种子）。在 Banking77 上，其 16 条的臂所返回的程序比种子差 1.6 个百分点，尽管它们的验证分数意味着 5 个百分点的增益（高估为 6.6，对照 256 条时的 2.4 个百分点，$p=0.27$；附录 C）。命题 1 以 $K$ 为池的大小，噪声项取每个优化器所记录的验证条目向量，能够追踪高估如何随 $n_{\mathrm{val}}$ 与预算变化，但并没有校准。在 GEPA 的高估较大之处，观测值是预测值的 1.1 至 1.9 倍（GEPA 的池本身就是在验证集上选出来的），而该模型对 MIPROv2 的 16 条 TREC 单元格预测偏高（9.0 对 5.5 个百分点；附录 C）。

## 6 讨论与局限

#### 应当记录并报告什么。

我们最稳健的发现是，循环对其自身进展的估计过于乐观，在 Qwen 循环中如此，在 GEPA 与 MIPROv2 的验证分数中亦然。循环的条目级日志支持一项估计：某个候选相对于其同代均值的优势应当收缩多少（第 5.1 节）。预测一项已被采纳的增益被高估了多少，还要使用关于效应的先验；我们的原生循环分析从另一次预实验取得该先验（第 5.2 节），而这两者都识别不出在复用选择集上累积起来的膨胀。按该循环自身的 $\hat{h}_{w}^{2}$ 收缩每一次采纳，在我们 $n=16$ 的 151 次验证性运行与当前模型运行上，大体消除了报告增益的平均高估，但在同一设定内部仍留下 $-8.5$ 到 $+5.8$ 分，因为该误差为所有候选所共享。一个小的审计集确实能估计它。在从未用于选择的 64 个条目上，为种子以及经过 19 代之后的现任解打分，花费 128 次评估，是 $n=16$ 的循环之名义预算——1,280 次候选评估——的十分之一。它消除了报告增益的偏差，并把均方根误差从 15.5 分降到 5.7 分。该误差仍然与一次典型增益一样大；按我们任务的不一致率，要分辨几个百分点的增益，需要几百个条目（附录 C）。我们建议，自我改进研究在用于选择的分数之外，同时报告留出分数及其标准误。

#### 接受规则能做什么、不能做什么。

在我们的实验中，校准阈值改善了在全新评估上的孤立决策，主要是靠更少地采纳（附录 C），但所检验的规则并没有改善在复用选择集上的完整运行。当后来的候选并不共享同样大的误差时，现任膨胀可能使贪心变得更保守。更大的选择集提高了 TREC 上的留出增益，这既出现在 Qwen2.5 运行中，也出现在评估代价为十六倍的 MIPROv2 中，但在 GEPA 中则未能检测到；在 GEPA 里，小集合允许了十倍之多的搜索。因此，我们不把所检验的规则推荐为自适应复用的一般补救。

#### 局限。

我们的产物是指令，我们的任务使用确定性评分规则（尽管模型输出并非按位可复现；附录 B），我们最大的提议器有 27B 参数，而且我们尚未考察智能体任务或代码任务。随机展开会引入额外的测量噪声；就该噪声随候选而异的部分而言，命题 1 在效应分布保持固定的条件下预测出更大的赢家诅咒。验证性研究按预注册使用 Qwen2.5 模型。图 1 中的两项检验都使用神谕量，第 5.2 节的原生循环检验覆盖四个 Qwen2.5 设定并且是事后的，贝叶斯门需要一次带大型评估集的预实验，而 PACE 以其已发表的固定赌注和 $n$ 个条目运行。我们不对锁定的动态建立模型，也不对递归自我改进作出断言。

## 代码与数据可用性

代码、保存的运行记录以及复现说明，由本预印本所附的附属文件提供。附录 B 描述实验设计与来源。

## 参考文献

  * Agrawal et al., (2025) Agrawal, L. A., Tan, S., Soylu, D., Ziems, N., Khare, R., Opsahl-Ong, K., Singhvi, A., Shandilya, H., Ryan, M. J., Jiang, M., Potts, C., Sen, K., Dimakis, A. G., Stoica, I., Klein, D., Zaharia, M., and Khattab, O. (2025).  GEPA: Reflective prompt evolution can outperform reinforcement learning.  arXiv preprint arXiv:2507.19457.
  * Andrews et al., (2024) Andrews, I., Kitagawa, T., and McCloskey, A. (2024).  Inference on winners.  The Quarterly Journal of Economics, 139(1):305–358.
  * Arce, (2026) Arce, L. (2026).  Most self-improving LLM-agent papers fail two or more evaluation traps: A methods audit of forty-one papers.  alphaXiv preprint, <https://www.alphaxiv.org/abs/2609.self-improving-llm-agent-evaluation-audits>.
  * Arnold, (2002) Arnold, D. V. (2002).  Noisy Optimization with Evolution Strategies.  Genetic Algorithms and Evolutionary Computation. Springer.
  * Arnold and Beyer, (2006) Arnold, D. V. and Beyer, H.-G. (2006).  A general noise model and its effects on evolution strategy performance.  IEEE Transactions on Evolutionary Computation, 10(4):380–391.
  * Azevedo et al., (2020) Azevedo, E. M., Deng, A., Montiel Olea, J. L., Rao, J., and Weyl, E. G. (2020).  A/b testing with fat tails.  Journal of Political Economy, 128(12):4614–4672.
  * Berman and Van den Bulte, (2022) Berman, R. and Van den Bulte, C. (2022).  False discovery in A/B testing.  Management Science, 68(9):6762–6782.
  * Beyer, (2000) Beyer, H.-G. (2000).  Evolutionary algorithms in noisy environments: Theoretical issues and guidelines for practice.  Computer Methods in Applied Mechanics and Engineering, 186(2–4):239–267.
  * Blum and Hardt, (2015) Blum, A. and Hardt, M. (2015).  The ladder: A reliable leaderboard for machine learning competitions.  In International Conference on Machine Learning (ICML), pages 1006–1014.
  * Casanueva et al., (2020) Casanueva, I., Temčinas, T., Gerz, D., Henderson, M., and Vulić, I. (2020).  Efficient intent detection with dual sentence encoders.  In Proceedings of the 2nd Workshop on Natural Language Processing for Conversational AI, pages 38–45.
  * Cawley and Talbot, (2010) Cawley, G. C. and Talbot, N. L. C. (2010).  On over-fitting in model selection and subsequent selection bias in performance evaluation.  Journal of Machine Learning Research, 11:2079–2107.
  * Cobbe et al., (2021) Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C., and Schulman, J. (2021).  Training verifiers to solve math word problems.  arXiv preprint arXiv:2110.14168.
  * Dawid, (1994) Dawid, A. P. (1994).  Selection paradoxes of Bayesian inference.  In Multivariate Analysis and Its Applications, volume 24 of IMS Lecture Notes–Monograph Series, pages 211–220.
  * Dwork et al., (2015) Dwork, C., Feldman, V., Hardt, M., Pitassi, T., Reingold, O., and Roth, A. (2015).  The reusable holdout: Preserving validity in adaptive data analysis.  Science, 349(6248):636–638.
  * Efron, (2011) Efron, B. (2011).  Tweedie’s formula and selection bias.  Journal of the American Statistical Association, 106(496):1602–1614.
  * Falconer and Mackay, (1996) Falconer, D. S. and Mackay, T. F. C. (1996).  Introduction to Quantitative Genetics.  Longman, 4th edition.
  * Fernando et al., (2024) Fernando, C., Banarse, D., Michalewski, H., Osindero, S., and Rocktäschel, T. (2024).  Promptbreeder: Self-referential self-improvement via prompt evolution.  In International Conference on Machine Learning (ICML).
  * Gao et al., (2023) Gao, L., Schulman, J., and Hilton, J. (2023).  Scaling laws for reward model overoptimization.  In International Conference on Machine Learning (ICML), pages 10835–10866.
  * Guo et al., (2024) Guo, Q., Wang, R., Guo, J., Li, B., Song, K., Tan, X., Liu, G., Bian, J., and Yang, Y. (2024).  Connecting large language models with evolutionary algorithms yields powerful prompt optimizers.  In International Conference on Learning Representations (ICLR).
  * Hammel and Bäck, (1994) Hammel, U. and Bäck, T. (1994).  Evolution strategies on noisy functions: How to improve convergence properties.  In Parallel Problem Solving from Nature – PPSN III, pages 159–168.
  * Harrison and March, (1984) Harrison, J. R. and March, J. G. (1984).  Decision making and postdecision surprises.  Administrative Science Quarterly, 29(1):26–42.
  * Hu et al., (2025) Hu, S., Lu, C., and Clune, J. (2025).  Automated design of agentic systems.  In International Conference on Learning Representations (ICLR).
  * Hutter et al., (2009) Hutter, F., Hoos, H. H., Leyton-Brown, K., and Stützle, T. (2009).  ParamILS: An automatic algorithm configuration framework.  Journal of Artificial Intelligence Research, 36:267–306.
  * Khattab et al., (2024) Khattab, O., Singhvi, A., Maheshwari, P., Zhang, Z., Santhanam, K., Vardhamanan, S., Haq, S., Sharma, A., Joshi, T. T., Moazam, H., Miller, H., Zaharia, M., and Potts, C. (2024).  DSPy: Compiling declarative language model calls into self-improving pipelines.  In International Conference on Learning Representations (ICLR).
  * Kwon et al., (2023) Kwon, W., Li, Z., Zhuang, S., Sheng, Y., Zheng, L., Yu, C. H., Gonzalez, J. E., Zhang, H., and Stoica, I. (2023).  Efficient memory management for large language model serving with PagedAttention.  In Proceedings of the ACM Symposium on Operating Systems Principles (SOSP).
  * Lee and Shen, (2018) Lee, M. R. and Shen, M. (2018).  Winner’s curse: Bias estimation for total effects of features in online controlled experiments.  In Proceedings of the 24th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining, pages 491–499.
  * Li and Roth, (2002) Li, X. and Roth, D. (2002).  Learning question classifiers.  In International Conference on Computational Linguistics (COLING).
  * López-Ibáñez et al., (2016) López-Ibáñez, M., Dubois-Lacoste, J., Pérez Cáceres, L., Birattari, M., and Stützle, T. (2016).  The irace package: Iterated racing for automatic algorithm configuration.  Operations Research Perspectives, 3:43–58.
  * Lush, (1937) Lush, J. L. (1937).  Animal Breeding Plans.  Collegiate Press, Ames, Iowa.
  * Manheim and Garrabrant, (2018) Manheim, D. and Garrabrant, S. (2018).  Categorizing variants of Goodhart’s law.  arXiv preprint arXiv:1803.04585.
  * Novikov et al., (2025) Novikov, A., Vũ, N., Eisenberger, M., Dupont, E., Huang, P.-S., Wagner, A. Z., Shirobokov, S., Kozlovskii, B., Ruiz, F. J. R., Mehrabian, A., Kumar, M. P., See, A., Chaudhuri, S., Holland, G., Davies, A., Nowozin, S., Kohli, P., and Balog, M. (2025).  AlphaEvolve: A coding agent for scientific and algorithmic discovery.  arXiv preprint arXiv:2506.13131.
  * Ohta, (1973) Ohta, T. (1973).  Slightly deleterious mutant substitutions in evolution.  Nature, 246:96–98.
  * Opsahl-Ong et al., (2024) Opsahl-Ong, K., Ryan, M. J., Purtell, J., Broman, D., Potts, C., Zaharia, M., and Khattab, O. (2024).  Optimizing instructions and demonstrations for multi-stage language model programs.  In Proceedings of the Conference on Empirical Methods in Natural Language Processing (EMNLP).
  * Pryzant et al., (2023) Pryzant, R., Iter, D., Li, J., Lee, Y. T., Zhu, C., and Zeng, M. (2023).  Automatic prompt optimization with “gradient descent” and beam search.  In Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing.
  * Qwen Team, (2024) Qwen Team (2024).  Qwen2.5 technical report.  arXiv preprint arXiv:2412.15115.
  * Qwen Team, (2026) Qwen Team (2026).  Qwen3.5-4B and Qwen3.8-27B model cards.  <https://huggingface.co/Qwen/Qwen3.5-4B>, <https://huggingface.co/Qwen/Qwen3.8-27B>.
  * Romera-Paredes et al., (2024) Romera-Paredes, B., Barekatain, M., Novikov, A., Balog, M., Kumar, M. P., Dupont, E., Ruiz, F. J. R., Ellenberg, J. S., Wang, P., Fawzi, O., Kohli, P., and Fawzi, A. (2024).  Mathematical discoveries from program search with large language models.  Nature, 625:468–475.
  * Shawn, (2026) Shawn, Z. (2026).  PACE: Anytime-valid acceptance tests for self-evolving agents.  arXiv preprint arXiv:2606.08106.
  * Shi et al., (2024) Shi, C., Yang, K., Chen, Z., Li, J., Yang, J., and Shen, C. (2024).  Efficient prompt optimization through the lens of best arm identification.  In Advances in Neural Information Processing Systems.
  * Smith and Winkler, (2006) Smith, J. E. and Winkler, R. L. (2006).  The optimizer’s curse: Skepticism and postdecision surprise in decision analysis.  Management Science, 52(3):311–322.
  * Sung et al., (2012) Sung, W., Ackerman, M. S., Miller, S. F., Doak, T. G., and Lynch, M. (2012).  Drift-barrier hypothesis and mutation-rate evolution.  Proceedings of the National Academy of Sciences, 109(45):18488–18492.
  * Wang et al., (2025) Wang, W., Piękos, P., Nanbo, L., Laakom, F., Chen, Y., Ostaszewski, M., Zhuge, M., and Schmidhuber, J. (2025).  Huxley-Gödel machine: Human-level coding agent development by an approximation of the optimal self-improving machine.  arXiv preprint arXiv:2510.21614.
  * Wang et al., (2026) Wang, Y., Zhu, H., Hu, Z., Yuan, Y., Chen, Z., Senthil, S., Hajishirzi, H., Tsvetkov, Y., Dasigi, P., and Xiao, T. (2026).  Rethinking the evaluation of harness evolution for agents.  arXiv preprint arXiv:2607.12227.
  * (44) Xu, C., Yan, N., Chen, L., and Kechadi, M.-T. (2026a).  Phantom gains: Auditing self-improvement against a measured null.  arXiv preprint arXiv:2608.20290.
  * (45) Xu, Y., Zhang, J., Sun, H., Zhou, Z., Cao, T., and Aggarwal, V. (2026b).  Towards reliable LLM evaluation: Correcting the winner’s curse in adaptive benchmarking.  arXiv preprint arXiv:2605.05973.
  * Yang et al., (2025) Yang, A. et al. (2025).  Qwen3 technical report.  arXiv preprint arXiv:2505.09388.
  * Yang et al., (2024) Yang, C., Wang, X., Lu, Y., Liu, H., Le, Q. V., Zhou, D., and Chen, X. (2024).  Large language models as optimizers.  In International Conference on Learning Representations (ICLR).
  * Ye et al., (2026) Ye, Q., Li, Y., Pruksachatkun, Y., Zhang, J., and Wu, C.-S. (2026).  On the fragility of self-improving agents: Variance, task order, and underspecification.  arXiv preprint arXiv:2608.18066.
  * Yin et al., (2024) Yin, X., Wang, X., Pan, L., Lin, L., Wan, X., and Wang, W. Y. (2024).  Gödel agent: A self-referential agent framework for recursive self-improvement.  arXiv preprint arXiv:2410.04444.
  * Yuksekgonul et al., (2024) Yuksekgonul, M., Bianchi, F., Boen, J., Liu, S., Huang, Z., Guestrin, C., and Zou, J. (2024).  TextGrad: Automatic “differentiation” via text.  arXiv preprint arXiv:2406.07496.
  * Zelikman et al., (2024) Zelikman, E., Lorch, E., Mackey, L., and Kalai, A. T. (2024).  Self-taught optimizer (STOP): Recursively self-improving code generation.  In Conference on Language Modeling (COLM).
  * Zhang et al., (2026) Zhang, J., Hu, S., Lu, C., Lange, R., and Clune, J. (2026).  Darwin Gödel machine: Open-ended evolution of self-improving agents.  In International Conference on Learning Representations (ICLR).
  * Zhou et al., (2023) Zhou, Y., Muresanu, A. I., Han, Z., Paster, K., Pitis, S., Chan, H., and Ba, J. (2023).  Large language models are human-level prompt engineers.  In International Conference on Learning Representations (ICLR).

## 附录 A 证明

全文中，$\phi,\Phi$ 是标准正态密度与累积分布函数，而 $Z_{(K)}=\max_{k\leq K}Z_{k}$ 是独立同分布标准正态随机变量的最大值。

#### 引理 A.1（随机效应模型下的后验）。

设 $\hat{\Delta}=\Delta+c\mathbf{1}+\eta$，其中 $\Delta_{k}\stackrel{{\scriptstyle iid}}{{\sim}}N(\mu,s^{2})$，$c\sim N(0,e_{c}^{2})$，且 $\eta_{k}\stackrel{{\scriptstyle iid}}{{\sim}}N(0,e_{\eta}^{2})$，彼此独立。于是 $\mathrm{Cov}(\Delta_{k},\hat{\Delta})=s^{2}e_{k}$，并且 $\Sigma=\mathrm{Var}(\hat{\Delta})=aI+e_{c}^{2}\mathbf{1}\mathbf{1}^{\top}$，其中 $a=\sigma_{w}^{2}=s^{2}+e_{\eta}^{2}$。由 Sherman–Morrison 公式，$\Sigma^{-1}=a^{-1}\big(I-\tfrac{e_{c}^{2}}{a+Ke_{c}^{2}}\mathbf{1}\mathbf{1}^{\top}\big)$，故

$$
\mathbb{E}[\Delta_{k}\mid\hat{\Delta}]=\mu+s^{2}e_{k}^{\top}\Sigma^{-1}(\hat{\Delta}-\mu\mathbf{1})=\mu+h_{w}^{2}(\hat{\Delta}_{k}-\bar{m})+\frac{h_{w}^{2}a}{a+Ke_{c}^{2}}(\bar{m}-\mu),
$$

此即式 (3)。对 $k$ 取平均，得到 $\mathbb{E}[\bar{\Delta}\mid\hat{\Delta}]=\mu+\frac{h_{w}^{2}a}{a+Ke_{c}^{2}}(\bar{m}-\mu)$，因而 $\mathbb{E}[\Delta_{k}-\bar{\Delta}\mid\hat{\Delta}]=h_{w}^{2}(\hat{\Delta}_{k}-\bar{m})$。$\square$

#### 命题 1 的证明。

$\hat{k}$ 是 $\hat{\Delta}$ 的函数，所以引理 A.1 与塔性质给出 $\mathbb{E}[\Delta_{\hat{k}}-\bar{\Delta}\mid\hat{\Delta}]=h_{w}^{2}(\hat{\Delta}_{\hat{k}}-\bar{m})$。代内偏差 $X_{k}=\hat{\Delta}_{k}-c=\Delta_{k}+\eta_{k}$ 独立同分布于 $N(\mu,\sigma_{w}^{2})$，并且 $\hat{\Delta}_{k}-\bar{m}=X_{k}-\bar{X}$，所以 $\mathbb{E}[\hat{\Delta}_{\hat{k}}-\bar{m}]=\mathbb{E}\max_{k}X_{k}-\mathbb{E}\bar{X}=\sigma_{w}\kappa_{K}$。对于高估，$\hat{\Delta}_{\hat{k}}-\Delta_{\hat{k}}=(\hat{\Delta}_{\hat{k}}-\bar{m})-(\Delta_{\hat{k}}-\bar{\Delta})+(\bar{m}-\bar{\Delta})$，并且 $\bar{m}-\bar{\Delta}=c+\bar{\eta}$；取期望得到 $(1-h_{w}^{2})\sigma_{w}\kappa_{K}+\mathbb{E}[c]$。当 $e_{c}=0$ 时，$h_{w}^{2}=h^{2}$，而第一项陈述就是育种者方程。$\square$

#### 命题 2 的证明。

对任意规则 $A(\hat{\Delta})\in\{1,\dots,K,\varnothing\}$，其中 $\Delta_{\varnothing}=0$，有 $\mathbb{E}\Delta_{A}=\mathbb{E}\big[\sum_{k}1\{A=k\}\mathbb{E}[\Delta_{k}\mid\hat{\Delta}]\big]$，其逐点最大由如下选择达到：若最大的后验均值为正，则选择它，否则选择 $\varnothing$。后验均值关于 $\hat{\Delta}_{k}$ 递增（其余各项对所有 $k$ 都相同），所以最大者属于 $\hat{k}$。当 $e_{c}=0$ 时，$\mathbb{E}[\Delta_{k}\mid\hat{\Delta}_{k}]=\mu+h^{2}(\hat{\Delta}_{k}-\mu)>0$ 当且仅当 $\hat{\Delta}_{k}>\mu-\mu/h^{2}=-\mu e^{2}/s^{2}$，这里 $s>0$。当 $s=0$ 时，每一个真实效应都等于已知的 $\mu$，所以当且仅当 $\mu>0$ 时才采纳，与观测无关。$\square$

#### 命题 3 的证明。

当 $s=0$ 时，$\hat{\Delta}_{k}=\mu+c+e_{\eta}Z_{k}$，并且贪心当且仅当 $c+e_{\eta}\max_{k}Z_{k}>|\mu|$ 时才采纳。这一概率随 $K$ 增大而增大（最大值增大），并随 $n$ 增大而减小（$e_{c}^{2}$ 与 $e_{\eta}^{2}$ 按 $1/n$ 缩放）。每一次采纳使 $f$ 恰好改变 $\mu$，并且采纳要求 $\hat{\Delta}_{\hat{k}}>0$。在选择集是全新的、并且提议分布与噪声分布保持不变时，这些事件跨代独立同分布，从而给出每代 $\mu p$ 的期望漂移。该陈述局部于这样一种平稳机制。$\square$

#### 一种启发式的分辨尺度（第 3 节）。

设提议分布依赖于现任解的表现 $f$，且 $b(f)=-\mu/s>0$（提议平均而言有害）。在独立噪声以及命题 2 的规则下，参数为 $(s,b,e)$ 的状态上的期望一步增益为 $G^{\star}=s\,\psi_{K,b}(s/e)$，其中 $\psi_{K,b}(r)=\mathbb{E}[(\rho Z_{(K)}-b)^{+}]$，$\rho=r/\sqrt{1+r^{2}}$。在标准化阈值 $\lambda=b/\rho=-\mu\sqrt{s^{2}+e^{2}}/s^{2}$ 超过 $\sqrt{2\ln(KT)}$ 的那些状态上，这一增益可以忽略：即便是 $T$ 个这样的步，累加起来也小于 $s$ 的一个小倍数。我们只把 $\lambda$ 达到该值的那个水平用作定性的、局部的分辨尺度。当 $\lambda$ 在交叉附近随表现上升时，该水平在噪声主导机制（$v/n\gtrsim s^{2}$）中随 $n$ 上升。它对 $K$ 与 $T$ 的依赖只经由 $\sqrt{2\ln(KT)}$，并且当 $n\to\infty$ 时只由提议分布决定，这类似于漂变屏障（Sung et al., 2012），也类似于含噪进化策略的进展极限（Arnold and Beyer, 2006）。它并不是循环在 $T$ 代之内所达到水平的一个界。这样的界需要对一切可达状态的一致控制，因为 $s$、$b$ 与 $\rho$ 随现任解而改变，准确率既可能下降，也可能上升。一步陈述的推导如下。当 $e_{c}=0$、$\mu=-bs$ 并采用命题 2 的规则时，一代的增益为 $\mathbb{E}[(\mu+h^{2}(\hat{\Delta}_{\hat{k}}-\mu))^{+}]$。由 $\hat{\Delta}_{\hat{k}}=\mu+\sigma Z_{(K)}$ 以及 $h^{2}\sigma=s\rho$、$\rho=s/\sigma=r/\sqrt{1+r^{2}}$，此式等于 $s\,\mathbb{E}[(\rho Z_{(K)}-b)^{+}]=s\psi_{K,b}(r)$。由于 $(\rho Z_{(K)}-b)^{+}\leq\sum_{k}(\rho Z_{k}-b)^{+}$，令 $t=b/\rho$，再由对 $t>0$ 成立的 Mills 比界 $1-\Phi(t)\geq\phi(t)(1/t-1/t^{3})$，得到 $\psi_{K,b}\leq K\rho[\phi(t)-t(1-\Phi(t))]\leq K\rho\phi(t)/t^{2}=K\rho^{3}\phi(b/\rho)/b^{2}$。因此，在 $b/\rho\geq\lambda_{0}$ 的状态上，期望一步增益至多为 $Ks\rho^{3}e^{-\lambda_{0}^{2}/2}/(\sqrt{2\pi}b^{2})$，而一旦 $\lambda_{0}\geq\sqrt{2\ln(KT)}$，$T$ 个这种大小的增益之和就小于 $s\rho^{3}/(\sqrt{2\pi}b^{2})$。特例：若 $b$ 为常数，且 $s=a(f^{\star}_{\infty}-f)^{\beta}$，$a,\beta>0$，则 $\lambda$ 达到 $\sqrt{2\ln(KT)}$ 的那个水平在噪声主导机制中满足 $f^{\star}_{\infty}-f^{\star}_{n}\approx\big(b^{2}v/(2a^{2}n\ln(KT))\big)^{1/(2\beta)}$；若 $s$ 为常数，$b(f^{\star}_{\infty})=\sqrt{2\ln(KT)}$，并且 $b$ 在 $f^{\star}_{\infty}$ 附近可微且 $0<b^{\prime}(f^{\star}_{\infty})<\infty$，则局部反演给出 $f^{\star}_{\infty}-f^{\star}_{n}\propto(1-\rho(n))\approx v/(2ns^{2})$。

#### 注记（模型遗漏了什么）。

(i) 候选噪声是异方差的（$v_{k}$ 既依赖于效应，也依赖于候选与现任的不一致率），并且真实效应可以是重尾的；第 5 节中依分布的预测用经验先验替换了高斯先验。(ii) 在复用的选择集上，共享误差的均值可以不为零，而根据在 $D$ 上的失败所形成的提议可以适应 $D$ 本身。仅凭现任膨胀并不能识别净增益误差；第 5.2 节报告了与锁定相一致的模式。(iii) 随机的逐条目奖励（智能体任务）向 $v$ 增加一个条目内的方差项。

## 附录 B 实验细节

### B.1 循环、提示与规则

#### 求解器提示。

产物是系统消息；用户消息是固定的：

> Possible labels: ABBR:abb, ABBR:exp, … (all labels)
> Question: {question}
> Label:

（Banking77：原文用 “Possible intents” / “Customer message” / “Intent”；GSM8K：原文用 “Question: {question}”。）解码是贪心的（分类 48 个新词元，GSM8K 为 320 个）。分类预测若第一行与某个标签匹配，则取第一行，否则取输出中的最后一个标签字符串；GSM8K 预测取紧随 “####” 之后的最后一个数，若不存在该标记，则取紧随 “answer is” 之后的最后一个数，否则取最后一个数。

#### 提议器提示。

提议器（除非另有说明，就是求解器本身）收到

> A small language model is given the following INSTRUCTION as its system prompt and then solves one task input. TASK: {task description}. CURRENT INSTRUCTION: <<<{artifact}>>>. Here are inputs where the model made mistakes with the current instruction: {4 failures: input, model output, correct answer}. Write an improved INSTRUCTION that will make the model more accurate on this task in general (not only on these examples). You may add strategies, rules, definitions, or output-format requirements. Keep it under 120 words. Return only the new instruction between <<< and >>>.

系统消息为 “You are an expert prompt engineer who improves instructions for a smaller language model”，温度 0.9，top-$p$ 为 0.95，至多 400 个新词元。一条提议是有效的，若它包含一个由 <<< 打开、由 >>>（或第二个 <<<）关闭的完整块，且至多 180 个词；无效提议与重复提议由新的样本替换（探索性运行中为 3 轮，验证性运行中为 5 轮）。即便如此，有些代的有效候选仍少于预定的 $K$ 个：探索性运行中为 26.0%（1.5B，vLLM）、25.8%（GSM8K）、0.0%（7B）与 39.3%（1.5B，llama.cpp），验证性运行中为 6.6%，而 14B 与 Qwen3.5-4B 提议器几乎没有这种情况（各臂见表 3）。探索性运行中，失败从选择集抽样；验证性运行中，失败来自一个单独的、含 256 个条目的反馈池（与 $n$ 无关）。

#### 种子指令与任务描述。

种子指令是 “Classify the question.”（TREC）、“Classify the customer message.”（Banking77）以及 “Answer the question.”（GSM8K）。展示给提议器的任务描述原文是：TREC，“Classify a question by the type of answer it seeks into one of 50 fine-grained TREC labels of the form COARSE:fine (the label list is given in the input). The first line of the response is matched against the label set; otherwise the last label mentioned is used.”；Banking77，“Classify an online-banking customer message into one of 77 intent labels (the label list is given in the input). The first line of the response is matched against the label set; otherwise the last label mentioned anywhere in the response is used.”；GSM8K，“Grade-school math word problems. The final numeric answer is extracted from the response (a number after ‘####’ if present, otherwise the last number in the response) and compared to the reference.”第 5.4 节的强起点循环从下列指令开始，原文照录（换行与原文一致）。

> TREC: Classify question intent using TREC COARSE:FINE labels. Strictly match the fine-grained label to the question type:
> 1\. Entities: COARSE:ENTY (words, names, objects). If asking for a specific *instance* (e.g., "which novel?", "what nickname?"), use ENTY:cremat. If asking for an *event occurrence* (e.g., "when did X happen?"), use ENTY:event.
> 2\. People: HUM:ind for specific individuals/nicknames; HUM:gr for groups/organizations.
> 3\. Attributes: DESC:def for abstract meanings; DESC:desc for general facts/origin; NUM:date for specific timepoints.
> 4\. Rules: Prioritize the question’s target entity over descriptive content. Output ONLY the label.
>  Banking77: You are an intent classifier. Map the input to exactly one label from the provided list. Prioritize the MOST SPECIFIC label describing the exact transaction event or specific charge (e.g., ’topping_up_by_card’ for card top-up failures, ’wrong_exchange_rate_for_cash_withdrawal’ for exchange errors). Avoid broad categories like ’fee charged’ if a specific event label exists. Match the core user issue precisely, ignoring filler. If no exact match, choose the most semantically similar. Output ONLY the label name on the first line. Do not use explanations, quotes, or markdown.

强起点 GEPA 程序（755 与 2151 个词）在补充材料包中。

#### 池与划分。

一项任务的池按公开发布的顺序拼接其公开的训练划分与测试划分；划分 $s$ 的条目 $i$ 的标识符为 task-s-i（Banking77 带前缀 b77；TREC：CogComp/trec，细标签；Banking77：PolyAI/banking77；GSM8K：openai/gsm8k，配置 main；前两者取自 refs/convert/parquet 修订版）。使用 Python 的 random 模块，探索性金标准集由以 Random(1234) 打乱该池之后的前 600 个条目组成。保持其余条目处于该打乱顺序，再用 Random(2027) 将它们重新打乱，并取前 600 个作为验证性金标准集。选择池在探索性运行中是去掉探索性金标准集之后的池，在其余情况下是去掉两套金标准集之后的池。在种子为 $r$ 的循环中，Random(r) 先抽取含 256 个条目的反馈池（验证性设计），再对剩余条目给出一个固定顺序，其中前 $n$ 个条目构成选择集，因此选择集随 $n$ 嵌套。GEPA 与 MIPROv2 用 Random(1000+r) 打乱验证性选择池，取前 256 个条目作为训练池，再取其后的 $n_{\mathrm{val}}$ 个作为验证集。补充材料包列出条目池的顺序以及所记录的划分标识符；标识符未被保存的优化器划分，可以依据清单、任务与运行种子，用上述过程重建。模型权重是正文中点名的 Hugging Face 发行版（Qwen/Qwen2.5-{1.5B,7B,14B}-Instruct、Qwen/Qwen3.5-4B、Qwen/Qwen3.8-27B-FP8）；我们没有固定它们的修订版。

#### 重复问题审计。

互不相交的条目标识符并不能保证问题文本互不相同。一次事后审计发现，验证性金标准集与其剩余池共享 12 条完全相同的 TREC 问题文本；144 次 TREC 验证性运行中有 58 次在其选择集或反馈集中包含一条匹配的问题，每次运行至多影响 600 个金标准条目中的两个。去掉全部 12 条匹配的金标准条目后剩下 588 个条目，并且主要结论保持不变：TREC 上更大集合的优势为 10.44 与 6.21 分（Holm 校正的 $p=0.010$ 与 $<0.001$），合并的预实验相对贪心比较为 $-2.10\pm 1.08$ 分（$p=0.061$）。补充材料包包含重复标识符以及一份可复现的敏感性分析。没有任何原始运行记录或主要估计被替换。Banking77 在裁剪周围空白之后有一条金标准集与剩余池的文本重复，它没有出现在任何一次验证性选择集或反馈集中；GSM8K 则没有。因此，留出指的是条目 ID 的分离，并带有已记录的 TREC 例外，即内容分离并不成立。运行记录中缺失的一个最终条目向量，由保留下来的缓存模型输出重建；其总分与所记录的分数一致，这在非确定性推断下并不能确立逐条目的同一性。对该次运行而言，在被排除的 12 个条目上全部 13 种可能的正确答案个数之下，敏感性结论仍然成立，并且不依赖这一重建。

#### 规则细节。

McNemar：对最佳候选的单侧精确检验，没有对在 $K$ 个候选之中所作的选择进行校正。PACE（决策级分析）：已发表的检验，对每个不一致条目下固定赌注 $\lambda=1/2$（平局跳过），一旦财富在 $n$ 个条目之内达到 $1/\alpha$（$\alpha=0.05$）就采纳，若条目先耗尽则拒绝；它至少需要比失败多出八次成功，而一个 16 条目的集合很少能提供这一点。贝叶斯门使用 $\hat{v}=\overline{\max\{\hat{q}_{k}-\hat{\Delta}_{k}^{2},1/n\}}$ 与 $\hat{v}_{cc}=\min\{\hat{v},\tfrac{1}{2}\overline{\max\{\hat{q}_{jk}-(\hat{\Delta}_{j}-\hat{\Delta}_{k})^{2},1/n\}}\}$，分别对候选以及对候选对取平均。只有一个候选时，实现使用 $\hat{v}_{cc}=0.6\hat{v}$。这些数值下限与上界先于第 4 节中的噪声分解；原生采纳分析改用未加下限的矩。先验按阶段索引，因为改写一行式的种子常常有益，而此后的改写大多无益。还要求测得的增益为正；这一要求是在如下情况之后加上的：在线门在我们的开发种子上、当其对 $s^{2}$ 的估计坍缩时，采纳了测得的退步（附录 C）。探索性完整运行中的 E-过程：一个可预测的 GRAPA 式赌注 $\lambda_{j}=\min\{1/2,\max(0,\bar{d}_{j}/(\hat{\sigma}^{2}_{j}+\bar{d}_{j}^{2}))\}$，由此前的条目计算，当且仅当财富的运行最大值达到 $1/\alpha$ 时才采纳。在线经验贝叶斯门：窗口为 8 代；第一代回退到 McNemar；当 $K=1$ 时，$s^{2}$ 取跨代方差减去总噪声。预实验校准门：按阶段索引的先验（第 1 代相对此后各代），来自同一模型与同一任务的神谕运行的留出效应，在探索性金标准集上测量（与验证性金标准集不相交）。以百分点计，第 1 代与此后各代的 $(\mu,s)$ 为：1.5B TREC 上 $(5.6,8.6)$ 与 $(-9.4,7.1)$，1.5B Banking77 上 $(-6.8,6.1)$ 与 $(-4.6,5.1)$，7B TREC 上 $(4.9,4.8)$ 与 $(-7.2,7.2)$，以及 1.5B GSM8K 上 $(-7.7,5.1)$ 与 $(-2.8,7.6)$。三次 1.5B TREC 源运行之一（llama.cpp，20 代）在先验被冻结时仍在运行，所以其先验使用该运行的前 17 代；若用全部 20 代，此后各代的值将为 $(-8.9,7.0)$。*有害采纳*：在金标准集上留出效应为负的一次采纳；金标准集把成对差异分辨到大约 1.5–2 分，所以我们也报告差于 $-1.8$ 分的采纳。

冻结的预实验估计量从代内散布中减去了全部的候选–现任抽样方差。在假设 1 下，对候选作中心化时，共享的现任误差会相消，所以这一历史近似减去了过多的噪声。我们保留这些冻结值，因为它们支配了已经完成的实验，以及基于预实验的采纳参照。图 1 中的事后噪声模型分析，以及表 2 中的提议散布，改为减去候选–候选噪声，权重与合并后的代内方差相匹配。经验分布预测器在每一个模拟代中重抽样四个效应偏差；当某个观测到的代拥有更少的有效候选时，这是一种固定 $K$ 的近似。图 1 的观测区间，是从用于所绘观测的同一 200 个划分中，对代作自助法得到的。

### B.2 设计

#### 探索性运行（v1）。

金标准集：每项任务 600 个条目（固定的随机子集，种子 1234）。选择集跨代复用，并且对给定种子随 $n$ 嵌套。主栈：一台租用的 A100-SXM4-80GB 上的 vLLM 0.30.0（bf16），$T=20$ 代，3 个种子。神谕装备的运行（每一个候选都在金标准集上打分）使用贪心接受，$n=256$，$K=4$：1.5B-TREC、1.5B-Banking77 与 GSM8K 各一次（$T=20$、$T=20$、$T=15$），7B-TREC 与 7B-Banking77 各两次（$T=15$）。复现栈：一台共享的 RTX 3080 Ti 上的 llama.cpp（构建 b11211，8 位 GGUF），$T=25$（每项任务两次神谕运行，$T=20$）。

#### 验证性运行（v2）。

在启动之前，以一份注明日期的书面计划作了预注册：每项任务一套新的、含 600 个条目的金标准集，与 v1 金标准集不相交；种子 11–18；含 256 个条目的反馈池，与选择集及金标准集不相交；各臂为贪心（$n=16,64,256$）、McNemar、预实验校准的贝叶斯门，以及先选后确认（$n=16,64$）；设定为 1.5B-TREC、1.5B-Banking77、7B-TREC、1.5B-GSM8K；更强的提议器：由 Qwen2.5-14B 为 1.5B 求解器作提议（贪心，$n=16,64,256$，种子 11–15，每项任务一次神谕运行）。vLLM 在一台 A100 上，同上。

#### 相对预注册的偏离。

(i) 预注册文件，包括其修正，以及预实验先验，最后修改于 15:54；第一次运行于 16:07 开始。修正自身的页眉给出了错误的时间（「16:15」）。(ii) 实现预注册检验的分析脚本是在启动十分钟之后写成的（文件创建于 16:17:46，当时 336 次运行中已有 16 次完成），于 16:17:50 归档，并于 16:17:52 在这 16 次运行上执行。该次运行打印了各臂摘要，并在输出步骤因一个序列化错误而失败；修复该错误是对脚本所作的唯一改动。(iii) H4 与 H5 的合并检验被实现为对 32 个种子配对差值的单样本 $t$ 检验，而不是带有设定固定效应；带有设定固定效应时，$p=0.054$（H4）与 $0.17$（H5），结论没有改变。(iv) 驱动这些运行的机器发生了一次内存故障，使一次运行（1.5B GSM8K，贪心，$n=256$，种子 13）在第 6 代停滞；它被从头重跑，其余运行被移到 GPU 服务器本身上的驱动进程（代码与模型服务器相同）。(v) 有效候选少于 $K=4$ 的代所占的比例，也就是计划要求我们报告的那个量，按臂给在表 3 中。(vi) 计划中承诺的次要分析：贪心循环的 $n$ 与留出增益之间的 Spearman 相关，在 1.5B TREC 上为 0.62（$p=0.001$），在 7B TREC 上为 0.74（$p<0.001$），在 GSM8K 上为 0.17（$p=0.42$）；$n=16$ 时，预实验门减去 McNemar 的 95% 区间，对 1.5B TREC、Banking77、7B TREC 与 GSM8K 分别为 $[-5.1,17.3]$、$[-0.7,1.7]$、$[3.8,15.8]$ 与 $[-8.5,0.0]$ 分，而在 $n=64$ 时为 $[-1.6,13.0]$、$[-4.2,1.1]$、$[-0.2,4.6]$ 与 $[-10.9,1.7]$。(vii) 研究结束之后由观测到的方差算出的敏感性数字：合并的 H4 检验本可以以 80% 的功效检测到 3.2 分的改进（各设定为 2.4 到 9.9 分；H1 为 2.3 到 8.5；H2 为 8.9 到 15.4）。

#### 当前模型与 GEPA。

GEPA 运行使用 DSPy 3.4.0（dspy.GEPA，gepa 0.1.4），除预算（4000 次指标调用）、reflection_minibatch_size=3 与 track_stats 之外均为默认设置。程序是单个 Predict 模块（TREC、Banking77；标签列表是一个固定的输入字段），或一个 ChainOfThought 模块（GSM8K），其指令从那一句单句种子开始。任务模型 Qwen3.5-4B 贪心解码；反思模型 Qwen3.8-27B（FP8 权重）在温度 1 下抽样；两者都关闭思考。训练池（256 个条目）与验证集从排除了两套金标准集的任务池中抽取，验证集对一个种子随 $n_{\mathrm{val}}$ 嵌套。在分析任何 GEPA 结果之前，发现并修复了两个问题，并且每一次修复之后全部运行都从头重启。默认的 1024 个打开文件的上限，使语言模型调用在运行中途失败（DSPy 把一次失败的调用记为错误答案），而 8192 个词元的上下文窗口在演化后的指令变长之后使反思失败。最终运行对反思模型使用 32768 个词元的窗口，并且不含基础设施错误。$n_{\mathrm{val}}=16$ 的运行进展缓慢（每一次迭代都要等待一次反思），所以主要比较使用 1000 次指标调用的共同预算，由每一次运行所保存的状态精确重建；$n_{\mathrm{val}}=16$ 的运行在 1030 次调用之后停止，五次 $n_{\mathrm{val}}=64$ 的 GSM8K 运行在 2100 到 2900 次之后停止。留出集为起始程序、返回的程序、按验证分数计的另外五个最佳候选，以及随机抽取的五个候选打分；因此遗憾是一个下界。无法解析的答案计为错误，与 GEPA 自身的指标一致。运行之前记入注明日期的研究日志的计划（17:57）规定了全预算，以及全体候选的留出分数；出于运行时间的原因，预算与打分样本在任何一次 $n_{\mathrm{val}}\leq 64$ 的运行完成之前就被改动。GEPA 的报告增益（返回程序的验证分数减去起始程序的验证分数）是在前八次运行全部完成、并且其结果已经展示之后才作为一项度量加上的；这八次运行全都是 $n_{\mathrm{val}}=256$ 且为全预算（19:37–19:38；会话记录显示，分析脚本当时已经为它们计算了报告增益），并且其中一次运行曾在 1000 次调用处被重建，以测试重建脚本（20:09），这发生在预算改动（20:15）之前。在这些时点的任何一个，都还没有看到任何 $n_{\mathrm{val}}\leq 64$ 的结果。预先计划的膨胀度量与报告增益带来同一项发现。GEPA 只在开始一次迭代时检查其预算，并在完整的验证遍之前记录一个候选的发现计数，所以一次停在 1000 次调用的运行，也可能包含一个计数为 1001 到 1006 的候选。在有完整记录的 21 次运行中，存在两个这样的候选，两者都不改变所返回的程序。对 $n_{\mathrm{val}}=16$ 的运行，服务器被释放时保存的状态丢失了，但运行日志记录了每一次评估，以及每一个被接受候选的验证分数。仅从日志重建每一个池，就复现了全部 45 次在 1000 次调用处的重建的池、发现计数、验证分数与返回程序，包括这 15 次。同一批日志给出在 1000 次指标调用之内作出的反思调用：验证条目为 16 时，平均每次运行 38 到 45 次（按任务计），为 64 时是 14 到 16 次，为 256 时是 3 到 5 次，所以比较在指标调用上是匹配的，而不是在反思计算量上匹配。每一次重启之前已经完成的运行的输出，包括四个已经在留出集上打过分的，被保留下来，但没有加以分析。Qwen3.5-4B 的自我改进运行原样使用第 5 节的循环（同样的提示、反馈池与验证性金标准集），以 Qwen3.5-4B 同时作为求解器与提议器：在 TREC 与 Banking77 上，以种子 11–14 作贪心，$n=16,64,256$，并且每项任务有一次神谕装备的运行（$n=256$，$T=15$）。预算约束使我们放弃了计划中的一臂，即由 Qwen3.8-27B 为 1.5B 求解器作提议。

#### 后续研究。

应评审意见增加了四项研究，每一项都在其第一次运行之前写入日志；全部使用 GEPA 研究的软件（vLLM 0.30.0、DSPy 3.4.0、gepa 0.1.4），在两台 H100 实例上运行。(i) 循环与 GEPA 的强起点（附录 C）。(ii) 从原始种子出发、带 16 个验证条目的 GEPA，种子 1–5，在 TREC 与 Banking77 上朝 3000 次调用运行，并与在同一预算处截断的、带 64 个与 256 个条目的全预算运行相比较。从候选记录截断一次已经完成的运行，其精确性与从保存状态重建的理由相同，并且它复现了全部 17 次基于状态、在 1000 次调用处的重建，这些重建中被选中的候选已经作过留出打分，包括留出向量；没有留出分数的被选候选，用同一个评估器打分。(iii) MIPROv2（dspy.MIPROv2，optuna 5.0.0），仅优化指令，有 16 条被提议的指令与 24 次贝叶斯优化试验，每一次都在完整的验证集上打分，模型、训练池与验证集与 GEPA 相同，任务为 TREC 与 Banking77，$n_{\mathrm{val}}\in\{16,256\}$，种子 1–5；留出集为种子程序、返回的程序以及其他四个已评估候选打分。(iv) 强起点循环的另外四个种子，是在前四个种子的结果已经被看到、并被另行报告之后才决定的。在 04:30 仍在运行的 GEPA 运行被停止，并在其所属组的每一次运行都已经达到的、250 次调用的最大倍数处重建。这条规则在看到这些运行的任何留出分数之前就已经固定，尽管某一次运行的第 2 次迭代的临时验证分数曾经可见。

#### 推断的非确定性。

批解码并非按位可复现。在一次运行之内，每一个（指令，条目）对只打分一次并被缓存，所以一个产物有一个分数向量。跨运行时，种子指令的金标准向量差异如下（翻转条目的最大成对占比）。在探索性运行中：llama.cpp 为 0%，vLLM 在 Banking77 上为 0%，vLLM 在 TREC 上至多为 1%，GSM8K 上为 9.3%。在验证性运行中，它们是在更重、也更多变的服务器负载下进行的：0.7%（1.5B TREC）、1.2%（Banking77）、1.5%（7B TREC），GSM8K 上为 9.8%，其中种子的留出准确率在各次运行之间从 61.0% 到 64.3%。不同的 GEPA 服务器同样给出略有差别的分数。长的生成对批次组成最为敏感。对 GSM8K 而言，这一条目级噪声是 $v$ 的一个可观部分，也是那些「真实」效应的一个可观部分，并且它是 GSM8K 位居图 1 中最不一致的设定之列的可能原因。预注册的 GSM8K 比较对它并不敏感。若把每一次运行相对平均的种子准确率来度量，而不是相对它自己的种子准确率，则预实验相对贪心的差值从 $-5.75$ 变为 $-5.71$ 分（两种方式都是 $p=0.008$）。我们还把 72 次 GSM8K 运行的种子，以及全部 59 条互不相同的最终指令，重新打分三次，每一次都放在一个批次里，并绕过响应缓存，用的是一台 H100，而不是本研究的 A100；种子的分数为 62.3%、63.7% 与 63.3%，一条最终指令的分数在各次重复之间变化 0.7 分，一次运行的增益平均移动 0.7 分。用平均之后的重打分增益，GSM8K 上的 H1 为 $+1.8\pm 2.0$ 分（$p=0.40$；原先为 $+1.6$，$p=0.48$），预实验相对贪心的差值（H4）为 $-5.6\pm 1.7$（$p=0.013$；原先为 $-5.75$，$p=0.008$）。

#### 数据与许可。

TREC（Li and Roth, 2002）（5,952 个条目，50 个细标签；CogComp/trec），Banking77（Casanueva et al., 2020）（13,069 个条目；CC-BY-4.0；PolyAI/banking77），GSM8K（Cobbe et al., 2021）（8,792 个条目；MIT）。Qwen2.5、Qwen3.5 与 Qwen3.8 模型在 Qwen 许可或 Apache-2.0 许可下发布。计算：一块消费级 RTX 3080 Ti（llama.cpp 复现），以及大约 28 个租用的 GPU 小时（两台 A100-80GB 与四台 H100-80GB 实例，vLLM 0.30），其中大约 11 个小时用于后续研究。

## 附录 C 补充结果

#### 提议分布。

表 2 汇总了带 oracle 插桩的运行中所有候选的留出集效应,其中包括 14B 提议者的运行(验证性研究;效应基于验证性金标准集)。

表 2:带 oracle 插桩运行的提议分布:候选数量;相对于其在位者(incumbent)具有正留出集效应的候选占比,以及平均留出集效应 $\mu$(百分点),分别针对种子的首次改写和后续各代;代内离散度 $s$(已针对金标准集噪声校正);$v_{cc}/v$;以及代内偏差的超额峰度。

| 提议者 | 求解器,任务 | 候选数 | 第 1 代:更优占比 | 第 1 代:$\mu$ | 后续各代:更优占比 | 后续各代:$\mu$ | $s$ | $v_{cc}/v$ | 超额峰度 |
|---|---|---|---|---|---|---|---|---|---|
| 1.5B | 1.5B TREC | 64 | 100% | +6.8 | 8% | -13.5 | 7.1 | 0.44 | -0.4 |
| 1.5B | 1.5B Banking77 | 77 | 0% | -11.5 | 4% | -10.0 | 8.2 | 0.60 | -0.2 |
| 1.5B | 1.5B GSM8K | 55 | 0% | -7.7 | 39% | -2.3 | 7.5 | 0.52 | 3.4 |
| 7B | 7B TREC | 120 | 100% | +4.9 | 15% | -7.2 | 7.1 | 0.58 | 2.7 |
| 7B | 7B Banking77 | 120 | 25% | -0.3 | 38% | -0.2 | 0.6 | 0.57 | 2.5 |
| 1.5B | 1.5B TREC (llama.cpp) | 133 | 75% | +5.0 | 20% | -6.5 | 7.2 | 0.55 | 0.2 |
| 1.5B | 1.5B Banking77 (llama.cpp) | 152 | 25% | -4.5 | 26% | -1.9 | 2.7 | 0.56 | 3.2 |
| 14B | 1.5B TREC | 60 | 75% | +10.0 | 9% | -11.0 | 9.5 | 0.57 | -0.1 |
| 14B | 1.5B Banking77 | 60 | 25% | -6.7 | 20% | -1.4 | 2.5 | 0.55 | 7.6 |
| 14B | 1.5B GSM8K | 60 | 50% | -4.1 | 20% | -4.0 | 5.6 | 0.52 | 0.6 |
| Qwen3.5-4B | Qwen3.5-4B TREC | 60 | 100% | +25.9 | 7% | -9.8 | 7.0 | 0.44 | 0.5 |
| Qwen3.5-4B | Qwen3.5-4B Banking77 | 60 | 100% | +14.0 | 20% | -1.0 | 0.9 | 0.48 | 0.1 |

#### 收益从何而来。

在以一句话作为种子的情况下,一个循环的最终收益中有很大一部分来自其第一次提交。对于贪心循环,第 1 代之后的收益占最终收益的比例为:Qwen2.5 模型在 TREC 上为 38% 至 54%(因此后续各代仍使收益翻了一倍);Qwen2.5-14B 担任提议者时在 TREC 上为 70% 至 78%;Qwen3.5-4B 为 90% 至 112%,其后续提交平均而言没有增加任何收益(Qwen2.5 在 Banking77 和 GSM8K 上的最终收益太小,无法计算比例)。第 5.4 节的强起点运行在构造上去除了首次改写阶段。

#### 验证性运行,全部实验组。

表 3 列出了预注册研究的每一个实验组:相对于种子指令的最终留出集收益,以及最终的代理集-留出集差距(各随机种子上的均值 ± 标准误),有害提交(留出集效应 $<0$,以及 $\leq -1.8$ 个百分点,即一个金标准集标准误)数量占全部提交数量的比例(对随机种子求和),以及有效候选少于 $K=4$ 个的代所占比例。

表 3:验证性(v2)运行:全部实验组。

| 设置 | 规则($n$) | 运行数 | 留出集收益 | 代理集−留出集 | 有害 / 提交 | $\leq -1.8$ 点 | $<K$ 个候选 |
|---|---|---|---|---|---|---|---|
| 1.5B TREC | Greedy (16) | 8 | +13.7±2.9 | +17.3±2.5 | 4 / 22 | 3 | 19% |
|  | Greedy (64) | 8 | +18.5±1.5 | +8.0±1.2 | 8 / 35 | 5 | 20% |
|  | Greedy (256) | 8 | +24.2±0.7 | +1.4±0.8 | 8 / 46 | 4 | 12% |
|  | McNemar (16) | 8 | +6.3±4.1 | +5.9±2.4 | 0 / 2 | 0 | 2% |
|  | McNemar (64) | 8 | +14.6±2.7 | +6.2±2.5 | 2 / 12 | 1 | 9% |
|  | Select-confirm (16) | 8 | +15.8±3.2 | +11.3±4.8 | 0 / 10 | 0 | 13% |
|  | Select-confirm (64) | 8 | +17.7±1.9 | +7.0±1.3 | 3 / 23 | 2 | 13% |
|  | Pilot gate (16) | 8 | +12.4±2.9 | +13.9±3.1 | 1 / 12 | 1 | 11% |
|  | Pilot gate (64) | 8 | +20.2±1.0 | +5.5±1.9 | 4 / 22 | 2 | 9% |
| 1.5B Banking77 | Greedy (16) | 8 | -0.6±0.7 | +13.2±3.7 | 5 / 12 | 4 | 20% |
|  | Greedy (64) | 8 | +0.0±0.5 | +5.3±1.9 | 6 / 15 | 2 | 10% |
|  | Greedy (256) | 8 | -0.5±0.5 | +2.7±1.1 | 12 / 21 | 2 | 4% |
|  | McNemar (16) | 8 | +0.0±0.0 | +0.0±4.8 | 0 / 0 | 0 | 1% |
|  | McNemar (64) | 8 | -0.2±0.2 | +1.4±2.0 | 1 / 1 | 0 | 4% |
|  | Select-confirm (16) | 8 | -2.3±1.7 | +10.2±4.8 | 3 / 5 | 3 | 2% |
|  | Select-confirm (64) | 8 | -1.1±0.9 | +3.9±2.3 | 4 / 4 | 1 | 4% |
|  | Pilot gate (16) | 8 | +0.5±0.5 | +10.5±4.7 | 2 / 8 | 2 | 4% |
|  | Pilot gate (64) | 8 | -1.7±1.1 | +7.2±2.0 | 7 / 10 | 2 | 8% |
| 7B TREC | Greedy (16) | 8 | +14.0±0.7 | +16.6±4.4 | 3 / 19 | 1 | 0% |
|  | Greedy (64) | 8 | +19.8±0.5 | +5.4±1.1 | 5 / 34 | 0 | 0% |
|  | Greedy (256) | 8 | +20.4±0.6 | +1.3±0.8 | 16 / 54 | 3 | 0% |
|  | McNemar (16) | 8 | +1.8±1.8 | +8.6±4.5 | 0 / 1 | 0 | 0% |
|  | McNemar (64) | 8 | +13.6±1.3 | +1.9±1.4 | 0 / 8 | 0 | 0% |
|  | Select-confirm (16) | 8 | +7.6±1.9 | +17.5±3.7 | 1 / 10 | 0 | 0% |
|  | Select-confirm (64) | 8 | +15.9±0.9 | +4.0±1.2 | 3 / 21 | 0 | 0% |
|  | Pilot gate (16) | 8 | +11.5±2.4 | +16.8±3.5 | 3 / 15 | 3 | 0% |
|  | Pilot gate (64) | 8 | +15.8±1.7 | +5.8±0.9 | 1 / 23 | 1 | 0% |
| 1.5B GSM8K | Greedy (16) | 8 | +1.5±1.7 | +19.6±4.5 | 7 / 16 | 5 | 8% |
|  | Greedy (64) | 8 | -1.1±1.1 | +12.2±2.2 | 15 / 27 | 12 | 9% |
|  | Greedy (256) | 8 | +3.1±1.0 | +4.9±1.0 | 13 / 29 | 8 | 12% |
|  | McNemar (16) | 8 | +0.0±0.0 | +1.7±3.5 | 0 / 0 | 0 | 1% |
|  | McNemar (64) | 8 | +2.7±1.8 | +7.7±2.9 | 2 / 7 | 1 | 8% |
|  | Select-confirm (16) | 8 | +1.1±1.3 | +13.2±5.1 | 1 / 6 | 1 | 5% |
|  | Select-confirm (64) | 8 | +1.9±2.0 | +8.8±2.9 | 6 / 12 | 3 | 6% |
|  | Pilot gate (16) | 8 | -4.3±1.8 | +22.2±4.9 | 9 / 15 | 8 | 6% |
|  | Pilot gate (64) | 8 | -1.9±1.3 | +12.6±2.8 | 13 / 27 | 12 | 15% |
| 14B→1.5B TREC | Greedy (16) | 5 | +26.1±1.5 | +22.3±6.5 | 4 / 15 | 3 | 0% |
|  | Greedy (64) | 5 | +32.2±1.7 | +8.1±2.4 | 5 / 24 | 4 | 0% |
|  | Greedy (256) | 5 | +32.8±2.0 | -0.8±1.5 | 5 / 24 | 1 | 0% |
| 14B→1.5B Banking77 | Greedy (16) | 5 | -2.3±1.1 | +13.7±1.3 | 6 / 9 | 4 | 0% |
|  | Greedy (64) | 5 | -1.3±0.9 | +9.0±1.0 | 9 / 14 | 3 | 0% |
|  | Greedy (256) | 5 | +1.6±0.5 | +3.2±1.0 | 5 / 16 | 2 | 0% |
| 14B→1.5B GSM8K | Greedy (16) | 5 | -3.3±3.5 | +21.0±5.1 | 5 / 8 | 4 | 0% |
|  | Greedy (64) | 5 | +2.4±2.2 | +10.6±2.7 | 3 / 9 | 2 | 0% |
|  | Greedy (256) | 5 | +3.6±1.0 | +4.3±0.6 | 2 / 13 | 1 | 0% |

#### 按代数和选择集大小划分的在位者膨胀。

表 4 给出了第 5.2 节所述的在位者膨胀。在 vLLM 的 oracle 运行中,在被重复使用的选择集上测得效应为正的候选所占比例,从第一代的 61% 降至第 11 至 20 代的 1.7%。这一下降是否真实取决于阈值。留出集效应为正的候选占比仅从 50% 降至 16%,而改进幅度超过一个金标准集标准误(1.8 个百分点)的占比从 32% 降至 2.3%,其下降速度与测得的占比大致相当。后期候选中有 14% 的留出集效应介于 0 与 1.8 个百分点之间,处于距零一个金标准集标准误的范围内。如果它们是真实的收益,那么贪心接受会错过它们,这与锁定(lock-in)一致,但它们同样与没有收益一致。全部提交中有 42% 发生在前三代。

表 4:在位者膨胀。在位者在选择集上的得分减去其留出集得分,相对于种子的偏移量(百分点;贪心运行,所有任务和模型;探索性 vLLM 运行和预注册的验证性运行;第 11 至 20 代:各运行上的均值 ± 标准误)。探索性研究中 $n=32$ 和 $128$ 的行包含的设置较少(没有 7B Banking77)。

| 研究 | $n$ | 第 1–3 代 | 第 4–10 代 | 第 11–20 代 |
|---|---|---|---|---|
| 探索性 | 16 | 9.4 | 13.3 | 16.5±2.6 |
|  | 32 | 6.2 | 8.6 | 10.2±2.8 |
|  | 64 | 3.7 | 5.3 | 7.3±1.7 |
|  | 128 | 2.3 | 3.0 | 4.2±1.4 |
|  | 256 | 1.6 | 2.1 | 2.5±0.7 |
| 验证性 | 16 | 7.6 | 11.8 | 13.6±1.7 |
|  | 64 | 3.6 | 5.9 | 7.2±1.1 |
|  | 256 | 0.0 | 0.7 | 1.7±0.6 |

#### 原生循环中的提交。

表 5 和表 6 给出了第 5.2 节的比较,这是对验证性研究中 96 次贪心运行的每一次提交所做的事后描述性分析。对于第 $t$ 代的一次提交,先验 $(\mu,s^{2})$ 是同一模型和任务的冻结的、按阶段索引的试点先验(第 1 代或之后各代;见附录 B),它是在这些运行开始之前,根据另行进行的探索性 oracle 运行的留出集效应拟合得到的。噪声项为 $\hat{e}_{\eta}^{2}=\hat{v}_{cc}/n$ 和 $\hat{e}_{c}^{2}=\max(\hat{v}-\hat{v}_{cc},0)/n$,由该代候选在循环选择集上的逐项向量计算得出(对于只有一个有效候选的代,$\hat{v}_{cc}=\hat{v}/2$)。我们以该代的有效候选数模拟假设 1(40,000 次抽样),并在其测得收益为正的那些抽样中,对测得最优候选的高估程度取平均。观测到的高估是该次提交的测得收益减去在位者留出集准确率的变化。由于在位者选择集得分的同一量给出其膨胀,因此一次提交的观测高估恰好等于在位者膨胀的变化。提交分为三组。第 1 代的提交是唯一由假设 1 所描述的。延迟的首次提交同样以种子为在位者,但它们之前在相同的数据项上经历了一次或多次拒绝:如果在位者的共享误差 $C$ 持续存在,则在第 $t$ 代的首次接受会以 $(1-p(C))^{t-1}p(C)$ 而不是 $p(C)$ 对其分布重新加权,其中 $p(C)$ 是给定 $C$ 时的接受概率。这些提交也使用后期各代的先验,尽管它们的在位者仍是种子。后期提交是针对在相同数据项上选出的在位者做出的。对于后两组,模型值是一个新鲜集合的参照值,而不是预测。该分析不使用根据这些运行的留出集效应拟合的先验;其噪声估计来自当前代的选择集得分。区间通过对运行进行自助法(4,000 次重采样)得到,并以试点先验和固定的 600 个金标准项为条件,因此它们不反映关于新的试点或新的评估项总体的不确定性。包含零的区间表明平均残差与零相容,而不表明模型已被校准。

表 5:原生贪心循环中提交的高估(百分点;验证性运行,四种设置,八个随机种子,在各设置上汇总)。观测值:选择集上的测得收益减去留出集效应,对各提交取平均。模型:带有冻结的试点先验和该代自身噪声项、并以是否被接受为条件的假设 1。第 1 代:第一代中的提交;延迟的首次:一次运行在一次或多次拒绝之后做出的首次提交;后期:针对在相同数据项上选出的在位者做出的提交。运行数:贡献提交的运行数。95% 区间通过对运行做自助法得到,并以试点和金标准项为条件。

| $n$ | 提交 | 数量 | 运行数 | 观测值 | 模型,已接受 | 模型,无条件 | 观测 − 模型(95% CI) |
|---|---|---|---|---|---|---|---|
| 16 | 第 1 代 | 21 | 21 | 9.1 | 9.0 | 6.0 | +0.0 [-3.4, +4.0] |
|  | 延迟的首次 | 10 | 10 | 6.3 | 9.6 | 6.2 | -3.3 [-8.0, +1.8] |
|  | 后期 | 38 | 22 | 5.2 | 9.7 | 5.7 | -4.5 [-6.1, -2.6] |
| 64 | 第 1 代 | 27 | 27 | 3.2 | 3.8 | 2.2 | -0.5 [-2.5, +1.5] |
|  | 延迟的首次 | 4 | 4 | 3.5 | 3.8 | 2.1 | -0.2 [-1.9, +1.3] |
|  | 后期 | 80 | 28 | 1.9 | 3.7 | 1.9 | -1.8 [-2.5, -1.1] |
| 256 | 第 1 代 | 24 | 24 | 0.1 | 1.3 | 0.7 | -1.2 [-2.7, +0.4] |
|  | 延迟的首次 | 8 | 8 | 0.6 | 1.3 | 0.7 | -0.6 [-3.1, +1.3] |
|  | 后期 | 118 | 31 | 0.5 | 1.1 | 0.5 | -0.6 [-1.0, -0.2] |

表 6:各设置下原生贪心循环中的提交(百分点;观测值 / 以接受为条件的模型值,括号中为提交数;分组同表 5)。

| 设置 | $n$ | 第 1 代 | 延迟的首次 | 后期 |
|---|---|---|---|---|
| 1.5B TREC | 16 | 1.9 / 5.9 (6) | 9.3 / 10.3 (2) | 5.7 / 11.8 (14) |
|  | 64 | -0.4 / 1.5 (7) | 1.1 / 3.6 (1) | 1.9 / 4.4 (27) |
|  | 256 | -0.5 / 0.5 (8) | – | -0.0 / 1.4 (38) |
| 1.5B Banking77 | 16 | 12.9 / 7.0 (4) | 11.1 / 9.6 (3) | 4.1 / 7.5 (5) |
|  | 64 | 6.6 / 3.2 (5) | 3.4 / 3.6 (2) | 1.2 / 2.5 (8) |
|  | 256 | 0.4 / 1.0 (3) | 1.0 / 1.3 (5) | 1.4 / 1.0 (13) |
| 7B TREC | 16 | 9.6 / 7.6 (6) | -0.9 / 9.1 (2) | 3.4 / 7.9 (11) |
|  | 64 | 1.1 / 2.7 (8) | – | 1.0 / 3.5 (26) |
|  | 256 | -2.1 / 0.8 (7) | -7.1 / 1.3 (1) | 0.5 / 1.0 (46) |
| 1.5B GSM8K | 16 | 13.9 / 16.1 (5) | 4.4 / 9.5 (3) | 7.7 / 10.1 (8) |
|  | 64 | 6.8 / 7.7 (7) | 6.2 / 4.4 (1) | 3.4 / 3.6 (19) |
|  | 256 | 3.3 / 3.2 (6) | 3.6 / 1.2 (2) | 0.9 / 1.0 (21) |

#### 估计循环内的赢家诅咒与收益。

第 5.1 节的因果估计,对每次 oracle 运行,使用该运行自身选择集 $D$(其在位者即在此集合上选出)中随机抽取的 $n$ 个数据项的子集 $A$,并在整个运行期间保持固定。在第 $t$ 代,它仅利用该运行第 1 至 $t$ 代在 $A$ 上测得的效应,计算 $\hat{v}_{cc}$ 和代内离散度,对每个有 $K_{t}\geq 2$ 个有效候选的代,两者都以 $K_{t}-1$ 加权。它令 $\hat{s}^{2}=\max(\text{spread}-\hat{v}_{cc}/n,0)$ 以及 $\hat{h}_{t}=\hat{s}^{2}/(\hat{s}^{2}+\hat{v}_{cc}/n)$,并将该代赢家相对于该代均值的真实优势预测为 $\hat{h}_{t}$ 乘以其测得优势。真值是在 600 个金标准项上的优势(每次运行抽取 100 次 $A$)。所有误差都是针对某一设置的各代、各运行以及 $A$ 的各次抽样的平均值而言的;对单个赢家的预测要不精确得多。在九种设置以及 $n\in\{16,32,64,128\}$ 上,真实优势与测得优势之比的预测值的平均绝对误差为 0.10($n=16$ 时为 0.08),偏差为 $+0.02$;限于前三代(此时循环几乎没有历史)时,误差为 0.13,偏差为 $-0.01$。在候选之间几乎没有差异的情形(Banking77 上的 7B 和 Qwen3.5-4B,其中零下限使 $\hat{s}^{2}$ 保持为正)以及 GSM8K 上,该估计偏高;在 1.5B vLLM 设置中 $n=128$ 时(此时 $A$ 占被重复使用集合的一半),该估计偏低。由于 oracle 运行在 256 个数据项上做选择,$A$ 是更大的被重复使用集合的子集,其轨迹并不是只有 $n$ 个数据项的循环的轨迹;表 5 是对原生循环的检验。

表 7 探讨一个循环应当如何报告其收益。对于验证性研究中的每一次非 oracle 运行(14B 提议者的运行和 Qwen3.5-4B 的运行),我们取进入最后一代的在位者,并将其相对于种子的收益的三个估计与在金标准集随机一半上的收益进行比较:循环在其选择集上的自我报告;同样的各提交求和,但将每个被提交候选相对于其代均值的测得优势按 $\hat{h}_{t}$ 收缩(无需额外评估);以及在金标准集另一半的 $m$ 个审计项上的收益,这些审计项只对种子和 19 代之后的在位者打分,且从不用于选择(50 次随机对半划分)。收缩在很大程度上消除了平均偏差,但没有消除设置层面的偏差(在 $n=16$、采用贪心接受时为 $-8.5$ 至 $+5.8$ 个百分点)。在 TREC 上,首批提交是巨大的真实收益,收缩去除过多;而在 Banking77 和 GSM8K 上去除过少,因为在位者在被重复使用的集合上累积的共享误差会使所有候选的测得效应一起移动。审计在构造上是无偏的,为这两条指令打分的 64 个数据项(128 次评估)就已使 $n=16$ 时所报告收益的误差低于 $n=64$ 时的自我报告。

表 7:报告循环的收益。三种估计在 19 代之后在位者相对于种子的收益的偏差 / 均方根误差(百分点),与 300 个留出金标准项上的收益相比,按选择集大小 $n$(所有规则)。额外评估是在名义候选选择预算 $KnT=80n$ 之外的追加评估,该预算不包括种子和反馈打分;候选不足会减少实际数量。

| 收益的估计 | 额外评估 | $n=16$ | $n=64$ | $n=256$ |
|---|---|---|---|---|
| 自我报告(选择集) | 0 | +10.8 / 15.5 | +6.1 / 9.0 | +1.8 / 4.7 |
| 按循环自身的 $\hat{h}_{w}^{2}$ 收缩 | 0 | +0.7 / 10.7 | -0.7 / 7.6 | -1.4 / 5.3 |
| 审计,16 项 | 32 | -0.0 / 10.7 | -0.3 / 11.9 | +0.1 / 12.3 |
| 审计,32 项 | 64 | -0.1 / 7.7 | -0.2 / 8.7 | -0.0 / 9.0 |
| 审计,64 项 | 128 | -0.0 / 5.7 | -0.1 / 6.3 | -0.0 / 6.7 |
| 审计,128 项 | 256 | -0.1 / 4.4 | -0.1 / 4.9 | +0.0 / 5.1 |
| 运行数 |  | 151 | 151 | 55 |
| 平均真实收益 |  | 6.2 | 9.6 | 14.2 |

#### 强起点。

两项后续研究均在其首次运行之前(23:38–23:39)就已在带日期的研究日志中做了规划。循环从 Qwen3.5-4B 在 $n=256$、种子 11 下的贪心运行的最终指令出发,该选择是事先固定的,而不是依据留出集准确率挑选的,因为后者会因向均值回归而使后续收益偏低(TREC 上留出集准确率为 63.5%,Banking77 上为 65.0%)。这一事先选择恰好是 TREC 上四个候选运行中最弱的(其余达到 66.0% 至 68.5%),在 Banking77 上是第二弱的(64.5% 至 68.8%),因此两个实验组的绝对收益可能部分反映了反方向的向均值回归;实验组之间的比较不受影响。它运行贪心接受,取 $K=4$、$T=20$、$n\in\{16,256\}$ 和随机种子 21–24,外加每个任务一次带 oracle 插桩的运行($n=256$,$T=10$)。在看到前四个种子的结果之后,又增加了四个种子(25–28)。仅计划中的种子给出的留出集差异(256 项减 16 项)在 TREC 上为 $6.5\pm 5.3$ 个百分点,在 Banking77 上为 $1.8\pm 1.4$($p=0.31$,两者相同),经偏移校正的高估下降了 13.6 和 12.5 个百分点($p=0.095$ 和 $<0.001$);计划中的代理集-留出集差距(它在每个 16 项集合上带有起点的偏移)下降了 9.4 和 2.8 个百分点($p=0.12$ 和 $0.69$)。新增的四个种子给出了相似的结果,留出集差异为 5.6 和 1.9 个百分点($p=0.15$ 和 $0.16$),差距减少 7.2 和 12.9 个百分点($p=0.08$ 和 0.03),高估下降 16.2 和 7.5 个百分点($p=0.04$ 和 $0.09$)。第 5.4 节还报告了对两个阶段采用等权重的逆正态组合(单侧的分阶段 $p$ 值)。运行第二阶段的决定是在看到第一阶段之后做出的,而权重和单侧方向是在两个阶段都运行完之后选定的,因此组合 $p$ 值属于探索性的。该表汇总了全部八个种子。GEPA 从其使用 256 个验证项和种子 1 的全预算运行所返回的程序出发(留出集准确率在 TREC 上为 69.5%,在 Banking77 上为 64.0%;指令分别为 755 词和 2151 词),使用种子 6–10,$n_{\mathrm{val}}\in\{16,256\}$,预算为 3000 次调用,并按附录 B 的时间规则缩减为 TREC 上 2750 次调用、Banking77 上 2000 次调用。使用 16 个验证项时,GEPA 的池中分别有 114 和 80 个程序,使用 256 个验证项时为 11 和 8 个;所返回程序的验证得分在 16 个项时对两个任务都意味着 25.0 个百分点的收益,而真实收益为 $1.6\pm 0.6$ 和 $4.1\pm 0.7$;在 256 个项时意味着 3.0 和 3.7 个百分点,而真实收益为 $0.8\pm 0.8$ 和 $1.9\pm 1.1$。经偏移校正的高估下降了 21.2 和 19.2 个百分点($p=0.003$ 和 0.03),而留出集收益没有显著差异($p=0.51$ 和 $0.13$)。使用 GEPA 停止规则的六次调用窗口进行重建,不改变任何结论(没有任何运行的指标变化超过 0.2 个百分点;对于表 9 中匹配预算的运行,没有任何变化)。表 8 给出了全部指标。

表 8:强起点(百分点;均值 ± 标准误;循环为八个种子,GEPA 为五个)。报告值:相对于起点在选择(验证)集上的收益;留出集:在 600 个金标准项上的收益;差距 / 膨胀:最终选择集准确率减去留出集准确率。

| 任务 | $n$ | GEPA,$B^{\star}$ 次指标调用:报告值 | GEPA:留出集 | GEPA:膨胀 | 贪心循环,20 代:报告值 | 贪心循环:留出集 | 贪心循环:差距 |
|---|---|---|---|---|---|---|---|
| TREC | 16 | +25.0±4.4 | +1.6±0.6 | +16.4±3.1 | +14.8±3.7 | -1.5±2.7 | +9.1±2.5 |
|  | 256 | +3.0±1.1 | +0.8±0.8 | -0.9±0.8 | +6.0±1.0 | +4.6±0.9 | +0.7±1.0 |
| Banking77 | 16 | +25.0±5.2 | +4.1±0.7 | +22.1±2.1 | +11.7±2.2 | +0.2±0.4 | +9.0±3.6 |
|  | 256 | +3.7±0.6 | +1.9±1.1 | +3.1±1.3 | +3.6±0.7 | +2.0±0.5 | +1.2±0.6 |

#### GEPA,全部条件。

表 10 给出了 1000 次指标调用这一共同预算下的 GEPA 结果,表 11 给出了完成了 4000 次调用全预算的运行的结果。全预算呈现出相同的模式。所返回程序的膨胀在 64 个验证项时大于 256 个验证项时(TREC 为 $4.5$ 对 $0.0$ 个百分点,Banking77 为 $12.7$ 对 $5.2$),其收益的高估也是如此(分别为 7.2 对 1.4 和 10.5 对 4.6 个百分点),而其留出集收益并不更小。表 9 增加了后续运行。使用 16 个验证项、从一句话种子出发运行到 2250 次调用(种子 1–5,与在 2250 次调用处截断的全预算 64 和 256 项运行比较),池中有 91 个程序,而后两者为 32 和 9 个,并且 16 项实验组平均进行了 107 次反思调用,而 256 项时不超过 10 次。留出集收益并未随 $n_{\mathrm{val}}$ 增长(256 项减 16 项:TREC 上为 $+0.5\pm 2.5$,Banking77 上为 $-2.8\pm 1.6$;$p=0.86$ 和 0.15),而经偏移校正的高估下降了 28.4 和 15.1 个百分点($p=0.007$ 和 $0.002$)。在 1000 次调用处重建时,新的 16 项运行复现了原先的实验组(膨胀:TREC 上 19.8 对 14.1 个百分点,Banking77 上 23.1 对 24.6;留出集收益:17.5 对 19.2 以及 2.9 对 1.2)。对后续运行的每一次重建,在池、发现计数、验证得分和所返回程序方面都与该运行的日志一致。2250 次调用的 16 项实验组由新的运行组成,所用服务器不同于被截断的 64 项和 256 项运行(相同的种子、训练池和嵌套的验证集);服务器之间的差异会使留出集得分移动约一个百分点(附录 B)。命题 1 的预测(最后一列)现在从 GEPA 为每个被接受候选记录的逐项验证得分中取 $v_{cc}$,因此它只使用优化器所记录的内容。在高估较大的情形下,观测值是预测值的 1.1 至 1.9 倍,涵盖所有 GEPA 条件(中位数为 1.4;在 1000 次调用时为 1.1 至 1.4,在更大预算下的 16 项实验组为 1.5 至 1.9);在 256 个项时两者都很小(观测值 $-3.0$ 至 $+3.7$ 个百分点,预测值 0.5 至 1.9)。因此,该模型捕捉到了高估如何依赖于验证集大小、池大小和预算,但没有捕捉到 GEPA 的高估水平,因为 GEPA 的池本身就是在验证集上选出的。

表 9:GEPA 后续运行(百分点;五个种子上的均值 ± 标准误)。$B$:一组中所有运行进行比较时所处的指标调用次数。候选数:GEPA 池中的程序数。高估:报告收益减去留出集收益。命题 1:预测的高估。

| 起点 | 任务 | $n_{\mathrm{val}}$ | $B$ | 候选数 | 报告收益 | 留出集收益 | 高估 | 命题 1 |
|---|---|---|---|---|---|---|---|---|
| 一句话种子 | TREC | 16 | 2250 | 91 | +45.0±7.2 | +18.5±1.2 | +26.5±6.9 | 14.1 |
|  |  | 64 |  | 32 | +26.6±4.0 | +20.4±1.6 | +6.2±3.9 | 5.4 |
|  |  | 256 |  | 9 | +17.0±2.6 | +19.0±1.9 | -1.9±2.2 | 0.8 |
|  | Banking77 | 16 | 2250 | 91 | +22.5±3.2 | +3.7±1.4 | +18.8±2.5 | 12.6 |
|  |  | 64 |  | 32 | +11.2±2.3 | +1.3±1.1 | +10.0±2.1 | 5.5 |
|  |  | 256 |  | 9 | +4.5±1.0 | +0.8±0.6 | +3.7±0.7 | 1.8 |
| 强起点 | TREC | 16 | 2750 | 114 | +25.0±4.4 | +1.6±0.6 | +23.4±4.1 | 15.2 |
|  |  | 256 |  | 11 | +3.0±1.1 | +0.8±0.8 | +2.2±1.2 | 1.8 |
|  | Banking77 | 16 | 2000 | 80 | +25.0±5.2 | +4.1±0.7 | +20.9±5.1 | 12.2 |
|  |  | 256 |  | 8 | +3.7±0.6 | +1.9±1.1 | +1.7±1.6 | 1.7 |

表 10:1000 次指标调用共同预算下的 GEPA(百分点;各种子上的均值 ± 标准误)。膨胀:所返回程序的验证准确率减去留出集准确率。遗憾(regret):已打分候选中最佳留出集准确率减去所返回程序的留出集准确率;它是一个下界,在种子、所返回程序以及另外十个程序(5 个最佳、5 个随机)上计算,在 $n_{\mathrm{val}}=256$ 时这就是整个池,而在 $n_{\mathrm{val}}=16$ 时只占其四分之一,因此不同 $n_{\mathrm{val}}$ 之间不可比。

| 任务 | $n_{\mathrm{val}}$ | 运行数 | 膨胀 | 报告收益 | 留出集收益 | 遗憾 |
|---|---|---|---|---|---|---|
| TREC | 16 | 5 | +14.1±2.7 | +38.8±4.6 | +19.2±2.0 | +4.9±1.9 |
|  | 64 | 5 | +2.4±2.6 | +24.4±4.3 | +19.3±1.7 | +2.1±0.6 |
|  | 256 | 5 | -4.5±1.3 | +15.9±2.9 | +18.9±1.5 | +0.2±0.1 |
| Banking77 | 16 | 5 | +24.6±5.0 | +18.8±2.8 | +1.2±1.7 | +1.8±0.6 |
|  | 64 | 5 | +7.3±2.9 | +7.5±2.2 | +2.4±1.0 | +0.3±0.3 |
|  | 256 | 5 | +2.4±0.6 | +2.9±0.7 | +1.2±0.6 | +0.1±0.1 |
| GSM8K | 16 | 5 | +3.6±1.3 | +6.2±2.0 | -0.9±0.3 | +0.9±0.3 |
|  | 64 | 5 | +0.6±1.2 | +3.4±1.2 | -0.8±0.5 | +0.8±0.5 |
|  | 256 | 5 | -0.5±0.6 | +1.0±0.5 | -0.4±0.2 | +0.4±0.2 |

表 11:完成了 4000 次指标调用全预算的 GEPA 运行(格式同表 10;遗憾基于全预算下已打分的 12 个最佳和 12 个随机候选)。

| 任务 | $n_{\mathrm{val}}$ | 运行数 | 膨胀 | 报告收益 | 留出集收益 | 遗憾 |
|---|---|---|---|---|---|---|
| TREC | 64 | 5 | +4.5±3.0 | +28.8±2.8 | +21.6±1.0 | +1.4±0.6 |
|  | 256 | 5 | -0.0±1.1 | +19.5±2.4 | +18.1±1.8 | +2.0±0.8 |
| Banking77 | 64 | 5 | +12.7±1.6 | +13.4±2.8 | +2.9±1.3 | +2.3±0.7 |
|  | 256 | 5 | +5.2±0.5 | +5.1±1.0 | +0.5±0.7 | +1.8±0.3 |

#### MIPROv2

表 12 给出 MIPROv2 的结果。在每一个种子上，两种验证集规模从语言模型缓存中抽到的提议指令都相同（只有一个 Banking77 种子是 16 条中的 15 条），因此它们的差别主要在于 MIPROv2 用以评估和选择的验证集（从而也在于贝叶斯优化各轮试验的顺序）。在 TREC 上，更大的验证集使所返回程序的留出增益翻了一倍（11.4 对 5.8 分；配对差为 $5.6\pm 0.7$，$p=0.001$，五个种子全部如此），并消除了它相对于种子的高估（0.7 对 5.5 分，$p=0.05$）。那里的原始膨胀没有信息量，因为种子程序自身的验证偏移，在各个 16 条样本的集合上从 $-30$ 分到 $+7$ 分不等。在候选彼此几乎没有差别的 Banking77 上，16 条样本的实验臂所返回的程序比种子差 $1.6\pm 0.9$ 分，而其验证分数却意味着 5 分的增益；相对于种子的高估在 16 条时为 6.6 分，在 256 条时为 2.4 分（$p=0.27$）。原始膨胀（13.8 对 3.0 分，$p=0.02$）主要反映的是种子程序在 16 条样本集合上自身的验证偏移，其范围从 $-8$ 到 $+17$ 分。十次 16 条样本的运行中有三次返回了种子程序。在这些运行里，有六到九个候选在 16 条样本上的验证分数并列最佳，MIPROv2 保留了其中的第一个。命题 1 以取自候选验证条目向量的 $v_{cc}$ 为依据，对四个单元格的预测分别为 9.0、0.9、7.7 和 1.9 分，观测值则为 5.5、0.7、6.6 和 2.4；它对 16 条样本的 TREC 单元格预测偏高，而该单元格中的并列并非高斯模型所描述的情形。

表 12：MIPROv2（分；五个种子上的均值 $\pm$ 标准误）。候选：被评估的不同指令数。高估：报告增益减去留出增益。命题 1：预测的高估。

| 任务 | $n_{\mathrm{val}}$ | 候选 | 膨胀 | 报告增益 | 留出增益 | 高估 | 命题 1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TREC | 16 | 15.8 | $+0.2\pm 6.5$ | $+11.2\pm 4.1$ | $+5.8\pm 2.1$ | $+5.5\pm 2.6$ | 9.0 |
|  | 256 | 14.4 | $-0.8\pm 0.6$ | $+12.1\pm 3.1$ | $+11.4\pm 2.3$ | $+0.7\pm 1.3$ | 0.9 |
| Banking77 | 16 | 15.8 | $+13.8\pm 2.9$ | $+5.0\pm 2.3$ | $-1.6\pm 0.9$ | $+6.6\pm 3.3$ | 7.7 |
|  | 256 | 15.4 | $+3.0\pm 0.4$ | $+2.5\pm 0.3$ | $+0.1\pm 0.5$ | $+2.4\pm 0.4$ | 1.9 |

#### 探索性贪心循环

图 4 给出第 5 节所总结的探索性（v1）贪心循环。

图 4：探索性贪心循环（v1，vLLM）。（a–d）$n=16,64,256$ 时的留出准确率（实线）以及循环自身的选择集分数（虚线），为 3 个种子的均值。（e）相对于种子指令的最终留出增益随 $n$ 的变化（均值 $\pm$ 标准误）。

#### 一条手写的参考指令

为了衡量 TREC 增益的幅度，我们编写了一条指令，其中定义了 TREC 分类体系的全部 50 个标签（141 个词）。在确认性黄金集上，它使 1.5B 模型的准确率从 10.5% 提高到 11.3%，使 7B 模型从 37.2% 提高到 54.5%。对 7B 而言，这份标签词表的价值大约相当于大 $n$ 时贪心循环的增益；1.5B 的循环增益则远大于这份词表（它们最好的最终指令描述粗粒度类别、给出演算示例，并约束输出格式）。

#### 孤立考察的接受决策

命题 2 讨论的是在一个全新选择集上的单次决策，而完整运行还涉及自适应复用，因此我们也逐次决策地评估这些规则（探索性）。在 $n=16$ 时，平均而言贪心决策使留出准确率下降 0.47 分（95% 区间 $[-0.82,-0.03]$），这与命题 3 在 $\mu<0$ 时的结论一致，而且每一条带有校准阈值的规则都避免了这一损失。经试运行校准的贝叶斯门比贪心高出 0.77 分 $[0.60,0.93]$，经试运行调参 $\alpha$ 的 McNemar 检验高出 0.68 分 $[0.43,0.91]$，二者在每一个 $n$ 上都不可区分。二者避免损失，主要是因为采纳得更少，而且它们的采纳中仍约有一半是有害的。在 $n=16$ 时，贪心在 44% 的决策中采纳，试运行门为 8%，调参后的 McNemar 检验为 6%；它们的采纳中分别有 58%、59% 和 46% 是有害的（其中幅度超过 1.8 分的分别占 40%、35% 和 40%）。在 $n=128$ 时，三者的有害占比都在 34% 到 39% 之间。在复用选择集的完整运行中，同样这些规则没有带来增益（第 5.3 节）；回放为每一次决策都重新抽取选择集，因此其中没有锁定，而且它的候选来自高分辨率轨迹。完整比较见表 13。每个神谕设定的黄金集被一次性分成两半 $G_1$ 与 $G_2$。先验、调好的显著性水平以及调好的阈值，都只根据其他运行在 $G_1$ 上估计（留一运行；对 1.5B 模型则跨推理栈进行；GSM8K 只有一次神谕运行，因此没有试运行）。每一次决策都在从 $G_2$ 抽取的 $n$ 条样本上做出，并按其在 $G_2$ 剩余样本上的留出效应记分；区间是按代计算的聚类自助区间。由于每个设定只有一次或两次神谕运行，而这些运行的各代共享同一条轨迹，上述区间以这些轨迹为条件，因而低估了运行与运行之间的变异。该门的非参数变体用试运行的经验效应替换高斯先验，但把各候选的测量误差视为相互独立，从而忽略了现任方案的共享误差。本回放中的高斯先验从代内效应方差中减去一个固定的近似方差 $0.15/|G_1|$，并把结果的下限截在 $10^{-4}$；这一近似不同于原生循环中的试运行拟合。样本内参考先验使用被评估运行自身在 $G_1$ 上的留出效应；因此在只有一次神谕运行的设定中，其第一阶段先验包含了被评估的那一代自身的效应。调好的 $\alpha$ 在 TREC 上为 0.2 到 0.5，在 Banking77 上为 0.01。调参后的 McNemar 检验相对于 $\alpha=0.05$ 每次决策提高 0.07 到 0.21 分（$n\geq 32$ 时显著），贝叶斯门提高 0.09 到 0.20 分（没有一个区间把零排除在外）。

表 13：单次接受决策的价值（每次决策的留出改进，单位为分；在七个神谕设定上取均值，基于试运行的规则为六个设定，因为 GSM8K 只有一次神谕运行；差值在共同设定上计算；按代计算的 95% 聚类自助区间，以神谕轨迹为条件）。†仅此一行：先验由被评估运行在样本内估计，因此该规则不可部署。

| 规则 | $n=16$ | $n=32$ | $n=64$ | $n=128$ |
| --- | --- | --- | --- | --- |
| 贪心：更优则保留 | -0.47 [-0.82, -0.03] | -0.08 [-0.46, +0.37] | +0.24 [-0.15, +0.71] | +0.50 [+0.11, +0.96] |
| McNemar，$\alpha=.05$ | +0.06 [+0.00, +0.14] | +0.20 [+0.04, +0.43] | +0.34 [+0.09, +0.67] | +0.49 [+0.18, +0.88] |
| McNemar，$\alpha=.20$ | +0.12 [-0.05, +0.34] | +0.31 [+0.04, +0.63] | +0.43 [+0.12, +0.82] | +0.58 [+0.25, +0.99] |
| PACE e-过程 | +0.02 [+0.00, +0.04] | +0.09 [+0.01, +0.23] | +0.25 [+0.04, +0.52] | +0.43 [+0.13, +0.79] |
| 先选后确认 | +0.06 [-0.09, +0.26] | +0.21 [-0.03, +0.52] | +0.35 [+0.07, +0.71] | +0.55 [+0.21, +0.95] |
| McNemar，经试运行调参的 $\alpha$ | +0.19 [-0.02, +0.43] | +0.44 [+0.12, +0.82] | +0.56 [+0.17, +1.02] | +0.64 [+0.25, +1.10] |
| 关于 $\hat{\Delta}$ 的阈值，经试运行调参 | +0.15 [-0.07, +0.40] | +0.35 [-0.02, +0.78] | +0.49 [+0.09, +0.95] | +0.60 [+0.20, +1.03] |
| 经试运行校准的贝叶斯门 | +0.28 [-0.03, +0.63] | +0.40 [+0.06, +0.82] | +0.52 [+0.14, +0.97] | +0.66 [+0.25, +1.12] |
| 非参数先验 | +0.20 [-0.09, +0.52] | +0.31 [-0.02, +0.70] | +0.51 [+0.13, +0.95] | +0.64 [+0.25, +1.08] |
| 贝叶斯门，样本内先验† | +0.21 [-0.04, +0.52] | +0.31 [-0.01, +0.68] | +0.47 [+0.10, +0.90] | +0.64 [+0.25, +1.08] |
| 最佳候选（神谕） | +1.40 [+1.00, +1.87] | +1.42 [+1.02, +1.89] | +1.48 [+1.08, +1.96] | +1.61 [+1.21, +2.07] |
| 试运行门 $-$ 贪心 | +0.77 [+0.60, +0.93] | +0.43 [+0.30, +0.56] | +0.21 [+0.10, +0.30] | +0.08 [-0.00, +0.15] |
| 调参 McNemar $-$ 贪心 | +0.68 [+0.43, +0.91] | +0.47 [+0.31, +0.62] | +0.25 [+0.16, +0.34] | +0.06 [-0.04, +0.16] |
| 试运行门 $-$ McNemar $.05$ | +0.20 [-0.04, +0.48] | +0.17 [-0.03, +0.40] | +0.12 [-0.01, +0.27] | +0.09 [-0.01, +0.19] |
| 调参 McNemar $-$ McNemar $.05$ | +0.11 [-0.03, +0.28] | +0.21 [+0.06, +0.40] | +0.16 [+0.02, +0.32] | +0.07 [+0.01, +0.14] |
| 试运行门 $-$ 调参 McNemar | +0.09 [-0.04, +0.24] | -0.04 [-0.13, +0.05] | -0.04 [-0.10, +0.01] | +0.02 [-0.05, +0.09] |

#### 试运行能否迁移？

要求试运行使用同一模型、同一任务，并且评估集很大，这是一个很强的条件。表 14 用取自他处的先验、调好的 $\alpha$ 和调好的阈值，重复上面的决策级分析：或者取自同一任务上另一模型的运行（$1.5\mathrm{B}\leftrightarrow 7\mathrm{B}$），或者取自同一模型上另一任务的运行（$\mathrm{TREC}\leftrightarrow\mathrm{Banking77}$；GSM8K 取自 1.5B 的 TREC）。跨模型时，贝叶斯门在小 $n$ 下仍然避免了贪心的损失，并且至少与 $\alpha=0.05$ 的 McNemar 一样好（在 $n=128$ 时显著如此），而调参后的检验在 $n=16$ 时迁移较差。跨任务时，能够迁移的只有对贪心损失的防护，而且该门并不好于 $\alpha=0.05$ 的 McNemar。因此，试运行在与任务相匹配时是有用的；当它不匹配时，保守的默认选择更为稳妥。

表 14：先验迁移（探索性；每次决策的价值，单位为分；按代计算的 95% 聚类自助区间，以神谕轨迹为条件）。

| 规则 | $n=16$ | $n=32$ | $n=64$ | $n=128$ |
| --- | --- | --- | --- | --- |
| _试运行来自另一模型（同一任务；4 个设定）_ |  |  |  |  |
| 贪心 | -0.62 [-1.08, -0.08] | -0.10 [-0.57, +0.47] | +0.24 [-0.26, +0.83] | +0.48 [+0.01, +1.06] |
| McNemar $.05$ | +0.06 [+0.01, +0.15] | +0.16 [+0.01, +0.34] | +0.33 [+0.03, +0.75] | +0.51 [+0.09, +1.06] |
| McNemar，调参 $\alpha$ | -0.05 [-0.19, +0.14] | +0.28 [-0.08, +0.74] | +0.45 [+0.01, +1.00] | +0.56 [+0.12, +1.13] |
| 阈值，已调参 | +0.00 [-0.20, +0.28] | +0.16 [-0.15, +0.56] | +0.46 [+0.02, +1.03] | +0.51 [+0.07, +1.08] |
| 试运行贝叶斯门 | +0.25 [-0.08, +0.66] | +0.39 [-0.00, +0.90] | +0.50 [+0.05, +1.06] | +0.62 [+0.18, +1.19] |
| 试运行门 $-$ 贪心 | +0.88 [+0.63, +1.08] | +0.49 [+0.30, +0.67] | +0.27 [+0.14, +0.38] | +0.14 [+0.06, +0.22] |
| 试运行门 $-$ McNemar $.05$ | +0.19 [-0.09, +0.56] | +0.23 [-0.02, +0.57] | +0.17 [-0.00, +0.39] | +0.11 [+0.01, +0.22] |
| _试运行来自另一任务（同一模型；5 个设定）_ |  |  |  |  |
| 贪心 | -0.56 [-0.97, -0.05] | -0.16 [-0.56, +0.36] | +0.15 [-0.28, +0.70] | +0.39 [-0.01, +0.94] |
| McNemar $.05$ | +0.04 [-0.00, +0.12] | +0.13 [+0.02, +0.29] | +0.26 [+0.03, +0.62] | +0.41 [+0.08, +0.88] |
| McNemar，调参 $\alpha$ | -0.05 [-0.18, +0.11] | +0.02 [-0.13, +0.21] | +0.18 [-0.03, +0.45] | +0.46 [+0.14, +0.92] |
| 阈值，已调参 | +0.04 [-0.15, +0.31] | +0.18 [-0.09, +0.54] | +0.35 [+0.02, +0.80] | +0.52 [+0.18, +0.99] |
| 试运行贝叶斯门 | +0.04 [-0.16, +0.31] | +0.22 [-0.04, +0.59] | +0.31 [+0.01, +0.76] | +0.43 [+0.08, +0.92] |
| 试运行门 $-$ 贪心 | +0.60 [+0.26, +0.89] | +0.38 [+0.12, +0.63] | +0.16 [-0.07, +0.38] | +0.04 [-0.13, +0.21] |
| 试运行门 $-$ McNemar $.05$ | -0.01 [-0.18, +0.19] | +0.09 [-0.07, +0.30] | +0.05 [-0.06, +0.17] | +0.03 [-0.09, +0.15] |

#### 全部接受规则的完整运行结果（探索性）

表 15–16 针对探索性 vLLM 实验的每一条规则和每一个设定，报告相对于种子指令的最终留出增益（3 个种子上的均值 $\pm$ 标准误）、循环的选择集分数与其留出准确率之间的最终差距，以及全部采纳中的有害采纳次数（跨种子相加）。各规则之间的完整运行差异相对于种子变异大多较小，而且许多增益出现在最初几次决策之中，这与锁定相符。固定水平的检验在 $n=16$ 时很少采纳，因而停留在种子附近：这在 Banking77 上是最优的（那里很少有提议能带来帮助），在 TREC 上则代价很高；贪心的有害采纳和自报差距都在小 $n$ 时最大。在线经验贝叶斯（EB）门在完整运行中并未优于贪心。在 1.5B-TREC、$n=64$ 上，它最终比贪心低 6.2 分（$+11.6\pm 0.9$ 对 $+17.8\pm 0.7$；三个种子全都更低，配对 $t$ 检验 $p=0.016$）。我们把这一点归因于：它在单次运行内部仅凭少数几个候选估计 $s^2$，而这一估计并不可靠；因此在确认性研究中，我们只评估经试运行校准的门。

表 15：完整运行，1.5B，vLLM：相对于种子的留出增益（分；种子上的均值 $\pm$ 标准误），选择集分数减去留出准确率的最终差距（分），以及全部采纳中的有害采纳次数，跨种子相加。RM（分辨率匹配）运行使用每代预算 $B=256$，并列在 $n=64$ 之下。

| 设定 | 规则 | 运行数 | 留出增益 | 代理 $-$ 留出 | 有害 / 采纳 |
| --- | --- | --- | --- | --- | --- |
| 1.5B TREC，$n=16$ | 贪心 | 3 | $+15.8\pm 3.6$ | 15.8 | 2 / 10 |
|  | McNemar | 3 | $+11.3\pm 5.8$ | 7.7 | 0 / 2 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -1.8 | 0 / 0 |
|  | EB 门 | 3 | $+5.6\pm 1.5$ | 17.6 | 0 / 3 |
| 1.5B TREC，$n=64$ | 贪心 | 3 | $+17.8\pm 0.7$ | 7.4 | 2 / 13 |
|  | McNemar | 3 | $+12.6\pm 2.4$ | 1.6 | 0 / 3 |
|  | e-过程 | 3 | $+16.0\pm 1.4$ | 0.3 | 0 / 3 |
|  | EB 门 | 3 | $+11.6\pm 0.9$ | 4.5 | 1 / 4 |
|  | EB + RM | 3 | $+16.2\pm 1.9$ | 5.3 | 0 / 6 |
| 1.5B Banking77，$n=16$ | 贪心 | 3 | $-5.4\pm 2.9$ | 10.3 | 5 / 5 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -5.6 | 0 / 0 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -5.6 | 0 / 0 |
|  | EB 门 | 3 | $-0.5\pm 0.5$ | -3.0 | 1 / 1 |
| 1.5B Banking77，$n=64$ | 贪心 | 3 | $-3.6\pm 2.5$ | 1.1 | 5 / 5 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -6.6 | 0 / 0 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -6.6 | 0 / 0 |
|  | EB 门 | 3 | $-1.9\pm 1.2$ | 0.5 | 2 / 2 |
|  | EB + RM | 3 | $-0.6\pm 0.3$ | 0.4 | 2 / 3 |
| 1.5B GSM8K，$n=16$ | 贪心 | 3 | $-0.8\pm 3.7$ | 18.0 | 4 / 9 |
|  | EB 门 | 3 | $+2.4\pm 1.0$ | 14.9 | 2 / 5 |
| 1.5B GSM8K，$n=64$ | 贪心 | 3 | $+0.4\pm 1.9$ | 7.4 | 4 / 9 |
|  | McNemar | 3 | $+4.3\pm 1.6$ | 6.8 | 1 / 4 |
|  | EB 门 | 3 | $+1.8\pm 2.5$ | 6.8 | 3 / 7 |
|  | EB + RM | 3 | $-0.7\pm 1.9$ | 8.6 | 3 / 5 |

表 16：完整运行，7B，vLLM（格式同表 15）。

| 设定 | 规则 | 运行数 | 留出增益 | 代理 $-$ 留出 | 有害 / 采纳 |
| --- | --- | --- | --- | --- | --- |
| 7B TREC，$n=16$ | 贪心 | 3 | $+5.9\pm 1.7$ | 16.0 | 1 / 5 |
|  | McNemar | 3 | $+1.0\pm 1.0$ | 10.7 | 0 / 1 |
|  | EB 门 | 3 | $+8.9\pm 4.2$ | 25.7 | 3 / 7 |
| 7B TREC，$n=64$ | 贪心 | 3 | $+15.4\pm 3.2$ | 10.9 | 4 / 14 |
|  | McNemar | 3 | $+8.9\pm 3.0$ | 6.9 | 0 / 3 |
|  | e-过程 | 3 | $+5.9\pm 5.9$ | 2.6 | 0 / 1 |
|  | EB 门 | 3 | $+13.9\pm 4.2$ | 10.2 | 3 / 10 |
|  | EB + RM | 3 | $+14.1\pm 2.8$ | 4.7 | 2 / 10 |
| 7B Banking77，$n=16$ | 贪心 | 3 | $-0.2\pm 1.3$ | 3.1 | 1 / 3 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -3.4 | 0 / 0 |
|  | EB 门 | 3 | $+0.0\pm 0.6$ | 2.8 | 1 / 2 |
| 7B Banking77，$n=64$ | 贪心 | 3 | $+0.8\pm 0.3$ | -0.5 | 2 / 6 |
|  | McNemar | 3 | $+0.5\pm 0.5$ | -1.3 | 0 / 1 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -3.9 | 0 / 0 |
|  | EB 门 | 3 | $+1.2\pm 0.4$ | -2.0 | 0 / 3 |
|  | EB + RM | 3 | $+0.9\pm 0.2$ | -4.7 | 0 / 3 |

#### 全新选择集与复用选择集

每一代都重新抽取选择集（“贪心，全新 $D$”，llama.cpp 复现，TREC $n=64$），可以避免把现任方案早先的选择误差带进下一次比较。该循环的采纳次数是复用时的两倍（三个种子合计 33 次对 16 次采纳），其采纳中有 39% 是有害的（复用时为 25%；在 $-1.8$ 分的阈值上为 30% 对 12.5%），最终留出增益也更低（$+14.4\pm 1.1$ 对 $+21.3\pm 1.4$ 分）。当大多数提议有害、而且每次比较都是一次新的噪声时，这与命题 3 相符。这只是一个含三个种子的探索性设定，并且带有探索性设计的反馈混杂；有害占比之差并不显著（Fisher $p=0.36$），而且使用全新集合的运行每代有效候选更少（2.3 对 3.1）。

#### 无防护 EB 门的开发种子失败

在开发种子上（TREC，llama.cpp），在线 EB 门的第一版与_分辨率匹配_并用（逐代增大 $n$，以使所估计的 $h_w^2$ 保持不变），峰值达到 $+18.4$，最终落到 $+5.8$ 分，8 次采纳中有 3 次有害：两代之后，它对 $s^2$ 的估计坍缩为零，后验退化为一个陈旧的正值 $\hat{\mu}$，于是该门采纳了那些被_测得_为退步的候选。我们补上了第 4 节的防护（绝不采纳一次被测为退步的结果），并给 $n$ 的增长加上上限，然后在全部种子和全部任务上运行带防护的门；这一改动是事后做出的，而探索性完整运行表中的 EB 门条目是对全部种子（包括开发种子）取平均。无防护的运行作为消融列在表 18 中。

#### 分配扫描

表 17 把固定的每代预算 $B=256$ 次评估，拆成 $K$ 个各含 $n$ 条样本的候选（在线 EB 门）。差异都落在种子变异之内；对 7B 来说，两个极端（$K=1$ 与 $K=16$）表现最差，这与内部最优相符，但这次扫描太小，不能下定论。更早一次在 llama.cpp 栈上的扫描（1.5B TREC，两个种子，$T=25$）对 $(K,n)=(1,256)$、$(2,128)$、$(8,32)$ 和 $(16,16)$ 给出的留出增益分别为 16.7、13.9、24.2 和 14.6 分，同样落在种子变异之内。

表 17：把每代 $B=256$ 次评估做不同拆分时的留出增益（分，均值 $\pm$ 标准误，种子数）。

| 模型 | $K\times n=16\times 16$ | $8\times 32$ | $4\times 64$ | $2\times 128$ | $1\times 256$ |
| --- | --- | --- | --- | --- | --- |
| 1.5B TREC | $+16.3\pm 1.9$ (3) | $+13.6\pm 2.0$ (3) | $+11.6\pm 0.9$ (3) | $+14.8\pm 1.3$ (3) | $+13.7\pm 2.2$ (3) |
| 7B TREC | $+8.8\pm 4.2$ (2) | $+11.5\pm 7.3$ (2) | $+13.9\pm 4.2$ (3) | $+12.6\pm 5.3$ (2) | $+6.6\pm 3.8$ (2) |

## 附录 D 使用第二套推理栈的复现

我们用一套独立的推理栈重复了 1.5B 的分类实验：在消费级 GPU（RTX 3080 Ti）上由 llama.cpp 提供服务的 8 位 GGUF 权重，$T=25$ 代，其余代码相同。图 1 收录了两个神谕设定（“llama.cpp”）；分布感知预测的平均绝对误差在 TREC 上为 0.014，在 Banking77 上为 0.048。贪心循环在 TREC 上再次随着 $n$ 增大而获得更高的留出准确率（$n=16$ 时为 $+9.1$，对比 $n=256$ 时为 $+25.7$），同时它们的自报差距缩小（14.4 对 5.4 分）；在 Banking77 上，它们再次损失留出准确率（$n=16$ 与 $n=64$ 时为 $-3.8$，$n=256$ 时为 $-1.4$）。各接受规则的完整运行结果见表 18；在这一栈上，带防护的 EB 门在 $n=64$ 时与贪心持平，在 TREC 的 $n=16$ 上与贪心的差距处于种子变异之内（$+10.8\pm 4.0$ 对 $+9.1\pm 2.5$），而带分辨率匹配的 EB 在每代 256 次评估的规则中取得了最高的 TREC 增益（$+24.8\pm 0.3$；$n=256$ 的贪心每代评估 1024 次，达到 $+25.7$）。

表 18：完整运行，1.5B，llama.cpp 复现（$T=25$；格式同表 15）。

| 设定 | 规则 | 运行数 | 留出增益 | 代理 $-$ 留出 | 有害 / 采纳 |
| --- | --- | --- | --- | --- | --- |
| 1.5B TREC，$n=16$ | 贪心 | 3 | $+9.1\pm 2.5$ | 14.4 | 1 / 9 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -3.6 | 0 / 0 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -3.6 | 0 / 0 |
|  | EB 门 | 3 | $+10.8\pm 4.0$ | 4.3 | 0 / 5 |
|  | EB，无防护 | 2 | $+12.8\pm 7.4$ | 8.7 | 0 / 5 |
| 1.5B TREC，$n=64$ | 贪心 | 3 | $+21.3\pm 1.4$ | 6.9 | 4 / 16 |
|  | 贪心，全新 $D$ | 3 | $+14.4\pm 1.1$ | 6.0 | 13 / 33 |
|  | McNemar | 3 | $+17.6\pm 2.8$ | 6.5 | 2 / 6 |
|  | e-过程 | 3 | $+18.8\pm 3.0$ | 7.8 | 0 / 4 |
|  | EB 门 | 3 | $+21.2\pm 1.2$ | 7.0 | 3 / 15 |
|  | EB + RM | 3 | $+24.8\pm 0.3$ | 4.1 | 2 / 11 |
|  | EB，无防护 | 1 | $+21.7\pm 0.0$ | -0.3 | 0 / 6 |
|  | EB + RM，无防护 | 1 | $+5.8\pm 0.0$ | 5.8 | 3 / 8 |
| 1.5B Banking77，$n=16$ | 贪心 | 3 | $-3.8\pm 3.5$ | 12.0 | 3 / 4 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -2.3 | 0 / 0 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -2.3 | 0 / 0 |
|  | EB 门 | 3 | $-2.6\pm 2.6$ | 6.6 | 2 / 2 |
| 1.5B Banking77，$n=64$ | 贪心 | 3 | $-3.8\pm 2.7$ | 4.1 | 5 / 8 |
|  | McNemar | 3 | $+0.0\pm 0.0$ | -6.9 | 0 / 0 |
|  | e-过程 | 3 | $+0.0\pm 0.0$ | -6.9 | 0 / 0 |
|  | EB 门 | 3 | $-3.1\pm 2.4$ | 2.4 | 3 / 5 |
|  | EB + RM | 3 | $-1.7\pm 0.8$ | -3.8 | 3 / 4 |
