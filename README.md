# Pi 个人配置

这是 Pi 的配置选项与主题，不含模型配置或凭据。把下面这段话交给 LLM：

```text
请按 https://raw.githubusercontent.com/hyird/pi-config/main/README.md 的规则，从仓库同步我的 Pi 配置。保留本机模型配置和凭据；完成后更新插件、重启 Pi 并报告结果。
```

## 目标目录

优先使用启动 Pi 时的 `PI_CODING_AGENT_DIR`，否则使用当前用户的 `~/.pi/agent`（Windows 通常是 `%USERPROFILE%\.pi\agent`）。确认是当前用户实际使用的配置目录，不要写入 Pi 程序安装目录或项目 `.pi`。

## 覆盖规则

| 仓库文件 | 目标（相对配置目录） | 操作 |
|---|---|---|
| `settings.json` | `settings.json` | 按键递归覆盖 |
| `mcp.json` | `mcp.json` | 按键递归覆盖 |
| `themes/Codex.json` | `themes/Codex.json` | 整文件覆盖，不合并 |

按键递归覆盖：仓库中出现的选项，直接替换本机对应值；嵌套对象逐层处理；数组整体替换；仓库中未出现的本机选项保持不变（下节的插件/skills 清理除外）。如果目标文件不存在，则创建。不备份。

## 清理多余插件和 skills

- `settings.json` 中的 `packages` 数组是**唯一允许的包清单**，整体替换。覆盖前用 `pi list` 查看当前用户级安装，对不在仓库清单中的用户级包运行 `pi remove <来源>`；覆盖后运行 `pi update --extensions` 安装/更新清单内的包。包内附带的扩展和 skills 随包处理，不手动删除包缓存或依赖。
- 仓库没有配置独立的扩展或 skills：删除本机 `settings.json` 中的 `extensions`、`skills` 字段（若存在），清理当前用户配置目录下 `extensions/`、`skills/` 中的独立资源。只清理这两个目录中的内容，不动 `npm/`、`git/`、会话、凭据或项目级资源；发现符号链接或指向其他目录的资源时不要沿链接删除目标。
- 检查 Pi 自动发现的用户级 `~/.agents/skills/`：其中若有专用于 Pi、且不在清单包内的多余 skill，也应移除；若与其他代理共用或用途不明，不删除，报告给用户。
- 项目级 `.pi/`、`.agents/` 的插件和 skills 不属于本仓库清理范围；其他设置仍按前述覆盖规则保留。

**模型相关配置不在同步范围内：**保留本机 `models.json`、`settings.json` 中的 `defaultProvider`、`defaultModel`、`enabledModels`、`modelThinkingLevels` 等模型选项以及其他未列出的设置；不要把历史提交中的模型配置重新写入本机。

完成后运行 `pi update --extensions`，重启 Pi，使插件和主题生效。核对 `pi list`、`Codex` 主题及 MCP 连接；失败时报告错误，不擅自修改仓库配置。

`settings.json` 中的 `defaultProjectTrust: "always"` 会自动信任项目并加载其代码；只在可信项目目录运行 Pi。MCP 配置要求本机有 Bun 和 Chrome；仓库不包含这些程序。

**不要上传/覆盖** `auth.json`、`accounts.json`、会话、插件凭据、缓存、`npm/`、`git/`、`bin/` 等本机数据；插件/skills 清理仅限上述范围。仓库配置不得包含密钥、token、cookie 或私钥。
