---
title: "Harness Engineering: The Complete Guide to Building AI Agents That Don't Fall Apart"
author: "Lunar (@LunarResearcher)"
source_url: "https://x.com/LunarResearcher/article/2096570562625655088"
published_at: "2026-09-06T12:05:49.000Z"
archived_at: "2026-09-08"
x_post_id: "2096570562625655088"
---

# Harness Engineering: The Complete Guide to Building AI Agents That Don't Fall Apart

- **作者：** Lunar (@LunarResearcher)
- **出处：** https://x.com/LunarResearcher/article/2096570562625655088
- **发表：** 2026-09-06T12:05:49.000Z
- **存档：** 2026-09-08（API 原文 plain_text + 配图二进制；代码块另附 entities）

![Cover](./media/01-cover.jpg)

Most people are trying to improve AI agents at the wrong layer.
When an agent fails, they rewrite the prompt.
When it fails again, they add more instructions.
Before we start:
Follow my Substack for fresh AI alpha, agent workflows, and step-by-step guides before they hit X: https://substack.com/@lunarresearcher
Then they switch models, add more tools, increase the context window, and hope the next run behaves differently.
But many agent failures are not reasoning failures.
They are environment failures.
The agent did not know which files mattered.
It used the right tool in the wrong place.
It lost the decisions made in the previous session.
It claimed success without running the checks.
It repeated an action after a partial failure.
It had permission to do something that should have required approval.
The model was not necessarily the problem. The system around the model was incomplete.
That system is the harness.
And designing it is becoming its own engineering discipline.
Harness Engineering is the practice of building the environment that turns model intelligence into reliable work.
A prompt changes one attempt.
A harness changes every attempt.
This guide explains how to build one.
 
 
1. The Model Is Not the Agent
A model can reason, generate, compare, and choose.
But an agent must also interact with a real environment.
It needs to:
understand the task
find the relevant context
select and use tools
preserve state
respect permissions
inspect the result
recover from failure
prove that the work is complete
The model is the reasoning engine inside that system.
The harness is everything that makes the reasoning operational.
 
A powerful model inside a weak harness is still a weak agent.
 
It may produce impressive individual responses, but it will behave inconsistently across long tasks, changing environments, and partial failures.
The goal of harness engineering is not to remove uncertainty from the model.
It is to contain that uncertainty inside a system that can observe, verify, and recover.
 
2. Start With a Task Contract
Most agent tasks begin as vague intent:
Improve the onboarding flow.
That sentence may be enough for a conversation.
It is not enough for autonomous execution.
Before the agent acts, the harness should convert the request into a task contract.
A useful contract answers five questions:
 
What outcome must exist?

What is inside the scope?
What must not change?
What evidence proves completion?
Which actions require human approval?
 
This changes the agent's question from:
What should I do next?
to:
What action moves the environment toward the contracted outcome?
Without a contract, the agent optimizes for plausible activity.
With a contract, it can optimize for verified completion.
 
3. Give the Agent a Map, Not a Manual
Dumping the entire repository, documentation set, and conversation history into context is not good context engineering.
It is context flooding.
 
The harness should provide a small map first, then let the agent retrieve details when they become relevant.
 
This is progressive disclosure:
 
The context should expand because the task requires it, not because the information exists.
A good context compiler decides:
what is always needed
what can be retrieved later
what has become stale
what can be summarized
what must remain verbatim
The objective is not maximum context.
It is maximum signal per token.
 
4. Build a Tool Gateway, Not a Tool Pile
Giving an agent twenty tools does not make it capable.
It gives the agent twenty ways to make a mistake.
 
Every tool should have a clear contract:
 
The harness should control how tools are exposed and used.
It can:
hide irrelevant tools
validate arguments
restrict paths and domains
attach timeouts
make retries idempotent
normalize outputs
require confirmation for risky actions
return evidence, not just "success"
This creates an important separation:
 
The model can propose an action.
The tool gateway decides whether that action is valid enough to execute.
 
5. Separate the Brain, the Hands, and the History
Many fragile agents mix everything into one growing transcript.
Reasoning, tool calls, files, decisions, errors, and old observations all compete for the same context window.
A stronger system separates three responsibilities:
 
 
The model does not need every raw event in active context.
It needs the right current state.
The sandbox does not need to understand the entire objective.
It needs to safely execute a bounded action.
The session log does not need to reason.
It needs to preserve what happened after the current context disappears.
This separation makes long-running agents easier to resume, inspect, and repair.
It also lets you replace one part without rebuilding the entire system.
 
6. Memory Must Become Durable State
Conversation history is not reliable memory.
It is an event stream.
Useful memory should be converted into explicit state.
 
At minimum, preserve four categories:
 
For example:
 
This is far more useful than replaying fifty pages of transcript and hoping the model notices the important line.
Store raw history for auditability.
Compile durable state for execution.
 
7. Completion Requires Evidence
An agent saying "done" is not evidence that the task is done.
It is only another model output.
 
Completion must be decided by observable changes in the environment.
 
The harness should run the cheapest deterministic checks first.
 
Do not use another model where a compiler, schema, checksum, query, or test can answer the question.
Use models for ambiguity.
Use code for plumbing.
A model can propose that the task is complete.
Only the environment can prove it.
 
8. Verification Should Attack the Result
Workers and evaluators should not share the same objective.
The worker tries to create the strongest solution.
The evaluator tries to find the reason it should be rejected.
 
 
This asymmetry matters.
If you ask the same agent, in the same context, to "double-check its work," it often preserves the assumptions that created the mistake.
A useful verification stage should have:
an explicit rejection rubric
access to the produced artifact
access to the acceptance contract
independent tools or fresh context when needed
permission to reject without repairing
Verification is not a second opinion.
It is an attempted disproof.
 
9. The Model Proposes, the Policy Authorizes
Some rules should never depend on whether the model remembers them.
 
These are not prompt suggestions.
They are policy.
The safest design keeps policy outside the reasoning loop.
 
 
The stronger the consequence, the harder the gate.
Autonomy is not the absence of control.
It is the ability to operate freely inside a clearly enforced boundary.
 
10. Recovery Should Target the Failure Class
The most common recovery strategy is:
Something failed. Try again.
That is not recovery.
It is repetition.
 
The harness should classify the failure before selecting the next action.
 
A retry should change at least one relevant condition.
Otherwise the system is paying to reproduce the same failure.
A bounded agent loop looks like this:
 
Every loop needs a budget:
maximum attempts
maximum time
maximum spend
maximum destructive scope
escalation condition
Reliable agents know how to continue.
They also know when continuing is no longer rational.
 
11. Instructions Should Become Infrastructure
Agent instructions are useful when they explain local reality.
But instructions alone are weak enforcement.
If a rule matters repeatedly, move it down the stack.
 
This creates an instruction ladder:
 
Move important knowledge as far down that ladder as practical.
The prompt should explain judgment.
The harness should enforce invariants.
 
12. Observe the Run, Not Just the Final Answer
A clean final artifact can hide a terrible process.
The agent may have:
accessed the wrong data
ignored a failed command
retried an external action twice
consumed ten times the expected budget
reached the right answer for the wrong reason
You need traces that make the run reconstructable.
 
A useful trace records:
state transitions
context sources
tool inputs and outputs
environment changes
verification results
retry reasons
approval decisions
cost and latency
The goal is not surveillance.
The goal is local repair.
When a run fails at step 18, you should be able to restart from a trustworthy checkpoint instead of replaying the entire task.
 
13. Every Run Needs a Change Receipt
Long agent transcripts are difficult to review.
At the end of a run, the harness should compile a small change receipt.
 
The receipt is not a summary of what the model said.
It is a summary of what the system can prove.
This gives humans a compact review surface and gives the next agent session a trustworthy starting point.
The best handoff is not "here is the conversation."
It is "here is the state, the evidence, and the unresolved risk."
 
14. Every Failure Should Upgrade the Harness
The weakest teams fix the failed output.
The strongest teams also fix the system that allowed it.
After a failure, ask:
 
Then convert the lesson into a reusable improvement.
 
This is the harness flywheel.
The system becomes more reliable because failures leave infrastructure behind.
A corrected answer helps one run.
A corrected harness helps every future run.
 
 
15. Harnesses Decay Too
More harness is not always better.
Models improve. Tools improve. Tasks change. Old safeguards can become unnecessary friction.
A workaround created for yesterday's model may prevent today's model from using a better strategy.
This creates harness decay:
 
Treat harness components like production code.
Measure whether they still provide lift.
For every router, evaluator, memory layer, and retry rule, ask:
Which failure does this prevent?
How often does that failure still occur?
What latency and complexity does this add?
Can the same result now be achieved more simply?
What happens if we remove it?
The best harness is not the largest one.
It is the smallest system that reliably closes the gap between intent and evidence.
Build to delete.
 
16. The Minimum Viable Harness
You do not need an orchestration platform to begin.
Build the harness in layers.
Level 1: A bounded task
objective
scope
constraints
acceptance checks
Level 2: A legible environment
project map
commands
local instructions
known dependencies
Level 3: Controlled actions
typed tools
argument validation
path and permission boundaries
structured results
Level 4: Durable execution
explicit run state
checkpoints
decisions
lessons
Level 5: Evidence
deterministic checks
adversarial verification
change receipt
Level 6: Recovery and learning
failure classification
bounded retries
escalation
harness updates from recurring failures
Build the smallest layer that eliminates the failure you actually have.
Do not begin with a multi-agent architecture because a single prompt occasionally needs clarification.
Complexity should be earned by observed failure.
 
17. A Reusable Harness Specification
Before giving an agent meaningful autonomy, define this:
 
If these fields are undefined, the agent is not autonomous.
It is improvising.
 
18. Measure the System at the Right Level
Token count is not the final metric.
Neither is the number of tasks attempted.
The useful unit is accepted work.
A practical metric is:
 
Also track:
first-pass acceptance rate
recovery rate after tool failure
repeated failure rate
human interventions per task
unsupported completion claims
time from request to verified outcome
harness overhead by component
This prevents a common illusion:
An agent can look highly productive while creating expensive review work.
The objective is not more agent activity.
It is more trusted outcomes per unit of human attention.
 
19. When You Do Not Need a Heavy Harness
Not every model call needs an operating system.
Use a simple prompt when:
the task is short
the output is easy to inspect
failure is cheap
no external side effect occurs
the user remains in the loop
Add a harness when:
work spans multiple tools or sessions
the environment can change
actions have real consequences
completion is difficult to judge manually
the same failure appears repeatedly
human review becomes the bottleneck
The purpose of a harness is not to make a demo look sophisticated.
It is to make real work dependable.
 
The Real Shift
The first generation of AI products was built around prompts.
The next generation is being built around environments.
The question is no longer only:
How do we make the model answer better?
It is:
How do we build a system where good actions are easy, dangerous actions are controlled, failures are visible, and completion is provable?
That is the shift from prompt engineering to harness engineering.
The model supplies intelligence.
The harness supplies structure.
Together they produce reliable execution.
If your agent keeps falling apart, stop adding adjectives to the prompt.
Build the environment it needs to succeed.
 
If You Made It This Far
Bookmark this guide.
Follow @LunarResearcher on X
Subscribe to my Substack
Send this article to someone who is still trying to fix every agent failure with a longer prompt.


## Article images

![02.jpg](./media/02.jpg)

<!-- source: https://pbs.twimg.com/media/HRhsGHbWoAI7DMz.jpg media_key: 3_2096544478869757954 -->

![03.jpg](./media/03.jpg)

<!-- source: https://pbs.twimg.com/media/HRhrmMsbYAAdP50.jpg media_key: 3_2096543930527735808 -->

![04.jpg](./media/04.jpg)

<!-- source: https://pbs.twimg.com/media/HRhsAy3bsAAjePW.jpg media_key: 3_2096544387451039744 -->

![05.jpg](./media/05.jpg)

<!-- source: https://pbs.twimg.com/media/HRhr80OWgAAF953.jpg media_key: 3_2096544319096127488 -->

![06.jpg](./media/06.jpg)

<!-- source: https://pbs.twimg.com/media/HRhsQ62bMAEHsnR.jpg media_key: 3_2096544664472203265 -->

![07.jpg](./media/07.jpg)

<!-- source: https://pbs.twimg.com/media/HRhsMycXIAQRODq.jpg media_key: 3_2096544593495924740 -->

![08.jpg](./media/08.jpg)

<!-- source: https://pbs.twimg.com/media/HRhr4rxbsAA2VCm.jpg media_key: 3_2096544248107872256 -->

![09.jpg](./media/09.jpg)

<!-- source: https://pbs.twimg.com/media/HRhrurDaYAAgCWo.jpg media_key: 3_2096544076116156416 -->

![10.jpg](./media/10.jpg)

<!-- source: https://pbs.twimg.com/media/HRhskrUagAAzDrm.jpg media_key: 3_2096545003900403712 -->

![11.jpg](./media/11.jpg)

<!-- source: https://pbs.twimg.com/media/HRhrie5agAAwPyq.jpg media_key: 3_2096543866694565888 -->

![12.jpg](./media/12.jpg)

<!-- source: https://pbs.twimg.com/media/HRhrqmtWsAE_9Na.jpg media_key: 3_2096544006230421505 -->

![13.jpg](./media/13.jpg)

<!-- source: https://pbs.twimg.com/media/HRhsX__XsAEuCxY.jpg media_key: 3_2096544786110984193 -->

## Code blocks (from X article entities)

> 说明：X API 的 `plain_text` 不含代码块本体；以下从 `entities.code` 原样导出，保证不丢代码。

### Code block 1

```latex
explanation
  -> checklist
      -> template
          -> automated check
              -> enforced policy
```

### Code block 2

```latex
model decides intent
gateway validates action
tool changes environment
sensor observes result
```

### Code block 3

```latex
task
  -> project map
      -> relevant subsystem
          -> exact files
              -> local instructions
```

### Code block 4

```latex
OBJECTIVE
Fix duplicate coupon application during checkout.

CHANGED
- checkout validation logic
- focused regression test

VERIFIED
- lint passed
- unit tests passed
- checkout integration test passed

NOT VERIFIED
- production payment provider

DECISIONS
- preserved existing coupon priority order

RISKS
- legacy mobile client was not available locally

APPROVAL NEEDED
- deploy to staging
```

### Code block 5

```latex
LOW RISK
read files, search, inspect
-> automatic

REVERSIBLE CHANGE
edit workspace, run tests
-> automatic with trace

EXTERNAL EFFECT
send message, deploy, purchase
-> explicit approval

IRREVERSIBLE OR SENSITIVE
delete data, rotate credentials, publish globally
-> hard gate or prohibited
```

### Code block 6

```yaml
facts:
  - checkout validation lives in services/orders

decisions:
  - reuse the existing validation pipeline
  - reason: avoids a second source of truth

progress:
  completed:
    - added server-side rule
  remaining:
    - update integration test

lessons:
  - local test command requires TEST_DB_URL
```

### Code block 7

```latex
accepted outputs
------------------------------
human review minutes + run cost
```

### Code block 8

```latex
syntax
  -> types
      -> focused tests
          -> integration tests
              -> visual or semantic review
                  -> human approval
```

### Code block 9

```latex
old model limitation
  -> harness workaround
      -> model improves
          -> workaround remains
              -> system becomes slower or less capable
```

### Code block 10

```latex
observe
  -> decide
      -> act
          -> measure
              -> accept
              -> repair
              -> escalate
              -> stop
```

### Code block 11

```latex
Was the task contract ambiguous?
Was important context invisible?
Was the wrong tool exposed?
Was a precondition missing?
Was the result unverifiable?
Was policy left inside the prompt?
Was recovery too broad?
Was the trace insufficient?
```

### Code block 12

```latex
PROJECT MAP

product rules     -> docs/product/
architecture      -> docs/architecture.md
frontend          -> apps/web/
backend           -> services/api/
tests             -> tests/
commands          -> docs/commands.md
release rules     -> docs/release.md
```

### Code block 13

```latex
FACTS
stable information discovered about the environment

DECISIONS
choices made and the reason behind them

PROGRESS
completed, active, blocked, and remaining work

LESSONS
failures that should change future behavior
```

### Code block 14

```latex
tool timeout
-> retry with backoff

invalid arguments
-> repair the tool call

missing context
-> retrieve specific source

failed test
-> inspect failing behavior

permission denied
-> request approval or choose safe path

contradictory requirements
-> escalate to human

repeated unchanged failure
-> stop the loop
```

### Code block 15

```latex
"use the formatter"
-> run formatter automatically

"do not import across layers"
-> add architecture test

"include a migration rollback"
-> require rollback file in CI

"do not modify generated files"
-> block writes to generated paths

"cite every external claim"
-> validate citation coverage
```

### Code block 16

```latex
claim                         evidence
--------------------------------------------------
"the bug is fixed"            failing test now passes
"the page works"              browser flow completed
"the migration is safe"       dry run and rollback pass
"the report is correct"       values match source data
"the task is complete"        every acceptance check passes
```

### Code block 17

```latex
TOOL: edit_file

inputs:
  path
  patch

preconditions:
  path exists
  path is inside allowed workspace

success evidence:
  patch applied
  resulting diff returned

failure behavior:
  no partial overwrite
  structured error returned

risk class:
  reversible
```

### Code block 18

```latex
failure
  -> diagnosis
      -> new sensor, rule, map, test, or tool contract
          -> future runs improve automatically
```

### Code block 19

```latex
never publish without approval
never expose a secret
never write outside the workspace
never exceed the spend cap
never mark tests passed unless they ran
```

### Code block 20

```latex
AGENT HARNESS SPEC

1. CONTRACT
   objective:
   scope:
   constraints:
   acceptance evidence:

2. CONTEXT
   always-loaded map:
   retrieval sources:
   local instructions:
   freshness rules:

3. TOOLS
   allowed tools:
   preconditions:
   side effects:
   success evidence:
   timeout and retry policy:

4. STATE
   facts:
   decisions:
   progress:
   lessons:
   checkpoint format:

5. POLICY
   automatic actions:
   approval-required actions:
   prohibited actions:
   budget limits:

6. VERIFICATION
   deterministic checks:
   adversarial checks:
   acceptance rule:

7. RECOVERY
   failure classes:
   retry limits:
   escalation conditions:
   safe rollback:

8. OBSERVABILITY
   trace events:
   metrics:
   final change receipt:
```

### Code block 21

```latex
09:14  contract created
09:15  context source loaded: architecture.md
09:17  file edited: checkout.ts
09:18  focused test failed: duplicate coupon
09:21  implementation repaired
09:22  focused test passed
09:24  integration test passed
09:25  external deployment blocked: approval required
```

### Code block 22

```latex
BRAIN
plans, reasons, chooses

HANDS
execute tools inside a controlled environment

HISTORY
stores durable facts, decisions, and run state
```

### Code block 23

```yaml
objective: reduce onboarding drop-off

scope:
  - signup flow
  - onboarding analytics

constraints:
  - do not change authentication
  - preserve existing mobile behavior

acceptance:
  - tests pass
  - analytics event is emitted
  - screenshots cover desktop and mobile

approval_required:
  - production deployment
  - database migration
```

### Code block 24

```latex
worker
  -> produces candidate

verifier
  -> checks contract
  -> searches for missing cases
  -> tests unsupported claims
  -> attempts to break result

survives
  -> accept

fails
  -> return targeted evidence
```

### Code block 25

```latex
user request
     |
     v
+-----------------------------+
|           HARNESS           |
| contract | context | policy |
| tools    | state   | checks |
| traces   | recovery          |
+-----------------------------+
     |
     v
    model
     |
     v
real environment
```
