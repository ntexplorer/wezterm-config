# AGENTS.md

本文件供 opencode 读取，说明本 fork（wezterm 配置）的维护方式与个人定制键位。

> **新机器完整环境部署**（wezterm + opencode + sleev + cliamp + skills，含全部踩坑记录）见私有主手册仓库 `ntexplorer/ai-dev-setup` 的 `AGENTS.md`，本仓库仅是其 wezterm 步骤的落地仓库。部署本配置：
> ```powershell
> git clone -b personal https://github.com/ntexplorer/wezterm-config.git "$env:USERPROFILE\.config\wezterm"
> ```

## 仓库性质

- 本仓库是 [KevinSilvester/wezterm-config](https://github.com/KevinSilvester/wezterm-config) 的**个人 fork**，个人定制全部在 `personal` 分支上
- 远程布局：`origin` = 上游原仓库（用于同步更新），`mine` = fork（`https://github.com/ntexplorer/wezterm-config`，用于推送）
- 同步上游：`git fetch origin && git merge origin/master`，冲突时以个人定制为准保留 `config/bindings.lua`、`config/launch.lua` 的本地改动

## 个人定制键位速查（相对上游的改动，均在 `config/bindings.lua`）

| 键位 | 功能 | 备注 |
|---|---|---|
| `CTRL+SHIFT+h/j/k/l` | 方向切换 pane | 原上游为 `CTRL+ALT`（Windows 热键冲突，已弃用） |
| `CTRL+SHIFT+p` | 显示 pane 编号，按数字切换焦点（PaneSelect Activate） | 新增 |
| `CTRL+ALT+p` | Swap pane（与选中 pane 交换位置，保持焦点） | 上游原有 |
| `CTRL+SHIFT+a` | 三分屏工作区：左 70% 保留当前程序（opencode），右上开 cliamp，右下同路径开 sleev，焦点回左侧 | 新增 |
| `ALT+\` / `CTRL+ALT+\` | 垂直 / 水平分屏 | 上游原有 |
| `CTRL+ALT+Space` | LEADER 前缀（`LEADER+f` 字号、`LEADER+p` 调整 pane 尺寸） | 上游原有 |

## 维护规约

- Lua 代码风格遵循仓库 `.stylua.toml`（缩进 3 空格等）
- commit 风格沿用上游 conventional commits：`type(scope): message`，如 `feat(config.bindings): ...`
- 修改配置后用 `wezterm show-keys` 验证解析无错、键位已注册
  > 显示怪癖：`CTRL|SHIFT+小写字母` 会被折叠显示为大写字母，如 `CTRL H` = `CTRL+SHIFT+h`，不要误判为绑定丢失
- 改动提交到 `personal` 分支并 `git push mine personal`，不要动 `master`
- 本仓库为公开仓库：**不得写入任何密钥、账号、邮箱等私人信息**
