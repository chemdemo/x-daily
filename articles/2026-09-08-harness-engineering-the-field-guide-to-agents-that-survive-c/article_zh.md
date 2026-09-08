---
title: "Harness Engineering：让 Agent 扛得住现实的实战指南"
author: "spect (@spectnfa)"
source_url: "https://x.com/spectnfa/status/2097298431383417150"
published_at: "2026-09-08"
archived_at: "2026-09-08"
x_post_id: "2097298431383417150"
x_article_id: "2097289277788893184"
archive_note: "Complete article captured from the X page in a browser on 2026-09-08; media retained in ./media/."
lang: zh-Hans
translation_of: article.md
---

![Cover](./media/01-cover.jpg)

# Harness Engineering：让 Agent 扛得住现实的实战指南

你的 agent 挂了。

你打开 prompt，加一条规则。

它换了一种方式挂。你再加一条。

三周后，system prompt 成了九百字的伤疤组织，模型也换了两次，这个 agent 仍然无法独自完成两小时的任务。

那些规则大多从来不是真正的修法。

因为那些失败大多从来不是推理问题。

Prompt 是你唯一看得见的那一层，于是它成了你唯一去改的那一层。

真正的缺陷在别处：agent 能看见什么、被允许碰什么、记住了什么、什么才算做完。

那一层有名字。它叫 harness。

**Harness engineering，就是建造那种环境：把模型智能，变成你可以信任的工作。**

> 一条 prompt 改变一次运行。

> 一个 harness 改变之后的每一次运行。

---

## 第一部分：先诊断，再打补丁

先给失败命名。几乎每一次都会落进六个桶里。

![Six ways an agent fails: blind, clumsy, amnesiac, boastful, reckless, and stubborn](./media/02.png)

| 看上去像什么 | 缺的是什么 |
| --- | --- |
| 改错了文件，漏掉了正确文件 | Context |
| 对的命令，跑在错误的地方 | Tool gateway |
| 四十分钟前做的决策忘了 | Durable state |
| 什么都没跑就宣布成功 | Evidence |
| 部署了本该人类审批的东西 | Policy |
| 同一失败调用原样重试了十一次 | Recovery |

上面那张表里，没有任何一行是在说「模型不够聪明」。

把更强的模型丢进同一个破环境，你只是得到一个说得更流畅的同款错误。

没有结构的能力，只会失败得更流利。

---

## 第二部分：模型只是其中一个组件

模型会推理、比较、选择。

Agent 必须能运转。

运转意味着：找到关键信息、选工具、改变真实世界里的东西、跟踪发生了什么变化、守住边界、检查结果、错了能恢复。

![Model as one component inside a harness](./media/03.jpg)

模型是引擎。Harness 是车的其余部分。

一台强引擎焊在没有底盘的架子上，不是快车，是隐患。

Harness 并不能消除模型的不确定性。没有东西能。

它只是把不确定性关进一个能观察、能验证、能往回走的系统里。

七层各自干活。每一层对应关掉上面某一种失败。

---

### 第 1 层：契约（The contract）

多数 agent 工作始于一个愿望。

> 把退款积压清掉。

两人共享上下文时还行。

对自主执行毫无用处：没有成功定义，也没有「走得太远」的定义。

在 agent 动手之前，把意图编译成契约。

![Informal intent turned into a bounded run](./media/04.jpg)

```yaml
objective: clear refunds older than 7 days

scope:
  - services/refunds
  - refund analytics events

constraints:
  - do not touch the payment provider client
  - no schema changes

acceptance:
  - queue age p95 under 48h in local replay
  - refund regression suite green
  - one refund traced end to end

approval_required:
  - any production write
  - any customer-facing message
```

文件很小。问题却变了。

没有它，agent 优化的是「看起来忙」。

有了它，agent 优化的是「可检查的东西」。

契约也是唯一诚实的失败方式。若从未定义成功，「做完」只是一种心情。

---

### 第 2 层：Context 是地图，不是手册

把仓库、文档和完整历史一股脑倒进窗口，那不是 context engineering。

那是洪水。

真正重要的那一行仍在里面，却要和另外四百行无关内容抢注意力。

![Map-not-manual diagram](./media/05.jpg)

给一张地图。让 agent 按需拉取细节。

### PROJECT MAP

```latex
refund rules      -> docs/refunds.md
service code      -> services/refunds/
analytics events  -> packages/events/
test commands     -> docs/testing.md
release rules     -> docs/release.md
```

任务，然后地图，然后子系统，然后精确文件，然后本地规则。

Context 应因任务需要而扩展，而不是因为数据存在。

你不是在最大化 context。你是在最大化每 token 的信号。

---

### 第 3 层：网关，不是工具堆

二十个工具不是二十种能力。

它是二十种出错方式，外加每一步的选择问题。

![Gateway validates and authorizes tools](./media/06.jpg)

每个工具都要有契约，像一个 API endpoint。

```yaml
TOOL: edit_file

inputs:        path, patch
preconditions: path exists
               path inside workspace
               path is not generated code
success:       patch applied, diff returned
failure:       no partial write, structured error
risk class:    reversible
```

然后在所有工具前面放一个网关。

它校验参数、隐藏无关工具、限制路径与域名、设超时、让重试幂等，并返回证据，而不是那个词「success」。

### 真正要紧的分离：

**model**     提出意图  
**gateway**   授权动作  
**tool**      改变环境  
**sensor**    报告发生了什么  

模型可以有意见。它没有最终决定权。

---

### 第 4 层：记忆必须变成状态

Transcript 不是记忆。它是事件日志。

作为执行基底很差：关键行和随手一行看起来一模一样。

![Raw transcript compiled into state](./media/07.jpg)

把日志编译进四个桶。从那些桶跑。

```csharp
facts:
  - refund eligibility lives in services/refunds/policy.ts
  - the analytics pipeline drops events over 4KB

decisions:
  - reuse the existing eligibility check
  - reason: a second source of truth caused the last incident

progress:
  done:    [age based queue split]
  blocked: [staging replay needs a fresh dataset]
  left:    [regression test for partial refunds]

lessons:
  - the test runner needs TEST_DB_URL or it silently passes
```

原始日志留给审计。执行从编译后的状态出发。

这正是你如何得到一个可恢复的 agent。

> 一次运行死在第 14 步，应从状态重启，而不是从没人想重读的对话。

把三件关切在物理上分开：大脑规划，双手在沙箱里执行，历史在 context window 消失后仍存活。

分开它们，你就能替换其中一个，而不必重建另外两个。

---

### 第 5 层：策略放在 loop 之外

有些规则绝不能依赖模型是否记得它们。

- 未经审批绝不发布
- 绝不写到 workspace 之外
- 绝不暴露凭证
- 绝不突破花费上限
- 测试没跑过就绝不标成通过

那些不是指令。指令是建议。

那是策略（policy），策略属于代码，由网关强制执行。

![Autonomy ceiling with consequence gates](./media/08.jpg)

按后果分档。

**读、搜索、检查。** 自动。

**可逆变更。** 自动，但要留痕。

**外部效应**（发送、部署、花钱）。显式审批。

**不可逆或敏感**（删数据、轮换密钥）。硬门，或者根本不暴露。

自主不是没有边界。

它是在真正被强制执行的边界里跑得快。

---

### 第 6 层：完成需要证据

「Task complete」不是状态。它只是又一次 token 预测。

![Evidence gate before acceptance](./media/09.jpg)

完成是环境的属性。只有环境能授予它。

| 声称 | 证据 |
| --- | --- |
| 「bug 修好了」 | 原本失败的测试现在通过 |
| 「流程能跑」 | 一次真实会话把它走完 |
| 「迁移安全」 | dry run 与 rollback 都通过 |
| 「数字对」 | 输出与源数据对得上 |
| 「任务完成」 | 每一项验收检查都通过 |

> **先跑最便宜的确定性检查：语法、类型、聚焦测试、集成，然后才是判断，再然后才是人。**

凡是编译器、schema、checksum 或一条 SQL 能回答的问题，就别再花一次模型调用。

模型处理歧义。代码处理管道。

验证是一次攻击

让同一个 agent、在同一 context 里「再检查一遍自己」，它往往会重新推导出自己的那些假设。

那不是验证。那是置信度闭环。

![Worker and verifier attack loop](./media/10.jpg)

拆开目标。

Worker 造出最强候选。Verifier 试图把它打穿。

给 verifier 一份拒绝量规、产物、契约，以及新鲜 context。

还要给它「拒绝而不修复」的权利。

最后这一点很重要。一个还必须负责修好的 verifier，会悄悄把标准降到自己知道怎么修的程度。

---

### 第 7 层：恢复对准失败类别

默认的恢复策略是：坏了，再跑一遍。

那不是恢复。那是带着 API 账单的老虎机。

![Failure classification and recovery](./media/11.jpg)

先分类，再响应。

```javascript
tool timeout              -> back off, then repeat
invalid arguments         -> repair the call
missing context           -> retrieve the specific source
failed test               -> inspect the behavior
permission denied         -> request approval or take the safe path
contradictory constraints -> escalate to a human
same failure, unchanged   -> stop
```

底下的规则：一次重试必须改变至少一个相关条件。

否则你是在付费复现已知结果。

> 每个 loop 都需要预算。最大尝试次数、最大挂钟时间、最大花费、最大破坏范围，以及一个升级触发器。

可靠的 agent 知道如何继续。

好的 agent 知道何时继续已经不再合理。

---

## 第三部分：让运行可读

干净的答案可以掩盖丑陋的过程。

Agent 可能读了错误数据、忽略了失败命令、对外 API 调了两次、烧掉十倍预算，却因一个下周站不住的理由「碰巧对了」。

你要的是能重建这次运行的轨迹。

```python
09:14  contract created
09:15  context loaded: docs/refunds.md
09:17  edited services/refunds/queue.ts
09:18  focused test failed: partial refund double count
09:21  repair applied
09:22  focused test passed
09:24  integration suite passed
09:25  deploy blocked: approval required
```

记录状态转换、context 来源、工具输入输出、验证结果、重试原因、审批、成本、延迟。

不是监控。

是第 14 步重启与从零重启之间的差别。

交付回执，而不是 transcript

没人会审四百轮 agent 闲聊。

一次运行结束时，编译成这样。

| 字段 | 值 |
| --- | --- |
| Objective | clear refunds older than 7 days |
| Changed | queue split logic, one regression test |
| Verified | lint, unit, refund integration suite |
| Not verified | production payment provider, legacy mobile client |
| Decisions | kept existing eligibility check as single source |
| Risks | staging replay used a 3 week old dataset |
| Needs approval | deploy to staging |

回执不是模型说了什么的摘要。

它是系统能证明什么的摘要。

最好的交接从来不是「这是对话」。

而是「这是状态、证据，以及未解决的风险」。

---

## 第四部分：飞轮，然后修剪

弱团队修坏掉的输出。

强团队修让它坏掉的那个东西。

![Build it then delete it diagram](./media/12.jpg)

失败之后，跑一遍诊断。

> 契约是否含糊？  
> Context 是否看不见？  
> 是否暴露了错误工具？  
> 是否缺了前置条件？  
> 结果是否不可验证？  
> 策略是否还住在 prompt 里？  
> 恢复是否太宽？  
> 轨迹是否太薄？  

然后把教训往下移。

**explanation → checklist → template → automated check → enforced policy**

「用 formatter」变成自动跑的 formatter。

「不要跨层 import」变成架构测试。

Prompt 承载判断。Harness 承载不变量。

修好的答案帮一次运行。修好的 harness 帮之后每一次运行。

然后删掉一半

---

## Harness 也会腐烂。几乎没人写这部分。

你为去年的 context 上限做的变通，现在挡住了更好的策略。

你在某个糟糕周加的重试规则，现在每次调用多四秒，而那种失败三月就停了。

把 harness 组件当生产代码对待。衡量它们是否仍赚回成本。

对每个路由器、评估器、记忆层和重试规则：

- 它防止哪种失败？
- 那种失败现在还多常发生？
- 它增加了多少延迟与复杂度？
- 去掉它会坏什么？

最好的 harness 不是最大的那个。

而是能可靠弥合意图与证据之间缺口的最小系统。

为删除而构建。

---

## 第五部分：构建顺序

你不需要编排平台。你需要一次加一层，而且每一层都是在你亲眼见过失败之后才加。

**1. 有界任务。** Objective、scope、constraints、acceptance。

**2. 可读环境。** 地图、命令、本地规则、依赖。

**3. 受控动作。** 类型化工具、校验、路径限制、结构化结果。

**4. 持久执行。** 显式状态、检查点、决策、教训。

**5. 证据。** 确定性检查、对抗式验证、回执。

**6. 恢复与学习。** 失败类别、有界重试、升级、harness 更新。

别因为一条 prompt 偶尔需要澄清，就开局上多 agent 架构。

复杂度应由观察到的失败挣得，而不是提前臆想。

规格表

在授予真正自主性之前填完这些。

**1. CONTRACT**              objective, scope, constraints, acceptance evidence  
**2. CONTEXT**               always-loaded map, retrieval sources, freshness rules  
**3. TOOLS**                 allowed set, preconditions, side effects, timeouts  
**4. STATE**                 facts, decisions, progress, lessons, checkpoints  
**5. POLICY**                automatic, approval-required, prohibited, budget caps  
**6. VERIFICATION**          deterministic checks, adversarial checks, acceptance  
**7. RECOVERY**              failure classes, retry limits, escalation, rollback  
**8. OBSERVABILITY**         trace events, metrics, final receipt  

空白字段意味着 agent 并不自主。

它只是在用你的凭证即兴发挥。

衡量正确的东西

Token 数不是指标。尝试的任务数也不是。

单位是被接受的工作。

```yaml
accepted outputs
--------------------------------------------
    human review minutes + run cost
```

同时跟踪首通接受率、工具失败后的恢复率、反复失败率、每任务干预次数，以及无支撑的完成声明。

这能杀死一种特定幻觉。

Agent 可以看起来极其高产，同时给人类制造昂贵的审查工作。

活动量不是产出。

何时跳过这一切

并非每次模型调用都需要操作系统。

当任务很短、输出容易肉眼检查、失败代价低、没有外部变化、而且你在看着时，用一条普通 prompt 就行。

> 当工作跨工具或会话、环境会在脚下移动、动作有后果、完成很难靠阅读判断、同一失败反复出现，或审查成了瓶颈时，再构建 harness。

---

## 真正的转变

第一代 AI 产品围绕 prompt 构建。

下一代正围绕环境构建。

问题不再是「我们如何得到更好的答案」。

而变成：我们如何构建一个系统，让好动作容易、危险动作受控、失败可见、完成可证明。

模型提供智能。Harness 提供结构。

可靠性住在第二个里。

如果你的 agent 不断散架，别再给 prompt 加形容词。

建造它成功所需的环境。

---

## 关掉页签之前

你不会靠读一遍就记住七层。

> **收藏这篇，下次 agent 又在你面前失败时打开它，用第一部分给坏掉的那一层命名。**

这才是本指南的全部意义。

### **在 X 上关注 [@spectnfa](https://x.com/spectnfa)，获取更多关于 agent 架构与生产中会坏掉的东西。**

也把这篇发给团队里那个仍在靠加长 prompt 修每一次失败的人。
