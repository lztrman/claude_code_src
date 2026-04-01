# Claude Code 源码研究索引

这份 `docs/` 不是“看完仓库后的随手笔记”，而是面向多轮继续研究的长期结构化档案。目标有三个：

1. 把 `src/` 和 `vendor/` 的所有顶层模块都落到明确位置。
2. 把最值得深入的异常点单独抽出来，避免被目录树淹没。
3. 给下一轮继续研究留下前线和未决问题，而不是每次从零开始。

## 当前结论

- 这份仓库的本体不是“CLI 命令集合”，而是一个运行在终端里的 agent platform。
- 最关键的三条主线是：
  - `entrypoints/cli.tsx -> main.tsx`：启动、快路径、模式分流、总控装配。
  - `screens/REPL.tsx`：交互式前台壳，吸纳消息流、权限、任务、远程、IDE、voice、plugins。
  - `query.ts / QueryEngine.ts`：真正的 agent turn engine。
- 最反常的几块不是业务功能，而是底层能力：
  - 自带一整套终端渲染内核 `src/ink/`
  - 纯 TypeScript 版 Yoga / native 替代层 `src/native-ts/`
  - transcript-aware 权限分类器 `src/utils/permissions/`
  - 把 computer-use 包成动态 MCP 的兼容设计 `src/utils/computerUse/`
  - 容器内 CONNECT-over-WebSocket relay `src/upstreamproxy/`

## 文档结构

- [模块地图](./module-map.md)
  - 所有顶层模块、文件数量、职责、对应研究章节
- [运行时与入口](./modules/00-runtime-and-entrypoints.md)
  - `src-root`、`entrypoints`、`cli`、`bootstrap`、`state`、`screens`、`query`、`migrations`、`assistant`
- [UI 与终端内核](./modules/01-ui-and-terminal.md)
  - `components`、`hooks`、`context`、`ink`、`keybindings`、`vim`、`buddy`、`voice`、`moreright`
- [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md)
  - `commands`、`tools`、`tasks`、`coordinator`
- [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md)
  - `services`、`skills`、`plugins`、`bridge`、`remote`、`server`、`upstreamproxy`
- [基础设施与策略底座](./modules/04-infrastructure-and-policy.md)
  - `utils`、`constants`、`types`、`schemas`、`memdir`、`native-ts`、`outputStyles`、`vendor/*`
- [异常点专题](./findings/interesting-anomalies.md)
  - 跨模块排序后的“最有意思/最反常”实现
- [下一轮前线](./frontier/next-pass.md)
  - 下一次执行应该补齐什么、往哪里追加

## 阅读顺序

如果第一次进入这份仓库，建议按下面顺序：

1. 先看 [模块地图](./module-map.md)，建立顶层分块和文件规模感。
2. 再看 [运行时与入口](./modules/00-runtime-and-entrypoints.md)，理解主链路。
3. 然后在 [命令、工具与任务面](./modules/02-commands-tools-and-tasks.md) 和 [服务、扩展与远程控制](./modules/03-services-extensions-and-remote.md) 之间来回看。
4. 最后看 [异常点专题](./findings/interesting-anomalies.md)，把值得深挖的部分单独拎出来。

## 覆盖说明

本轮已经覆盖 `src/` 与 `vendor/` 的所有顶层模块：

- `src-root`
- `assistant`
- `bootstrap`
- `bridge`
- `buddy`
- `cli`
- `commands`
- `components`
- `constants`
- `context`
- `coordinator`
- `entrypoints`
- `hooks`
- `ink`
- `keybindings`
- `memdir`
- `migrations`
- `moreright`
- `native-ts`
- `outputStyles`
- `plugins`
- `query`
- `remote`
- `schemas`
- `screens`
- `server`
- `services`
- `skills`
- `state`
- `tasks`
- `tools`
- `types`
- `upstreamproxy`
- `utils`
- `vim`
- `voice`
- `vendor/audio-capture-src`
- `vendor/image-processor-src`
- `vendor/modifiers-napi-src`
- `vendor/url-handler-src`

后续如果新增研究内容，优先更新对应模块章节；如果某个发现跨越多个模块，再补到 [异常点专题](./findings/interesting-anomalies.md)。
