# 最有意思、最反常的部分

这份列表不是按目录排，而是按“研究价值”和“异常程度”排。

## 1. build-time feature gating 在塑造源码形状

这里的 `feature(...)` 远不只是运行时布尔开关。

它实际上参与：

- dead-code elimination
- ant/internal 与 external 构建分化
- import 路径切换
- overlay / stub 留痕
- 某些字符串与实现的完全剔除

所以这份代码的“真实形状”本来就是 build-dependent 的。

## 2. import order 本身就是运行时契约

这是第二轮最值得记住的新结论。

入口层明显依赖：

- 在导入 `main.tsx` 之前先改 env
- 在 `main.tsx` 模块求值期就启动预热任务
- 之后再在 `preAction` 里同步等待这些预热结果

这是一种非常刻意的延迟隐藏策略，但代价是：正确性部分建立在模块求值顺序上，而不是只建立在显式函数调用上。

## 3. `main.tsx` + `REPL.tsx` 形成双核心

- `main.tsx` 负责进程级总控、模式分流、入口参数改写、interactive/headless 终局分叉
- `REPL.tsx` 负责会话级控制、消息流、权限、任务、overlay、远端与宿主集成

很多项目会努力把复杂度拆散；这份代码则在长期产品化后，把复杂度重新汇聚到两个中心文件。

## 4. `REPL.tsx` 不是 screen，而是 control plane

第二轮深挖后，这一点更加明确：

- `messagesRef + setMessages` 才像真实消息源
- `onSubmit` 是 transport 与输入路由器
- `getFocusedInputDialog` 是 overlay 仲裁器
- `onQuery/onQueryImpl` 是 turn 生命周期总控

它把 transport、permissions、tasks、history mutation、performance shaping 都拉进同一控制平面。

## 5. `src/ink/` 基本是一台私有终端引擎

它维护的内容已经远超“用 Ink”：

- reconciler
- DOM / layout
- screen buffer
- diff/render
- ANSI/OSC/CSI parser
- terminal query
- selection/search/cursor 语义

这解释了为什么终端前台能承载如此重的交互。

## 6. `src/native-ts/yoga-layout` 是纯 TypeScript Yoga

这是全库最不寻常的单点实现之一。

它说明团队不只是想“跑起来”，而是想把布局引擎拉回可控、可调试、可裁剪的 TS 世界。

## 7. commands / skills / plugins / MCP 被故意打平

最终系统更关心“能力池”，而不是目录边界：

- commands 合并 built-in、skills、plugins、workflow、MCP skill commands
- tools 又把 built-in 与 MCP tools 合池
- MCP prompt/resource/tool 最终都能投影到 Claude 自己的执行面

这是一种很强的“统一能力面优先”设计。

## 8. `Tool.ts` 比命令协议还重

工具在这里不是普通函数，而是正式运行时对象。

它们携带：

- permission 语义
- concurrency / interrupt 语义
- progress / result / transcript 语义
- UI 渲染
- context modifier
- MCP 元信息

研究这一层，比研究单个工具实现更重要。

## 9. `query.ts` 是带多重续跑路径的状态机

它不是简单的 LLM request loop，而是持续判断：

- 什么时候 compact
- 什么时候 reactive recovery
- 什么时候继续下一 turn
- 什么时候 stop hook 重试
- 什么时候 token-budget continuation
- 什么时候终止

这也是为什么 `QueryEngine.ts` 目前还不能把 REPL 逻辑完全吃掉。

## 10. `services/mcp/client.ts` 是平台主干，不是 SDK adapter

它统一了：

- transport
- auth
- metadata fetch
- result transformation
- cache / reconnect
- Claude-in-Chrome / Computer Use 特例

这意味着 MCP 在 Claude Code 里已经是一等平台平面。

## 11. 权限系统已经引入 transcript-aware classifier

`utils/permissions/yoloClassifier.ts`、Bash classifier、`useCanUseTool()` 这一组逻辑说明：

- 权限不只是规则表
- 它会吃 transcript / context
- 会结合 classifier 与 hook 自动放行或拒绝

这已经不是 ACL，而是“上下文感知安全代理”。

## 12. remote control 明显不是一条路径

并存的至少有：

- `bridge/`
- `remote/`
- `server/direct connect`
- `ssh`
- `upstreamproxy/`

它们看起来不像重复实现，更像不同宿主与网络条件下并存的多套方案。

## 13. `upstreamproxy/` 里有真正的安全/网络基础设施代码

包括：

- session token 处理
- CA bundle
- CONNECT-over-WebSocket relay
- 代理环境注入
- `prctl(PR_SET_DUMPABLE, 0)`

这已经完全超出了人们对“AI coding CLI”最直觉的预期。

## 14. `memdir/` 把长期记忆做成文件协议

不是 opaque state，也不是向量库优先，而是：

- `MEMORY.md`
- topic 文件
- 类型与大小约束
- team memory

这让记忆成为可审计、可版本化、可人工理解的资产。

## 15. `buddy/`、`voice/`、`moreright/` 暴露出强烈的产品实验层

- `buddy/` 是 companion/pet 系统
- `voice/` 是横切式产品能力
- `moreright/` 是 external stub / internal overlay 留痕

这说明 Claude Code 想做的不只是工程代理，还包括长期陪伴式终端体验和内部产品试验台。

## 一句话总判断

Claude Code 最值得研究的地方，不是“功能很多”，而是它把下面这些通常属于不同产品的能力强行压进了同一棵运行时树里：

- 终端渲染引擎
- agent turn 状态机
- 权限与分类器
- 插件 / skill / MCP 平台
- 多执行器任务系统
- 远程控制与企业网络边界
- 文件化长期记忆
- 持续产品实验层
