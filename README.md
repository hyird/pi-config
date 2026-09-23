# Pi 个人配置

这是当前用户的 Pi 配置快照，不是安装脚本或插件。把下面这段话交给 LLM：

```text
请按 https://raw.githubusercontent.com/hyird/pi-config/main/README.md 安装我的 Pi 配置。
从 https://github.com/hyird/pi-config 获取仓库文件，将 README 列出的每个文件直接覆盖到当前用户实际使用的 Pi 配置目录对应路径。不要合并、挑选字段或保留这些目标文件的原有内容；不要删除未列出的文件。不要备份。完成后运行 pi update --extensions，重启 Pi 并报告结果。不要上传或打印本机凭据。
```

## 覆盖范围

配置目录：优先使用启动 Pi 时的 `PI_CODING_AGENT_DIR`，否则为当前用户的 `~/.pi/agent`（Windows 通常是 `%USERPROFILE%\.pi\agent`）。确认是当前用户实际使用的目录，不要写进 Pi 程序安装目录或项目 `.pi`。

| 仓库文件 | 目标（相对配置目录） |
|---|---|
| `settings.json` | `settings.json` |
| `mcp.json` | `mcp.json` |
| `themes/Codex.json` | `themes/Codex.json` |

对表中每个文件 **无条件整文件覆盖**，不做字段级合并、不备份；目标不存在就创建。`models.json` 不在同步范围内，保留本机原文件。只覆盖表中列出的文件，其他文件不清理、不改动。完成覆盖后运行 `pi update --extensions`，重启 Pi，使插件和主题生效。核对 `pi list`、`Codex` 主题及 MCP 连接；如果某项不能运行，报告错误，不擅自更改仓库配置。

`settings.json` 中的 `defaultProjectTrust: "always"` 会自动信任项目并加载其代码；覆盖即接受这一行为，只在可信项目目录运行 Pi。MCP 配置要求本机有 Bun 和 Chrome；仓库不包含这些程序。

**不要上传/覆盖** `auth.json`、`accounts.json`、会话、插件凭据、缓存、`npm/`、`git/`、`bin/` 等本机数据。仓库内的配置不得包含密钥、token、cookie 或私钥；今后增加文件时先检查是否有秘密。
