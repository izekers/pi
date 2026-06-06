<p align="center">
  <a href="https://pi.dev">
    <img alt="pi logo" src="https://pi.dev/logo-auto.svg" width="128">
  </a>
</p>
<p align="center">
  <a href="https://discord.com/invite/3cU7Bz4UPx"><img alt="Discord" src="https://img.shields.io/badge/discord-community-5865F2?style=flat-square&logo=discord&logoColor=white" /></a>
</p>
<p align="center">
  <a href="https://pi.dev">pi.dev</a> 域名由
  <br /><br />
  <a href="https://exe.dev"><img src="packages/coding-agent/docs/images/exy.png" alt="Exy mascot" width="48" /><br />exe.dev</a> 慷慨捐赠
</p>

> 新贡献者提交的 issue 和 PR 默认会被自动关闭。维护者每天会审核自动关闭的 issue。详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

# Pi Agent Harness 单体仓库

这是 pi agent harness 项目的主仓库，包含我们可自扩展的编程代理。

* **[@earendil-works/pi-coding-agent](packages/coding-agent)**：交互式编程代理 CLI
* **[@earendil-works/pi-agent-core](packages/agent)**：具备工具调用和状态管理的代理运行时
* **[@earendil-works/pi-ai](packages/ai)**：统一的多提供商 LLM API（OpenAI、Anthropic、Google……）

了解更多关于 pi 的信息：

* [访问 pi.dev](https://pi.dev)，项目官网，包含演示
* [阅读文档](https://pi.dev/docs/latest)，你也可以让代理自行解释

## 分享你的开源编程代理会话

如果你使用 pi 或其他编程代理进行开源工作，请分享你的会话。

公开的开源会话数据有助于通过真实任务、工具使用、失败和修复来改进编程代理，而非依赖玩具级基准测试。

完整说明请参见 [这篇 X 帖子](https://x.com/badlogicgames/status/2037811643774652911)。

要发布会话，请使用 [`badlogic/pi-share-hf`](https://github.com/badlogic/pi-share-hf)。请阅读其 README.md 了解设置说明。你只需要一个 Hugging Face 账户、Hugging Face CLI 和 `pi-share-hf`。

你也可以观看 [这个视频](https://x.com/badlogicgames/status/2041151967695634619)，我在其中演示了如何发布 `pi-mono` 会话。

我定期在此发布我自己的 `pi-mono` 工作会话：

- [badlogicgames/pi-mono on Hugging Face](https://huggingface.co/datasets/badlogicgames/pi-mono)

## 所有包

| 包名 | 描述 |
|------|------|
| **[@earendil-works/pi-ai](packages/ai)** | 统一的多提供商 LLM API（OpenAI、Anthropic、Google 等） |
| **[@earendil-works/pi-agent-core](packages/agent)** | 具备工具调用和状态管理的代理运行时 |
| **[@earendil-works/pi-coding-agent](packages/coding-agent)** | 交互式编程代理 CLI |
| **[@earendil-works/pi-tui](packages/tui)** | 带差异渲染的终端 UI 库 |

如需 Slack/聊天自动化和工作流，请参见 [earendil-works/pi-chat](https://github.com/earendil-works/pi-chat)。

## 权限与容器化

Pi 不包含用于限制文件系统、进程、网络或凭据访问的内置权限系统。默认情况下，它以启动它的用户和进程的权限运行。

如需更强的隔离边界，请对 Pi 进行容器化或沙箱化。参见 [packages/coding-agent/docs/containerization.md](packages/coding-agent/docs/containerization.md) 了解三种模式：

- **OpenShell**：在策略控制的沙箱中运行整个 `pi` 进程。
- **Gondolin 扩展**：将 `pi` 和提供商认证保留在宿主机上，同时将内置工具和 `!` 命令路由到本地 Linux 微型虚拟机。
- **Plain Docker**：在本地容器中运行整个 `pi` 进程以实现简单隔离。

## 贡献

请参见 [CONTRIBUTING.md](CONTRIBUTING.md) 了解贡献指南，以及 [AGENTS.md](AGENTS.md) 了解项目特定规则（适用于人类和代理）。

## 开发

```bash
npm install --ignore-scripts  # 安装所有依赖，不运行生命周期脚本
npm run build        # 构建所有包
npm run check        # 检查格式和类型
./test.sh            # 运行测试（无 API 密钥时跳过依赖 LLM 的测试）
./pi-test.sh         # 从源码运行 pi（可从任意目录运行）
```

## 供应链安全加固

我们将 npm 依赖变更视为经过审查的代码变更。

- 直接外部依赖锁定到精确版本。内部工作区包保持版本范围。
- `.npmrc` 设置了 `save-exact=true` 和 `min-release-age=2`，以避免 npm 解析过程中同日发布的依赖。
- `package-lock.json` 是依赖的真实来源。Pre-commit 阻止意外的 lockfile 提交，除非设置了 `PI_ALLOW_LOCKFILE_CHANGE=1`。
- `npm run check` 验证锁定的直接依赖、原生 TypeScript 导入兼容性以及生成的 coding-agent shrinkwrap。
- 发布的 CLI 包包含 `packages/coding-agent/npm-shrinkwrap.json`（从根 lockfile 生成），以为 npm 用户锁定传递依赖。
- 发布冒烟测试使用 `npm run release:local` 来构建、打包并在仓库外创建隔离的 npm 和 Bun 安装，然后再打发布标签。
- 本地发布安装、文档化的 npm 安装和 `pi update --self` 在支持的情况下使用 `--ignore-scripts`。
- CI 使用 `npm ci --ignore-scripts` 安装，定期运行的 GitHub 工作流执行 `npm audit --omit=dev` 和 `npm audit signatures --omit=dev`。
- Shrinkwrap 生成对依赖生命周期脚本有明确的白名单；新的生命周期脚本依赖在审查通过前会检查失败。

## 许可证

MIT
