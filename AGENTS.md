# AGENTS.md

本文件供 opencode 在新机器上读取并自动执行环境部署。请按顺序完成下方检查与安装步骤，每步均为幂等操作（已满足则跳过）。

## 仓库性质

- 本仓库是 [KevinSilvester/wezterm-config](https://github.com/KevinSilvester/wezterm-config) 的**个人 fork**，个人定制全部在 `personal` 分支上
- 远程布局：`origin` = 上游原仓库（用于同步更新），`mine` = fork（`https://github.com/ntexplorer/wezterm-config`，用于推送）
- 同步上游：`git fetch origin && git merge origin/master`，冲突时以个人定制为准保留 `config/bindings.lua`、`config/launch.lua` 的本地改动

## 新机器 Bootstrap

前提：用户已自行安装 node/npm、opencode、wezterm、git。以下由你（opencode）完成：

1. **部署本配置**（若 `~/.config/wezterm` 尚非本 fork）：
   ```powershell
   git clone -b personal https://github.com/ntexplorer/wezterm-config.git "$env:USERPROFILE\.config\wezterm"
   ```
   若目录已存在且是 Kevin 原仓库，可将 remote 改指 fork 并 `git fetch && git checkout personal`。

2. **安装字体**（配置使用 JetBrainsMono Nerd Font，缺失会字体回退）：
   ```powershell
   winget install DEVCOM.JetBrainsMonoNerdFont
   ```
   > 注：旧 ID `nerd-fonts.JetBrainsMonoNerdFont` 已从 winget 源下架（2026-09 实测找不到），若 `DEVCOM.JetBrainsMonoNerdFont` 也失效，从 https://github.com/ryanoasis/nerd-fonts/releases 手动下载安装。

3. **安装 cliamp**（TUI 音乐播放器）：
   - 从 https://github.com/bjarneo/cliamp/releases 下载最新 **`cliamp-windows-amd64.zip`**（不要下裸 `cliamp-windows-amd64.exe`）
   - 解压**全部内容**（`cliamp.exe` + `libmpg123-0.dll` 等 6 个编解码 DLL）到 `$env:USERPROFILE\.local\bin\`（目录不存在则创建；配置中按 `~\.local\bin\cliamp.exe` 全路径引用，DLL 必须与 exe 同目录）
   - 下载后用 release 的 `checksums.txt` 做 SHA256 校验
   > 踩坑记录（2026-09）：裸 exe 动态链接 libmpg123-0.dll 等编解码库，缺失会弹「系统错误：找不到 libmpg123-0.dll」。官方 README 明确 Windows 应使用 zip 版。

4. **追加 PATH**（幂等，已含则跳过）：
   ```powershell
   $p = [Environment]::GetEnvironmentVariable('Path','User')
   if ($p -notlike '*.local\bin*') { [Environment]::SetEnvironmentVariable('Path', $p.TrimEnd(';') + ';' + "$env:USERPROFILE\.local\bin", 'User') }
   ```

5. **安装 sleev**：
   ```powershell
   npm i -g sleev
   ```

6. **安装 ffmpeg 与 yt-dlp**（cliamp 可选运行时依赖：ffmpeg 补 AAC/ALAC/Opus/WMA 格式，yt-dlp 补 YouTube/B站/SoundCloud 等流媒体源）：
   ```powershell
   winget install Gyan.FFmpeg
   winget install yt-dlp.yt-dlp
   ```
   > 注：yt-dlp 会自动带上 Deno 与 yt-dlp.FFmpeg 两个依赖，属正常现象。

## ⚠️ opencode Provider 配置坑位

用户订阅的是**智谱 GLM 编码套餐**（provider：`zhipuai-coding-plan`，上游 `https://open.bigmodel.cn/api/coding/paas/v4`）。sleev 1.8.0 的 `harness opencode` 自动接线列表**只支持 `zai-coding-plan` 不支持智谱**，且网关对 `sleev-provider` 头有白名单（乱填报 `Unknown sleev-provider header`）。

**正确做法：走 sleev 的 custom provider 机制**（官方文档 `/docs/config/custom-providers`），`~/.config/opencode/opencode.jsonc` 完整配置（2026-09-03 实测验证 200 OK）：

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "compaction": { "prune": false },
  "provider": {
    "zhipuai-coding-plan": {
      "options": {
        "headers": {
          "sleev-harness": "opencode",
          "sleev-base-url": "https://open.bigmodel.cn/api/coding/paas/v4"
        },
        "baseURL": "http://127.0.0.1:17321"
      }
    }
  }
}
```

要点：

- **用 `sleev-base-url` 头指定上游，不要发 `sleev-provider` 头**（custom provider 场景下发它会撞白名单）
- `compaction.prune: false` 必须保留——禁用 opencode 自带裁剪，避免与 sleev 的压缩冲突（sleev 官方 enable 时也会写这一项）
- API 密钥留在 `~/.local/share/opencode/auth.json`（`/connect` 命令存储），网关透传 Authorization，**任何密钥不得写入本仓库**
- **sleev 未运行时 opencode 无法发起任何模型请求**——遇到连接错误先检查 sleev（网关装成计划任务常驻，一般无需手动启动）
- 验证方法：网关日志 `%LOCALAPPDATA%\sleev\debug-logs\server\gateway.err.log` 应出现 `POST open.bigmodel.cn/... OK`；`sleev usage --json` 可查用量

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
- 改动提交到 `personal` 分支并 `git push mine personal`，不要动 `master`
