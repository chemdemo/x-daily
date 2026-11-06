---
title: "Agent-Engineering — 第一次 Pi Durable 会话（结构化笔记）"
author: "Nelson Lin"
source_url: "https://www.agent-engineering.dev/series/comprehensive-pi-durable-agent-from-zero-to-hero/article/setting-up-a-project-and-running-your-first-pi-durable-conversation"
published_at: "2026-11-03"
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

系列第二篇（Nelson Lin，Agent-Engineering.dev，2026-11-03）：搭项目并用内存存储 + 第三方 OpenAI 兼容 API 跑第一次 Pi Durable 会话。持久化仅作预告。

## 先决条件与项目搭建

- Node.js **≥22.19.0**（包 `engines` 下限）
- npm/pnpm/yarn/bun；必须 ESM（`"type": "module"`）
- 任意 OpenAI 兼容端点的 API key

脚手架：`mkdir` → `npm init -y` → 设 type module → 安装 `pi-durable`、`pi-ai`、`chord`。可选 TypeScript（`tsx`，ES2022 / NodeNext）。

## 方法：第一次会话逐步说明

1. **环境变量**：在 `src/hello.ts` 顶部 `loadEnvFile()`（或 CLI `--env-file`），确保 provider 构造前密钥已在。
2. **Model 对象**：构造 `Model<"openai-completions">`，含精确远程 `id`、匹配的 `provider` 字符串、`baseUrl`、reasoning/input、成本元数据、`contextWindow`、`maxTokens`。元数据不改线上协议，但影响预算/UI。
3. **Provider**：`createProvider` + `envApiKeyAuth` + `openAICompletionsApi()`。
4. **注册**：`createModels().setProvider(...)`——必须是传入 `Harness.open` 的同一实例。漏注册 → “Provider is not configured.”
5. **打开 harness**：`Harness.open(new MemoryStorage(), { models, registry: createRegistry() }, BACKGROUND_CONTEXT)`。
6. **Root 会话**：`harness.root(..., { agent: { model: { provider, modelId } } })`——只存名称，请求时解析。
7. **提交 / 等待 / 读取**：`submit` → `wait` → `done` 时 `commit` 读 `AssistantEntry` 并打印。`harness.close` 收尾。

用 `npx tsx` 或 `node --experimental-strip-types` 运行。内存存储在进程退出后消失——符合预期。

## 持久化预告

换成 `openNodeSqliteStorage("./session.sqlite")` 并 `harness.resume()`，同一 root 可跨重启。第三篇深入 kill -9 恢复。

## 常见坑清单

Node 过旧；非 ESM；环境未加载；model/provider/root 的 provider id 不一致；远程 model id 写错（含 `/` 段）；实验版本未钉；日后两进程打开同一 SQLite。

## 对自研 harness 的启示

用 MemoryStorage 作教学后端，让工程师先学会 `Context` / Harness / Conversation / Submission，不被 WAL 文件干扰。在 DI 文档写明「每个 harness open 一个 models 注册表」。把 OpenAI 兼容网关当一等公民——很多生产栈不是纯 OpenAI。

建议布局：`package.json`、`tsconfig.json`、gitignore 的 `.env`、`src/hello.ts`，日后出现 `session.sqlite`。

## 摘录

> “Even with MemoryStorage, the commit protocol is the same one that will later survive crashes when you switch to SQLite or JSONL.”
> — （见 front matter URL）
