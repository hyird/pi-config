# Pi 配置

交给 AI：

```text
请按此说明配置我的 Pi：https://raw.githubusercontent.com/hyird/pi-config/main/README.md
```

按顺序执行；本文不是插件。只处理用户级 Pi 配置，不改项目配置。

## 1. 确认环境

- 定位实际使用的 Pi：Linux/macOS 用 `command -v pi`，Windows 用 `Get-Command pi`；检查启动器、版本、Bun 和 Git。Pi 不在 PATH 时使用启动器绝对路径。
- 配置目录取启动环境的 `PI_CODING_AGENT_DIR`，否则为当前用户的 `~/.pi/agent`；不要与程序安装目录混淆。下文文件路径均相对此目录。不确定时先询问。
- Windows 必须使用 PowerShell 7：`Get-Command pwsh` 检查；缺失时用 `winget install --id Microsoft.PowerShell --exact --source winget` 安装，无 winget 则用 Microsoft 官方安装包。刷新 PATH 后验证 `pwsh --version`，不回退到 5.1。Linux/macOS 使用 Bash。
- 缺失依赖或版本不兼容先解决，再清理配置；不卸载已有 Node。

## 2. 清理旧配置

**不备份，保留登录，清理后重建。**

- 确认其他 Pi 实例未在写入。先用 `pi list` 列出用户级旧插件，再用 `pi remove <来源>` 卸载；确认不会删除账号存储后执行。
- 清理旧 `settings.json`、`models.json`、`mcp.json`、全局 `AGENTS.md`、`SYSTEM.md`、`APPEND_SYSTEM.md`，以及 `extensions/`、`agents/`、`prompts/`、`themes/`、`skills/` 中的旧配置和自定义资源。
- **保留全部登录信息**，包括 `auth.json`、`multiprovider-auth.json`、插件/MCP 凭据、token、cookie、私钥；不退出登录，不打印或上传秘密。配置混有凭据且无法安全分离时，停止该项并报告。
- 不整目录删除配置目录，不跟随符号链接；不动会话、项目、程序和运行依赖，不手动清空 `npm/`、`git/`、`bin/`。

## 3. 安装插件

用 `pi install <来源>` 安装以下插件，不钉版本、不重复安装、不复制源码。不要用 Bun 安装 Pi 插件。

```text
npm:@juicesharp/rpiv-ask-user-question
npm:pi-goal
npm:pi-mcp-adapter
npm:pi-multiprovider
npm:@monotykamary/pi-better-openai
npm:@monotykamary/pi-better-grok
git:github.com/hyird/pi-better-opencode-go
npm:@bacnh85/pi-subagent
```

按插件文档编辑下列选项：

| `extensions/` 下的文件 | 非默认设置 |
|---|---|
| `pi-better-openai.json` | `usage.autoRedeemBankedResets = false`、`footer.mode = "status"` |
| `pi-better-grok.json` | `footer.mode = "status"` |
| `opencode-go-usage.json` | `footer.mode = "status"` |

## 4. 集成独立 CLI

RTK、CodeGraph 只用命令行，不安装 Pi 插件、不注册 MCP。优先复用本机命令，缺失时按各自官方文档安装到 PATH。

| 工具 | 平台 | 验证 |
|---|---|---|
| RTK | Linux/macOS；Windows 不安装、不调用 | `rtk --version` |
| CodeGraph | 各平台；不支持当前环境时报告 | `codegraph --version` |

在全局 `AGENTS.md` 写入：

- Linux/macOS 优先使用 `rtk git status`、`rtk ls` 等受支持命令；不支持时用原生命令。Windows 使用 pwsh。
- 结构查询使用 `codegraph query "符号"`、`codegraph explore "问题"`、`codegraph node "符号"`；调用关系使用 `callers`、`callees`，影响分析使用 `impact` 子命令。
- CodeGraph 在目标项目根目录运行：无索引时 `codegraph init`，代码变化后 `codegraph sync`；不要在主目录初始化。索引不足时再搜索或读取源码。

配置阶段不为无关项目建索引。

## 5. Pi 设置

编辑 `settings.json`，保留安装命令写入的包清单，仅调整：

| 字段 | 值 |
|---|---|
| `theme` | `"Codex"`，完成第 6 节后启用 |
| `quietStartup`, `showHardwareCursor` | `true` |
| `images.blockImages` | `true` |
| `terminal.clearOnShrink`, `terminal.showTerminalProgress` | `true` |
| `tuiMode` | `"fullscreen"` |
| `editorPaddingX` | `1` |
| `hideThinkingBlock`, `showCacheMissNotices`, `collapseChangelog` | `true` |
| `defaultProjectTrust` | `"always"`；会自动信任并加载项目代码，须说明风险并获使用者确认，否则保持默认 |

不指定模型，其余设置沿用默认值。Windows 设置 `defaultTools = ["read", "powershell", "edit", "write"]`，确认 Pi 支持该工具并实际调用 `pwsh`；Linux/macOS 不设置 `defaultTools`。

## 6. Codex 主题

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

## 7. Playwright MCP

在配置目录的 `npm/` 下执行 `bun add @playwright/mcp`，保留其他依赖。确认 Chrome 已安装，定位 `node_modules/@playwright/mcp/cli.js` 的绝对路径。

编辑 `mcp.json`：

| 路径 | 值 |
|---|---|
| `settings.mcpFooterStatus` | `"full"` |
| `mcpServers.playwright.command` | 本机 Bun 绝对路径 |
| `mcpServers.playwright.args` | CLI 文件绝对路径，随后为 `--browser`, `chrome`, `--isolated`, `--headless`, `--caps`, `devtools` |
| `mcpServers.playwright.lifecycle` | `"lazy"` |
| `mcpServers.playwright.directTools` | `false` |

## 8. 验证

运行 `pi update --extensions`，重启 Pi 后检查：

- `pi list`：仅包含目标插件，无 RTK/CodeGraph 插件或重复加载。
- CLI：CodeGraph 可执行；Linux/macOS 的 RTK 可运行；Windows 终端实际为 pwsh。
- 界面：Codex 主题、Pi 设置和用量选项生效，无启动错误。
- MCP：Playwright 能发现工具并打开测试页面。
- 清理：旧配置无残留，登录信息仍在，未创建备份。

沿用已有登录，仅缺失或失效时提示 `/login`、`/multilogin`。简报安装、清理和验证结果，单列阻塞项，不把未验证项记为成功。
