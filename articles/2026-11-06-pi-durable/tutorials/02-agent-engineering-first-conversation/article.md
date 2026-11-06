---
title: "Agent-Engineering — First Pi Durable conversation (structured notes)"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/setting-up-a-project-and-running-your-first-pi-durable-conversation"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Series article 2 (Nelson Lin, Agent-Engineering.dev, 2026-11-03): create a project and run the first Pi Durable conversation using in-memory storage and a third-party OpenAI-compatible API. Persistence is previewed only.

## Prerequisites and project setup

- Node.js **≥22.19.0** (package `engines` floor); verify with `node --version`
- npm/pnpm/yarn/bun; ESM required (`"type": "module"`)
- An API key for any OpenAI-compatible endpoint (official OpenAI, gateway, etc.)

Scaffold: `mkdir` → `npm init -y` → set type module → install `@earendil-works/pi-durable`, `pi-ai`, `chord`. Optional TypeScript with `tsx` and `tsconfig` targeting ES2022 / NodeNext.

## Method: first conversation walkthrough

1. **Env**: `loadEnvFile()` at top of `src/hello.ts` (or CLI `--env-file`) so keys exist before providers construct.
2. **Model object**: build a `Model<"openai-completions">` with exact remote `id`, matching `provider` string, `baseUrl`, `reasoning`/`input` flags, cost metadata, `contextWindow`, `maxTokens`. Metadata does not change the wire protocol but drives budgeting/UI.
3. **Provider**: `createProvider({ id, name, baseUrl, auth: { apiKey: envApiKeyAuth(...) }, models: [...], api: openAICompletionsApi() })`.
4. **Register**: `const models = createModels(); models.setProvider(LLMProvider);` — must be the instance passed into `Harness.open`. Missing this → “Provider is not configured.”
5. **Open harness**: `Harness.open(new MemoryStorage(), { models, registry: createRegistry() }, BACKGROUND_CONTEXT)`.
6. **Root conversation**: `harness.root(context, { agent: { model: { provider, modelId } } })` — stores names only; resolves at request time.
7. **Submit / wait / read**: `submit({ type: "input", content })` → `submission.wait` → on `done`, `commit(tx => tx.entry(AssistantEntry, settled.answer))` and print. Close with `harness.close`.

Run via `npx tsx src/hello.ts` or `node --experimental-strip-types`. Memory storage vanishes when the process exits—that is expected.

## Persistence preview

Swap storage for `openNodeSqliteStorage("./session.sqlite")`, call `harness.resume()`, and the same root returns across restarts. Article 3 deep-dives kill -9 recovery.

## Gotchas checklist

Node too old; not ESM; env not loaded; provider id mismatch between model/provider/root; wrong remote model id (including `/` segments); unpinned experimental versions; later: two processes opening one SQLite file.

## Takeaways for an in-house harness

Use MemoryStorage as the teaching backend so engineers learn `Context` / Harness / Conversation / Submission without WAL files. Codify “one models registry per harness open” in your DI docs. Document OpenAI-compatible gateways as first-class—many prod stacks are not pure OpenAI.

Suggested layout: `package.json`, `tsconfig.json`, gitignored `.env`, `src/hello.ts`, later `session.sqlite`.

## Quote

> “Even with MemoryStorage, the commit protocol is the same one that will later survive crashes when you switch to SQLite or JSONL.”
> — https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/setting-up-a-project-and-running-your-first-pi-durable-conversation
