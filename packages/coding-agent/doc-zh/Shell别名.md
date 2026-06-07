---
description: 配置 Shell 别名以在 Pi 中生效
---

# Shell 别名

Pi 以非交互模式运行 bash（`bash -c`），默认不会展开别名。

要启用你的 Shell 别名，请在 `~/.pi/agent/settings.json` 中添加：

```json
{
  "shellCommandPrefix": "shopt -s expand_aliases\neval \"$(grep '^alias ' ~/.zshrc)\""
}
```

调整路径（`~/.zshrc`、`~/.bashrc` 等）以匹配你的 Shell 配置。