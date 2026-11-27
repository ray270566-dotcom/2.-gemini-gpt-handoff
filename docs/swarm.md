# Swarm 模式：Carbon 当总指挥（Antigravity 内）

> Swarm 是预览功能（Preview & Quota Heavy）。每个 worker 都是独立 agent，都耗额度，并行跑起来额度烧得比单聊快得多。只在任务够大、能干净拆分时用，小任务直接单聊。

## Swarm 原生能做到什么

- 在 Antigravity App / CLI 里输入 `/swarm <任务>`，当前对话就变成 coordinator。
- Coordinator 拆任务、给每个 worker 起名（例如 `@parser`、`@api-tests`、`@docs`），每个 worker 用干净上下文、在自己的对话里跑，任务简报发到共享 message board 上，所有人都能看到谁在干什么。
- 通常每 2–4 个 worker 配一个 `@verifier`，读 diff、跑测试，有问题打回给写代码的 worker 改。
- 所有人只在 board 上用 `@name` / `@group` / `@all` 沟通，不私聊；没事做的 worker 会睡过去，不耗 token；还有一个 `@monitor`，全员安静但活没干完时才会出来催。
- Coordinator 本人不干技术活，只分活、评判、向你汇报。
- Worker 默认在隔离的 git worktree 里跑，互相不撞文件；Swarm UI 里有 Board、Roster、Graph，可以实时看、随时插手改方向。

官方说明：`/swarm` 把任务交给一组并行 agent，“a coordinator that spins up specialized parallel workers in isolated git worktrees so edits never collide, and adversarial verifiers dedicated to running tests and challenging diffs”。

## 模型怎么配：Carbon 总指挥 + Neon / Hydrogen 执行

不要和 Orchestrator 模式搞混：

- **Orchestrator 模式**是单会话内自动切模型，2.34.0 版本里 Executor 只能选 Hydrogen 和 Claude Sonnet 5.5，选不到 Neon；验证类 subagent 还会悄悄降到 Hydrogen。所以想让 Carbon 规划、Neon 执行，不要用 Orchestrator，用 `/swarm`。
- **Swarm 模式**可以在 prompt 里给每个 worker 指定模型，`/swarm` 接受具体的模型 ID（例如 `gemini-4-neon-high`、`gemini-4-carbon-high`），也可以中途给等着的 worker 换模型、保留对话历史。

先确认你的套餐里能看到 Carbon。Carbon 最初是 Ultra 才能用，Neon 是 Pro / Ultra 都能用；以你打开模型选择器实际看到的为准，看不到就别在 prompt 里硬写。

推荐配法：

| Swarm 角色 | 模型 | 干什么 |
|---|---|---|
| coordinator | Gemini 4 Carbon | 只拆任务、定标准、验收，不写代码 |
| @plan / @scout | Gemini 4 Carbon 或 Neon | 读仓库、定方案、拆文件清单（只读） |
| @author-核心 | Gemini 4 Neon High | 复杂模块、跨模块状态、核心抽象 |
| @author-常规 | Gemini 4 Hydrogen High | 常规切片、组件扩展、单元测试，跑得快 |
| @verifier | Gemini 4 Neon High， 高风险再升 Carbon High | 读 diff、跑测试、挑错，不和作者用同一个模型 |

## 直接能复制的 /swarm prompt

在 Antigravity 里先把模型切到 Gemini 4 Carbon，再发：

```
/swarm 按 handoffs/02-plan-final.md 执行，仓库根目录就是本项目。

分工（每个 worker 一个独立 worktree，不要互相改对方的文件）：
- @plan on Gemini 4 Carbon：只读，先读 brief.md 和 02-plan-final.md，输出文件清单和验收标准，写到 handoffs/swarm-plan.md，不写代码。
- @author-core on Gemini 4 Neon High：做核心模块（在 02-plan-final.md 里标出来的那部分）。
- @author-fast on Gemini 4 Hydrogen High：做常规切片和单元测试。
- @verifier on Gemini 4 Neon High：每 2 个 worker 配一个，读 diff、跑测试，有问题直接在 board 上 @ 对应的 author 打回。

规则：
- 每个 worker 只改自己分到的文件，改之前先在 board 上报自己要动的文件清单。
- 所有结论写进对应的 handoffs/ 文件，不要只留在聊天里。
- 严重问题才拦：算错钱、丢数据、越权、发错消息、上线会出事故；其他只记建议。
- 最多返工两轮，两轮不过就停下来，等我拍板。
```

发完打开 Swarm 面板（`/swarm board` 或 Alt+S）看 Board、Roster、Graph，随时可以点进某个 worker 的对话插话。

## GPT Codex：Swarm 原生派不过去，要单独接

这一点要先说清楚，不然会以为一点就能全自动：

- Swarm 的 worker 都是 Antigravity 自己的 agent，只能用模型选择器里的模型。Antigravity 里没有真正的 GPT-6.2，只有快下架的 GPT-OSS，所以 **Swarm 不能原生把任务派给 GPT Codex**。
- 想让 Codex 也进这个队，有两条路，**优先用 A，B 暂不用**：

**A. 文件交接（默认，已选用）**

Swarm 里加一个任务：让 coordinator 写一份 `handoffs/codex-task.md`，里面写清：背景、最终方案链接、要 Codex 做的具体事、输入文件、输出文件、验收标准、不要动的文件。
你把这份文件（或整个分支）交给 ChatGPT 桌面端 Codex 模式，Codex 做完把结果写回 `handoffs/04-backend-gpt.md` 和 `handoffs/04-review-gpt.md`，push 回 Git，你再 pull 回 Antigravity。
Swarm 的 coordinator 负责准备这份 brief，Codex 负责干活，人负责在中间传一次。

**B. MCP / CLI 桥（暂不用，想全自动再考虑）**

社区有现成桥，例如把 Antigravity（`agy`）和 Codex（`codex exec`）都暴露成 MCP 工具，让一个 agent 中途把任务派给另一个模型家族；也有 Antigravity + Codex 的 playbook，让 Antigravity 只做调度、Codex 做执行。
这类桥都要在本机装好 Codex 并登录、能跑 shell 命令，安全边界要自己管（Codex 的 sandbox 是真的边界，`agy -p` 的 sandbox 只拦 shell 命令，不拦文件写和联网）。先跑通 A，再决定要不要上 B。

## 什么时候用 Swarm，什么时候不用

- 用：任务能干净拆成 3 个以上互不撞的块，例如多文件重构、多包并行、多个独立调查。
- 不用：小任务、单文件改动、只需要一个模型想清楚的事。Swarm 每个 worker 都要烧额度，小活用它就是浪费。
- 额度提醒：官方原话是“even with zero-token idle waits and dynamic model hot-swapping, parallel multi-agent systems consume quota much faster than a standard chat”。跑之前先看一眼剩余额度。
