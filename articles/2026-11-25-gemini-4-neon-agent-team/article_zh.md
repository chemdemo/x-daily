---
title: "Gemini 4 Neon：八步搭起一支 AI agent 团队（结构化笔记）"
author: "Movez (@0xMovez)"
source_url: "https://x.com/0xMovez/status/2125577359269019791"
article_url: "https://x.com/i/article/2125560293870669824"
substack_url: "https://substack.com/@0xmovez"
published_at: "2026-11-25"
published_at_utc: "2026-11-25T13:08:28.000Z"
archived_at: "2026-11-27"
kind: "structured-notes"
license_allows_full_reprint_and_translation: false
archive_note: "根据随任务提供的原文写成的结构化研究笔记的中文稿。不是原文转载，不是逐段近译，也不是全文翻译。该 X Article 受版权保护，帖子与文章均未给出允许再分发或翻译的许可。长提示词、长 /swarm 简报和截图只在原文。"
---

## 来源

Movez（@0xMovez）。《Gemini 4 Neon: Build an AI Agent team with the best Google model in 8 Steps (Full-course)》。X Article：https://x.com/i/article/2125560293870669824。帖子：https://x.com/0xMovez/status/2125577359269019791。发布于 2026-11-25 13:08:28 UTC（2026-11-25 21:08，Asia/Shanghai）。文中称 Google 于 2026 年 11 月 18 日发布 Neon。Substack：https://substack.com/@0xmovez。帖子元数据里的身份是内容创作者、AI 研究者与 agent 构建者，以及 @beyond_xai 的 CEO。

笔记根据 2026-11-27 随本任务提供的正文、帖子元数据和代码块摘录写成，没有重新抓取线上页面。原文有 18 张图，这里一张都不存。截图和完整提示词请回原文。

这是受版权保护的 X Article，没有再分发或翻译的许可。笔记只记主张、八个步骤、角色、价格和文中报告的结果，不收录九段长提示、长篇 `/swarm` 简报和插图。短引标明出处。只保留一行官方安装命令。价格和分数按原文抄录，没有对照 Google 文档改过。步骤编号跟着原文，方便跳回。

## 主张

Movez 的前提是：人们把 Neon 当成聊天机器人，或当成单独一个模型在用；Google 造它，是让它带领一支团队，负责规划、分派、检查和签收。他说自己一开始也把它看成 Google 对标 Opus 的更便宜答案。正文写，99% 的人只用到三分之一的能力：一半人在聊天里得到一个还行的回答就回到 Claude，另一半把它单独开在 xhigh，额度到午饭就没了。帖子正文把同一个 99% 写成只用到 20%。两个数字都是他的说法，这里不取舍。

他用一行引述 Google 发布帖里的目标形态：一个又快、判断力又高的 Orchestrator，指挥并行的 Hydrogen 子代理。

## 席位

**Gemini 4 Neon（High）** 读仓库、写简报、在工人卡住时修好计划，并签收。**Gemini 4 Hydrogen** 在自己的工作副本上做一件划定范围的活，待命时不耗额度。**另一个模型、一份新上下文上的核验者** 核对路径、行号和测试，因此不会共享工人的盲区。**`gemini-api` CLI** 做深度研究、图像、视频、语音和音乐。任何有 shell 的 agent 都能调用。

## 他引用的结果

他转述的 Google 数字：同一模型单独做 SWE-sweep 是 12.8%，放进团队 harness 是 39.0%。权重不变，大约三倍。

Theo，按他的叙述：不到 10 小时，一支 Neon swarm 超过了另一个模型追了 7 天以上的结果，二进制小了 5.95 倍，测试套件快了 48.6%。那个模型没有点名。

另一则 Theo 的事对应单点排错，不是 swarm：Neon 把 6.3 秒 worktree 延迟里的 3.8 秒，追到一块按机械硬盘格式化、并且被漏看了三周的磁盘上。

## 1. 四条入口

选哪扇门，决定账单怎么算。每扇门都读同一份 `AGENTS.md` 和同一个 `.agents/`，以后再加一条入口，不用重写团队。Google AI Pro 每月 19.99 美元，也就是开头说的「19.99 美元方案」。

**Antigravity 应用或 `agy`。** 记在 AI Pro 或 Ultra 额度上。应用适合看 swarm，CLI 适合服务器或 SSH。文中说 Neon 正在向已付费的 Google AI Pro 与 Ultra 用户铺开，企业侧走 Gemini for business。`/swarm` 所有方案都能用，免费方案可以先在已有模型上练流程，升级后再把 Orchestrator 指到 Neon。应用里是 New Project、Add Folder、Create，然后选 Local 或新建 worktree。首次启动可以从 Claude Code、Codex、Cursor 或 Gemini CLI 导入 MCP 服务器、skills 或 rules。

**T3 Code 经 ACP。** 同一套 Antigravity 运行时，同一份订阅。路径是 Settings → Providers → Antigravity，安装运行时并登录。Neon 随后出现在和 Claude、GPT 同一张模型列表里。细节在第 5 步。

**Gemini API 和 ADK。** 按 token 计费。

**付费 API key** 给买不了 AI Pro 的 Workspace 账号。文中的 Richie 只到这一句：他撞上了这堵墙。在 `~/.gemini/antigravity-cli/settings.json` 写入 `"modelProvider": "gemini"`，并导出 `GEMINI_API_KEY`，让 `agy` 页眉显示 “Gemini API key”。在 T3 Code 里，这类账号走 API key 或 Enterprise 登录。

`agy` 的安装命令，按原文：

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

装到 `~/.local/bin/agy`。Windows：`irm https://antigravity.google/cli/install.ps1 | iex`。然后 `agy --version`，再 `agy`。

## 2. Orchestrator 与思考档位

他认为高于任何提示词的设置，是在模型选择器里选 **Orchestrator**。一个会话、两个模型：Neon 做计划，Hydrogen 动手。遇到阻力，控制权回到 Neon，由它查看现状、修好计划、再继续。判断按 Neon 的价格计，击键按 Hydrogen 的价格计。开头说工人是「它一半的价格」，和 API 标价一致：Hydrogen 的输入和输出都是 Neon 的一半。

先打开 **Orchestrator · Gemini 4 Neon High**。换配对的位置是 Settings → Models → Orchestrator Settings。规划者：Gemini 4 Neon（默认）、Carbon、Argon、Claude Opus 5.5。执行者：Gemini 4 Hydrogen（默认）、Claude Sonnet 5.5。第一周保持默认。只有多日、长程的工作才把规划者换成 Carbon。

思考档位是 low、medium、high（默认）和 xhigh。向 API 发送 `thinking_budget` 或 `MINIMAL` 会返回 HTTP 400，从 Gemini 3.x 迁出时要删掉这些字段。

他转述的 Adam Holter 曲线：相对 low，每项任务成本是 1 倍、2.5 倍、5 倍、8.6 倍。从 high 到 xhigh，每项任务大约贵 72%，换来 Terminal-Bench Science +4.3、FrontierCode +1.1、AutomationBench +0.1，以及 GDP.pdf 上的 −5.4。他引用了 Philipp Schmid（Google DeepMind）的一条思考档位规则，那条规则只在截图里，这里不抄。

## 3. 共用目录

任何 agent 启动前，Antigravity 会从仓库根目录和上级目录读取 `AGENTS.md` 与可选的 `GEMINI.md`，并读入 `.agents/`。个人默认在 `~/.gemini/config/`。扫描只到一层。需要旁边放辅助文件的子代理，可以放在 `.agents/agents/<name>/agent.md`。

他指定的布局：`AGENTS.md`（共用规则）、可选的 `GEMINI.md`，以及 `.agents/` 下的 `agents/`（每个一份 markdown；例子是 `planner`、`implementer`、`verifier`、`motion-designer`）、`rules/`、`skills/<name>/SKILL.md`、`hooks.json` 和 `mcp_config.json`。四条入口共用这个目录。

一份简报有三块：工人拥有什么、不许碰什么、怎样算做完。`AGENTS.md` 按这个形状写，每份简报都会继承。子代理文件是 YAML frontmatter 加系统提示，必填只有 `name` 和 `description`。他还点出 `mainAgent` 与 `subagent`（可以用在哪里）、`mcpServers`，以及 `commandExecutionPolicy`（`off`、`auto`、`eager` 或默认的 `sandbox`）。在信任之前，工人留在 sandbox。

核验者是他说能赚回成本的那个文件：只读；打开每一条被引用的路径；重跑工人声称已通过的命令；没有命令或文件支撑的数字一律拒绝。系统提示留在原文。应用里，Customizations 列出这些 agent，`@verifier` 在自己的上下文中回答。

## 4. 第一次 `/swarm`，以及两处错误发现

输入 `/swarm`，按 Tab，把工作写成几条轨道加上一个核验者。对话变成 `@coordinator`。它用大约一分钟查看仓库，拉起有名字的工人，并在共享看板上给每人一份公开简报。待命的工人不烧 token。

第一次 swarm 不该改文件。形状是：编号轨道、只读、核验者核对路径和行号、最后一份排好序的摘要。具体例子是 Flavio Copes 在 Kevin Hou 推荐之后，对自己的网站做的审计（链接、图片、构建）。那段提示留在原文。

开头把这次运行概括为：核验者在结果发出之前抓住了两处错误发现。成员是 `@links-auditor`、`@images-auditor`、`@build-auditor`、`@verifier` 和 `@monitor`，全部在 `gemini-4-hydrogen` 上。核验者在第 7 号报告里标出两处错误说法。9 分 42 秒后链接审计员贴出更正，核验者签收，协调者把一份 10 条的排序摘要作为第 14 帖发出。他称为实锤的发现：233 MiB 未使用或被遮住的资源；图像审计脚本漏掉 3646 张里的 3021 张；链接检查器看不见 79 个付费应用；构建以并发 2 跑过大约 8721 个产物。他写道，单独一次聊天会很有把握地报出前两项，并且永远不会去核对。

只读这一跑干净之后，他给出两段改编自 Antigravity 文档的写代码 swarm。工人默认和当前对话用同一个模型，所以换模型的核验者必须显式要求。那两段提示留在原文。

## 5. T3 Code，以及不该交给 Neon 的席位

他更喜欢的接法并不把每个席位都给 Neon。T3 Code 把各家自己的 harness 并排放：Claude Code、Codex，以及经官方 ACP agent 接上的 Antigravity（t3dotgg 的 PR #9348）。

Theo 的分工，按他的转述。Neon 写得好，也服从否定指令：让它忽略某个 skill 的文风，它照做了。代码审查是 21 条成立、5 条次要、0 条错误。架构问题是 72 条说法里错了 10 条。因此 Neon 负责写和审，另一个模型主导实现，第三个模型扮演用户。

连接：Settings → Providers，打开 Antigravity，安装运行时。需要 1.5 或更新，推荐 1.5.12，大约 1.25 GiB。登录可以是 Google 账号、Gemini Enterprise、Gemini API key 或 Agent Platform（Vertex）。买不了 AI Pro 的 Workspace 账号用 API key 或 Enterprise。选中 Gemini 4 Neon 后，思考档位就在旁边。`/plan`、`/compact`、`/goal` 会透传。Skills 从 `.agents/skills` 加载。

他说能用的分工：High 上的 Neon 做规划和审查；大量实现交给 Orchestrator 里的 Hydrogen，或交给已经在付钱的 Claude 席位；试玩者来自另一个实验室，而且没有读过计划；三维和着色器很重的活不要只靠 Neon。Theo 说它的三维输出弱。看起来好的演示，外观都是人先指定的。

Theo 的安装故事：Neon 在他的 Windows 机器上装好带认证的 Antigravity，在 T3 Code 里验证，并通知另一个线程，用了 8 分钟。同一过程里它发现 T3 Code 的 Antigravity 适配器在丢 `html_render` 帧，绕开之后开了一个 pull request。他还说有人从别的编辑器转向 Neon；例子在截图里，正文没有列出。

## 6. 新的 Gemini 工具，从 Claude Code 或 Codex 调用

发帖（2026-11-25）前 72 小时里有两次发布。11 月 23 日，Gemini API 开始把视频、音频和文档当作函数结果接受。11 月 24 日，开源的 `gemini-api` CLI 传开，Claude Code 或 Codex 可以从 shell 调用 Gemini 的媒体栈。他的读法是：这不是让 Antigravity 退场，而是团队里属于 Gemini 的那一半不必再住在里面。

`gemini-api` v0.5.0 被标在 11 月 20 日。安装：

```bash
curl -fsSL https://raw.githubusercontent.com/google-gemini/gemini-api-cli/main/scripts/install.sh | bash
```

Windows：`iwr -useb https://raw.githubusercontent.com/google-gemini/gemini-api-cli/main/scripts/install.ps1 | iex`。或 `go install github.com/google-gemini/gemini-api-cli/cmd/gemini-api@latest`。他接着演示 `gemini-api configure` 和 `gemini-api whoami`。

这个 CLI 故意无状态。计划和上下文留在 agent 这边，CLI 返回文件路径或 interaction id。他说团队最常用的命令，这里只记名字：后台 Deep Research（示例 agent id 为 `deep-research-preview-04-2026`）和状态查询；对 PDF 或截图做 `analyze`；以及 `image`、`video`、`tts`、`music`，文中标成 Nano Banana 2.1、Omni 1.2 Flash、3.8 Flash TTS 和 Lyria。他让人贴进 `AGENTS.md`、`CLAUDE.md` 或 `.cursorrules` 的那段是 README 摘录，这里不抄。

11 月 23 日之后，UI 子代理可以把 bug 的 mp4 作为函数结果返回，Neon 直接看这段视频。通话录音、`.docx` 或表格页走同一条路。媒体要留在工具结果里。把它贴回成新的用户回合，函数调用链会断。

## 7. 同一支团队写进 ADK

Antigravity 是你自己跑这支团队的地方。要给别人跑（Slack 机器人、夜间作业、API）时，把同一张图在 ADK 2.0 里重建，部署到 Cloud Run。Neon 规划和核验，Hydrogen 工人扇出，条件边把被拒绝的工作送回去。

最小交接来自他转述的 Google Neon 开发指南：Neon 把计划存在一次 interaction 上，Hydrogen 用 `previous_interaction_id` 接上同一条线程，计划不用再发。示例客户端是企业客户端。示例任务的原文不抄。

ADK 2.0 的 workflow 是边。元组负责扇出。`JoinNode` 等每条分支结束，再交出一个以节点名为键的字典。一组路由字典把边变成条件边。回去的边若是条件边，循环就允许。Agent 节点默认 `mode="single_turn"`，这正是他要给工人的：一份干净简报，没有残留上下文。`google.adk.labs.antigravity` 里的 `AntigravityAgent` 可以从 ADK 驱动真正的 Antigravity agent。名字要按所用版本的 ADK 文档核对。Python 草稿只在原文。

## 8. 动效，只记到高层

那一周被转发最多的 Neon 演示是视频，好的那些都是代码：一个 HTML 文件，用 Canvas、Three.js 或 WebGL，再渲染成帧。写成代码的影片可以按代码来验收：时长、切点是否落在节拍上、音频是否削波。

四段公开要求，只留约束：

- @IshanJX（Extra Effort）：15 秒、60 帧的展示片，一个代码文件，不要外部库或素材。
- @atomtanstudio：纯代码介绍 Gemini 4 Neon，不要外部素材。
- @cfenghub：Neon 先写自己的提示再写影片，单页 Three.js 加 GLSL，15 秒，明亮的高调发布风格，Gemini 渐变，人工编辑为零。
- @Jonathascosta89，开在 xhigh：一个项目文件夹，外加一句 “make a presentation video of my project”。

雨景：没有人要求深度模型。Neon 自己判断这张图需要深度，取了 Depth Anything V2 Large，在本地跑完，再做出分层。他的寓意是，一个目标加一个核验者，团队就会自己造工具。Noel 的抱怨被他称为公平，而且是简报的问题。不给品牌色时，Neon 会退回自己的渐变。把品牌色和节拍时间写进简报。

作为 swarm：先把简报存成 `film/brief.md`，再拆成画面、文案、声音（经 `gemini-api`）和渲染，核验者用另一个模型。提示里的具体时长和音频上限留在原文。

## 价格，按原文

**订阅**（应用、`agy`、T3 Code）：Google AI Pro 每月 19.99 美元。固定方案吸收思考 token，所以 xhigh 显得像免费。额度并不免费：一支 swarm 是单个 agent 的 4 到 5 倍，一次运行可能中途耗尽。

**API，每 100 万 token：**

- Gemini 4 Neon：输入 3.00 美元，输出 15.00 美元，缓存输入 0.15 美元（他称为 95% 折扣）。隐式缓存从 4096 个 token 起，所以 `AGENTS.md` 和简报应放在提示顶部并保持稳定。
- Gemini 4 Hydrogen：输入 1.50 美元，输出 7.50 美元，缓存 0.075 美元。这是工人。
- Gemini 4 Carbon：介绍价到 11 月 30 日为 2 / 10 美元，12 月 1 日起为 4 / 20 美元，顺序同样是先输入后输出。那个月用 Carbon 做过原型的人，下周要再对一次账单。文中没有 Carbon 的缓存价。

他说按 token 的标价会误导，因为 Neon 想得很多。SimpleBench 上它每题平均输出 2385 个 token，Opus 5.5 是 575 个，所以完整跑一遍 Neon 是 7.99 美元，Opus 5.5 是 2.74 美元。他的切分：规划和审查用 High 上的 Neon，量放在 Hydrogen，xhigh 只留到卡住的那个晚上。在 High 上，Neon 每项任务比 Carbon 便宜 32%。

## 第一晚

他的时间盒，收成清单。指向的提示在原文。

- 15 分钟。安装 `agy` 或应用，登录，选择 Orchestrator · Neon High。
- 10 分钟。在主仓库上，先拿到代码地图和拆成轨道的方案，任何编辑之前读完。
- 25 分钟。写 `AGENTS.md`（命令、所有权、「完成」的含义）和 `.agents/agents/verifier.md`。
- 30 分钟。三条轨道的只读 `/swarm`。只读最后的摘要，再亲手抽查两条发现。
- 15 分钟。安装 `gemini-api`，在 `AGENTS.md` 里说明它，让平时用的 agent 做一份竞争对手的 Deep Research。
- 10 分钟。拿审计里的一条发现跑第一次写代码的 swarm，核验者放在另一个模型上。

## 九段提示

文章以九段可复制提示收尾（尖括号占位符）。这里不收录。按他的标题，分别是：编辑前先画仓库地图的 Orchestrator 开场；只读审计 swarm；带所有权的功能 swarm；一段短的核验者系统提示；工人简报；单点排错简报（他把它和 Theo 的 worktree 追查连在一起）；调用 Gemini 做图像、旁白和竞品研究的落地页；由已存简报驱动的动效；以及一段短回顾，说明改了什么、核验了什么、拒绝了什么、该亲手查的那一件事。

## 收束

他的结论是：提示词现在是在做人员配置。同一模型在 12.8% 和 39.0% 之间的差距是 harness。Theo 的 7 天以上和他不到 10 小时之间的差距，是一支有目标的团队。Flavio 那份排序摘要和一份自信但错误的摘要之间的差距，是一个核对了行号的核验者。要写下来的组织图是：谁规划、谁打字、谁检查，以及「完成」是什么意思。

## 短引

> "But Neon wasn't built to be a chatbot. Google built it to be the boss of a team: plan the work, hand it out, check it, sign off"
> — Movez，https://x.com/i/article/2125560293870669824

> "a fast, high-judgment Orchestrator directing parallel Hydrogen subagents."
> — 文章所引 Google 发布帖

> "Let Neon decide. Let Hydrogen type. Let a stranger check."
> — Movez，同一篇文章
