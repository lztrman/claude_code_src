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

如果把 `commands/tools/tasks` 看成前台执行面，那么这里就是后端能力面：

- `services` 提供 API、MCP、policy、analytics、tool execution、LSP 等主干服务
- `skills/plugins` 把系统做成可扩展平台
- `bridge/remote/server/upstreamproxy` 把本地会话扩展成远端、容器、桥接与代理环境

这一层最重要的信号是：Claude Code 不是单机 CLI，而是一个可被本地、远端、云端、插件和 MCP 共同驱动的平台。

## `services`

`services/` 不是薄服务层，而是第二个运行时核心。

最重要的几组子系统包括：

- `services/api/*`
- `services/mcp/*`
- `services/tools/*`
- `services/compact/*`
- `services/analytics/*`
- `services/policyLimits/*`
- `services/remoteManagedSettings/*`
- `services/lsp/*`

其中最重的主干之一就是 `services/mcp/client.ts`。

## `skills` 与 `plugins`

这两层共同说明：prompt 资产、插件命令、工作流、hooks、agent 定义最终都被编进同一能力池。

这不是“支持插件”，而是：

- skills 是可加载 prompt 资产
- plugins 是可安装扩展包
- MCP 又能继续往能力池里注入 tool/resource/prompt/skill

系统最终优先统一能力面，而不是维护目录边界纯洁性。

## `bridge`、`remote`、`server`

这三层说明远程控制不是一条额外路径，而是正式架构面：

- `bridge/` 偏 host 侧控制和 remote-control
- `remote/` 偏会话客户端与状态适配
- `server/` 暴露 direct-connect 路径

因此，remote 并不是“加个 websocket”，而是正式的会话模型。

## `upstreamproxy`

`upstreamproxy/` 是整个仓库里最反常的目录之一。

它处理的不是普通 CLI 问题，而是：

- session token
- CA bundle 拼接
- CONNECT-over-WebSocket relay
- 代理环境注入
- Linux 下的进程级安全加固

单看这个目录，很难把它和“代码助手 CLI”联系起来；但它说明 Claude Code 已经深入到远端容器、企业网络与安全边界中。

## 第二轮补充研究：`services/mcp/client.ts`

`services/mcp/client.ts` 不是单纯的 SDK adapter，而是 MCP 平台核心。

### 1. transport matrix

| server type | transport | 备注 |
| --- | --- | --- |
| `sse` | `SSEClientTransport` | 带 auth provider，单独处理长连接 fetch |
| `http` | `StreamableHTTPClientTransport` | 带 fresh timeout、proxy、step-up detection |
| `ws` | 自定义 `WebSocketTransport` | 合并 headers、代理、TLS 选项 |
| `sse-ide` | `SSEClientTransport` | IDE 特化，无常规 auth |
| `ws-ide` | `WebSocketTransport` | 走 `X-Claude-Code-Ide-Authorization` |
| `stdio` | `StdioClientTransport` | 默认本地 MCP 形态 |
| `claudeai-proxy` | `StreamableHTTPClientTransport` | 经 Claude.ai OAuth 与 proxy URL |
| Chrome / Computer Use | in-process linked transport | 明确绕开外部 stdio 子进程 |

最有意思的点不是“支持很多 transport”，而是所有这些 transport 最终都被揉进同一个客户端抽象里。

### 2. auth 不是统一层，而是 transport-specific

这里的认证策略明显按 transport 分裂：

- SSE/HTTP：`ClaudeAuthProvider`
- HTTP/WS：可叠加 session-ingress bearer token
- `ws-ide`：IDE 专用头
- `sse-ide`：显式无常规 auth
- `claudeai-proxy`：Claude.ai OAuth + session id 头
- 各 server 的静态/动态头再通过 `getMcpServerHeaders()` 合并

这说明 auth 不是“外围通用 middleware”，而是 MCP transport 协议的一部分。

### 3. 缓存与 memoization 是三层结构

目前至少能看到三层缓存：

- `mcp-needs-auth-cache.json`
  - 15 分钟 auth failure 缓存
- `connectToServer`
  - 按 `server name + serialized config` memoize
- `fetchToolsForClient` / `fetchResourcesForClient` / `fetchCommandsForClient`
  - LRU memoization

这层缓存结构说明 MCP client 不只是“调用远端”，而是长期承载连接生命周期和 metadata 生命周期。

### 4. tool / resource / command 获取逻辑

`services/mcp/client.ts` 不只拉 tools：

- `fetchToolsForClient`
  - 调 `tools/list`
  - 清洗 Unicode
  - 截断过长 description
  - 映射注解到 Claude tool flags
  - 包装成可调用工具
- `fetchResourcesForClient`
  - 调 `resources/list`
  - 绑定 server 来源
- `fetchCommandsForClient`
  - 调 `prompts/list`
  - 把 MCP prompt 转成命令
  - 与 MCP skills 合流

也就是说，MCP 在这里不仅是 tool provider，还是 resource provider 和 prompt/command provider。

### 5. 结果处理不是简单 passthrough

结果转换路径大致是：

1. `transformResultContent`
2. `transformMCPResult`
3. `processMCPResult`

处理中会区分：

- text
- image
- audio
- binary blob
- resource / resource_link

大结果会被截断或落盘，再把“如何读取”作为文本结果回给模型或 UI。

所以 MCP 输出在这里已经被做成“Claude 自己的内容协议”。

### 6. 特殊分支：Claude-in-Chrome 与 Computer Use

这两条路径非常反常：

- 它们在 transport 层就被特殊拦截
- 直接走 in-process linked transport
- Computer Use 甚至明确写着 package 自己的 `CallTool` handler 是 stub，真实调度走 wrapper override

这说明 MCP 在这里既是标准协议，也被拿来当本地能力统一外壳。

### 7. 错误与重连策略非常激进

这份文件内建了大量恢复逻辑：

- 连接超时
- 每次请求 fresh timeout
- tool 调用独立 timeout
- auth failure -> needs-auth
- session expired -> clear cache + reconnect
- URL elicitation retry
- SSE/HTTP/claudeai-proxy 的 session 过期检测
- stdio 子进程清理升级链

它明显不是“失败了就抛错”，而是在自己维护一套正式的连接状态机。

## 这一层目前最重要的判断

`services/mcp/client.ts` 暴露出一个核心事实：

- Claude Code 已经把 MCP 当成平台主干，而不是外接协议。

它统一了：

- transport
- auth
- metadata 生命周期
- 结果协议变换
- 重连与恢复
- 内建特化能力注入

## 这一层最反常的地方

1. MCP 在这里是主干，不是插件协议。
2. tools / resources / prompts / skills 会在能力池里汇合。
3. 远程控制并不只有一套路径：bridge、remote、direct connect、upstream proxy 并存。
4. `upstreamproxy/` 让这份代码显露出明显的企业网络与安全基础设施气质。
5. Claude-in-Chrome 与 Computer Use 被强行包装进同一 MCP 语义面，说明协议兼容优先于架构洁癖。
