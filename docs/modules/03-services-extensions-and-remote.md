# 服务、扩展与远程控制

## 覆盖模块

- `services`
- `skills`
- `plugins`
- `bridge`
- `remote`
- `server`
- `upstreamproxy`

## 总体判断

如果把 `commands/tools/tasks` 看成前台执行面，那么这一层就是后台能力平面：

- `services` 提供 API、MCP、tool execution、analytics、policy 等服务
- `skills/plugins` 把系统做成可扩展平台
- `bridge/remote/server/upstreamproxy` 把本地会话扩展成远程控制宿主

这一层最强烈的信号是：Claude Code 不是单机 CLI，而是一个可以被本地、远程、云端、插件、MCP、多代理共同驱动的平台。

## `services`

`services/` 不是薄服务层，而是第二个运行时核心。

### 最重要的几组子系统

#### 1. API 与模型调用

- `services/api/claude.ts`
- `services/api/client.ts`
- `services/api/errors.ts`
- `services/api/logging.ts`

这层负责：

- Claude API 调用
- betas
- thinking
- structured output
- usage/cost
- retry
- provider 差异

它几乎就是 LLM transport 内核。

#### 2. MCP

- `services/mcp/client.ts`
- `services/mcp/config.ts`
- `services/mcp/auth.ts`
- `services/mcp/officialRegistry.ts`
- `services/mcp/claudeai.ts`

这是整棵树里最重要的子系统之一。MCP 在这里不只是插件协议，而是：

- tool 来源
- resource 来源
- prompt 来源
- auth 路径
- UI/permissions/remote 的一部分

### 结论

MCP 在 Claude Code 里是一等公民，不是外挂。

#### 3. tool execution

- `services/tools/toolExecution.ts`
- `services/tools/toolOrchestration.ts`
- `services/tools/StreamingToolExecutor.ts`

这层把模型发出的 tool use 变成：

- 可中断
- 可追踪
- 可并发
- 可带 hook/telemetry
- 可进入任务平面

#### 4. analytics / policy / managed settings

- `services/analytics/*`
- `services/policyLimits/*`
- `services/remoteManagedSettings/*`

这说明 Claude Code 不是孤立工具，而是持续在线产品：

- feature gate
- 组织策略
- 远程下发设置
- telemetry

都是主干能力。

#### 5. 其它产品化子系统

- `services/compact/*`
- `services/tips/*`
- `services/voice*`
- `services/PromptSuggestion/*`
- `services/lsp/*`
- `services/SessionMemory/*`

这些子系统数量多、职责杂，但共同说明一件事：这是长期演化产品，不是单点工具。

## `skills`

`skills/` 是本地 prompt 能力层。

### 组成

- `loadSkillsDir.ts`
- `bundledSkills.ts`
- `bundled/*`
- `mcpSkillBuilders.ts`

### 关键事实

- skill 本质上会被编译成一种 `prompt command`
- bundled skills 是同步注册的
- 本地技能、bundled skills、plugin skills、MCP skills 最终都可能进入统一能力池

### 反常点

这套系统把“prompt 资产”产品化了，而不是把 prompt 留在代码字面量里。

## `plugins`

`src/plugins/` 本身很薄，真正的重量在 `utils/plugins/`，但从顶层模块看，它承担的是“内建插件壳”。

### 信号

- Claude Code 不只是能“装插件”
- 它本身就带内建插件机制
- 插件还可以进一步提供：
  - commands
  - skills
  - output styles
  - hooks
  - agents

### 结论

插件不是附加层，而是平台扩展面的正式入口。

## `bridge`

`bridge/` 是远程控制主栈。

### 它做的事

- remote-control / bridge mode
- 会话创建与恢复
- bridge permission callbacks
- session runner
- bridge API / config / status
- REPL bridge
- remote bridge core

### 为什么重要

这不是单纯 websocket 层，而是“把本地机器暴露成远程可控工作环境”的宿主。

### 反常点

- bridge 并不是一条额外路径，而是和主运行时同等级的模式。
- 很多逻辑围绕 feature gate、auth、version、policy 和 session ingress 组织，明显是产品级远控系统。

## `remote`

`remote/` 更像 bridge 的客户端侧观察和控制层：

- `RemoteSessionManager.ts`
- `SessionsWebSocket.ts`
- `sdkMessageAdapter.ts`
- `remotePermissionBridge.ts`

### 作用

- 维持远程会话状态
- 做消息适配
- 跨会话桥接权限/事件

它说明“远程”不是一个按钮，而是会话级运行模型。

## `server`

`server/` 规模很小，但含义明确：

- `createDirectConnectSession.ts`
- `directConnectManager.ts`
- `types.ts`

### 为什么值得记

这代表另一条远程路径：direct connect。

也就是说，远程控制并不只有 bridge 一套做法。

## `upstreamproxy`

这是整个仓库里最反常的一块之一。

### 它做什么

- 在 CCR 容器内读取 session token
- 拼接 CA bundle
- 起本地 CONNECT -> WebSocket relay
- 给子进程继承代理环境变量
- 在 Linux 下通过 `prctl(PR_SET_DUMPABLE, 0)` 减少 token 被同 UID ptrace 抓堆的风险

### 为什么反常

这已经是安全/网络基础设施代码，而不是普通 CLI 逻辑。

### 研究结论

如果只看 `upstreamproxy/`，很难把它和“终端 AI 编码工具”联系起来。但它恰恰说明 Claude Code 已经深入到远程容器与企业网络环境。

## 这一层最反常的地方

1. `services/mcp` 把 MCP 做成了平台主干，而不是 SDK 集成点。
2. `skills`、`plugins`、`MCP` 最终会在能力池里互相打平。
3. `bridge`、`remote`、`server/direct connect`、`upstreamproxy` 同时存在，说明远程控制不止一条实现路径。
4. `upstreamproxy` 暴露了很强的云端/安全基础设施气质。
5. `services/analytics`、`policyLimits`、`remoteManagedSettings` 说明这份代码是长期产品内核，而不是离线工具。
