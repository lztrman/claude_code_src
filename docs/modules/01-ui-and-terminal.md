# UI 与终端内核

## 覆盖模块

- `components`
- `hooks`
- `context`
- `ink`
- `keybindings`
- `vim`
- `buddy`
- `voice`
- `moreright`

## 总体判断

这一层不是“用 Ink 画几个终端组件”，而是：

- `components/` 负责前台壳
- `hooks/` 负责行为编排
- `context/` 负责全局信号线
- `ink/` 负责终端渲染内核
- `keybindings/` 和 `vim/` 负责输入状态机

从结果上看，这一层已经像一个小型终端操作系统，而不只是一个 React TUI。

## `components`

`components/` 的重量不在“数量多”，而在它承载的是多个产品子系统的 UI 投影：

- transcript / message rendering
- 权限与审批
- tasks / agents
- onboarding / survey / notification
- IDE / remote / plugin / desktop handoff

它不是纯展示层，而是运行时能力的可视化外壳。

## `hooks`

`hooks/` 不是轻量封装，而是前台行为编排层。

尤其关键的 hook 类型有：

- 工具与命令合并：`useMergedTools`、`useMergedCommands`
- 权限：`useCanUseTool`
- 队列与会话：`useQueueProcessor`、`useCommandQueue`
- 远端：`useReplBridge`、`useRemoteSession`、`useDirectConnect`、`useSSHSession`
- IDE / browser / voice：`useIDEIntegration`、`usePromptsFromClaudeInChrome`、`useVoiceIntegration`
- 背景任务：`useTasksV2`、`useInboxPoller`、`useSessionBackgrounding`

所以复杂度没有消失，只是从组件树被拆进了 hook 编排层。

## `context`

`context/` 文件不多，但它们承担的是全局 UI 信号总线：

- notifications
- mailbox
- modal / overlay
- prompt overlay
- stats / fps metrics
- voice

它们的作用不是“方便传参”，而是把终端前台里跨系统的状态接到一起。

## `ink`

`src/ink/` 是整棵树里最反常的目录之一。

这不是“依赖 Ink”，而是“在 Ink 之上维护一套私有终端内核”。

它自己处理的内容包括：

- reconciler
- DOM / layout 抽象
- typed-array screen buffer
- render diff
- ANSI / OSC / CSI 解析
- terminal querying
- selection / search highlight / cursor 语义

这解释了为什么 Claude Code 可以在终端里承载这么重的交互。

## `keybindings` 与 `vim`

这两层说明“输入系统”是正式产品能力，而不是彩蛋：

- `keybindings/` 维护 schema、用户配置、冲突检查、上下文分层
- `vim/` 维护相对完整的 normal-mode 语义

这对于 AI agent CLI 来说很不寻常，因为它意味着团队把长期终端使用体验当成了一等需求。

## `buddy`、`voice`、`moreright`

这几个目录暴露了很强的产品实验痕迹：

- `buddy/companion.ts` 是带 rarity/species/stats 的 companion 系统
- `voice/` 是顶层 gate，真正的语音能力散在 hooks/services/vendor
- `moreright/useMoreRight.tsx` 明确提示 external build stub / overlay 缺失

这类模块提醒我们：这份恢复源码不是单一真实源树，而是“外部构建可见层 + 内部 overlay 缺失痕迹”的组合。

## 第二轮补充研究：`REPL.tsx` concern map

`REPL.tsx` 不是 screen，而是前台控制平面。

### 几个真正的 choke point

- `messagesRef + setMessages`
  - React state 更像渲染投影
  - 真正的会话真相被这对同步封装握在手里
- `getToolUseContext`
  - 每轮 turn 都在这里重新拼装 tools、MCP、权限、通知、IDE、resume 钩子
- `onSubmit`
  - 输入路由、transport 路由、历史写入、speculation 接受、query handoff 都在这里汇合
- `onQuery / onQueryImpl`
  - 真正的一轮 turn 生命周期
- `getFocusedInputDialog`
  - 几乎所有 overlay 的总仲裁器

### concern map

#### 1. query / message flow

主路径是：

- `onSubmit`
- `handlePromptSubmit`
- `onQuery`
- `onQueryImpl`
- `query()`
- streamed events 回流本地状态

而 compact boundary、duration message、bridge completion、auto-restore 都插在这个路径的前后。

#### 2. permissions

同一个文件里同时调度：

- tool permission
- sandbox permission
- worker sandbox
- prompt dialog
- elicitation dialog

而且这些请求最终进入同一套 dialog 优先级树。

#### 3. tasks / agents

这里还承担：

- swarm 初始化
- main-thread agent 限制
- local agent transcript bootstrap
- teammate / inbox / mailbox / task list
- background session handoff

它不是“显示任务列表”，而是在真正调度任务前台。

#### 4. bridge / remote / direct connect / ssh

`REPL.tsx` 同时兼容：

- 本地 REPL
- remote session
- direct connect
- SSH session
- REPL bridge

这几条远端路径都不是在外围包一层，而是深度嵌进提交、权限、消息镜像和回调收尾流程里。

#### 5. IDE / browser / voice

这一文件里还绑着：

- IDE selection / install / onboarding / logging
- Claude in Chrome prompt injection 与权限同步
- voice integration 与 keybinding

所以它既是 agent 会话控制平面，又是宿主环境集成平面。

#### 6. search / history / rewind

这里的“历史”不是只读能力：

- transcript search 是虚拟滚动模式
- rewind 会恢复权限上下文
- file history 也会一起回滚
- selector UI 甚至能触发 partial compact

换句话说，history 在这里是运行时变异能力，不是浏览功能。

#### 7. surveys / overlays

feedback、memory、frustration、skill-improvement、idle-return、desktop upsell、plugin/LSP hint、permission overlay 都在同一套优先级树里竞争显示。

也就是说，安全审批 UI 和产品增长/反馈 UI 共用一个控制栈。

#### 8. rendering / performance

文件里大量逻辑是性能塑形，不是业务逻辑：

- deferred rendering
- virtual transcript
- ref-mirrored callback
- scroll repinning
- progress/compact dedupe
- sync/deferred message switching

这说明团队长期在和“终端里做重交互”的成本对抗。

### 第二轮最重要的结论

`REPL.tsx` 的反常点不只是大，而是它把下面这些东西强行汇在同一个控制文件里：

- transport
- permissions
- agents/tasks
- overlays
- history mutation
- performance shaping

所以它不是 UI screen，而是前台 control plane。

## 这一层最反常的地方

1. `REPL.tsx` 的真实身份是控制平面，不是界面组件。
2. `messagesRef + setMessages` 说明 React state 不是唯一真相源。
3. sandbox / tool approval / product overlay 共享同一优先级树。
4. history 与 rewind 会直接修改运行时，而不是只读浏览。
5. `ink/` 已经接近一套私有终端引擎。
