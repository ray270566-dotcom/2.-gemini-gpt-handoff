# Gemini × GPT 双模型工作流

目标：不用 Claude，只用 Antigravity 里的 Gemini 和 ChatGPT 桌面端里的 GPT，打通两个模型。
Antigravity 内用 Swarm：Carbon 当总指挥，Neon / Hydrogen 当执行；GPT Codex 是外面的 worker，用文件交接进来。

## 角色

| 角色 | 工具 | 负责 |
|---|---|---|
| 总指挥（最终拍板） | 你 | 写 brief、合并两份方案、拍板 |
| Swarm coordinator | Antigravity，Gemini 4 Carbon | 拆任务、定标准、验收，不写代码 |
| Swarm worker | Antigravity，Neon High / Hydrogen High | 按分工落地，互不撞文件 |
| Swarm verifier | Antigravity，Neon High（高风险升 Carbon） | 读 diff、跑测试、打回 |
| GPT | ChatGPT 桌面端 Codex / Work 模式 | 独立出方案、写后端 / 算法、修 bug、上线前检查 |

GPT 接入优先用 A 方案（文件交接，见下），B 方案（MCP / CLI 桥）暂不用。

## Swarm 预设（不用每次贴 prompt）

`.agents/skills/swarm-run/SKILL.md` 就是预设。用法：

```
/swarm-run <任务一句话>
```

例如 `/swarm-run 给 Shopee 订单导出加检查`。它会先确认 `brief.md` 和 `02-plan-final.md` 齐了，再按预设发 `/swarm`，Carbon 当 coordinator，Neon / Hydrogen 当 worker。
`.agents/agents/coordinator-carbon.md` 是 coordinator 的角色定义。

为什么这样分：Gemini 可沟通性最好、创意最灵、Antigravity 能直接改本地文件；GPT 最严谨，后端和检查最放心。Swarm 补上了原来 Claude 总指挥的位置，Carbon 只做调度和验收，不自己写代码。

## 同步方式：Git 为准，云盘只做冷备份

- **Git 私有仓库是唯一同步通道。** Mac 和 Windows 都 clone 同一个仓库，改完 commit + push，换机器先 pull。
- **云盘不同步、不做实时同步。** 只做冷备份：每周或每个大任务结束时，把仓库打一个 bundle 存到云盘，同时把原始数据（视频、录屏、原始文档等）也拷一份到云盘。云盘里的东西不直接打开改，丢了 Git 还能从 bundle 恢复。

冷备份命令（在仓库根目录跑一次即可）：

```bash
git bundle create backup-2026-11-27.bundle --all
```

把生成的 `.bundle` 文件拷到云盘。恢复时：`git clone backup-2026-11-27.bundle <新目录>`。

## 目录

```
.
├── AGENTS.md                  # 唯一规则源，两个模型都读它
├── GEMINI.md                  # Antigravity 读：导入 AGENTS.md
├── brief.md                   # 每次任务的原始需求，只写你的原话 + 必要背景
├── handoffs/
│   ├── 01-plan-gemini.md     # Gemini 独立出的方案
│   ├── 01-plan-gpt.md        # GPT 独立出的方案
│   ├── 02-plan-final.md      # 你合并后的最终方案
│   ├── 03-implementation-gemini.md  # Gemini 落地记录
│   ├── 04-backend-gpt.md     # GPT 后端 / 硬活记录
│   ├── 04-review-gpt.md      # GPT 上线前检查
│   └── 05-decision.md        # 你的拍板记录
└── docs/
    ├── setup-mac-windows.md  # 两台机器的安装与配置
    ├── swarm.md              # Swarm 模式：Carbon 总指挥 + Neon/Hydrogen 执行 + Codex 怎么接
    └── mcp-filesystem.md     # ChatGPT 桌面端 filesystem MCP 配置
```

## 四步流程（每步只让一个模型写，另一个只读）

### 第 1 步：定方案 —— 两份独立方案，你来合并

1. 你写好 `brief.md`，commit + push。
2. Antigravity 里开新对话，模型选 Gemini 4 Neon，只给它 `brief.md`，让它写 `handoffs/01-plan-gemini.md`。
3. ChatGPT 桌面端开 Codex / Work 模式，只给它 `brief.md`（不要给 Gemini 的方案），让它写 `handoffs/01-plan-gpt.md`。
4. 你对照两份，挑各自没想到的点，写 `handoffs/02-plan-final.md`，commit + push。
5. 关键点：两份方案必须独立出，不能一个抄另一个，否则对照就没意义。

### 第 2 步：做设计 + 落地 —— Swarm 在 Antigravity 里做

任务够大、能干净拆成 3 块以上时，用 `/swarm`（详见 `docs/swarm.md`，直接复制里面的 prompt）：
Carbon 当 coordinator，只拆任务、验收，不写代码；Neon High 做核心模块，Hydrogen High 做常规切片和测试；每 2 个 worker 配一个 Neon verifier。
小任务别用 Swarm，单聊更省额度——Swarm 是预览功能，每个 worker 都烧额度，并行起来烧得很快。

Swarm 做完写 `handoffs/03-implementation-gemini.md`：改了哪些文件、怎么跑、跑通没有、还剩什么问题。
commit + push，分支建议用 `gemini/<任务名>`。

### 第 3 步：后端 / 硬活 —— GPT 做

ChatGPT 桌面端 Codex 模式打开同一个仓库（先 pull 到最新），照着 `02-plan-final.md` 写后端、算法、数据处理。
小改动别派给 GPT，派一次活的沟通成本比活本身大；小改动让 Gemini 顺手做。
做完写 `handoffs/04-backend-gpt.md`，commit + push，分支建议用 `gpt/<任务名>`。

### 第 4 步：上线前检查 —— GPT 查，最多两轮

每次改动上线前都过 GPT 这一关。让它读 diff，写 `handoffs/04-review-gpt.md`。
问题分两级，只有算错钱、丢数据、越权、发错消息、上线会出事故才拦着不让上，其他只记成建议。
最多查两轮，两轮还过不去就写进 `handoffs/05-decision.md`，由你拍板，不要无限返工。

## 防撞规则

- 同一时间只有一个模型在写代码，另一个只读交接文档。
- 两个模型不要在同一个分支上改同一个文件：Gemini 用 `gemini/<任务名>`，GPT 用 `gpt/<任务名>`，你合到 `main`。
- 并行出方案时可以用 git worktree 各开一个工作区，互不干扰。
- 每次换机器、换模型前先 `git pull`，写完马上 `git push`，不要攒到第二天。

## 规则沉淀

AI 犯过的错被纠正一次，就把这一类错拦住：能写成自动检查就写成检查，实在做不到才写成一条文字规则，记进 `AGENTS.md`。
`AGENTS.md` 是唯一规则源，改规则只改这一份，不要在两个客户端里各记一份。
