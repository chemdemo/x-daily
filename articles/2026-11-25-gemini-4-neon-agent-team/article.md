---
title: "Gemini 4 Neon: Build an AI Agent team with the best Google model in 8 Steps (structured notes)"
author: "Movez (@0xMovez)"
source_url: "https://x.com/0xMovez/status/2125577359269019791"
article_url: "https://x.com/i/article/2125560293870669824"
substack_url: "https://substack.com/@0xmovez"
published_at: "2026-11-25"
published_at_utc: "2026-11-25T13:08:28.000Z"
archived_at: "2026-11-27"
kind: "structured-notes"
license_allows_full_reprint_and_translation: false
archive_note: "Structured research notes written from the supplied article text. This is not a reprint, not a close paraphrase of the full text, and not a full translation. The X Article is copyrighted and states no license that allows redistribution or translation. Long prompts, /swarm briefs, and screenshots stay in the original."
---

## Source

Movez (@0xMovez). *Gemini 4 Neon: Build an AI Agent team with the best Google model in 8 Steps (Full-course).* X Article: https://x.com/i/article/2125560293870669824. Post: https://x.com/0xMovez/status/2125577359269019791. Published 2026-11-25 13:08:28 UTC (2026-11-25 21:08 Asia/Shanghai). The article says Google shipped Neon on November 18, 2026. Substack: https://substack.com/@0xmovez. Post metadata: content creator, AI researcher and agentic builder, CEO @beyond_xai.

These notes were written from the article text, post metadata, and code-block extracts supplied with this archive on 2026-11-27. The live page was not refetched. The original has 18 images; none are stored here. Open the article for screenshots and for every full prompt.

This is a copyrighted X Article with no license to redistribute or translate. The notes record claims, the eight steps, roles, prices, and reported results. They omit the nine long copy-paste prompts, the long `/swarm` briefs, and the figures. Short quotations are attributed. One-line install commands are kept as published install facts. Prices and scores are copied as stated and were not checked against Google's docs. Step numbers follow the article so a reader can jump back.

## The claim

Movez's premise: people run Neon as a chatbot or as a solo model, while Google built it to boss a team — plan the work, hand it out, check it, and sign off. He says he first read it as Google's cheaper answer to Opus. The article text says 99% of people run it at a third of its power: half got a decent chat answer and went back to Claude; the other half run it solo on xhigh and burn the quota by lunch. The post text puts that same 99% at 20% of its power. Both figures are his. These notes do not choose between them.

He quotes Google's launch line for the intended setup: a fast, high-judgment Orchestrator directing parallel Hydrogen subagents.

## Seats

**Gemini 4 Neon (High)** reads the repo, writes briefs, repairs the plan when a worker is stuck, and signs off. **Gemini 4 Hydrogen** does one scoped job on its own working copy and costs nothing while it waits to be mentioned. **A verifier on a different model, with fresh context,** checks paths, line numbers, and tests, so it does not share the workers' blind spots. **The `gemini-api` CLI** covers deep research, images, video, voice, and music. Any agent with a shell can call it.

## Results he cites

Google's numbers, as he states them: the same model scores 12.8% on SWE-sweep alone and 39.0% inside its team harness. Same weights, about 3x.

Theo, as he tells it: in under 10 hours a Neon swarm beat a result another model had chased for 7+ days — a 5.95x smaller binary and a 48.6% faster test suite. The other model is not named.

A separate Theo anecdote, tied to a single-bug brief rather than a swarm: Neon traced 3.8s of a 6.3s worktree delay to a disk formatted for spinning drives, missed for three weeks.

## 1. Four ways in

The door decides the bill. Every door reads the same `AGENTS.md` and `.agents/` folder, so a later door does not mean rewriting the team. Google AI Pro is $19.99/month (the intro's "$19.99 plan").

**Antigravity app or `agy`.** Billed to an AI Pro or Ultra quota. The app is for watching a swarm; the CLI is for a server or SSH. Neon is described as rolling out to paid Google AI Pro and Ultra subscribers, and to enterprise through Gemini for business. `/swarm` works on all plans, so a free plan can practice on the models it already sees and point the Orchestrator at Neon after an upgrade. In the app: New Project, Add Folder, Create, then Local or a new worktree. First launch can import MCP servers, skills, or rules from Claude Code, Codex, Cursor, or Gemini CLI.

**T3 Code via ACP.** Same Antigravity runtime, same subscription. Settings → Providers → Antigravity, install the runtime, sign in. Neon then sits in the same model list as the Claude and GPT seats. Detail is in step 5.

**Gemini API and ADK.** Per token.

**Paid API key** for a Workspace account that cannot buy AI Pro. That is the whole Richie anecdote as written: he hit that wall. Set `"modelProvider": "gemini"` in `~/.gemini/antigravity-cli/settings.json`, and export `GEMINI_API_KEY` so the `agy` header reads "Gemini API key". In T3 Code those accounts use the API key or Enterprise sign-in.

`agy` install, as given:

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

Installs to `~/.local/bin/agy`. Windows: `irm https://antigravity.google/cli/install.ps1 | iex`. Then `agy --version` and `agy`.

## 2. Orchestrator and thinking level

The setting he ranks above any prompt is **Orchestrator** in the model picker. One session, two models: Neon plans, Hydrogen types. On friction, control returns to Neon to inspect state, repair the plan, and resume. Judgment is billed at Neon's rate, keystrokes at Hydrogen's. The intro calls the workers "at half its price," which matches the API list (Hydrogen input and output are half of Neon's).

Open **Orchestrator · Gemini 4 Neon High**. Change the pair under Settings → Models → Orchestrator Settings. Planners: Gemini 4 Neon (default), Carbon, Argon, Claude Opus 5.5. Executors: Gemini 4 Hydrogen (default), Claude Sonnet 5.5. Keep the defaults for the first week. Move the planner to Carbon only for multi-day, long-horizon work.

Thinking levels: low, medium, high (default), xhigh. `thinking_budget` or `MINIMAL` on the API returns HTTP 400; delete them when leaving Gemini 3.x.

Adam Holter's curve, as reported: cost per task relative to low is 1x, 2.5x, 5x, 8.6x. High to xhigh is about 72% more per task, for +4.3 on Terminal-Bench Science, +1.1 on FrontierCode, +0.1 on AutomationBench, and −5.4 on GDP.pdf. A thinking-level rule from Philipp Schmid (Google DeepMind) is cited and appears only as a screenshot, so it is not copied here.

## 3. The shared folder

Antigravity loads `AGENTS.md` and optional `GEMINI.md` from the repo root and parent folders, plus `.agents/`, before any agent starts. Personal defaults live in `~/.gemini/config/`. The scan is one level deep. A subagent with helper files can live at `.agents/agents/<name>/agent.md`.

Layout he specifies: `AGENTS.md` (shared rules), optional `GEMINI.md`, and `.agents/` with `agents/` (one markdown file each; the example names `planner`, `implementer`, `verifier`, `motion-designer`), `rules/`, `skills/<name>/SKILL.md`, `hooks.json`, and `mcp_config.json`. All four doors share this folder.

A brief has three parts: what the worker owns, what it must not touch, and when it is done. Write `AGENTS.md` in that shape and every brief inherits it. A subagent file is YAML frontmatter plus a system prompt. Only `name` and `description` are required. Other fields he flags: `mainAgent` and `subagent` (where it may run), `mcpServers`, and `commandExecutionPolicy` (`off`, `auto`, `eager`, or `sandbox`, the default). Keep workers on sandbox until they are trusted.

The verifier is the file he says pays for itself: read-only; open every cited path; re-run commands the worker claims passed; reject numbers with no command or file behind them. The system prompt stays in the original. In the app, Customizations lists the agents, and `@verifier` answers in its own context.

## 4. First `/swarm`, and two wrong findings

`/swarm`, then Tab, then the work as separate tracks plus a verifier. The conversation becomes `@coordinator`. It inspects the repo for about a minute, spawns named workers, and posts a public brief to each on a shared board. Idle workers burn no tokens.

The first swarm should not edit. The shape: numbered tracks, read-only, a verifier on paths and lines, one ranked summary. The concrete case is an audit Flavio Copes ran on his own site after Kevin Hou recommended it (links, images, build). That prompt stays in the original.

The intro calls this the swarm whose verifier caught two wrong findings before they shipped. The run: `@links-auditor`, `@images-auditor`, `@build-auditor`, `@verifier`, and `@monitor`, all on `gemini-4-hydrogen`. The verifier flagged two wrong claims in report #7. The links auditor posted a correction 9 minutes 42 seconds later, the verifier signed off, and the coordinator posted a ranked 10-item summary as post #14. Findings he calls real: 233 MiB of unused or shadowed assets; an image-audit script that missed 3,021 of 3,646 images; a link checker blind to 79 paid apps; a build at concurrency 2 across roughly 8,721 artifacts. A single chat, he writes, would have reported the first two with confidence and never checked them.

After a clean read-only run, he shows two writing swarms adapted from the Antigravity docs. Workers default to the conversation's model, so a verifier on a different model has to be requested. Those prompts stay in the original.

## 5. T3 Code, and seats that are not Neon's

The setups he prefers do not give Neon every seat. T3 Code runs each lab's own harness side by side: Claude Code, Codex, and Antigravity through the official ACP agent (t3dotgg's PR #9348).

Theo's casting, as reported. Neon writes well and follows negative instructions: told to ignore a skill's writing style, it did. Code review: 21 valid findings, 5 minor, 0 wrong. Architecture questions: 10 of 72 claims wrong. So Neon writes and reviews, another model leads implementation, and a third plays the user.

Connect: Settings → Providers, enable Antigravity, install the runtime. Antigravity 1.5+ required, 1.5.12 recommended, about 1.25 GiB. Sign in with a Google account, Gemini Enterprise, a Gemini API key, or Agent Platform (Vertex). Workspace accounts that cannot buy AI Pro use the API key or Enterprise. Pick Gemini 4 Neon; the thinking-level control sits beside it. `/plan`, `/compact`, and `/goal` pass through. Skills load from `.agents/skills`.

A casting he says works: Neon on High as planner and reviewer; Hydrogen via the Orchestrator, or a Claude seat already paid for, on bulk implementation; a playtester from another lab that has not read the plan; and not Neon alone on 3D or shader-heavy work. Theo called its 3D output weak. Demos that look good had a human-specified look.

Theo's setup story: Neon installed Antigravity with auth on his Windows machine, verified it inside T3 Code, and notified another thread, in 8 minutes. It also found T3 Code's Antigravity adapter dropping `html_render` frames, worked around that, and opened a pull request. He says people are leaving other editors for Neon; the examples are in a screenshot and are not in the text.

## 6. New Gemini tools, from Claude Code or Codex

Two releases in the 72 hours before the post (2026-11-25). On November 23 the Gemini API began accepting video, audio, and documents as function results. On November 24 the open-source `gemini-api` CLI spread far enough that Claude Code or Codex can call Gemini's media stack from a shell. His reading: this does not retire Antigravity; the Gemini half of the team no longer has to live inside it.

`gemini-api` v0.5.0 is dated November 20. Install:

```bash
curl -fsSL https://raw.githubusercontent.com/google-gemini/gemini-api-cli/main/scripts/install.sh | bash
```

Windows: `iwr -useb https://raw.githubusercontent.com/google-gemini/gemini-api-cli/main/scripts/install.ps1 | iex`. Or `go install github.com/google-gemini/gemini-api-cli/cmd/gemini-api@latest`. He then shows `gemini-api configure` and `gemini-api whoami`.

The CLI is stateless on purpose. The agent keeps the plan and the context; the CLI returns a file path or an interaction id. Commands he says the team uses most, by name only: background Deep Research (sample agent id `deep-research-preview-04-2026`) and a status check; `analyze` on a PDF or screenshots; `image`, `video`, `tts`, and `music`, labeled Nano Banana 2.1, Omni 1.2 Flash, 3.8 Flash TTS, and Lyria. The block he says to paste into `AGENTS.md`, `CLAUDE.md`, or `.cursorrules` is a README excerpt and is not copied here.

After November 23 a UI subagent can return an mp4 of a bug as the function result, and Neon watches the clip. The same path is described for a call recording, a `.docx`, or a spreadsheet tab. Keep the media inside the tool result. Pasting it back as a new user turn breaks the function-call chain.

## 7. The same team in ADK

Antigravity is for running the team yourself. For other people — a Slack bot, a nightly job, an API — rebuild the same graph in ADK 2.0 and deploy it to Cloud Run. Neon plans and verifies, Hydrogen workers fan out, and a conditional edge sends rejected work back.

Smallest handoff, from Google's Neon developer guide as he presents it: Neon stores a plan on an interaction; Hydrogen continues that thread with `previous_interaction_id`, so the plan is not resent. The sample client is an enterprise client. The sample task text is not copied.

ADK 2.0 workflows are edges. A tuple fans out. A `JoinNode` waits for every branch and passes on a dict keyed by node name. A dict of routes makes an edge conditional. A loop is allowed when the edge back is conditional. An Agent node defaults to `mode="single_turn"`, which is what he wants for a worker: a clean brief, no leftover context. `AntigravityAgent` in `google.adk.labs.antigravity` can drive a real Antigravity agent from ADK. Check names against the ADK docs for your version. The Python sketch stays in the original.

## 8. Motion design, high level

The most shared Neon demos that week were videos, and the good ones were code: one HTML file with Canvas, Three.js, or WebGL, rendered to frames. A film in code can be checked like code: duration, cuts on beats, no clipped audio.

Four public asks, constraints only:

- @IshanJX (Extra Effort): a 15-second, 60fps showreel in one code file, no external libraries or assets.
- @atomtanstudio: a code-only introduction of Gemini 4 Neon, no external assets.
- @cfenghub: Neon wrote its own prompt, then a 15-second single-page Three.js and GLSL ad, bright high-key launch style, Gemini gradients, zero human edits.
- @Jonathascosta89, on xhigh: a project folder and the line "make a presentation video of my project".

The rain scene: nobody asked for a depth model. Neon decided the image needed depth, fetched Depth Anything V2 Large, ran it locally, and built the layers. His moral: a goal plus a verifier, and the team builds its own tools. Noel's complaint, which he calls fair, is a brief problem. With no brand colors, Neon falls back to its own gradients. Put brand tokens and beat timings in the brief.

As a swarm: save `film/brief.md` first, then split picture, copy, sound (via `gemini-api`), and render, verifier on a different model. The prompt's exact timings and audio ceiling stay in the original.

## Prices, as stated

**Subscription** (app, `agy`, T3 Code): Google AI Pro is $19.99/month. The flat plan absorbs thinking tokens, which is why xhigh feels free. Quota is not free: a swarm burns 4x to 5x one agent, and can run out mid-run.

**API, per 1M tokens:**

- Gemini 4 Neon: $3.00 in, $15.00 out, cached input $0.15 (he calls this a 95% discount). Implicit caching starts at 4,096+ tokens, so keep `AGENTS.md` and briefs at the top and stable.
- Gemini 4 Hydrogen: $1.50 in, $7.50 out, cached $0.075. The workers.
- Gemini 4 Carbon: intro $2 / $10 through Nov 30, then $4 / $20 from Dec 1, in the same input-then-output order. Re-check a Carbon prototype from that month the following week. No cached price for Carbon is stated.

Per-token price, he says, misleads, because Neon thinks a lot. On SimpleBench it averaged 2,385 output tokens per question against Opus 5.5's 575, so a full run cost $7.99 on Neon versus $2.74 on Opus 5.5. His split: Neon on High for planning and review, volume on Hydrogen, xhigh only on the evening you are stuck. On High, Neon costs 32% less per task than Carbon.

## First evening

His time boxes, as a checklist. The prompts they point at are in the original.

- 15 min. Install `agy` or the app, sign in, select Orchestrator · Neon High.
- 10 min. On the main repo, get a map and a proposed split into tracks before any edit, and read it.
- 25 min. Write `AGENTS.md` (commands, ownership, what "done" means) and `.agents/agents/verifier.md`.
- 30 min. Read-only `/swarm` with three tracks. Read only the final summary, then spot-check two findings by hand.
- 15 min. Install `gemini-api`, point `AGENTS.md` at it, and ask your usual agent for a Deep Research report on a competitor.
- 10 min. Take one audit finding into a first writing swarm, verifier on a different model.

## The nine prompts

Nine copy-paste prompts close the article (angle-bracket placeholders). Not reproduced here. By his labels: an Orchestrator kickoff that maps the repo before edits; a read-only audit swarm; a feature swarm with ownership; a short verifier system prompt; a worker brief; a single-bug debugging brief (the shape he ties to Theo's worktree hunt); a landing page that calls Gemini for an image, a voiceover, and competitor research; a motion film from a saved brief; and a short recap of changes, checks, rejections, and the one thing to verify by hand.

## Closing thesis

Prompting, he concludes, is now staffing. The gap between 12.8% and 39.0% on the same model was the harness. The gap between Theo's 7+ days and his run under 10 hours was a team with a target. The gap between Flavio's ranked summary and a confident wrong one was a verifier that checked line numbers. The org chart to write down: who plans, who types, who checks, and what "done" means.

## Quotes

> "But Neon wasn't built to be a chatbot. Google built it to be the boss of a team: plan the work, hand it out, check it, sign off"
> — Movez, https://x.com/i/article/2125560293870669824

> "a fast, high-judgment Orchestrator directing parallel Hydrogen subagents."
> — Google's launch post, as quoted in the article

> "Let Neon decide. Let Hydrogen type. Let a stranger check."
> — Movez, same article
