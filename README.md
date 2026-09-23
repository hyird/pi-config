# Pi 个人配置

这是 Pi 的配置选项与主题，不含模型配置或凭据。把下面这段话交给 LLM：

```text
请按 https://raw.githubusercontent.com/hyird/pi-config/main/README.md 配置我的 Pi。
从 https://github.com/hyird/pi-config 获取仓库文件：对于 settings.json 和 mcp.json，只覆盖仓库文件中明确列出的选项，保留本机其他选项；themes/Codex.json 整文件覆盖。不要更改模型配置或凭据，不备份。完成后运行 pi update --extensions，重启 Pi 并报告结果。
```

## 目标目录

优先使用启动 Pi 时的 `PI_CODING_AGENT_DIR`，否则使用当前用户的 `~/.pi/agent`（Windows 通常是 `%USERPROFILE%\.pi\agent`）。确认是当前用户实际使用的配置目录，不要写入 Pi 程序安装目录或项目 `.pi`。

## 覆盖规则

| 仓库文件 | 目标（相对配置目录） | 操作 |
|---|---|---|
| `settings.json` | `settings.json` | 按键递归覆盖 |
| `mcp.json` | `mcp.json` | 按键递归覆盖 |
| `themes/Codex.json` | `themes/Codex.json` | 整文件覆盖，不合并 |

按键递归覆盖：仓库中出现的选项，直接替换本机对应值；嵌套对象逐层处理；数组整体替换；仓库中未出现的本机选项保持不变。如果目标文件不存在，则创建。不要删除未列出的文件，也不要备份。`settings.json` 中的 `packages` 数组会整体替换：请据此同步插件，不要将本机其他旧包自动并入该数组。

**模型相关配置不在同步范围内：**保留本机 `models.json`、`settings.json` 中的 `defaultProvider`、`defaultModel`、`enabledModels`、`modelThinkingLevels` 等模型选项以及其他未列出的设置；不要把历史提交中的模型配置重新写入本机。

完成后运行 `pi update --extensions`，重启 Pi，使插件和主题生效。核对 `pi list`、`Codex` 主题及 MCP 连接；失败时报告错误，不擅自修改仓库配置。

`settings.json` 中的 `defaultProjectTrust: "always"` 会自动信任项目并加载其代码；只在可信项目目录运行 Pi。MCP 配置要求本机有 Bun 和 Chrome；仓库不包含这些程序。

**不要上传/覆盖** `auth.json`、`accounts.json`、会话、插件凭据、缓存、`npm/`、`git/`、`bin/` 等本机数据。仓库配置不得包含密钥、token、cookie 或私钥。
