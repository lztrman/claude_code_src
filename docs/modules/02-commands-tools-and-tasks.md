# 命令、工具与任务面

## 覆盖模块

- `commands`
- `tools`
- `tasks`
- `coordinator`

## 总体判断

Claude Code 有两个并行控制面：

- `commands`：用户通过 slash command 驱动系统
- `tools`：模型通过 tool use 驱动系统

`tasks` 则是两者共用的执行平面，负责承载后台 shell、子代理、远程代理和 teammate 运行态。

这三层加在一起，才是 Claude Code 的“代理执行面”。

## `commands`

### 命令模型

`Command` 并不是单一函数签名，而至少分三类：

- `prompt`
- `local`
- `local-jsx`

这意味着 slash command 既可能只是 prompt 宏，也可能执行本地逻辑，甚至直接挂 UI。

### `commands.ts` 做了什么

它不是简单导出数组，而是统一装配：

- 内建命令
- 内部命令
- bundled skills
- 用户 skills
- plugin commands
- workflow commands
- 动态发现的 skills
- MCP skill commands

还会再根据：

- 可用性
- feature gate
- remote mode
- bridge mode
- auth/provider

做二次裁剪。

### 这层最值得记住的事实

- 命令和技能的边界被故意打平了。
- “命令系统”其实是用户控制面的能力注册表。
- 一部分看似独立的功能，其实只是 `prompt` 命令的不同来源。

### 反常点

- 仓库里存在一批 external build stub 命令，说明内外部构建不是一套完整相同的产品。
- `REMOTE_SAFE_COMMANDS` / `BRIDGE_SAFE_COMMANDS` 这种显式 allowlist 很像移动/远程客户端的专门适配层，而不是普通 CLI 逻辑。

## `tools`

### `Tool` 协议比想象中重得多

`Tool.ts` 定义的工具接口不只包括 schema 和执行逻辑，还包括：

- prompt 文本
- permission checks
- destructive / read-only 语义
- UI 渲染
- 进度与结果映射
- 搜索索引
- tool result collapsing
- auto-classifier 输入
- MCP 元信息

这说明工具不是“命令的下层”，而是模型执行面的正式协议对象。

### `tools.ts` 做了什么

它是工具总装厂：

- 内建工具
- feature-gated 工具
- REPL-only 工具
- plan/worktree/task 工具
- ToolSearch/MCP 资源工具
- AgentTool/SkillTool
- Bash/PowerShell/File tools
- MCP tools 合池

并且会处理：

- deny-rule 预过滤
- REPL 模式下隐藏底层原语
- simple mode
- built-in 与 MCP tool 的排序稳定性

### 关键工具簇

#### 1. Shell 与文件工具

- `BashTool`
- `PowerShellTool`
- `FileReadTool`
- `FileEditTool`
- `FileWriteTool`
- `NotebookEditTool`
- `GlobTool`
- `GrepTool`

这层不是简单文件/命令接口，而是被权限系统深度控制的“受管执行面”。

#### 2. Agent 与任务工具

- `AgentTool`
- `TaskCreateTool`
- `TaskGetTool`
- `TaskUpdateTool`
- `TaskListTool`
- `TaskOutputTool`
- `TaskStopTool`

这层说明“任务”和“代理”不是隐藏实现，而是显式暴露给模型的一级能力。

#### 3. 扩展与策略工具

- `SkillTool`
- `MCPTool`
- `ListMcpResourcesTool`
- `ReadMcpResourceTool`
- `ToolSearchTool`
- `ConfigTool`
- `EnterPlanModeTool`
- `ExitPlanModeTool`
- `EnterWorktreeTool`
- `ExitWorktreeTool`

这类工具很特殊：它们操作的不是项目文件，而是 Claude Code 自己的运行时能力边界。

## `tasks`

`tasks/` 是第二执行平面，不属于 UI，也不完全属于 tools。

### 统一抽象

根级：

- `Task.ts`
- `tasks.ts`

给整个系统提供了统一任务接口和任务状态语义。

### 任务模型分化

- `LocalShellTask`
  - 本地 shell 后台任务
- `LocalAgentTask`
  - 本地子代理任务
- `RemoteAgentTask`
  - 云端远程代理任务
- `InProcessTeammateTask`
  - 同进程 teammate 任务
- `DreamTask`
  - 更偏实验/产品态的任务模型

### 重要意义

这说明团队没有把“agent”简化成一种执行器，而是把不同隔离级别、成本模型和宿主环境都产品化了。

### 反常点

- 所有任务都能归并成统一通知流。
- shell 任务、子代理、远程代理、同进程 teammate 可以共存在一个前台会话中。

这在普通 CLI 里很少见，更像一个小型 job system。

## `coordinator`

`coordinator/` 目录只有一个文件，但异常重要：`coordinatorMode.ts`。

### 为什么它反常

- 它直接把 coordinator system prompt 写进代码。
- 它还定义 worker 能力边界、resume 时如何切回 coordinator mode、scratchpad 提示和任务工作流。

### 它暴露出的事实

- “多 worker 编排”不是临时策略，而是正式产品模式。
- coordinator 不是外部 prompt 拼装，而是运行时内建角色。

## 这一层最反常的地方

1. `commands` 和 `tools` 不是上下层，而是两个并行控制面。
2. `Tool` 协议重到足以自成平台接口。
3. `commands.ts` 会把 skills/plugins/workflows/MCP 命令都打平进一个命令宇宙。
4. `tasks` 不是实现细节，而是正式的多执行器任务平面。
5. `coordinatorMode.ts` 把“如何做总控代理”直接写成代码内规则和系统提示。

## 本层最值得优先深读的文件

- `src/commands.ts`
- `src/types/command.ts`
- `src/tools.ts`
- `src/Tool.ts`
- `src/tools/AgentTool/AgentTool.tsx`
- `src/tools/BashTool/BashTool.tsx`
- `src/tools/PowerShellTool/PowerShellTool.tsx`
- `src/services/tools/toolExecution.ts`
- `src/services/tools/toolOrchestration.ts`
- `src/services/tools/StreamingToolExecutor.ts`
- `src/tasks/LocalShellTask/LocalShellTask.tsx`
- `src/tasks/LocalAgentTask/LocalAgentTask.tsx`
- `src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`
- `src/coordinator/coordinatorMode.ts`
