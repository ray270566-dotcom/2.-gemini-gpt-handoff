# swarm-run

用 Swarm 跑一个任务。Carbon 当 coordinator，Neon / Hydrogen 当 worker，GPT Codex 走 A 方案（文件交接），不走 B 方案（MCP / CLI 桥）。

## 什么时候用

- 任务能干净拆成 3 块以上、互不撞文件时才用。
- 小任务、单文件改动、只想清楚一件事时不要用，直接单聊。

## 用法

用户只说任务名 / 一句话，例如：

```
/swarm-run 给 Shopee 订单导出加检查
```

你先做这三件事，再发 `/swarm`：

1. 读 `brief.md`，确认任务已经写清楚；没写清楚先让用户补，不要猜。
2. 确认 `handoffs/02-plan-final.md` 已经存在；不存在就先停下来，走第 1 步定方案（Gemini 和 GPT 各出一份独立方案，用户合并）。
3. 把下面 prompt 里的 `<任务>` 换成用户说的任务和最终方案的章节，照原样发 `/swarm`。

## Swarm prompt（照抄，替换 <任务> 和 <核心模块>）

```
/swarm <任务>，按 handoffs/02-plan-final.md 执行，仓库根目录就是本项目。

分工（每个 worker 一个独立 worktree，不要互相改对方的文件）：
- @plan on Gemini 4 Carbon：只读，先读 brief.md 和 02-plan-final.md，输出文件清单和验收标准，写到 handoffs/swarm-plan.md，不写代码。
- @author-core on Gemini 4 Neon High：做核心模块 <核心模块>。
- @author-fast on Gemini 4 Hydrogen High：做常规切片和单元测试。
- @verifier on Gemini 4 Neon High：每 2 个 worker 配一个，读 diff、跑测试，有问题直接在 board 上 @ 对应的 author 打回。

规则：
- 每个 worker 只改自己分到的文件，改之前先在 board 上报自己要动的文件清单。
- 所有结论写进对应的 handoffs/ 文件，不要只留在聊天里。
- 严重问题才拦：算错钱、丢数据、越权、发错消息、上线会出事故；其他只记建议。
- 最多返工两轮，两轮不过就停下来，等用户拍板。
- 需要 GPT Codex 做的事不要直接派：写进 handoffs/codex-task.md，等用户把这份文件交给 Codex。
```

## GPT Codex（A 方案，默认）

- Swarm 里任何需要 Codex 的活，都先写 `handoffs/codex-task.md`，不要试图在 Swarm 里直接调用 Codex。
- Codex 做完后，用户把 `handoffs/04-backend-gpt.md` 和 `handoffs/04-review-gpt.md` push 回来，Swarm 再 pull 继续。
- B 方案（MCP / CLI 桥）目前不用。除非用户明确说要上全自动，才再讨论。

## 额度提醒

Swarm 是预览功能，每个 worker 都烧额度，并行起来烧得很快。发之前提醒用户看一眼剩余额度。
