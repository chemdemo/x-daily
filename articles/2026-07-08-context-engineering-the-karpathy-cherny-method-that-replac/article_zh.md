---
title: "Context Engineering：取代 prompting 的 Karpathy-Cherny 方法"
author: "vartekx (@vartekxx)"
source_url: "https://x.com/vartekxx/status/2074864291568664646"
article_url: "https://x.com/i/article/2074824432510521344"
published_at: "2026-07-08"
archived_at: "2026-09-19"
x_post_id: "2074864291568664646"
x_article_id: "2074824432510521344"
archive_note: "Complete article captured from the X page in a browser on 2026-09-19; media retained in ./media/. Direct /i/article/ URL returns 404; status URL used."
lang: zh-Hans
translation_of: article.md
---

# Context Engineering：取代 prompting 的 Karpathy-Cherny 方法

![Article cover](./media/01-cover.webp)

> Prompt engineering 是一条指令。Context engineering 是整套操作系统。

---

### 同一个模型，在同一套 benchmark 上可以得 0.637，也可以得 0.488。

同样的权重。同样的问题。唯一的差别，是模型作答时 context window 里还装着什么。

这不是四舍五入的误差。这是「好用的工具」和「昂贵的自动补全」之间的差距。

所有人都在吵哪个模型最好。GPT vs Claude vs Gemini。Benchmarks、排行榜、价目表。与此同时，真正在用 AI 交付产品的人，几个月前就已经不再纠结模型了。

> 他们在做的是 engineering the context。

Context engineering 是 2025 年取代 prompt engineering 的那项技能。到了 2026 年，两个人把它做成了一套系统：Andrej Karpathy 定义了框架。Boris Cherny 做出了工具。

本文将说明：

1 - context engineering 究竟是什么，以及为什么你的 prompt 连真正重要因素的 5% 都不到

2 - Karpathy 的框架：context window 就是你的程序

3 - Cherny 的框架：别再 prompting，开始设计 loops

4 - 如何在 Claude Code 上自己搭起来，附可直接复制粘贴的 prompts

5 - 诚实的部分：这套方法修不了什么

### 先收藏这篇文章。

---

## Part 1 | 大多数人错过的演进

有一条时间线，大多数 AI 用户都没注意到。四年里三次范式切换：

![Evolution of prompt, context, and loop engineering](./media/02-evolution.png)

Prompt engineering 是写一条好指令。你雕琢完美的句子，按下回车，然后祈祷一切顺利。

Context engineering 是设计模型看到的一切：哪些文件、哪些历史、哪些工具结果、哪些规则。Prompt 只是其中一个组件。Context 才是整套操作系统。

Loop engineering 是设计那个替你自动、反复做 context engineering 的系统——你睡觉时它也在跑。

每一层都不取代上一层。它叠在上面。完美 loop 里塞一条潦草的 prompt，仍然只会更快地产出潦草的工作。但杠杆已经转移了。而大多数人还没意识到。

---

## Part 2 | Karpathy

## Context window 就是你的程序

![Karpathy / context window](./media/03-karpathy.jpg)

2026 年 4 月，Andrej Karpathy 在 Sequoia AI Ascent 上做了一场演讲，重塑了整个行业对 AI 的思考方式。

他的框架：Software 3.0。

- Software 1.0：人类写显式代码
- Software 2.0：人类用数据训练神经网络
- Software 3.0：人类通过 context 给模型「编程」

重点不在「写更好的 prompts」。重点在于：context window 成了新的编程表面。你不再写确定性指令。你是在给一个能读、能推理、能调工具、能检查环境、能适应的智能解释器提供 context。

> "Context engineering is the delicate art and science of filling the context window with just the right information for the next step."  - Andrej Karpathy

把 LLM 想成一套新操作系统。模型是 CPU。Context window 是 RAM。就像操作系统决定什么装进 CPU 的 RAM，context engineering 决定什么装进模型的工作记忆。

好 agent 和坏 agent 的差别不在模型。而在模型运行时，context window 里面装了什么。

![Four operations of context engineering](./media/04-operations.png)

Context engineering 的四种操作

Karpathy 与研究社区收敛到了四种核心操作。你对 context 做的一切，都落在其中之一：

- Write（写入）——把 context 持久化到窗口之外。CLAUDE.md、skills、state files。让 agent 之后可以读回来，而不是一直占着内存。
- Select（选择）——只检索此刻相关的东西。不是全部。不是随机切块。是从 50,000 份文档里挑对的 5 份。
- Compress（压缩）——总结旧信息以节省 tokens。历史太长时就压紧。新鲜的工具结果永远优先于陈旧对话。
- Isolate（隔离）——给子任务各自干净的 context。这就是 Boris Cherny 发明的「context firewall」。每个 subagent 拿到一个全新窗口。只有结构化输出流回来。

跳过这些操作，就会撞上一个真实问题，叫「context rot」（上下文腐烂）。对话变长时，无关 tokens 堆积，信噪比下降，模型开始做出更差的决策。窗口并没有变小。它变乱了。

> 同一个模型在 MMLU 上可以得 0.637 或 0.488，差别仅仅取决于你如何组织 context。同一模型。同一问题。不同 context。不同结果。

---

## Part 3 | Cherny

### 别再 prompting。开始设计 loops。

![Cherny's loop engineering](./media/05-cherny.png)

Boris Cherny 构建了 Claude Code。他每天都在用。而在 2026 年 6 月，他说了一句传遍整个开发者社区的话：

> "I don't prompt Claude anymore. I have loops running that prompt Claude and figuring out what to do. My job is to write loops." - Boris Cherny, creator of Claude Code

这不是营销话术。Cherny 在自己的工作里跑着持续 loops：一个 agent 猎取架构改进，另一个统一重复的抽象，两者无限地提交 PRs。Loop 负责 prompting。他负责设计系统。

Loop Engineering 如何接到 Context Engineering

大多数文章把 loops 说成 context engineering 之后的「下一层」。这种说法是反的。没有好的 context engineering 的 loop，只是一种昂贵的自动量产垃圾的方式。Context engineering 是菜谱。Loop 是反复按菜谱下厨的厨房。

Loop 的每个周期都在做 Karpathy 描述的那四种操作：

- Write：每次运行后，loop 把状态存到磁盘（什么奏效了、什么失败了、下一步是什么）
- Select：下一轮周期，loop 只加载相关状态，而不是完整历史
- Compress：旧运行被总结。新鲜结果获得优先权。
- Isolate：Subagents 在各自的 context windows 里处理子任务。主 loop 保持干净。

如果这四种操作很差，loop 会让它们更快地变差。如果它们很好，loop 会让它们永远很好。Context engineering 是地基。Loop 是驱动它运转的引擎。

这就是为什么本文先讲 context engineering，再讲 loops。先把 context 做对。然后再自动化它。

![Building blocks of a working loop](./media/06-loop.png)

Cherny 的工作 loop 五大构建块

每一个能跑起来的 loop——无论你用 Claude Code、Codex 还是 bash scripts 来搭——都由五块拼成：

- Automation（自动化）——心跳。/loop 管节奏，/goal 管停止条件。
- Skill（技能）——写在 markdown 文件里的项目知识。写一次，每次运行都读。
- Sub-agents（子代理）——把做的人和查的人拆开。一个写，另一个验。
- Connectors（连接器）——让 loop 在真实环境里动手。开 PRs、ping Slack、更新工单。
- Verifier（验证器）——闸门。Tests、type checks、builds，自动拒绝劣质产出。

Cherny 分享过一个数据点：只要给 Claude 有效的 verification 方法，最终输出质量通常能提升 2–3 倍。

> 没有 verifier，你就没有 loop。你只有 agent 在一遍遍跟自己达成一致。

---

## Part 4 | 自己动手搭

### 在 Claude Code 上做 Context Engineering：一步步来

![Claude Code context engineering](./media/07-claude-code.jpg)

理论到此为止。下面是怎么搭。下面每条 prompt 都可以直接复制粘贴。

> 4.1 - 搭好 context 层（CLAUDE.md）

CLAUDE.md 是持久化的 context 层。每次会话开始都会加载。没有它，每次会话都从零开始，agent 会重新发明你的约定。

```bash
claude /init
```

Claude 会根据你的 codebase 脚手架出一份基线。然后你来精修：

```markdown
# Project: e-commerce-api
# Stack: Node.js, TypeScript, PostgreSQL, Prisma

Test command: bun test
Lint command: bun run lint

# Rules
- Functional style, no classes
- Explicit error handling, no silent catches
- All API routes require Zod validation
- Payment routes require idempotency keys
- Never use `any` type
- Always run tests before committing
```

控制在 200 行以内。Claude 大约能可靠关注约 150 条指令。每一行都必须挣来自己的位置。

> 4.2 - 写 specs，而不是 prompts

Prompt 是愿望。Spec 是命令。Context engineering 版的「写一条好 prompt」，是「设计模型产出正确结果所需的完整输入」。

```plaintext
❌ bad prompt:

Refactor the auth system
```

```plaintext
✅ good spec:

# Task: Migrate auth from session cookies to JWT

GOAL: Replace session-based auth with JWT tokens
      across all 23 API routes in /src/routes/

SCOPE:
- IN: /src/routes/, /src/middleware/auth.ts
- OUT: /src/routes/webhooks/ (leave untouched)

OUTPUT:
- Modified route files with JWT validation
- New /src/middleware/jwt.ts
- Updated tests for every changed route

ON CONFLICT: flag the file, do not resolve silently
STOP CONDITION: all tests green, no type errors
```

> 4.3 - 用 subagents 隔离 context

这就是 Cherny 的「context firewall」。每个 subagent 拿到自己干净的 context window。主 agent 保持专注。

```plaintext
Break this migration into subtasks.
Spawn a subagent for each route file.
Each subagent: migrate the route, update
its tests, verify tests pass.
Report only failures back to me.
```

没有 subagents，单个 agent 做复杂任务会把 context 填满直到淹死。Subagents 让每个子任务保持范围清晰、上下文干净。

> 4.4 - 加上 verification（闸门）

CLAUDE.md 是建议性的，大约 70% 会被遵守。Hooks 是确定性的，100% 强制执行。凡是不可妥协的事，用 hooks。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $FILE_PATH"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Run full test suite. If any test fails, fix it. Do not stop until all tests pass."
          }
        ]
      }
    ]
  }
}
```

那个 Stop hook 就是 verifier 闸门。测试不过，Claude 就不能收工。不需要人盯。正是这一块把 prompt 变成了 loop。

> 4.5 - 搭建自我改进的 context loop

Karpathy 与 Cherny 在这里汇合。Agent 看到的 context 应该每跑一轮都更好，而不是重置归零。

```plaintext
After completing the task:
1. Review what succeeded and what failed
2. Score output against the spec criteria
3. Write 2-3 short entries to learnings.md
4. If any failure repeated a known mistake
   from learnings.md, escalate it to CLAUDE.md
   as a hard rule

Keep entries action-oriented.
One-liners get used. Dense paragraphs get skipped.
```

```markdown
# learnings.md - loaded every session

2026-07-01: Payment API expects idempotency key
  in the header, not the body. Added to CLAUDE.md.

2026-07-03: Auth middleware expects token in
  x-auth-token, not Authorization. Three subagents
  hit this independently. Now a hard rule.

2026-07-05: Test suite takes 45s full run.
  Use --filter for iteration, full suite on
  final commit only.
```

每轮运行写入新的 learnings。下一轮读它们。Context 会复利增长。这就是把 Karpathy Loop 应用到 context 本身。

![Learnings file example](./media/08-learnings.png)

> 4.6 - 用 /loop 自动化

现在把它变成真正的 loop。/loop 给 Claude 一个心跳。它会反复跑你的任务，并在迭代之间维持 session context。

```plaintext
/goal all tests pass and coverage is above 80%

/loop every 10 minutes, check test results.
If any test fails, read the error, fix it,
and re-run. If coverage is below 80%, find
untested functions and add tests.
```

/goal 定义何时停止。/loop 定义节奏。两者一起，形成一个会一直干到完工的自主 agent。

```plaintext
/loop every 5 minutes, check my open PR.
If CI is red, pull the failing job log,
diagnose, fix the issue, and push.
If new review comments arrived, address
each one and resolve the thread.
If everything is green, say so in one line.
```

> 4.7 - 升级到 Routines，实现 24/7

关掉终端，/loop 就死了。Routines 跑在 Anthropic 的云端基础设施上，即使你的笔记本关机也在跑。

```plaintext
Every Monday at 9am:
Scan all PRs merged in the past 7 days.
For each PR, check if documentation files
reference modified functions or APIs.
If docs are outdated, open a PR with fixes.
If everything is current, report "no drift".
```

三种触发类型：schedule（cron）、API call（webhook）、GitHub event（PR opened、push to main）。在 claude.ai/code/routines 配置，或在 CLI 里用 /schedule。

> 4.8 - 用 Dynamic Workflows 扩展规模

当一个 agent 不够用时。Claude 会写自己的编排脚本，扇出 tens 到 hundreds 个并行 subagents，交叉核对结果，再交付一份经验证的输出。

```plaintext
Audit every API route in /src/routes/ for
missing input validation.
Spawn one agent per route file.
Each agent: check for Zod schema, report
any missing validation.
Cross-check findings with an adversarial
reviewer before reporting.
Output: one markdown report with file,
line number, and severity.
```

关键洞察：编排活在代码里，而不是模型的 context window 里。协调不花 tokens。所有 tokens 都花在真正干活上。这是 context engineering 最极致的形态：每个 subagent 窗口里的每一个 token 都在干活，而不是在协调。

---

## Part 5 | 诚实的部分

### Context engineering 修不了什么

- 更多 context 并不总是更好的 context。过了某个点，噪声会淹没信号。一份每行都挣来位置的 200 行 CLAUDE.md，胜过一份 2,000 行的倾倒。
- 即使在完美的 context 里，模型仍会幻觉。Context 减少错误。它并不消灭错误。Verifier 不是可选项。
- 这是大多数团队还没有的新技能。不是写 prompt。也不是软件工程。而是去想模型需要看到什么，而不是你想说什么。

> Context window 是你的程序。但你仍然是那个程序员。

---

## 结语：

> Prompt engineering 给你最初的 10%。Context engineering 给你接下来的 90%

大多数人会继续写一行 prompts。会继续手动把文件复制进聊天窗口。会继续纳闷为什么模型「不懂」他们的项目。

理解这场转变的建造者会看得很清楚：context window 就是程序。模型是解释器。Loop 是那个自动填充 context、验证输出、从结果中学习、并且每一轮都变得更锋利的系统。

Karpathy 给了我们框架。Cherny 给了我们工具。剩下的就是 engineering。

设计 context。搭建 loop。让系统去做 prompting。
