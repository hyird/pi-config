# Pi 配置

交给 AI：

```text
请按此说明配置我的 Pi：https://raw.githubusercontent.com/hyird/pi-config/main/README.md
```

## 1. 定位与清理

- 自行定位 Pi：Linux/macOS 用 `command -v pi`，Windows 用 `Get-Command pi`；确认启动器、版本及 Bun、Git。
- 配置目录取实际启动环境的 `PI_CODING_AGENT_DIR`，否则为当前用户的 `~/.pi/agent`，与程序安装目录区分。路径不明确时询问。下文路径均相对此目录。
- **先清理旧配置，不备份，再重新配置。** 确认其他 Pi 实例未在写入；用 `pi list` 列出旧插件，经 `pi remove <来源>` 卸载，确保账号存储不被删除。
- 清理旧 `settings.json`、`models.json`、`mcp.json`、全局 `AGENTS.md`、`SYSTEM.md`、`APPEND_SYSTEM.md`，以及 `extensions/`、`agents/`、`prompts/`、`themes/`、`skills/` 中的旧配置和自定义资源。
- **保留全部登录信息**，包括 `auth.json`、`multiprovider-auth.json`、插件/MCP 凭据、token、cookie、私钥；不退出登录，不打印或上传秘密。配置混有凭据且无法安全分离时，停止该项并报告。
- 不整目录删除配置目录，不跟随符号链接；不动会话、项目、程序和运行依赖，不手动清空 `npm/`、`git/`、`bin/`。

## 2. 插件

用 `pi install <来源>` 安装以下插件，不钉版本、不重复安装、不复制源码；检查版本兼容性，不兼容时报告。本文不是 Pi 插件。

```text
npm:@juicesharp/rpiv-ask-user-question
npm:pi-goal
npm:@vndv/pi-codegraph
npm:pi-mcp-adapter
npm:pi-multiprovider
npm:@monotykamary/pi-better-openai
npm:@monotykamary/pi-better-grok
git:github.com/hyird/pi-better-opencode-go
npm:@bacnh85/pi-rtk
npm:@bacnh85/pi-subagent
```

RTK 需在 PATH 中，最低 0.23.0，推荐 ≥ 0.46.0；仅改写 Bash。CodeGraph 在需要索引的项目执行 `codegraph init`。

按插件文档编辑下列选项：

| `extensions/` 下的文件 | 非默认设置 |
|---|---|
| `pi-better-openai.json` | `usage.autoRedeemBankedResets = false`、`footer.mode = "status"` |
| `pi-better-grok.json` | `footer.mode = "status"` |
| `opencode-go-usage.json` | `footer.mode = "status"` |

## 3. Pi 设置

编辑 `settings.json`，保留安装命令写入的包清单，仅调整：

| 字段 | 值 |
|---|---|
| `theme` | `"Codex"` |
| `quietStartup`, `showHardwareCursor` | `true` |
| `images.blockImages` | `true` |
| `terminal.clearOnShrink`, `terminal.showTerminalProgress` | `true` |
| `tuiMode` | `"fullscreen"` |
| `editorPaddingX` | `1` |
| `hideThinkingBlock`, `showCacheMissNotices`, `collapseChangelog` | `true` |
| `defaultProjectTrust` | `"always"`；会自动信任并加载项目代码，须说明风险并获使用者确认，否则保持默认 |

不指定模型。Windows 使用 `defaultTools = ["read", "powershell", "edit", "write"]`（先确认版本支持）；Linux/macOS 使用默认 Bash。命令语法跟随系统。

## 4. Codex 主题

创建 `themes/Codex.json`，`name = "Codex"`，按当前 Pi schema 设置 `colors`；同一行的 token 使用同一颜色：

| token | 颜色 |
|---|---|
| `accent`, `borderAccent`, `scrollbarThumb`, `mdQuoteBorder`, `mdListBullet`, `syntaxKeyword`, `thinkingMinimal` | `#d0a0ff` |
| `customMessageLabel`, `toolTitle`, `mdHeading`, `syntaxType` | `#e6c7ff` |
| `mdLink`, `syntaxFunction`, `syntaxOperator`, `thinkingLow` | `#8be9fd` |
| `success`, `mdCode`, `toolDiffAdded`, `syntaxString`, `thinkingMedium` | `#50fa7b` |
| `warning`, `thinkingHigh` | `#f1fa8c` |
| `syntaxVariable`, `syntaxNumber`, `thinkingXhigh`, `bashMode` | `#ffb86c` |
| `error`, `toolDiffRemoved`, `thinkingMax` | `#ff6e6e` |
| `text`, `searchMatchText`, `userMessageText`, `customMessageText`, `mdCodeBlock` | `#f5f5f5` |
| `muted`, `toolDiffContext`, `syntaxPunctuation` | `#9b9ba5` |
| `dim`, `mdHr`, `syntaxComment`, `thinkingOff` | `#666674` |
| `thinkingText`, `toolOutput`, `mdQuote` | `#c6c6d0` |
| `border`, `mdCodeBlockBorder` | `#383842` |
| `borderMuted`, `scrollbarTrack` | `#25252c` |
| `selectedBg`, `toolPendingBg` | `#222228` |
| `userMessageBg`, `customMessageBg` | `#17171b` |
| `searchMatchBg` | `#3b2d4d` |
| `toolSuccessBg` | `#17291e` |
| `toolErrorBg` | `#321d24` |
| `mdLinkUrl` | `#b8dfff` |

`export`：`pageBg = "#0d0d0f"`、`cardBg = "#17171b"`、`infoBg = "#222228"`。

## 5. Playwright MCP

在配置目录的 `npm/` 下用 Bun 添加独立服务 `@playwright/mcp`，保留其他依赖；确认 Chrome 和 `node_modules/@playwright/mcp/cli.js` 可用。不卸载已有 Node。Pi 插件仍由 `pi install` 安装。

编辑 `mcp.json`：

| 路径 | 值 |
|---|---|
| `settings.mcpFooterStatus` | `"full"` |
| `mcpServers.playwright.command` | 本机 Bun 绝对路径 |
| `mcpServers.playwright.args` | CLI 文件绝对路径，随后为 `--browser`, `chrome`, `--isolated`, `--headless`, `--caps`, `devtools` |
| `mcpServers.playwright.lifecycle` | `"lazy"` |
| `mcpServers.playwright.directTools` | `false` |

## 6. 验证

运行 `pi update --extensions` 后重启 Pi。用 `pi list`、`/rtk status` 检查插件；验证主题、界面、用量选项及 MCP 打开测试页面。确认无旧配置残留、重复工具或启动错误。

沿用已有登录，仅缺失或失效时提示 `/login`、`/multilogin`。报告已完成、已清理和未解决项，不将未验证内容记为成功。
