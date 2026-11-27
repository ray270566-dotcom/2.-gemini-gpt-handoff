# ChatGPT 桌面端 filesystem MCP（可选，半自动那步用）

目的：让 ChatGPT 桌面端 Codex 模式能直接读写仓库文件夹，不用每次拖文件。

> 注意：本地 MCP 只在 Codex / Work 模式里生效，普通聊天不走它。
> 只授权这一个仓库文件夹，千万别指到 home 目录或整个硬盘。

## Mac

ChatGPT 桌面端：设置 → MCP servers → Add server

- Name：`filesystem`
- Type：STDIO
- Command：`npx`
- Arguments（逐条加）：
  - `-y`
  - `@modelcontextprotocol/server-filesystem`
  - `/Users/<你>/Projects/<项目名>`（改成你仓库的实际路径）

保存后 Restart。聊天里输入 `/mcp` 能看到它在列表里就是连上了。

同一套配置也写在 `~/.codex/config.toml`，Codex CLI 共用：

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/Users/<你>/Projects/<项目名>"]
```

## Windows

同样在设置 → MCP servers 里加，路径改成 Windows 的，例如：

```
D:\Projects\<项目名>
```

`~/.codex/config.toml` 里对应写：

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "D:\\Projects\\<项目名>"]
```

## 不用 MCP 也能跑

如果这一步配起来麻烦，先跳过：Codex 模式里直接打开仓库文件夹，或者把 `brief.md` / `02-plan-final.md` 拖进对话，效果一样，只是多一步手动操作。
先跑通一次完整流程，再决定要不要加 MCP。
