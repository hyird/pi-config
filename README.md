# Pi 配置

把这句话交给 AI：

```text
请按这份说明安装插件并配置我的 Pi：https://raw.githubusercontent.com/hyird/pi-config/main/README.md
```

## 1. 准备

先自行查找本机正在使用的 Pi：Linux/macOS 使用 `command -v pi`，Windows 使用 `Get-Command pi`，必要时检查启动器、符号链接和安装记录，确认实际程序路径与版本。同时检查 Bun、Git 和操作系统。

Pi 程序安装位置与用户配置目录不是同一个目录。根据实际启动环境中的 `PI_CODING_AGENT_DIR` 确认配置目录；未设置时使用当前用户主目录下的 `.pi/agent`。检查启动器是否覆盖该环境变量，有多个安装或无法确定当前配置目录时先询问，不猜测盘符或路径。

下文的文件路径均相对于已确认的 **Pi 配置目录**。本文是配置说明，不是 Pi 插件。

### 先清理，再配置（不备份）

先清理旧的本地配置，仅保留登录信息，再执行后续安装和配置。此过程不创建备份，删除的配置不能靠本流程恢复。

1. 确认没有其他 Pi 实例正在写入配置。先通过 `pi list` 获取旧插件清单，通过 `pi remove <安装来源>` 移除旧插件；卸载前确认不会删除其账号存储。
2. 保留 `auth.json`、`multiprovider-auth.json` 及其他插件/MCP 的登录信息、API key、token、cookie、私钥，不退出登录、不撤销授权，不将秘密打印到对话或上传。不能确定文件是否包含登录信息时先询问。
3. 清理旧 `settings.json`、`models.json`、`mcp.json`、全局 `AGENTS.md`、`SYSTEM.md`、`APPEND_SYSTEM.md`，以及用户级 `extensions/`、`agents/`、`prompts/`、`themes/`、`skills/` 中的旧配置和自定义资源。若其中混有凭据，先在本机保留对应登录字段或存储；无法安全分离时停止该项清理并报告，不删除凭据。
4. 不整目录删除 Pi 配置目录，不递归跟随符号链接。会话、项目文件、程序安装和运行依赖不属于配置清理范围；不手动清空 `npm/`、`git/`、`bin/`。插件资源由 Pi 包管理命令处理。
5. 从清理后的状态按下文重建，不合并旧的非登录配置。

## 2. 安装插件

Pi 插件统一通过本机已确认的 Pi 命令管理：

- 查看已安装插件：`pi list`
- 安装缺少的插件：`pi install <安装来源>`
- 更新插件：`pi update --extensions`
- 移除多余插件：`pi remove <安装来源>`

已有插件不重复安装。不用 `bun add`、`npm install` 或手动复制源码来代替 Pi 插件安装，也不只往 `settings.json` 写包名。若 `pi` 不在 PATH，使用已找到的 Pi 启动器路径。Bun 仅用于单独的 JavaScript 依赖，例如下文的 Playwright MCP；不要卸载其他程序依赖的 Node。

| 安装来源 | 用途与配置 |
|---|---|
| `npm:@juicesharp/rpiv-ask-user-question` | 结构化提问 |
| `npm:pi-goal` | `/goal` 长任务管理 |
| `npm:@vndv/pi-codegraph` | 代码结构分析；需要索引的项目单独执行 `codegraph init`，不复制其他项目的索引 |
| `npm:pi-mcp-adapter` | MCP 接入，按第 5 节配置 |
| `npm:pi-multiprovider` | 多账号管理，用户在本机通过 `/multilogin` 登录 |
| `npm:@monotykamary/pi-better-openai` | OpenAI 用量与状态栏，按下表配置 |
| `npm:@monotykamary/pi-better-grok` | Grok 用量与状态栏，按下表配置 |
| `git:github.com/hyird/pi-better-opencode-go` | OpenCode Go 用量，按下表配置 |
| `npm:@bacnh85/pi-rtk` | Bash 命令优化；确保 PATH 中有 RTK ≥ 0.23.0，推荐 ≥ 0.46.0，使用 `/rtk status` 验证；不改写 PowerShell |
| `npm:@bacnh85/pi-subagent` | 子代理 |

包不钉版本。安装前检查插件与本机 Pi 的版本兼容性；不兼容时报告，不静默换包。

三个用量插件的本地选项：

| 配置目录中 `extensions/` 下的文件 | 设置 |
|---|---|
| `pi-better-openai.json` | `usage.autoRedeemBankedResets = false`、`footer.mode = "status"` |
| `pi-better-grok.json` | `footer.mode = "status"` |
| `opencode-go-usage.json` | `footer.mode = "status"` |

按插件已安装版本的文档编辑这些选项。

## 3. Pi 本身的设置

配置目录中的 `settings.json` 只设置以下非默认值，并保留 Pi 安装命令写入的包清单；其余使用 Pi 默认值：

| 字段 | 目标值 |
|---|---|
| `theme` | `"Codex"`，先完成第 4 节 |
| `quietStartup` | `true` |
| `showHardwareCursor` | `true` |
| `images.blockImages` | `true`（不向模型发送图片） |
| `terminal.clearOnShrink` | `true` |
| `terminal.showTerminalProgress` | `true` |
| `tuiMode` | `"fullscreen"` |
| `editorPaddingX` | `1` |
| `hideThinkingBlock` | `true` |
| `showCacheMissNotices` | `true` |
| `collapseChangelog` | `true` |
| `defaultProjectTrust` | 原配置为 `"always"`；会自动信任项目并允许加载项目代码，说明风险并经使用者确认后设置，否则使用 Pi 默认值 |

不指定默认模型或模型列表，由使用者自行选择。

Windows 在当前 Pi 支持时将 `defaultTools` 设为 `["read", "powershell", "edit", "write"]`；Linux/macOS 使用默认 Bash 工具，去掉迁移遗留的 PowerShell 专用设置。命令语法跟随实际系统；不要硬编码作者的主目录。

## 4. Codex 主题

Codex 是自定义主题，不是 Pi 内置主题。在配置目录中创建或编辑 `themes/Codex.json`，`name` 为 `Codex`；按照当前 Pi 主题 schema 设置下列颜色。随后选择 `theme = "Codex"`。

每一行列出的 token 都使用该行颜色：

| `colors` 中的 token | 颜色 |
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

HTML 导出颜色：`export.pageBg = "#0d0d0f"`、`export.cardBg = "#17171b"`、`export.infoBg = "#222228"`。检查当前版本必填 token 是否齐全，不能只写主题名称而没有主题定义。

## 5. Playwright MCP

`@playwright/mcp` 是独立 MCP 服务，不是 Pi 插件；`pi-mcp-adapter` 则通过上面的 `pi install` 安装。使用 Bun 安装 `@playwright/mcp` 到配置目录下 `npm/` 的依赖中，例如进入该目录后使用 Bun 添加该包。保留该目录其他依赖，确认 `node_modules/@playwright/mcp/cli.js` 存在。检查 Chrome 是否已安装；缺失时向使用者说明并安装所需浏览器。

通过 `pi-mcp-adapter` 编辑配置目录中的 `mcp.json`：

| 路径 | 值 |
|---|---|
| `settings.mcpFooterStatus` | `"full"` |
| `mcpServers.playwright.command` | 本机 Bun 可执行文件的绝对路径 |
| `mcpServers.playwright.args` | CLI 文件绝对路径，随后依次为 `--browser`, `chrome`, `--isolated`, `--headless`, `--caps`, `devtools` |
| `mcpServers.playwright.lifecycle` | `"lazy"` |
| `mcpServers.playwright.directTools` | `false` |

路径按使用者系统生成，不留占位符。启动并检查 MCP 工具发现，确认能够打开测试页面；若本机 Bun、Chrome 或包版本组合不兼容，报告具体错误，不能只写好 JSON 就算完成。

## 6. 验收

- 确认旧配置已先行清理，没有旧插件、重复加载项或旧自定义资源残留；凭据保留，未创建备份。
- 不把保留的登录信息当成多余配置删除；未能安全清理的项目单独报告。
- 运行 `pi update --extensions`，重启 Pi；用 `pi list`、`/rtk status` 及 MCP 状态检查安装与加载结果。核对主题、界面和插件用量选项，保证没有重复工具或启动错误。
- 用量查询使用保留的登录状态；仅在缺失或失效时让使用者通过 `/login`、`/multilogin` 完成登录。最后报告已完成、已清理、待登录及不兼容项。不要把未验证的项目写成成功。
