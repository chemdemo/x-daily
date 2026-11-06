---
title: "Pi Durable 速查表 — luongnv（结构化笔记）"
author: "luongnv.com"
source_url: "https://luongnv.com/awesome-cheatsheets/cheatsheets/pi-durable/"
published_at: null
archived_at: "2026-11-06"
category: "tutorials"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

luongnv.com 社区速查，汇总 Pi Durable 快速开始、API 表、Cloudflare `PiHarness` 说明、主动 agent 模式，并指向 `disler/durable-pi` 演示网络。当作**地图与清单**，不能替代官方 README——API 会漂；部分片段仍用旧字段名（如工具结果 `content` vs 新的 `output`）。

## 先决条件回顾

Node ≥22.19（或带适配器的 Bun/DO）；ESM TypeScript；供应商密钥；可选 Pi 仓 30+ 示例；可选 Cloudflare SQLite DO；可选社区 demo（Node 24、`just`、OpenRouter）。Durable 不替代短时有人监督终端会话的 `pi-dev`。包是实验性的——钉精确版本，并按已装版本重读 README。

## 九级阶梯

1. MemoryStorage 快速问答  
2. SQLite + `resume` + `requestId` + 失败时重开  
3. CodingTools + 执行环境；带 replay 的 `defineTool`；按名 wrap/覆盖  
4. 每会话 `configure` / fork / `whenBusy`；harness `settings`  
5. Hooks + `api.memo` 审批；子 agent 所有权；`defineTask` 等待/`failFast`；后台任务  
6. 嵌套 `executeTool`、callers/modelTools、codemode、deferred tool_search  
7. `defineDoc`、compaction/reset/handoff、同名热安装  
8. 多人 watch；Cloudflare PiHarness + operationId  
9. 主动：事件指纹 → commit 事故+会话 → 睡眠/轮询决策文档的审批子任务  

## 最佳实践 vs 不要做（精炼）

要做：把 `replay: "safe"` 当契约；从稳定 id 派生 requestId/operationId；memo 钩子决策；section 稳定以保缓存；documents 优于旁路表；显式选 fork 策略；子 agent 用更便宜模型并去掉递归扩展；远程存储提高 progress 间隔；查 usage；限制递归 callers；主动 agent 前加廉价分类器；为守卫钩子决定 fail-open/closed。

不要：双开同一存储；把挂钟塞进 section；假定 `abort()` 立即返回；混淆 PiHarness 与核心 API；测试外依赖 MemoryStorage；浮动版本；把分类器守卫当沙箱，或把设备 token 看得低于 shell。

## 对自研 harness 的启示

把此页放在官方 README 旁作运维清单。每段复制的 snippet 合入生产模板前必须与已装包版本 diff。「审批作为耐久子任务」模式对会部署的 coding agent 尤其相关。

## 关于社区 demo 的警告

速查表提到 `disler/durable-pi` 与主动 `just` 配方。那些是**社区** demo，自有钉版本（常更旧）、未沙箱的 bash，以及仅 localhost 的假设。可偷模式（事件指纹 → 一会话；审批作子任务），但要在你钉住的官方包版本与安全边界上重实现——不要把 demo token 或 fail-open bash 门闸粘进生产。

## 摘录

> “The package is experimental: ‘The API changes without notice between releases.’ Pin exact versions…”
> — （同上）
