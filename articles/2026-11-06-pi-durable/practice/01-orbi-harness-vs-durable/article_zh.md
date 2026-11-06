---
title: "Orbi — Pi harness vs Pi Durable 与 kill -9（结构化笔记）"
author: "Larry (Orbi)"
source_url: "https://orbi.build/blog/pi-agent-harness/"
published_at: "2026-10-06"
archived_at: "2026-11-06"
category: "practice"
archive_note: "面向自研 coding-agent harness 工程师的结构化研究笔记。并非原文全文转载或逐句翻译。"
---

## 来源

Larry（Orbi），2026-10-06：对三种无人值守 Pi 部署做 kill 实测——`pi --print`、AI SDK `HarnessAgent`、Cloudflare `PiHarness`——同一「创建 step1–5 并 8 秒 sleep」任务，模型 DeepSeek flash。属 Orbi「如何基于 Pi 构建」系列。

## 这里的「Pi harness」指什么

自 2026-10-01 起，「Pi harness」可指**编码 harness**（Pi 1.0 CLI/coding agent：进程死了由人看了继续），或 **Pi Durable**（面向无人值守的带 checkpoint 库）。Orbi 自身主要跑 `pi --print` + 监督进程；本文测量各路径在 `SIGKILL` 后留下什么。

## 对比（结果）

| | `pi --print` | AI SDK | CF `PiHarness` |
|---|---|---|---|
| SIGKILL | `--continue` 完成 (2/2) | 会话丢失 (2/2) | 重启本地 runtime 后对象自行恢复 (3/3) |
| SIGTERM | — | 配合 `session.stop()` 可恢复 (7/7) | — |
| 谁重启 | 你的监督（+ 收尸） | 你的代码 + 已存状态 | 心跳 alarm ~30s（文档） |

并非完全对等基准：Cloudflare 用自定义工具（Worker 无 shell）。AI SDK 中断的 sleep 被记成 exit 0。PiHarness 默认 unsafe 的 sleep 写入 interrupted；模型常**再次调用 sleep**——故 unsafe ≠ 无应用守卫就不会跑两次。

## CLI 运维教训

必须关闭 stdin，否则 print 模式可能在开着的管道上永远等待。`SIGKILL` 留下 detached bash 进程组——监督必须收尸。孤儿 tool call 在组下一请求时被合成 “No result provided”。

## 选型建议

有 git checkout + 真 shell + 机器 → `pi --print` + 监督。Node 产品内嵌 + 每用户沙箱 → AI SDK（接受崩溃缺口；接 SIGTERM）。无自备监督的崩溃续跑 + API 形工具 → Cloudflare 上 Pi Durable。三者都把调度、互斥领取、评审、合并留给外层（Orbi 的工作）。

## 对自研 harness 的启示

选型前自跑 kill 矩阵。分清「会话文件还在」与「工具副作用诚实」。为 CLI 拉起的 bash 规划孤儿清理。把 interrupted 后模型再调用当作对幂等设计的预期对抗行为。

## 摘录

> “I compared one thing: what is left after a hard kill, and who has to act.”
> — （同上）
