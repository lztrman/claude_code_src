# 下一轮前线

这份文件的目的不是记录“还没看完”，而是给下一次执行留一个高质量起点。

## 本轮已经完成的事

- 建立了完整的顶层模块地图
- 把所有顶层模块落到了对应章节
- 把几条主线明确下来：
  - 启动/入口
  - UI/终端内核
  - 命令/工具/任务
  - 服务/扩展/远程
  - 基础设施/策略底座
- 抽出了跨模块异常点专题

## 下一轮最值得优先做的事

### 1. 把 `main.tsx` 的启动路径画成精确调用图

当前只完成了高层描述。下一轮应做：

- 列出所有 fast path
- 列出进入完整 REPL 前的关键初始化顺序
- 标明哪些初始化依赖 `feature()`，哪些依赖 `USER_TYPE`
- 专门梳理 top-level side effects

最值得补的文件：

- `src/entrypoints/cli.tsx`
- `src/main.tsx`
- `src/setup.ts`
- `src/entrypoints/init.ts`

### 2. 把 `REPL.tsx` 拆成“关注点地图”

当前知道它是巨型前台壳，但还缺一份真正可用的 concern map。

建议下一轮按这些维度切：

- query/message flow
- permissions
- tasks/agents
- bridge/remote
- IDE/browser/voice
- search/history/navigation
- surveys/product overlays

### 3. 把 `query.ts` 做成正式 sequence 文档

要回答的问题：

- 一次 user message 进入后，怎么变成 API 请求
- tool use 是怎么被串行/并行执行的
- stop hooks、compact、budget、structured output 是在哪些节点插入的
- `QueryEngine.ts` 与 `query.ts` 的真正边界是什么

### 4. 深挖 `services/mcp/client.ts`

MCP 是主干之一，但本轮只完成高层定位。

下一轮应补：

- transport 形态总表
- auth 形态总表
- resources/prompts/tools 的统一抽象
- Claude-in-Chrome / computer-use 特判路径
- MCP 输出存储与 UI 映射

### 5. 追踪工具执行的完整生命周期

建议专门写一篇附加文档，覆盖：

- `Tool.ts`
- `tools.ts`
- `services/tools/toolOrchestration.ts`
- `services/tools/toolExecution.ts`
- `services/tools/StreamingToolExecutor.ts`
- `hooks/useCanUseTool.tsx`

目标是搞清楚：

- 谁决定能不能执行
- 谁决定如何展示
- 谁决定如何写回 transcript
- 谁决定变成 task / background work

### 6. 把 external stub 和内部 overlay 痕迹单列出来

这块很重要，因为它直接影响对 recovered 源码的判断。

下一轮建议建立一个清单：

- 哪些文件明显是 stub
- 哪些模块只剩 external 版壳
- 哪些注释明确提到 ant-only / overlay / external build
- 哪些 feature gate 只在内部版有意义

## 可追加的专题

如果下一轮时间足够，优先扩展这些专题：

- `utils/permissions` 的完整裁决流程图
- `utils/plugins` 的版本缓存与安装路径
- `memdir` 的文件协议与 prompt 注入机制
- `upstreamproxy` 的容器网络与安全设计
- `native-ts` 从 native 迁回 TS 的工程动机

## 写作约定

下次继续研究时，优先遵守下面这几条：

1. 新内容优先追加到已有章节，不要随意新开散乱文件。
2. 如果发现跨模块异常点，先更新 `findings/interesting-anomalies.md`，再补对应模块章节。
3. 如果发现新的顶层模块或目录变化，先更新 `module-map.md`。
4. 如果只是局部深挖，不要重写总览，直接在章节里新增“补充研究”小节。

## 当前未决问题

- `REPL.tsx` 中真正的 concern 边界在哪里，哪些只是历史堆叠，哪些是当前必要耦合。
- `QueryEngine.ts` 最终是否意图完全替代交互式 query 主循环，还是只服务 SDK/headless。
- internal overlay 缺失对 `assistant`、`moreright`、若干 stub commands 的影响到底有多大。
- bridge、remote、direct connect、upstreamproxy 四套远程机制之间的长期关系是什么：互补、替代，还是历史叠加。
