# Claude Code 源码研究

这套文档不是“按目录抄一遍”，而是按运行时主干、控制平面和异常设计点来组织。

当前已经完成两轮研究：

- 第一轮：铺开全库模块地图，覆盖 `src` 与 `vendor` 的顶层模块。
- 第二轮：沿主执行链深挖启动入口、`REPL.tsx`、`query.ts`、`QueryEngine.ts`、工具生命周期、`services/mcp/client.ts`。

## 阅读顺序

1. `module-map.md`
2. `modules/00-runtime-and-entrypoints.md`
3. `modules/01-ui-and-terminal.md`
4. `modules/02-commands-tools-and-tasks.md`
5. `modules/03-services-extensions-and-remote.md`
6. `modules/04-infrastructure-and-policy.md`
7. `findings/interesting-anomalies.md`
8. `frontier/next-pass.md`

## 当前最重要的结论

- 这不是“一个 CLI 项目”，而是一套终端原生 agent 平台。
- `main.tsx` 和 `REPL.tsx` 形成双核心：前者控制进程级启动与模式分流，后者控制会话级运行时与前台交互。
- `query.ts` 才是真正的一轮 agent turn 状态机；`QueryEngine.ts` 是对它的 headless/SDK 封装，不是完整替代。
- `Tool.ts` 定义的不是轻量工具接口，而是模型执行面的正式协议对象。
- `services/mcp/client.ts` 不是普通 SDK 包装层，而是 transport、auth、缓存、结果映射、重连策略汇聚的主干模块。
- `src/ink/`、`src/native-ts/yoga-layout/`、`upstreamproxy/` 这些目录说明，这份代码库的技术野心明显超出“代码补全 CLI”。

## 第二轮新增深挖主题

- 启动链精确调用图：`entrypoints/cli.tsx -> main.tsx -> init.ts -> setup.ts -> REPL/print`
- `REPL.tsx` concern map：消息流、权限、任务、远端、IDE、overlay、性能优化都汇在同一控制平面
- `query.ts` 序列化分析：预处理、流式采样、工具执行、续跑路径、终止路径
- `QueryEngine.ts` 边界：它拥有什么、不拥有什么、为什么还不能替掉 REPL
- 工具生命周期：注册、权限决策、执行、转写 transcript、背景任务语义
- MCP 客户端架构：transport、auth、缓存、tool/resource/command 拉取、结果处理、特殊分支

## 文档组织原则

- 模块级结论优先写进 `modules/`，不随意开散乱文档。
- 跨模块异常先写进 `findings/interesting-anomalies.md`。
- 下一轮待挖方向写进 `frontier/next-pass.md`。
- 如果源码恢复状态与真实内部实现可能不一致，文档会明确标注“stub / overlay / external build 痕迹”。
