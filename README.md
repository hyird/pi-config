# Pi 个人配置

把这段话交给 LLM：

```text
请按 https://raw.githubusercontent.com/hyird/pi-config/main/README.md 从仓库同步我的 Pi 配置。保留本机模型配置、凭据及其他未列出的设置；按文档清理多余插件，完成后核对结果并提示我重启 Pi。
```

## 同步范围

确认 Pi 实际使用的用户配置目录：优先使用启动时的 `PI_CODING_AGENT_DIR`，否则为 `~/.pi/agent`（Windows 通常为 `%USERPROFILE%\.pi\agent`）。不要写入 Pi 安装目录或项目级 `.pi/`。

| 仓库文件 | 用户配置目录下的目标 | 规则 |
|---|---|---|
| `settings.json` | `settings.json` | 按键递归覆盖 |
| `mcp.json` | `mcp.json` | 按键递归覆盖 |
| `themes/Codex.json` | `themes/Codex.json` | 整文件覆盖 |

从仓库获取**当前版本**文件。递归覆盖时，对象逐层合并，数组整体替换；仓库未列出的键保留（下述 `extensions`、`skills` 除外），目标文件不存在则创建。**模型配置不参与同步**：保留 `models.json`、`defaultProvider`、`defaultModel`、`enabledModels`、`modelThinkingLevels` 等本机选项；不要从历史提交恢复它们。不备份。

## 插件与 skills

1. 仓库 `settings.json` 的 `packages` 是唯一允许的**用户级**包清单。先用 `pi list` 区分用户级与项目级包，对清单外的用户级包执行 `pi remove <来源>`（不要加 `--local`）；再按上表覆盖设置，执行 `pi update --extensions` 安装或更新清单中的包。包附带的资源随包管理，不手动清理 `npm/`、`git/` 或依赖。
2. 删除用户级 `settings.json` 中独立的 `extensions`、`skills` 键，清空用户配置目录下 `extensions/`、`skills/` 中的独立资源；仅限这两处。遇到符号链接或指向外部的路径，不跟随、不删除目标，报告用户。
3. 检查用户级 `~/.agents/skills/`：只移除能确认专用于 Pi 且不属于清单包的多余 skill；共用或用途不明的保留并报告。项目级 `.pi/`、`.agents/` 不动。

## 完成检查与安全边界

核对 `pi list` 的用户级包、`Codex` 主题和 MCP 连接；失败则报告原因，不擅自改仓库配置。提示用户重启 Pi 使配置生效；若能安全地重启，也须报告结果。

不得上传或覆盖 `auth.json`、`accounts.json`、会话、插件凭据、缓存及 `npm/`、`git/`、`bin/` 等本机数据；仓库不得包含密钥、token、cookie 或私钥。`defaultProjectTrust: "always"` 会自动信任并加载项目代码，只在可信目录运行 Pi。MCP 配置需要本机安装 Bun 和 Chrome。
