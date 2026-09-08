---
title: "Harness Engineering: How to Build AI Agents That Don't Fall Apart"
author: "rari (@0xwhrrari)"
source_url: "https://x.com/0xwhrrari/article/2093685107534000560"
published_at: "2026-08-29"
archived_at: "2026-09-08"
x_post_id: "2093685107534000560"
archive_note: "Browser-captured: H3, 16 inline code blocks, 3 images, quotes + source links. Media in ./media/."
---

# Harness Engineering: How to Build AI Agents That Don't Fall Apart

![Cover](./media/01-cover.jpg)

Most people respond to a failing agent by changing the prompt

Then they change the model

Then they add a larger context window

The agent still forgets decisions

It still uses the wrong tool

It still skips verification

It still gets stuck in the same loop

The problem is not always the intelligence

The problem is the environment around it

That environment is the harness

And designing it is harness engineering

Dario Amodei, CEO of Anthropic, said it directly while explaining how Claude Code emerged

> "Of course, you need an interface, you need a harness to use them"

I publish practical breakdowns of AI agents, workflows, and production systems on Substack [Join the newsletter here](https://whrrari.substack.com/subscribe?next=https%3A%2F%2Fsubstack.com%2F%40whrrari%2Fnotes&utm_source=profile-page&utm_medium=web&utm_campaign=substack_profile&just_signed_up=true)

### The model is only the reasoning engine

A model can suggest the next action

It cannot create a reliable operating environment by itself

The harness decides what the model can see, what it can touch, what survives between sessions, what counts as evidence, and when the run must stop

```text
MODEL
reasons and proposes actions

HARNESS
selects context
exposes tools
stores state
enforces permissions
checks results
records traces
recovers from failure
```

The prompt is one component inside this system

The model is another

The product is what happens when every surrounding component works together

Prompt engineering improves the instruction

Harness engineering improves the conditions under which the instruction is executed

### The same model can become a completely different agent

Put the same model inside a chat box and it answers questions

Put it inside a repository with terminal access, tests, browser tools, project memory, isolated worktrees, and a review loop and it can ship software

The weights did not change

The harness did

OpenAI described the same shift while building an agent-first codebase with Codex

Their early progress was slow because the environment was underspecified, not because the model lacked raw capability

The response was not to tell the agent to try harder

It was to ask what capability was missing and make that capability both legible and enforceable

> "The environment was underspecified"

[OpenAI, Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)

This is the central idea

When an agent fails repeatedly, stop editing adjectives in the prompt

Inspect the system around the model

### A production harness has seven jobs

![figure](./media/02.jpg)

### 1. Turn the request into a contract

Before the agent acts, convert the request into a bounded object

```text
{
  "goal": "ship the feature",
  "inputs": ["issue", "repository", "design"],
  "output": "reviewable pull request",
  "constraints": ["no schema changes", "preserve public API"],
  "done_when": ["tests pass", "visual check passes", "review passes"]
}
```

The contract protects the task from silent redefinition

Without it, the agent can complete a different job and still declare success

### 2. Give the agent a map

Agents need project knowledge

They do not need every document in every context window

Use a small root guide that tells the agent where to look

```text
AGENTS.md
  -> architecture map
  -> testing map
  -> product rules
  -> security rules
  -> task-specific guides
```

A map preserves context

A giant manual consumes it

Keep the detailed knowledge close to the code, tool, or workflow it governs

Load it only when the current task needs it

### 3. Expose the right tools inside the right environment

Tool access is not a list of buttons

It is an interface between the model and the real world

Every tool needs a clear purpose, predictable output, explicit failure state, and a permission boundary

```text
READ FILES       allowed by default
RUN TESTS        allowed inside sandbox
WRITE FILES      allowed inside workspace
ACCESS NETWORK   scoped by task
DEPLOY           requires approval
DELETE DATA      requires approval
```

Good tools reduce ambiguity before the model has a chance to reason badly

Bad tools force the model to guess what happened

### 4. Externalize memory into durable state

The conversation is not the system of record

Store decisions, artifacts, failures, and open risks outside the context window

```text
{
  "task_id": "task_042",
  "current_step": "verify_ui",
  "artifacts": ["build.zip", "report.md", "screenshot.png"],
  "decisions": ["keep existing schema"],
  "failures": ["mobile overflow at 390px"],
  "pending": ["human approval"]
}
```

The next session should inherit the state of the work, not a lossy retelling of the conversation

This is how an agent survives context resets, crashes, and handoffs

### 5. Add sensors before adding autonomy

An agent cannot correct what it cannot observe

Tests, linters, screenshots, logs, metrics, and schema validators turn vague quality into evidence

```text
CODE       -> tests + type checks + lint
UI         -> render + screenshot + visual inspection
RESEARCH   -> source check + contradiction check
DATA       -> schema + range + freshness checks
```

The model creates an artifact

The environment produces evidence about the artifact

The harness decides whether that evidence is enough to continue

### 6. Enforce permissions outside the model

The model can recommend an action

The harness must authorize it

```text
MODEL SUGGESTS -> POLICY CHECKS -> TOOL EXECUTES
```

This separation matters most when the action is expensive, irreversible, or touches another person

Do not ask the same probabilistic system to invent the plan, approve the risk, and execute the side effect

### 7. Record traces and recover locally

Every run should leave a readable trail

```text
request
selected context
tool calls
state changes
verification results
retries
cost
final artifact
rollback point
```

Without traces, failure becomes a mystery

With traces, failure becomes input for the next harness improvement

### Instructions should become infrastructure

Most teams keep important rules in prose

The agent reads them

Then eventually ignores one

The stronger pattern is to encode the important rule twice

First as guidance the agent can understand

Then as a mechanical check the agent cannot bypass

```text
GUIDE
"UI code may not query the database directly"

CHECK
lint fails when UI imports the repository layer
```

The guide explains the reason

The check enforces the boundary

This turns a past failure into a permanent system improvement

The next agent does not need to remember the incident

The harness remembers for it

### The loop belongs to the harness

Long-running work needs iteration

But "keep trying until it works" is not a control system

A useful loop has evidence, bounded retries, a budget, and an escalation path

```text
for (let attempt = 1; attempt <= 3; attempt += 1) {
  const artifact = await build(state)
  const evidence = await verify(artifact)

  if (evidence.pass) return artifact

  state.failures.push(evidence.gap)
  state.repair = evidence.repair
}

return requestHumanReview(state)
```

The model should decide how to repair the local gap

The harness should decide whether another attempt is allowed

Anthropic reached a similar conclusion in its work on long-running agents

Structured artifacts preserve continuity across sessions, while a separate evaluator gives the builder concrete feedback instead of letting it approve its own work

> "Find the simplest solution possible, and only increase complexity when needed"

[Anthropic, Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)

### Failure should upgrade the system

![figure](./media/03.jpg)

Most people repair the current output

Harness engineers repair the class of failure

```text
MISSING CONTEXT   -> add a map or retrieval rule
WRONG TOOL        -> improve tool description or routing
BAD OUTPUT        -> add a validator or stronger contract
REPEATED LOOP     -> add a retry cap and escalation
UNSAFE ACTION     -> add a permission gate
LOST DECISION     -> store it in durable state
UNKNOWN FAILURE   -> add tracing and evidence capture
```

The immediate patch fixes one run

The harness change improves every run after it

That is the compounding advantage

A good harness converts agent mistakes into infrastructure

### Separate the brain, the hands, and the history

A reliable agent is easier to reason about when three components are separate

```text
BRAIN
the model that reasons

HANDS
the sandbox and tools that act

HISTORY
the append-only record of what happened
```

If the sandbox dies, the history survives

If the model changes, the tools and policy remain inspectable

If a task resumes, a new session can reconstruct the state from artifacts and traces

Anthropic's Managed Agents architecture makes this separation explicit through the session, harness, and sandbox

[https://x.com/i/web/status/2041927687460024721](https://x.com/i/web/status/2041927687460024721)

The important part is not the vendor

It is the architecture

The reasoning engine should not also be the filesystem, permission system, memory database, and audit log

### Give every run a change receipt

When the agent finishes, do not keep only the final output

Keep a compact receipt that explains how the output was produced

```text
{
  "context_sources": ["issue", "repo_map", "design_spec"],
  "policy_version": "v12",
  "model_route": "complex_coding",
  "tools_used": ["shell", "browser", "tests"],
  "tests": { "passed": 42, "failed": 0 },
  "human_corrections": 1,
  "retries": 2,
  "cost_usd": 3.84,
  "accepted_artifact": "pr_1842",
  "rollback_point": "commit_7f3a"
}
```

This makes model upgrades comparable

It makes regressions attributable

It makes audits possible

And it prevents the final answer from hiding a broken process

### Start with the smallest harness that closes the loop

Harness engineering does not mean building a platform before the first task

Start with the smallest system that can observe, verify, and recover

```text
LEVEL 0
prompt + model

LEVEL 1
project guide + tools

LEVEL 2
structured state + tests + bounded loop

LEVEL 3
permissions + traces + recovery + human gates
```

Move up only when the task earns the complexity

A short low-risk task may need one prompt and one review

A six-hour coding run that can edit files, access the network, and open a pull request needs a real harness

The harness should be smaller than the failure surface it controls

### The harness engineering checklist

Before you trust an agent with real work, ask

```text
[ ] Is success defined before execution begins
[ ] Can the agent find the right project knowledge without loading everything
[ ] Does every tool have a clear contract and failure state
[ ] Is execution isolated from production systems
[ ] Are important decisions stored outside the conversation
[ ] Does every risky transition have evidence
[ ] Are irreversible actions protected by approval
[ ] Does every loop have a retry cap and budget
[ ] Can the run resume after interruption
[ ] Can you explain every tool call and state change
[ ] Does failure update a guide, test, tool, or policy
[ ] Can the final artifact be rolled back
```

If several answers are no, a stronger model will not make the system reliable

It will only make the failure more expensive

### The real shift

Prompt engineering tells the model what to do

Context engineering decides what the model sees

Harness engineering builds the world in which the model acts

```text
PROMPT      -> instruction
CONTEXT     -> working view
HARNESS     -> operating system
LOOP        -> local improvement
GRAPH       -> coordination
```

The model may change next month

The tools, tests, state, policies, and traces can keep improving

That is why the durable advantage is moving out of the prompt and into the system around it

The best builders will not only ask which model is smartest

They will ask which environment makes that intelligence reliable

That is harness engineering

### If you read this far

-> [Subscribe to my Substack](https://whrrari.substack.com/subscribe?next=https%3A%2F%2Fsubstack.com%2F%40whrrari%2Fnotes&utm_source=profile-page&utm_medium=web&utm_campaign=substack_profile&just_signed_up=true)

-> [Join my Telegram](https://t.me/+qqS3Qn-x1305ZmUy)

-> Bookmark the article so you can use the checklist when you build your next agent

-> [Follow @0xwhrrari](https://x.com/0xwhrrari) for more practical breakdowns of agent systems
