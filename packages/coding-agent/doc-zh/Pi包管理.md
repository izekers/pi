---
description: 通过 npm 或 git 打包和分享扩展、技能、提示词模板和主题
---

> pi 可以帮助你创建 pi 包。请让它打包你的扩展、技能、提示词模板或主题。

# Pi 包管理

Pi 包将扩展、技能、提示词模板和主题打包，以便你可以通过 npm 或 git 进行分享。包可以在 `package.json` 的 `pi` 键下声明资源，也可以使用约定的目录结构。

## 目录

- [安装和管理](#安装和管理)
- [包来源](#包来源)
- [创建 Pi 包](#创建-pi-包)
- [包结构](#包结构)
- [依赖](#依赖)
- [包过滤](#包过滤)
- [启用和禁用资源](#启用和禁用资源)
- [作用域和去重](#作用域和去重)

## 安装和管理

> **安全注意：** Pi 包以完整系统权限运行。扩展执行任意代码，技能可以指示模型执行包括运行可执行文件在内的任何操作。在安装第三方包之前请审查源代码。

```bash
pi install npm:@foo/bar@1.0.0
pi install git:github.com/user/repo@v1
pi install https://github.com/user/repo  # 原始 URL 也可以
pi install /absolute/path/to/package
pi install ./relative/path/to/package

pi remove npm:@foo/bar
pi list                     # 从设置中显示已安装的包
pi update                   # 更新 pi、更新包并协调固定的 git 引用
pi update --extensions      # 仅更新包并协调固定的 git 引用
pi update --self            # 仅更新 pi
pi update --self --force    # 即使是当前版本也重新安装 pi
pi update npm:@foo/bar      # 更新一个包
pi update --extension npm:@foo/bar
```

这些命令管理 pi 包，不是 pi CLI 安装本身。要卸载 pi 本身，请参阅[快速入门](quickstart.md#卸载)。

默认情况下，`install` 和 `remove` 写入用户设置（`~/.pi/agent/settings.json`）。使用 `-l` 改为写入项目设置（`.pi/settings.json`）。项目设置可以与你的团队共享，在项目被信任后，pi 会在启动时自动安装任何缺失的包。

只有在项目被信任时，项目包命令才会读取项目设置。使用 `--approve` 信任项目本地文件以执行一次命令，或使用 `--no-approve` 在执行一次命令时忽略它们。

要试用包而不安装它，请使用 `--extension` 或 `-e`。这会将包安装到当前运行的临时目录：

```bash
pi -e npm:@foo/bar
pi -e git:github.com/user/repo
```

## 包来源

Pi 在设置和 `pi install` 中接受三种来源类型。

### npm

```
npm:@scope/pkg@1.2.3
npm:pkg
```

- 带版本的规格会被固定并在包更新时跳过（`pi update`、`pi update --extensions`）。
- 用户安装放在 `~/.pi/agent/npm/` 下。
- 项目安装放在 `.pi/npm/` 下。
- 在 `settings.json` 中设置 `npmCommand` 以将 npm 包查找和安装操作固定到特定包装命令，例如 `mise` 或 `asdf`。

示例：

```json
{
  "npmCommand": ["mise", "exec", "node@20", "--", "npm"]
}
```

### git

```
git:github.com/user/repo@v1
git:git@github.com:user/repo@v1
https://github.com/user/repo@v1
ssh://git@github.com/user/repo@v1
```

- 没有 `git:` 前缀时，只接受协议 URL（`https://`、`http://`、`ssh://`、`git://`）。
- 有 `git:` 前缀时，接受简写格式，包括 `github.com/user/repo` 和 `git@github.com:user/repo`。
- 支持 HTTPS 和 SSH URL。
- SSH URL 自动使用你配置的 SSH 密钥（尊重 `~/.ssh/config`）。
- 对于非交互式运行（例如 CI），你可以设置 `GIT_TERMINAL_PROMPT=0` 来禁用凭证提示，并设置 `GIT_SSH_COMMAND`（例如 `ssh -o BatchMode=yes -o ConnectTimeout=5`）以快速失败。
- 引用是固定的标签或提交。`pi update` 和 `pi update --extensions` 不会将它们移动到更新的引用，但它们确实会将现有克隆协调到配置的引用。
- 使用 `pi install git:host/user/repo@new-ref` 来更新设置并将现有包移动到新的固定引用。
- 克隆到 `~/.pi/agent/git/<host>/<path>`（全局）或 `.pi/git/<host>/<path>`（项目）。
- 当协调更改检出时，pi 会重置并清理克隆，然后如果存在 `package.json` 则运行 `npm install`。

**SSH 示例：**
```bash
# git@host:path 简写（需要 git: 前缀）
pi install git:git@github.com:user/repo

# ssh:// 协议格式
pi install ssh://git@github.com/user/repo

# 带版本引用
pi install git:git@github.com:user/repo@v1.0.0
```

### 本地路径

```
/absolute/path/to/package
./relative/path/to/package
```

本地路径指向磁盘上的文件或目录，并在不加复制的情况下添加到设置。相对路径相对于它们出现的设置文件进行解析。如果路径是文件，它作为单个扩展加载。如果它是目录，pi 使用包规则加载资源。

## 创建 Pi 包

在 `package.json` 中添加 `pi` 清单或使用约定的目录。包括 `pi-package` 关键字以便发现。

```json
{
  "name": "my-package",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "skills": ["./skills"],
    "prompts": ["./prompts"],
    "themes": ["./themes"]
  }
}
```

路径相对于包根目录。数组支持 glob 模式和 `!排除`。

### 画廊元数据

[包画廊](https://pi.dev/packages) 显示带有 `pi-package` 标签的包。添加 `video` 或 `image` 字段以显示预览：

```json
{
  "name": "my-package",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "video": "https://example.com/demo.mp4",
    "image": "https://example.com/screenshot.png"
  }
}
```

- **video**：仅限 MP4。在桌面上，悬停时自动播放。点击打开全屏播放器。
- **image**：PNG、JPEG、GIF 或 WebP。显示为静态预览。

如果同时设置，video 优先于 image。

## 包结构

### 约定目录

如果没有 `pi` 清单，pi 会从这些目录自动发现资源：

- `extensions/` 加载 `.ts` 和 `.js` 文件
- `skills/` 递归查找 `SKILL.md` 文件夹并加载顶级 `.md` 文件作为技能
- `prompts/` 加载 `.md` 文件
- `themes/` 加载 `.json` 文件

## 依赖

第三方运行时依赖应放在 `package.json` 的 `dependencies` 中。不注册扩展、技能、提示词模板或主题的依赖也应放在 `dependencies` 中。当 pi 从 npm 或 git 安装包时，它会运行 `npm install`，因此这些依赖会自动安装。

Pi 为扩展和技能绑定了核心包。如果你导入其中任何一个，请在 `peerDependencies` 中将它们列为 `"*"` 范围且不要捆绑它们：`@earendil-works/pi-ai`、`@earendil-works/pi-agent-core`、`@earendil-works/pi-coding-agent`、`@earendil-works/pi-tui`、`typebox`。

其他 pi 包必须捆绑在你的 tarball 中。将它们添加到 `dependencies` 和 `bundledDependencies`，然后通过 `node_modules/` 路径引用它们的资源。Pi 使用单独的模块根加载包，因此单独的安装不会冲突或共享模块。

示例：

```json
{
  "dependencies": {
    "shitty-extensions": "^1.0.1"
  },
  "bundledDependencies": ["shitty-extensions"],
  "pi": {
    "extensions": ["extensions", "node_modules/shitty-extensions/extensions"],
    "skills": ["skills", "node_modules/shitty-extensions/skills"]
  }
}
```

## 包过滤

在设置中使用对象形式过滤包加载的内容：

```json
{
  "packages": [
    "npm:simple-pkg",
    {
      "source": "npm:my-package",
      "extensions": ["extensions/*.ts", "!extensions/legacy.ts"],
      "skills": [],
      "prompts": ["prompts/review.md"],
      "themes": ["+themes/legacy.json"]
    }
  ]
}
```

`+path` 和 `-path` 是相对于包根目录的精确路径。

- 省略键以加载该类型的所有内容。
- 使用 `[]` 不加载该类型的任何内容。
- `!pattern` 排除匹配项。
- `+path` 强制包含精确路径。
- `-path` 强制排除精确路径。
- 过滤器在清单之上分层。它们缩小已有内容的范围。

## 启用和禁用资源

使用 `pi config` 启用或禁用已安装包和本地目录中的扩展、技能、提示词模板和主题。适用于全局（`~/.pi/agent`）和项目（`.pi/`）作用域。

## 作用域和去重

包可以同时出现在全局和项目设置中。如果同一个包同时出现在两者中，项目条目优先。身份由以下决定：

- npm：包名称
- git：去除引用的仓库 URL
- local：解析后的绝对路径