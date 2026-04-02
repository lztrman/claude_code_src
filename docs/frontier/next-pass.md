# 下一轮前线

这份文件不是记录“还没看完”，而是给下一次执行留下高质量起点。

## 本轮已完成

- 顶层模块地图已经建立并落到 `module-map.md`
- `src` 与 `vendor` 的顶层模块已经全部归类到模块章节
- 第二轮已经补完下面这些主干深挖：
  - 启动精确路径：`entrypoints/cli.tsx -> main.tsx -> init.ts -> setup.ts`
  - `REPL.tsx` concern map
  - `query.ts` 执行序列
  - `QueryEngine.ts` 的真实边界
  - 工具生命周期
  - `services/mcp/client.ts` 架构拆解

## 下一轮最值得优先做的事

### 1. 把 `REPL.tsx` 的状态 choke points 画成正式地图

这一轮已经确认它是 control plane，但下一轮还需要更精确：

- `messagesRef`
- `setMessages`
- `AppState`
- command queue
- permission queues
- 各类 refs 与 bridge callback refs

目标不是再说一次“它很大”，而是画出真正的状态真相源和写入边界。

### 2. 把 `onSubmit` 做成分支表

这是 REPL 中最值得单独拆解的函数之一。

下一轮建议按这些维度做表：

- 本地 query
- remote session
- direct connect
- ssh
- queued command
- speculation accept
- history write
- attachment serialization
- idle-return / survey / permission 交互影响

### 3. 追完整条 permissions 流程

建议专门拉一轮：

- `hasPermissionsToUseTool`
- `useCanUseTool`
- `PermissionContext`
- `toolExecution.ts`
- sandbox / worker sandbox / bridge remote approval
- `yoloClassifier` 与 Bash speculative classifier

目标是弄清：

- 谁先判规则
- 谁触发 classifier
- 谁落日志
- 谁弹 UI
- 谁能远端代批

### 4. 继续深挖 `services/mcp/client.ts` 的调用后半段

这一轮主要完成了 transport / auth / metadata / result pipeline 总览。

下一轮可以继续拆：

- `callMCPToolWithUrlElicitationRetry`
- `callMCPTool`
- `transformMCPResult`
- `processMCPResult`
- `setupSdkMcpClients`

目标是形成“从工具名到最终 transcript/UI 呈现”的精确路径图。

### 5. 回答 `QueryEngine.ts` 到底是不是迁移终点

现在能确认：

- 它已经是 headless/SDK 主壳
- 但 REPL 仍直接调用 `query()`

下一轮需要继续判断：

- 还有哪些 REPL 责任没法被引擎化
- 哪些只是技术债
- 哪些是交互式前台天然无法抽离的职责

### 6. 系统性清点 external stub / internal overlay 痕迹

这一块非常重要，因为它直接影响对 recovered 源码的判断。

下一轮建议建立清单：

- 明显 stub 的文件
- 只剩 external 壳的模块
- 注释里明确提到 ant-only / overlay / external build 的地方
- 哪些 feature gate 只在内部版本有实际意义

## 可追加专题

如果时间够，优先加这些：

- `bridge`、`remote`、`direct connect`、`ssh`、`upstreamproxy` 的长期关系图
- `memdir` 的文件协议与 prompt 注入机制
- `utils/plugins` 的版本缓存、安装和热重载路径
- `native-ts` 为什么要从 native 迁回 TS

## 写作约定

1. 新内容优先并回已有 `modules/` 章节，不随意开散文件。
2. 如果发现跨模块异常，先更新 `findings/interesting-anomalies.md`。
3. 如果发现顶层目录变化，先更新 `module-map.md`。
4. 如果只是局部深挖，优先追加“补充研究”小节，而不是重写总览。
5. 对复杂调用链，优先用阶段图或状态图，而不是只堆段落。
