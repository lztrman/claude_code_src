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

这份仓库的 UI 层不是“用 Ink 画点终端组件”，而是：

- 用 `components/` 搭前台壳
- 用 `hooks/` 编排行为和副作用
- 用 `context/` 做横向总线
- 用 `ink/` 自建终端渲染内核
- 用 `keybindings/` 和 `vim/` 管输入状态机

换句话说，UI 层本身就包含一个小型终端操作系统。

## `components`

`components/` 文件最多，仅次于 `utils`。但它不是单纯的展示层，而是多个子系统的前台投影。

### 组件分片方式

从目录和文件命名看，它大致分成几块：

- 权限与确认
  - `permissions/*`
  - `TrustDialog/*`
  - `MCPServerApprovalDialog.tsx`
- 消息与 transcript
  - `Messages.tsx`
  - `MessageSelector.tsx`
  - `VirtualMessageList.tsx`
  - `HighlightedCode.tsx`
- 任务与代理
  - `tasks/*`
  - `agents/*`
  - `TaskListV2.tsx`
- 输入与导航
  - `TextInput.tsx`
  - `VimTextInput.tsx`
  - `HistorySearchDialog.tsx`
  - `GlobalSearchDialog.tsx`
- 设计系统
  - `design-system/*`
  - `ui/*`
- 产品型对话框
  - onboarding、theme、model、output style、desktop handoff、IDE、feedback、survey、teleport、worktree、team 等

### 最值得记住的点

- 权限 UI 是完整子系统，不是一个弹窗。
- 任务和 agent 的可视化被长期产品化了，不是调试工具。
- “终端里做产品” 的气质很强，能看到大量 onboarding、survey、desktop upsell、team、grove、trust、teleport 等产品态。

## `hooks`

`hooks/` 是前台运行时真正的行为层，而不是 React 小工具合集。

### 典型角色

- 连接运行时和 UI：
  - `useMergedTools`
  - `useMergedCommands`
  - `useCanUseTool`
  - `useQueueProcessor`
- 连接远程能力：
  - `useReplBridge`
  - `useRemoteSession`
  - `useDirectConnect`
  - `useMailboxBridge`
- 连接本地宿主：
  - `useIDEIntegration`
  - `useIdeLogging`
  - `useDiffInIDE`
  - `useSSHSession`
- 连接后台任务与通知：
  - `useTasksV2`
  - `useScheduledTasks`
  - `useInboxPoller`
  - `hooks/notifs/*`

### 结论

REPL 没有把复杂度消掉，它只是把复杂度拆进大量 hook。`hooks/` 更像行为编排层，而不是组件辅助层。

## `context`

`context/` 文件数很少，但承担跨树总线：

- `notifications.tsx`
- `mailbox.tsx`
- `modalContext.tsx`
- `overlayContext.tsx`
- `promptOverlayContext.tsx`
- `stats.tsx`
- `fpsMetrics.tsx`
- `voice.tsx`

### 作用

- 给大范围组件树注入全局能力
- 避免把所有横切状态都塞进单一 React prop 链
- 在 `App.tsx` 级别装配终端前台基础设施

这层并不复杂，但很关键，因为它承担的是“全局 UI 信号线”。

## `ink`

这是整个仓库最反常的一层之一。

### 核心判断

这不是“使用 Ink”，而是“把 Ink 改造成私有终端渲染内核”。

### `ink/` 实际自带了什么

- 自定义 reconciler
- 自定义 DOM 模型
- 自定义 layout 抽象
- 自定义 screen buffer
- 自定义 render diff
- 自定义 ANSI/OSC/CSI parser
- 自定义 terminal query 机制
- 自定义 selection / search highlight / cursor 语义

### 几个最值得记住的实现

- `ink.tsx`
  - 终端实例本体
  - 处理 alt-screen、selection、search overlay、cursor park、SIGCONT/resize
- `screen.ts`
  - typed-array 屏幕表示
  - 不是字符串拼接，也不是对象数组
- `renderer.ts` / `output.ts`
  - 从 DOM 布局结果生成虚拟屏幕，再 diff 到真实终端
- `terminal-querier.ts`
  - 用 DA1 哨兵实现“无 timeout 的终端能力探测”
- `selection.ts`
  - 终端全文选择子系统

### 为什么这层重要

它解释了为什么 Claude Code 能在终端里做这么重的交互：不是依赖现成 CLI 库，而是自己接管了终端这个设备。

## `keybindings`

`keybindings/` 是一等系统，不是点缀。

### 特征

- 支持 schema
- 支持模板
- 支持用户配置加载和校验
- 有保留快捷键与冲突检查
- 支持按上下文分层

### 意义

这不是“全局快捷键表”，而是局部状态机输入路由系统。

## `vim`

`vim/` 规模很小，但完成度很高：

- `motions.ts`
- `operators.ts`
- `textObjects.ts`
- `transitions.ts`
- `types.ts`

### 为什么反常

这是一个 AI agent CLI，不是文本编辑器项目，但仍然实现了较完整的 Vim normal-mode 状态机语义。说明团队把“长期终端使用体验”当成正式需求，而不是彩蛋。

## `buddy`

`buddy/` 是产品气质最跳的一块。

### 关键事实

- 它不是单纯的装饰图标，而是一个持久 companion 系统。
- `companion.ts` 里有 rarity、species、hat、eye、stats、seed。
- 生成逻辑还按用户 ID 稳定映射，意味着它是“长期绑定用户”的产品层对象。

### 研究意义

它说明这套仓库不是只服务工程执行，还在试图建立“伴随式终端体验”。

## `voice`

`src/voice/` 目录本身只有一个 gate 文件，但它的存在提醒一件事：

- 语音不是孤立功能
- 真正的 voice 能力散在：
  - `services/voice*`
  - `hooks/useVoice*`
  - `context/voice.tsx`
  - `vendor/audio-capture-src`

也就是说，`voice` 更像一个顶层 feature flag 入口。

## `moreright`

`moreright/useMoreRight.tsx` 是 external build stub。

### 为什么重要

- 文件本身几乎不做事。
- 但注释明确说明“真实实现只在内部 overlay 中存在”。

这说明 recovered 源码不是单一真实源树，而是“外部构建可见层 + 内部 overlay 缺失”的拼接结果。

## 这一层最反常的地方

1. `REPL.tsx` 不是某个 screen，而是前台应用内核。
2. `components` 不是纯展示层，而是多个产品子系统的 UI 载体。
3. `hooks` 是行为总线，不是轻量封装。
4. `ink` 已经自成终端引擎。
5. `keybindings` 和 `vim` 说明输入系统被长期产品化。
6. `buddy`、`voice`、`moreright` 暴露了强烈的内部实验与产品叠层痕迹。
