---
title: "Harness Engineering：用 6 层构建可靠的 AI Agent"
author: "Ichigo (@iiiichigo_chan)"
source_url: https://x.com/iiiichigo_chan/article/2093765205276713218
published_at: 2026-08-29
archived_at: 2026-09-08
x_post_id: 2093765205276713218
archive_note: browser capture
lang: zh-Hans
translation_of: article.md
---

# Harness Engineering：用 6 层构建可靠的 AI Agent

![Cover image](./media/01-cover.jpg)

更好的 prompt 可以改进一个答案。

更好的 harness 可以改进每一次运行。

如果你的 agent 能推理，却仍然忘记约束、选错工具、跳过验证，或 loop 到预算耗尽——模型并不是全部问题。

模型周围的环境定义不足。

本指南给你一套实用的六层 harness，可以套在编码、研究、支持或运维 agent 外。

读完之后，你将拥有：

- 一份任务契约

- 一个 context 编译器

- 一个带权限的工具网关

- 持久状态

- 证据门控

- 一个轨迹与恢复 loop

不是又一份巨型 prompt。而是 agent 的操作系统。

## 为什么现在重要

2026 年 2 月，OpenAI 描述了一款内部产品，**零行人工手写代码**。

五个月后，仓库大约有一百万行代码、约 1500 个已合并的 pull request。OpenAI 估计，该产品的构建时间约为人工开发所需时间的十分之一。

有趣的不只是 Codex 能写代码。

而是人类工程师在那些代码变得有用之前，必须在 Codex 周围构建什么。

早期进展缓慢，是因为环境定义不足。agent 缺少工具、内部结构、可观察反馈和可强制执行的规则。当它失败时，有用的问题不是「如何让 prompt 听起来更强？」而是：

> 缺了哪项能力，我们如何让它对 agent 既可读、又可强制执行？

这就是 harness engineering。

模型提供概率推理。

harness 把推理变成受控执行。

```plaintext
MODEL
proposes the next action

HARNESS
selects context
authorizes tools
stores state
collects evidence
enforces limits
recovers from failure
```

prompt 是系统的一个输入。

它不是系统本身。

![Article illustration](./media/02.png)

## 最小可用 harness

有用的 harness 不需要二十个服务或多 agent swarm。

它需要显式处理六项工作。

### 1. 把请求变成契约

自然语言请求很灵活。生产任务不能如此。

在模型行动之前，harness 应将请求翻译成有边界的任务对象：

```yaml
task_id: feature_042
goal: Add CSV export to the analytics dashboard

inputs:
  - issue.md
  - repository
  - design/export-flow.png

constraints:
  - preserve the public API
  - do not change the database schema
  - do not add a new dependency

deliverable:
  type: pull_request

done_when:
  - tests pass
  - typecheck passes
  - exported CSV matches the fixture
  - UI screenshot passes review

escalate_when:
  - schema change appears necessary
  - tests fail three times for the same reason
  - requested behavior conflicts with an existing product rule
```

这可以防止静默的任务替换。

没有契约，agent 可以解决一个更容易的问题版本，并自信地宣称成功。

契约还让 harness 有客观可评估的东西。「看起来不错」不是停止条件。「四项检查全部通过」才是。

### 2. 编译 context，而不是倾倒 context

Context 是有限的注意力预算。

常见错误是注入一切：完整对话、每个工具结果、全部项目文档，以及一份 `1,000-line` 的指令文件。

更多 context 并不自动等于更多理解。

OpenAI 的实用规则很简单：**给 agent 一张地图，而不是一本手册**。Anthropic 推荐同样的总体方向：保持 context 高信号，并在恰当时机检索额外信息。

构建一个 context 编译器，只组装当前步骤所需：

```typescript
function buildContext(task, state) {
  return [
    load("AGENTS.md"),                 // small project map
    load(task.relevantProductSpec),    // task-specific rules
    load(task.relevantArchitecture),   // local boundaries
    summarize(state.completedSteps),   // compact history
    state.openRisks,
    state.currentArtifacts
  ];
}
```

使用渐进式披露：

```plaintext
AGENTS.md
  -> architecture index
  -> product rules
  -> task-specific guide
  -> exact files and evidence
```

根指南告诉 agent 知识住在哪里。

工具仅在变得相关时才检索更深的材料。

对话不应是你的数据库，系统 prompt 也不应是你的文件柜。

### 3. 在模型与每个工具之间放一个网关

模型可以请求一个动作。

harness 决定该动作是否有效、是否被允许、是否安全可执行。

```typescript
async function handleToolRequest(request, run) {
  validateSchema(request);

  const decision = policy.authorize({
    tool: request.name,
    args: request.args,
    task: run.contract,
    risk: classifyRisk(request)
  });

  if (decision === "deny") {
    return observation("permission_denied");
  }

  if (decision === "approval_required") {
    return pauseForHumanApproval(request);
  }

  const result = await sandbox.execute(request);
  return normalizeObservation(result);
}
```

每个工具需要：

- 一个清晰用途

- 无歧义的 schema

- 有范围的权限边界

- 可预期的成功响应

- 结构化的失败响应

- 超时

工具结果应返回模型可以推理的观察，而不是无界的终端输出墙。

```json
{
  "status": "failed",
  "tool": "run_tests",
  "reason": "2 snapshot mismatches",
  "evidence": [
    "artifacts/home-mobile-before.png",
    "artifacts/home-mobile-after.png"
  ],
  "retryable": true
}
```

好的工具设计减少模型必须猜测的决策数量。

坏的工具设计让每个动作都变成另一个推理问题。

### 4. 把记忆外置为持久状态

长时运行的 agent 最终会撞上 context 限制、崩溃、重启，或把工作交给另一个 agent。

如果关键状态只存在于 transcript 中，这次运行就很脆弱。

把工作状态持久化到模型之外：

```json
{
  "task_id": "feature_042",
  "status": "verifying",
  "current_step": "mobile_visual_check",
  "completed": [
    "implementation",
    "unit_tests",
    "desktop_visual_check"
  ],
  "decisions": [
    "reuse existing export endpoint",
    "preserve current date format"
  ],
  "artifacts": [
    "export.csv",
    "desktop-after.png"
  ],
  "open_risks": [
    "mobile toolbar may overflow at 390px"
  ],
  "next_action": "render mobile viewport"
}
```

分开存储四类记忆：

```plaintext
FACTS       stable project knowledge
DECISIONS   choices made during this task
STATE       where the current run is now
LESSONS     failures that should change future runs
```

这种区分很重要。

临时工具输出在被摘要后应消失。架构决策应在每次 context 重置后存活。反复失败的教训应变为规则或测试。

记忆不是「保存整段聊天」。

记忆是保留正确继续所需的最小信息集。

### 5. 让证据成为完成的门控

模型产出产物。

环境产出关于该产物的证据。

harness 决定证据是否充分。

```typescript
async function verify(artifact, contract) {
  const evidence = await Promise.all([
    runTests(),
    runTypecheck(),
    validateOutputSchema(artifact),
    renderAndCaptureScreenshots(),
    checkScope(contract.constraints)
  ]);

  const failed = evidence.filter(check => !check.passed);

  if (failed.length === 0) return { status: "accept", evidence };
  if (canRepairLocally(failed)) return { status: "retry", failed };
  return { status: "escalate", failed };
}
```

先使用确定性检查：

```plaintext
text
CODE       tests + types + lint + dependency rules
UI         render + screenshot + interaction replay
RESEARCH   source coverage + citation match + contradiction check
DATA       schema + range + freshness + reconciliation
SUPPORT    policy check + PII check + approval boundary
```

然后对需要判断的工作使用基于模型的审查者。

制作者与检查者不应共享完全相同的激励。写了答案的模型仍可审查它，但带有不同指令和新鲜 context 的独立验证器更难被糊弄。

自主性应仅在证据质量随之扩展时扩展。

![Article illustration](./media/03.png)

### 6. 记录运行并从精确失败中恢复

没有轨迹，失败变成故事。

有了轨迹，它变成可复现的测试用例。

记录：

```json
{
  "run_id": "run_2026_08_29_0142",
  "contract_version": "3",
  "model_route": "reasoning-large",
  "context_sources": ["AGENTS.md", "docs/export.md"],
  "tool_calls": 17,
  "state_changes": 6,
  "verification": {
    "passed": 4,
    "failed": 1
  },
  "retries": 1,
  "cost_usd": 2.84,
  "stop_reason": "human_approval_required",
  "rollback_point": "git:9cf31d2"
}
```

然后在重试前对失败分类：

```typescript
switch (failure.type) {
  case "missing_context":
    updateProjectMap(failure.source);
    break;
  case "bad_tool_contract":
    improveToolSchema(failure.tool);
    break;
  case "missing_guardrail":
    addPolicyCheck(failure.action);
    break;
  case "weak_verification":
    addRegressionTest(failure.example);
    break;
  default:
    escalateWithEvidence(failure);
}
```

不要用更情绪化的 prompt 盲目重跑同一个环境。

修复缺失的能力，重跑精确失败用例，并让修复永久化。

![Article illustration](./media/04.png)

最好的 harness 会复利。

一次失败改进未来每一次运行。

## 实用的权限阶梯

模型不应批准自己的高风险动作。

分离提议、授权与执行：

```plaintext
MODEL PROPOSES
      ↓
POLICY AUTHORIZES
      ↓
TOOL EXECUTES
      ↓
HARNESS RECORDS THE RESULT
```

一个简单的起始策略：

```yaml
permissions:
  read_files:
    mode: automatic

  write_workspace:
    mode: automatic
    requires:
      - isolated_workspace
      - diff_recorded

  send_message:
    mode: approval_required
    requires:
      - final_content_preview

  deploy_production:
    mode: approval_required
    requires:
      - tests_pass
      - rollback_ready

  delete_data:
    mode: approval_required
    requires:
      - exact_targets
      - recovery_plan
```

不要对每个任务施加最大摩擦。

阅读公开文档与删除客户记录，不应走同一条审批路径。

让控制匹配后果。

## 最小有用的项目结构

你可以不依赖框架就构建第一版：

```plaintext
agent-harness/
├── AGENTS.md              # small map, not an encyclopedia
├── contracts/
│   └── task.schema.json
├── context/
│   ├── architecture.md
│   ├── product-rules.md
│   └── security.md
├── tools/
│   ├── registry.json
│   └── permissions.yaml
├── state/
│   ├── current.json
│   └── decisions.md
├── checks/
│   ├── verify.ts
│   └── regression-cases/
├── runs/
│   └── traces.jsonl
└── lessons/
    └── harness-updates.md
```

文件夹名字不重要。

职责分离才重要。

## 按这个顺序构建

不要从 swarm 开始。

从能证明自身工作的最小 loop 开始。

### 步骤 1 — 定义「完成」

写下契约，以及两三项判定成功的检查。

### 步骤 2 — 包装一个工具

给它 schema、超时、权限规则和结构化结果。

### 步骤 3 — 持久化一个状态文件

存储已完成步骤、决策、产物、未决风险和下一步动作。

### 步骤 4 — 加一条恢复路径

当检查失败时，返回精确证据，并允许一次有界修复尝试。

### 步骤 5 — 保存轨迹

记录加载了什么 context、运行了哪些工具、什么变了、哪些检查通过了，以及为什么停止。

### 步骤 6 — 把反复失败变成基础设施

每一个反复出现的错误都应变成以下四者之一：

```plaintext
text
a clearer map
a better tool
a stricter permission
a new test
```

只有在此之后，你才应增加更多自主性、更多工具或更多 agent。

## Harness engineering 不是什么

它不是一份 5000 行的系统 prompt。

它不是把你能连上的每个工具都给 agent。

它不是永远存储原始 transcript 并称之为记忆。

它不是给没有客观验收标准的工作再加一个 reviewer agent。

它不是重试到某次随机运行看起来不错为止。

它也不是从每个决策中移除人类。

harness 的存在，是为了把人类注意力花在判断重要之处，并自动化其余部分。

## 真正重要的指标

不要为生成的 token、发起的工具调用或开始的任务优化。

优化：

```plaintext
accepted outputs
----------------
human review minutes
```

这个比率捕捉了 harness 应当做的事：把模型能力转化为有用、可审查的工作，而不是在出口处消耗等量的人力。

## 真正的转变

Prompt engineering 问：

我该告诉模型什么？

Context engineering 问：

模型现在该知道什么？

Harness engineering 问：

什么系统让模型能够行动、证明工作、恢复，并安全改进？

模型会不断变化。

你的 harness 是运营知识复利的地方。

构建契约。

编译 context。

门控工具。

持久化状态。

要求证据。

把失败变成基础设施。

这就是有能力的模型如何变成可靠的 agent。

感谢阅读。

如果你喜欢本文，请关注 [@iiiichigo_chan](https://x.com/iiiichigo_chan)

## 延伸阅读

OpenAI — Harness engineering: leveraging Codex in an agent-first world ([https://openai.com/index/harness-engineering/](https://openai.com/index/harness-engineering/))

OpenAI — Unrolling the Codex agent loop ([https://openai.com/index/unrolling-the-codex-agent-loop/](https://openai.com/index/unrolling-the-codex-agent-loop/))

Anthropic — Effective context engineering for AI agents ([https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))

Anthropic — Writing effective tools for AI agents ([https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents))
