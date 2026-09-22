# Pi 配置

把这句话交给 AI：

```text
请按这份说明安装插件并配置我的 Pi：https://raw.githubusercontent.com/hyird/pi-config/main/README.md
```

## 1. 准备

配置目录记为 `D`：使用 `PI_CODING_AGENT_DIR`，未设置时为 `~/.pi/agent`。检查 Pi、Bun、Git 和操作系统；先备份将修改、删除的配置，再按下面的目标逐项编辑。本文是配置说明，不是 Pi 插件。

## 2. 安装插件

对缺少的条目执行 `pi install <安装来源>`，已有条目不重复安装；使用 Bun 管理 JavaScript 依赖，不卸载其他程序依赖的 Node。Pi 包管理命令是否调用 Bun，应按当前版本确认并配置，不能只因本机有 Bun 就假定已经使用它。

| 安装来源 | 用途与配置 |
|---|---|
| `npm:@juicesharp/rpiv-ask-user-question` | 结构化提问，使用默认配置 |
| `npm:pi-goal` | `/goal` 长任务管理，使用默认配置 |
| `npm:@vndv/pi-codegraph` | 代码结构分析；需要索引的项目单独执行 `codegraph init`，不复制其他项目的索引 |
| `npm:pi-mcp-adapter` | MCP 接入，按第 5 节配置 |
| `npm:pi-multiprovider` | 多账号管理，用户在本机通过 `/multilogin` 登录 |
| `npm:@monotykamary/pi-better-openai` | OpenAI 用量与状态栏，按下表配置 |
| `npm:@monotykamary/pi-better-grok` | Grok 用量与状态栏，按下表配置 |
| `git:github.com/hyird/pi-better-opencode-go` | OpenCode Go 用量，按下表配置 |
| `npm:@bacnh85/pi-rtk` | Bash 命令优化；确保 PATH 中有 RTK ≥ 0.23.0，推荐 ≥ 0.46.0，使用 `/rtk status` 验证；不改写 PowerShell |
| `npm:@bacnh85/pi-subagent` | 子代理，使用默认配置 |

包不钉版本。安装前检查插件与本机 Pi 的版本兼容性；不兼容时报告，不静默换包。

三个用量插件的本地选项：

| `D/extensions/` 下的文件 | 设置 |
|---|---|
| `pi-better-openai.json` | `usage.enabled = true`、`usage.refreshIntervalMs = 60000`、`usage.autoRedeemBankedResets = false`、`footer.mode = "status"` |
| `pi-better-grok.json` | `usage.enabled = true`、`usage.refreshIntervalMs = 60000`、`footer.mode = "status"` |
| `opencode-go-usage.json` | `usage.enabled = true`、`usage.refreshIntervalMs = 60000`、`footer.mode = "status"` |

按插件已安装版本的文档编辑这些选项。

## 3. Pi 本身的设置

逐项修改 `D/settings.json`：

| 字段 | 目标值 |
|---|---|
| `packages` | 第 2 节的十个插件，去重 |
| `transport` | `"auto"` |
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
| `treeFilterMode` | `"default"` |
| `defaultProjectTrust` | 原配置为 `"always"`；会自动信任项目并允许加载项目代码，说明风险并经使用者确认后设置，否则保留原值 |

模型相关设置保持使用者原样。`lastChangelogVersion` 是机器运行状态，保留本机值。

Windows 在当前 Pi 支持时将 `defaultTools` 设为 `["read", "powershell", "edit", "write"]`；Linux/macOS 使用默认 Bash 工具，去掉迁移遗留的 PowerShell 专用设置。命令语法跟随实际系统；不要硬编码作者的主目录。

## 4. Codex 主题

Codex 是自定义主题，不是 Pi 内置主题。创建或编辑 `D/themes/Codex.json`，`name` 为 `Codex`；按照当前 Pi 主题 schema 设置下列颜色。随后选择 `theme = "Codex"`。

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

安装 `@playwright/mcp` 到 `D/npm` 的依赖中，例如进入该目录后使用 Bun 添加该包。保留该目录其他依赖，确认 `node_modules/@playwright/mcp/cli.js` 存在。检查 Chrome 是否已安装；缺失时向使用者说明并安装所需浏览器。

通过 `pi-mcp-adapter` 配置 `D/mcp.json`：

| 路径 | 值 |
|---|---|
| `settings.mcpFooterStatus` | `"full"` |
| `mcpServers.playwright.command` | 本机 Bun 可执行文件的绝对路径 |
| `mcpServers.playwright.args` | CLI 文件绝对路径，随后依次为 `--browser`, `chrome`, `--isolated`, `--headless`, `--caps`, `devtools` |
| `mcpServers.playwright.lifecycle` | `"lazy"` |
| `mcpServers.playwright.directTools` | `false` |

路径按使用者系统生成，不留占位符。启动并检查 MCP 工具发现，确认能够打开测试页面；若本机 Bun、Chrome 或包版本组合不兼容，报告具体错误，不能只写好 JSON 就算完成。

## 6. 独立安装的 Skills

将 `https://github.com/badlogic/pi-skills.git` 克隆到 `D/skills/pi-skills`；已存在就保留，不 reset。由 Pi 自动发现，不额外添加重复的 skills 路径或 package 项。

包含浏览器、搜索、Google Calendar/Drive/Gmail、转录、VS Code、YouTube 等技能。依赖按各自 `SKILL.md` 按需安装；平台不支持的技能不强行安装，账号由使用者在本机授权。后续更新使用 `git pull --ff-only`。

## 7. 清理和验收

- 按上述目标列出多余包、旧扩展和配置项；备份并确认清理清单后执行。
- 多余包通过 `pi remove <来源>` 移除；不要整目录删除 `npm/`、`git/`。
- 清理重复加载项、多余 MCP server、主题和过期配置；有独立用途或无法判定的条目先询问，不用“仓库没列出”代替依赖检查。模型相关配置不参与清理。
- 不动凭据、会话、项目文件、缓存、trust 记录、机器 `AGENTS.md`；不读取或上传 `auth.json`、`multiprovider-auth.json`、API key、token、cookie、SSH 私钥。遇到混有秘密的配置，仅在本机脱敏处理。
- 运行 `pi update --extensions`，重启 Pi；用 `pi list`、`/rtk status` 及 MCP 状态检查安装与加载结果。核对主题、界面和插件用量选项，保证没有重复工具或启动错误。
- 用量查询需要登录；让使用者通过 `/login`、`/multilogin` 完成。最后报告已完成、已清理、待登录及不兼容项。不要把未验证的项目写成成功。
