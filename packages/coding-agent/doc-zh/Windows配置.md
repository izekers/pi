---
description: 在 Windows 上配置 Bash Shell
---

# Windows 配置

Pi 需要 Windows 上的 bash shell。检查位置（按顺序）：

1. `~/.pi/agent/settings.json` 中的自定义路径
2. Git Bash（`C:\Program Files\Git\bin\bash.exe`）
3. PATH 上的 `bash.exe`（Cygwin、MSYS2、WSL）

对于大多数用户，[Git for Windows](https://git-scm.com/download/win) 已足够。

## 自定义 Shell 路径

```json
{
  "shellPath": "C:\\cygwin64\\bin\\bash.exe"
}
```