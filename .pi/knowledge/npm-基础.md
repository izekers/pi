# npm 基础

> Pi 项目中涉及的 npm 核心概念

## package.json

每个 Node.js 包的核心描述文件，定义了包名、版本、依赖、入口命令等。

##关键字段

| 字段 | 作用 |
|------|------|
| `name` | 包名，如 `@earendil-works/pi-coding-agent` |
| `version` | 语义化版本，如 `0.78.1` |
| `type` | 模块类型：`module`（ESM）或 `commonjs` |
| `main` | 包的默认入口文件 |
| `bin` | 命令行入口映射：`{ "pi": "dist/cli.js" }` |
| `dependencies` | 生产环境依赖 |
| `devDependencies` | 开发环境依赖 |

## npm workspaces

npm v7+ 支持的 monorepo 功能。在根目录 `package.json` 中声明：

```json
{
  "workspaces": ["packages/*"]
}
```

所有子包的依赖统一安装到根目录 `node_modules/`，子包之间互相引用时直接符号链接到本地包，无需从 npm registry 下载。

## 常用命令

```bash
npm install # 安装所有依赖
npm install --ignore-scripts # 跳过生命周期脚本（编译等）
npm run build                # 构建所有包（TypeScript → JS）
npm link # 将本地包的 bin 链接到全局命令
npm list -g --depth=0        # 查看全局已链接的命令
```

## 在 Pi 项目中的应用

- `npm install --ignore-scripts` — 跳过 native addon 构建，快速安装依赖
- `npm run build` — 按顺序构建 tui → ai → agent → coding-agent
- `cd packages/coding-agent && npm link` — 全局安装 `pi` 命令