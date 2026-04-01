# 顶层模块地图

## 先看整体

这份仓库的拓扑不是“按业务模块切包”，而是“按运行时职责堆在一棵树里”：

- 大体量核心是六块：
  - `utils` 564 files
  - `components` 389 files
  - `commands` 207 files
  - `tools` 184 files
  - `services` 130 files
  - `hooks` 104 files
- 真正的超节点不是按文件数算，而是按耦合度算：
  - `main.tsx`
  - `screens/REPL.tsx`
  - `query.ts`
  - `QueryEngine.ts`
  - `commands.ts`
  - `tools.ts`
  - `services/mcp/client.ts`
  - `utils/permissions/*`

## 顶层依赖印象

- `utils` 是所有大模块的共同底座，也是最大的耦合源。
- `components` 不是纯展示层，它高度依赖 `utils/hooks/tools/services/ink`。
- `commands` 和 `tools` 是两个并行的控制面，都依赖 `utils` 和 `services`，同时彼此通过 `query`/`Tool`/`Command` 协议互相渗透。
- `services/mcp`、`bridge`、`remote`、`upstreamproxy` 说明远程控制不是边缘能力，而是系统主干之一。
- `ink` 在架构上像“终端内核”，不是普通 UI 依赖。

## 模块总表

| 模块 | 文件数 | 一句话职责 | 主要归档 |
| --- | ---: | --- | --- |
| `src-root` | 18 | 根级总线文件，含 `main/query/commands/tools/setup` | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `assistant` | 1 | assistant/kairos 模式的残留入口与会话历史 | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `bootstrap` | 1 | 进程级全局状态与启动期 latch | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `bridge` | 31 | CCR / remote-control bridge 主栈 | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `buddy` | 6 | Companion/宠物系统与终端 sprite | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `cli` | 19 | 无头输出、transport、CLI handlers | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `commands` | 207 | slash command 实现 | [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md) |
| `components` | 389 | Ink/TUI 组件与对话框子系统 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `constants` | 21 | prompt、tools、betas、产品常量 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `context` | 9 | 通知、modal、overlay、mailbox 等 provider 总线 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `coordinator` | 1 | 多 worker 协调器模式规则与系统提示 | [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md) |
| `entrypoints` | 8 | 冷启动入口、SDK/mcp/sandbox 类型 | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `hooks` | 104 | REPL 行为编排层，接系统副作用 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `ink` | 96 | 私有终端渲染内核 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `keybindings` | 14 | 自定义快捷键系统 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `memdir` | 8 | 文件原生长期记忆协议 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `migrations` | 11 | 设置与模型迁移脚本 | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `moreright` | 1 | external build stub / 内部 overlay 占位 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `native-ts` | 4 | TypeScript 版 native 替代层 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `outputStyles` | 1 | 输出风格 prompt 插件入口 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `plugins` | 2 | 内建插件壳与 bundled plugin 注册 | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `query` | 4 | query config、token budget、stop hooks | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `remote` | 4 | 远程会话客户端与消息适配 | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `schemas` | 1 | hooks 等运行时 schema | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `screens` | 3 | REPL/Doctor/ResumeConversation 主屏 | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `server` | 3 | direct-connect server/client 配置 | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `services` | 130 | API、MCP、analytics、tool execution、policy 等服务层 | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `skills` | 20 | 本地技能、bundled skills、MCP skill builder | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `state` | 6 | 自定义 store、selectors、AppState | [运行时与入口](./modules/00-runtime-and-entrypoints.md) |
| `tasks` | 12 | 后台任务与子代理执行模型 | [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md) |
| `tools` | 184 | 模型可调用工具实现 | [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md) |
| `types` | 11 | 运行时协议类型与生成类型 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `upstreamproxy` | 2 | 容器侧 CONNECT-over-WebSocket relay | [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) |
| `utils` | 564 | 权限、插件、hooks、model、session、git、sandbox 等总底座 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `vim` | 5 | Vim 输入状态机 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `voice` | 1 | voice mode 可见性/kill-switch 入口 | [UI 与终端内核](./modules/01-ui-and-terminal.md) |
| `vendor/audio-capture-src` | 1 | 原生录音/播放/麦克风状态桥 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `vendor/image-processor-src` | 1 | 原生图像处理与 clipboard bridge | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `vendor/modifiers-napi-src` | 1 | 原生修饰键状态桥 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |
| `vendor/url-handler-src` | 1 | 原生 URL/deep-link 事件桥 | [基础设施与策略底座](./modules/04-infrastructure-and-policy.md) |

## 需要记住的边界

### 1. `commands` 不是“用户命令层”那么简单

它会合并：

- 内建 slash commands
- bundled skills
- 用户 skills
- plugin commands
- workflow commands
- 动态发现技能
- MCP skill commands

### 2. `tools` 不是“命令的下层”

它是模型调用面，和 `commands` 并行存在：

- `commands` 决定用户如何驱动系统
- `tools` 决定模型如何驱动系统

### 3. `services/mcp` 不是可选扩展

MCP 已经渗透进：

- tool pool
- command pool
- resources
- prompts
- auth
- UI
- permissions
- remote integration

### 4. `utils` 是底座，也是耦合源

如果后续要继续研究复杂度来源，优先从这些子树切：

- `utils/permissions`
- `utils/plugins`
- `utils/hooks`
- `utils/model`
- `utils/computerUse`
- `utils/sessionStorage`

## 下一步怎么用这张图

- 想理解主链路：先看 [运行时与入口](./modules/00-runtime-and-entrypoints.md)
- 想理解 agent 如何做事：看 [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md)
- 想理解它为什么像“平台”而不是“CLI”：看 [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md)
- 想抓住最怪的地方：看 [异常点专题](./findings/interesting-anomalies.md)
