# 2026 年 RSI 十篇精选（2026-11-25）

面向 **x-daily** 归档的独立专题，来源是 elvis（@omarsar0）的 X Article[《My Top 10 RSI Papers of 2026》](https://x.com/omarsar0/status/2125234668396023943)（X Article：https://x.com/i/article/2125233081082613761），发布于 2026-11-24 22:26（Asia/Shanghai）。

下列十项是帖中编号列出的「今年最喜欢的十篇 RSI 论文」，顺序与原帖一致。帖中建议在十篇之前先读的综述标注在文末，**不是**这十篇之一。

归档日期：**2026-11-25**（Asia/Shanghai 日历日）。

## 许可与收录范围

每项为 `article.md` + `article_zh.md` + `meta.json`。

- **全文转载 + 中文翻译**：仅当 arXiv 摘要页明确给出允许再分发 *并且* 允许翻译的许可（CC BY 4.0，或附带非商业与相同方式共享条件的 CC BY-NC-SA 4.0）。英文 `article.md` 是从公开 HTML 或 PDF 整理的全文，自摘要至附录。中文 `article_zh.md` 是对该文本的非官方、不删减翻译：摘要至结论的全部正文段落、表格、公式、脚注，以及附录散文。代码、论文标明原文照录的提示词，以及参考文献条目保持英文。不是官方或作者审定译本。压缩译述不标为全文翻译。
- **仅结构化研究笔记**：当作品使用 arXiv.org 非独占许可（不授予第三方全文再分发或翻译权）或 CC BY-NC-ND（禁止衍生作品，包括全文翻译）时。这些笔记是显著短于原文的独立摘要，并明确标明不是转载、不是全文翻译。带出处的短引不超过 25 个英文词。

开源代码许可不被当作覆盖论文正文。

## 建议阅读顺序

按源帖的分组。

1. [Meta-Harness](01-meta-harness/article_zh.md) — 改什么：部署前搜索中的 harness。
2. [Exo](02-exo/article_zh.md) — 改什么：使用过程中的 harness，并放在受保护层之后。
3. [Hyperagents](03-hyperagents/article_zh.md) — 改什么：提出修改的程序本身。
4. [Harness Evolution Hits a Ceiling](04-harness-evolution-ceiling/article_zh.md) — 何时从改 harness 转向训练权重。
5. [Harness-Delta Attribution](05-harness-delta-attribution/article_zh.md) — 把报告的 harness 增益拆成过拟合、额外测试时计算，以及剩余项。
6. [赢家诅咒](06-winners-curse/article_zh.md) — 在被反复使用的小评估集上「更好就保留」，会夸大真实增益。
7. [谁来验证验证器？](07-who-verifies-the-verifier/article_zh.md) — agent 分数上升不能为评分器背书。
8. [Trusting Trust, Revisited](08-trusting-trust-revisited/article_zh.md) — 被污染的自我评测集可以左右自修改 agent 保留什么。
9. [Spotify 对话推荐](09-spotify-conversational-agents/article_zh.md) — 生产循环里的 RSI，每项改动经人工评审。
10. [没有第二次运行](10-no-second-run/article_zh.md) — 把不可重复的前沿预训练拆成可闭合的小循环。

若只采纳一个做法：单辟一块只用于评估的留出集，在相信任何增益之前，用它同时给起始版本和当前版本打分。

## 目录

| # | 标题 | 文件 | 作者 | 许可 | 收录形式 |
|---|---|---|---|---|---|
| 1 | Meta-Harness: End-to-End Optimization of Model Harnesses | [article.md](01-meta-harness/article.md) · [article_zh.md](01-meta-harness/article_zh.md) | Yoonho Lee 等 | CC BY 4.0 | 全文 + 全文翻译 |
| 2 | Exo: A Harness Architecture for Runtime Self-Improvement | [article.md](02-exo/article.md) · [article_zh.md](02-exo/article_zh.md) | Alexander Krentsel 等 | CC BY 4.0 | 全文 + 全文翻译 |
| 3 | Hyperagents | [article.md](03-hyperagents/article.md) · [article_zh.md](03-hyperagents/article_zh.md) | Jenny Zhang 等 | CC BY 4.0 | 全文 + 全文翻译（v2） |
| 4 | Harness Evolution Hits a Ceiling | [article.md](04-harness-evolution-ceiling/article.md) · [article_zh.md](04-harness-evolution-ceiling/article_zh.md) | Yuan Tian 等 | arXiv 非独占 1.0 | 仅笔记 |
| 5 | Harness-Delta Attribution | [article.md](05-harness-delta-attribution/article.md) · [article_zh.md](05-harness-delta-attribution/article_zh.md) | Wenxuan Ding 等 | CC BY 4.0 | 全文 + 全文翻译 |
| 6 | LLM 自我改进循环中的赢家诅咒 | [article.md](06-winners-curse/article.md) · [article_zh.md](06-winners-curse/article_zh.md) | Litao Hu、Yutong Tang | CC BY 4.0 | 全文 + 全文翻译 |
| 7 | 谁来验证验证器？ | [article.md](07-who-verifies-the-verifier/article.md) · [article_zh.md](07-who-verifies-the-verifier/article_zh.md) | Xing Zhang 等 | CC BY-NC-SA 4.0 | 全文 + 全文翻译（非商业、相同方式共享） |
| 8 | Reflections on Trusting Trust, Revisited | [article.md](08-trusting-trust-revisited/article.md) · [article_zh.md](08-trusting-trust-revisited/article_zh.md) | Franziska Roesner、Tadayoshi Kohno | arXiv 非独占 1.0 | 仅笔记 |
| 9 | 在 Spotify 冷启动对话推荐 agent | [article.md](09-spotify-conversational-agents/article.md) · [article_zh.md](09-spotify-conversational-agents/article_zh.md) | Enrico Palumbo 等 | CC BY-NC-ND 4.0 | 仅笔记（ND：不可全文翻译） |
| 10 | 没有第二次运行 | [article.md](10-no-second-run/article.md) · [article_zh.md](10-no-second-run/article_zh.md) | Xiaojie Cai 等（字母序） | CC BY 4.0 | 全文 + 全文翻译 |

第 2 项 arXiv 摘要页标题是 “Exo: An Architecture for Self-Improving Systems at Runtime”（源帖清单也用这一用语）。论文正文标题是 “Exo: A Harness Architecture for Runtime Self-Improvement”，本归档按正文标题转载。

## 中英文范围

| # | 英文 `article.md` | 中文 `article_zh.md` | 有意保留英文 |
|---|---|---|---|
| 1 | 全文，摘要至附录 E | 对该文本的不删减翻译，含附录 A–E | 参考文献 |
| 2 | 全文。官方 HTML 在 §5.4 中途结束；自 §5.4 后半至附录 B 据 PDF 转录，含第 6 节、参考文献、AI 使用披露与附录 A–B。表 1 在 HTML 中表体为空，单元格据 PDF 补回。表 A1 的底色未复现 | 对该文本的不删减翻译，含两个附录与表 1 | 参考文献 |
| 3 | v2 全文，摘要至附录 F（ar5iv HTML；官方 `/html/2603.19461` 返回 404） | 散文译至附录 F，不删减 | 代码、提示词、伪代码、diff 与参考文献。一条文献姓名按 ar5iv HTML 原文照录（`Armengol-EstapÃŠ`） |
| 4 | 短篇独立笔记，不是转载 | 这些笔记的中文，不是论文全文翻译 | — |
| 5 | 全文，摘要至附录 G | 不删减翻译，含附录 A–G | 代码清单与参考文献 |
| 6 | 全文，摘要至附录 D | 不删减翻译，含证明与结果表 | 论文标明原文照录的提示词，以及参考文献 |
| 7 | 全文，摘要至附录 G。许可仍为 CC BY-NC-SA 4.0 | 同一许可下的不删减翻译（非商业、相同方式共享），文件内写明 | 逐字保留的验证器表达式与参考文献 |
| 8 | 高层次短篇笔记，不是转载，不含攻击步骤 | 这些笔记的中文，不是论文全文翻译 | — |
| 9 | 短篇独立笔记，不是转载。ND 禁止翻译论文 | 这些笔记的中文，不是论文全文翻译 | — |
| 10 | 全文，摘要至附录 D | 不删减翻译，含附录 A–D 与结果表 | shell 命令与参考文献 |

图只保留图注，不重绘。HTML 里重复导出的公式（如 `4×\times`）在中文稿中收成单一 LaTeX。

## 收获

- 给搜索 harness 的 agent 完整执行轨迹，而不只是一个分数（Meta-Harness）。
- 若允许 agent 在生产中修改自己，先冻结它不能改的受保护层（Exo）。
- 把提出修改的程序也当作可改进对象，并单独记录这些修改（Hyperagents）。
- 先给失败分类：过程失败改 harness，内容失败训练权重（Ceiling）。
- 相信 harness 增益之前，先让旧 harness 用新的计算预算跑，再在留出任务上比较（HDA）。
- 用从未用于选择的留出集同时给起始版本和当前版本打分（赢家诅咒；该文中 64 条去掉了平均夸大）。
- 保留一小份人工核对的例子，把评分器与 agent 分数分开衡量（谁来验证验证器）。
- 像审查代码一样审查进入自我评测集的内容，并审查 agent 为自己写下的工具与规则（Trusting Trust）。第 8 项笔记保持高层次，不复述攻击步骤。
- 产品循环里先收窄范围，每项改动保留人工评审，并用真实用户指标衡量（Spotify）。
- 若主循环慢到或贵到无法重复，就拆成带可快速测量指标的小循环（没有第二次运行）。

## 目录结构

```
articles/2026-11-25-rsi-top-10-papers/
  README.md / README_zh.md / meta.json
  01-meta-harness/ … 10-no-second-run/
    article.md  article_zh.md  meta.json
```

仓库索引 [`../README.md`](../README.md) 链到本专题。

## 抓取状态

十篇均已于 2026-11-25（Asia/Shanghai）从公开 arXiv 来源成功通读。无「抓取失败」占位。

| # | arXiv | 阅读来源 | 说明 |
|---|---|---|---|
| 1 | [2603.28052](https://arxiv.org/abs/2603.28052) | 官方 HTML + PDF | |
| 2 | [2611.00463](https://arxiv.org/abs/2611.00463) | 官方 HTML + PDF | 官方 HTML 在 §5.4 中途结束。自 §5.4 后半至附录 B 据 PDF 转录，含第 6 节、参考文献、AI 使用披露与附录 A–B |
| 3 | [2603.19461](https://arxiv.org/abs/2603.19461) v2 | ar5iv HTML + PDF | 官方 `/html/2603.19461` 返回 404 |
| 4 | [2610.11655](https://arxiv.org/abs/2610.11655) | 官方 HTML + PDF | 仅笔记（许可） |
| 5 | [2610.22466](https://arxiv.org/abs/2610.22466) | 官方 HTML + PDF | |
| 6 | [2610.09239](https://arxiv.org/abs/2610.09239) | 官方 HTML + PDF | |
| 7 | [2610.11464](https://arxiv.org/abs/2610.11464) | 官方 HTML + PDF | |
| 8 | [2609.17817](https://arxiv.org/abs/2609.17817) | 官方 HTML + PDF | 仅笔记（许可）；不复述攻击步骤 |
| 9 | [2609.30297](https://arxiv.org/abs/2609.30297) | 官方 HTML + PDF | 仅笔记（CC BY-NC-ND） |
| 10 | [2611.10719](https://arxiv.org/abs/2611.10719) | 官方 HTML + PDF | |

未下载视频。许可以各篇 arXiv 摘要页为准，不从代码仓库推断正文许可。

## 建议先读（不是这十篇之一）

源帖建议先读综述 *The Last AI Built by Humans: Toward Genuine Recursive Self-Improvement*，Yi Duan 等，2026，[arXiv:2609.11873](https://arxiv.org/abs/2609.11873)。它 **不是** 十篇编号精选之一。摘要页标明 CC BY-NC-ND 4.0，本专题不转载、不翻译。帖中还指向 DAIR 合集 https://academy.dair.ai/papers/collections/self-improving-agents ，并在第 8 项讨论中提及若干 2026 基准（PostTrainBench、SEAGym、Flex-RSI、Singularity-Bench、FrontierRSI），它们是上下文，不是这十篇的成员。
