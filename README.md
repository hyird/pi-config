# Pi 配置提示词

请根据这份说明配置使用者本机的 Pi。不要上传、复制或原样覆盖本地配置文件；使用 Pi 的配置命令和各插件自己的配置方式完成操作。

## 安装插件

安装并启用这些 Pi 插件：

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

使用 `pi install` 安装缺少的插件，安装后运行 `pi update --extensions`。不要重复安装已有插件，不要把本仓库安装成插件。

## 配置目标

通过 Pi 命令、插件文档和用户本机配置完成以下设置：

- 默认 provider：`xai`
- 默认 model：`grok-4.7`
- 启用模型：
  - `xai/grok-4.7`
  - `openai-codex/gpt-6-astra`
  - `openai-codex/gpt-5.6-sol`
  - `openai-codex/gpt-5.6-terra`
  - `openai-codex/gpt-5.6-luna`
  - `opencode-go/deepseek-v4.1-flash`
- 使用 Codex 主题。
- Windows 使用 PowerShell 工具；Linux/macOS 使用 Bash。
- 配置 Playwright MCP，使用本机 Bun 和实际的 `@playwright/mcp` 路径。
- 安装 `pi-skills` 到 Pi 的 skills 目录；已有检出不要重置。

## 插件配置

- `pi-goal`：启用 `/goal`，需要长期任务时使用它。
- `pi-codegraph`：项目需要代码结构分析时初始化并使用 CodeGraph。
- `pi-multiprovider`：让用户在本机运行 `/multilogin` 管理多个 provider 账号。
- `pi-better-openai`、`pi-better-grok`：按插件文档配置状态栏和用量显示，不复制其他机器的配置。
- `pi-better-opencode-go`：配置 `/go-usage`；账号由本机 `pi-multiprovider` 或 `/multilogin` 提供。
- `pi-rtk`：启用 RTK 命令优化，确认本机已安装并可执行 `rtk`。
- `pi-subagent`：启用 subagent，并按当前任务选择 explorer、researcher、reviewer、tester、worker 等代理。
- `rpiv-ask-user-question`：需要用户选择时使用结构化提问。

使用者已有配置应智能合并；清理不再使用的插件、重复扩展和无效配置，但不要破坏仍在使用的设置。

## 安全

不要读取、复制、上传或提交：

- `auth.json`
- `multiprovider-auth.json`
- API key、token、cookie、SSH 私钥
- 任何其他凭据

不要使用 root，不要删除 sessions、npm、git、skills、bin、缓存、trust 文件或本机 `AGENTS.md`。登录只让使用者在本机执行 `/login` 和 `/multilogin`。

配置完成后报告安装、修改和清理的内容，并提示使用者重启 Pi 或执行 `/reload`。