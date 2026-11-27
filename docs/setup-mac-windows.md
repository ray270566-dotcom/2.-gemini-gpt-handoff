# 两台机器的配置：Mac + Windows

前提：两台机器都装 Antigravity 客户端和 ChatGPT 桌面客户端，登录同一个账号。
Git 仓库是两台机器之间唯一的同步通道，云盘不参与同步。

## 1. 建仓库（只做一次，在常用那台机器上）

```bash
git init <项目名>
cd <项目名>
# 把本模板里的文件拷进来
git add .
git commit -m "init: gemini-gpt handoff"
git branch -M main
git remote add origin <你的私有仓库地址>
git push -u origin main
```

另一台机器：

```bash
git clone <你的私有仓库地址>
```

## 2. Antigravity（两台机器一样）

1. 打开 Antigravity，打开这个仓库文件夹作为 Project。
2. 需要时再把同一个仓库的 Google Drive 文件夹也连进同一个 Project（只连文档用，不做同步）。
3. 模型选择：
   - 出方案、想创意、写文案：Gemini 4 Neon
   - 任务复杂、推理链长：Gemini 4 Carbon
   - 不要用 Antigravity 里的 Claude，额度不够是已知问题，本流程不依赖它。
4. 每次开工前先看一眼右下角分支是不是自己的分支（`gemini/<任务名>`），不是就先切。

## 3. ChatGPT 桌面端（两台机器一样）

1. 打开 ChatGPT 桌面客户端，切换到 Codex / Work 模式（不是普通聊天）。
2. 打开同一个仓库文件夹。
3. 先 `git pull`，确认拿到 Gemini 刚 push 的最新内容，再动手。
4. 分支用 `gpt/<任务名>`，做完 commit + push。

## 4. 每天的顺序

```
Mac 上 Gemini 做完 -> push
Windows 上 GPT 开始前 -> pull
Windows 上 GPT 做完 -> push
Mac 上 Gemini 接手前 -> pull
```

一句话：换模型、换机器，先 pull；写完，马上 push。

## 5. 云盘冷备份（不做同步）

每周一次，或每个大任务结束时：

```bash
# 在仓库根目录
git bundle create backup-YYYY-MM-DD.bundle --all
```

把 `.bundle` 文件和当次的原始数据（录屏、原视频、原始文档）一起拷到云盘的一个日期文件夹里，例如 `冷备份/2026-11-27/`。
不要把云盘文件夹直接设成仓库的工作目录，也不要开云盘的实时同步去同步这个仓库——会和 Git 抢文件，容易出冲突和损坏。
