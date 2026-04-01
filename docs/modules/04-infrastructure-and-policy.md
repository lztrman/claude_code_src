# 基础设施与策略底座

## 覆盖模块

- `utils`
- `constants`
- `types`
- `schemas`
- `memdir`
- `native-ts`
- `outputStyles`
- `vendor/audio-capture-src`
- `vendor/image-processor-src`
- `vendor/modifiers-napi-src`
- `vendor/url-handler-src`

## 总体判断

这一层不是“杂项工具箱”，而是 Claude Code 的真实底座。

最重要的结论有两个：

1. `utils` 不是边角目录，而是第二核心。
2. 很多最反常、最有研究价值的设计都藏在这里，而不是在 UI 或命令层。

## `utils`

`utils/` 的最佳理解方式不是逐文件看，而是按层次看。

### 1. 协议与类型辅助层

- messages
- JSON/YAML/XML
- schema 辅助
- path/fs/cwd

作用：给运行时其他模块提供统一输入输出和低层抽象。

### 2. 权限与安全层

- `utils/permissions/*`
- `utils/powershell/*`
- `utils/bash/*`
- `utils/sandbox/*`

这是最重要的子树之一。

#### 关键结论

- 权限系统不是简单 allowlist/denylist。
- 它会综合：
  - 规则
  - mode
  - hook
  - sandbox
  - working dir
  - interactive dialog
  - async agent/coordinator
  - transcript-aware classifier

#### 为什么异常

`yoloClassifier.ts` 说明 Claude Code 已经把“语境感知权限决策”正式产品化了。命令是否危险，不只看命令本身，还看会话上下文。

### 3. 插件与扩展生态层

- `utils/plugins/*`

这层最像包管理器，而不是普通插件目录扫描器。

#### 重要能力

- versioned cache
- seed cache
- zip cache
- marketplace
- blocklist / policy
- multi-scope install
- plugin commands / skills / hooks / output styles / agents

#### 研究结论

插件系统已经接近“扩展平台”，而不是“读几个 markdown 文件”。

### 4. 模型与 provider 层

- `utils/model/*`

这层负责：

- alias
- provider 差异
- 内部代号与外部暴露
- 模型能力与上下文窗口
- 老型号兼容映射

#### 反常点

这里不是简单 model enum，而是一个完整的型号解释器。

### 5. 生命周期 hook 层

- `utils/hooks/*`
- `utils/hooks.ts`

这层把 CLI 生命周期开放成可编排总线：

- command hooks
- prompt hooks
- agent hooks
- HTTP hooks

而且是异步、带进度、带 stdout JSON 提取的“半结构化协程”，不是简单子进程 side effect。

### 6. computer use / Claude in Chrome / 桌面自动化层

- `utils/computerUse/*`
- `utils/claudeInChrome/*`

这是整棵树里最有意思的子树之一。

#### 关键事实

- computer use 没有直接作为 builtin tool 暴露
- 而是被包成动态 MCP server
- tool name 必须长成 `mcp__computer-use__*`
- 原因不是架构洁癖，而是为了触发后端 system prompt 可用性提示

这说明 MCP 在这里不只是协议，而是“能力命名兼容层”。

### 7. 会话、传送、Git、工作树、持久化层

- sessionStorage
- teleport
- git/worktree
- concurrent sessions
- settings
- release notes

这层最说明产品已经进入长期运行阶段：会话要恢复、要迁移、要跨工作树、要跨版本持续存在。

## `constants`

`constants/` 不是无聊常量目录，而是运行时政策集。

### 关键子类

- `prompts.ts`
- `tools.ts`
- `betas.ts`
- `product.ts`
- `messages.ts`

### 研究结论

这里混着：

- prompt 片段
- beta header
- rollout 常量
- 产品字符串
- 工具 allowlist/disallowlist

它暴露了一个事实：很多产品演进历史直接编码进运行时。

## `types`

`types/` 里最重要的不是生成类型，而是运行时协议类型：

- `types/command.ts`
- `types/permissions.ts`
- `types/plugin.ts`

### 最重要的发现

- command 被定义成三种执行模型
- permissions 被定义成多阶段原因链
- plugin 错误模型已经大到像一个小平台协议

它们说明 Claude Code 的复杂度已经逼到需要先稳定协议，再让实现围绕协议生长。

## `schemas`

`schemas/` 很小，但起的是“解耦导入拓扑”的作用。

例如 `schemas/hooks.ts` 就是为打断 import cycle 而拆出来的运行时 schema 层。

这说明仓库规模已经大到模块拓扑本身成为设计约束。

## `memdir`

`memdir/` 是另一块极有研究价值的异常点。

### 它不是数据库

它做的是：

- `MEMORY.md` 作为索引
- topic 文件作为正文
- 有类型学
- 有大小限制
- 有 team memory
- 有路径协议
- 有 prompt 注入规范

### 结论

Claude Code 把长期记忆实现成“可审计文件协议”，而不是黑箱 embedding store。

## `native-ts`

`native-ts/` 说明团队在主动摆脱原生依赖。

### 关键例子

- `native-ts/yoga-layout`
  - 纯 TypeScript 版 Yoga/flexbox 引擎
- `native-ts/file-index`
  - TypeScript 版文件索引替代
- `native-ts/color-diff`
  - TypeScript 版 color diff

### 研究结论

这不是简单降级，而是“把关键基础设施迁回 TS 以换取可控性、跨平台性和打包一致性”。

## `outputStyles`

`outputStyles/` 很小，但意义很大。

### 它做什么

- 从 `.claude/output-styles` 读 markdown
- frontmatter 提供 name/description/keep-coding-instructions
- 文件正文变成 style prompt

### 为什么有意思

这不是 UI theme，而是 prompt skin。

也就是说，Claude Code 把“输出人格和汇报格式”也做成了可扩展资产。

## `vendor/*`

这几块都是原生桥：

- `vendor/audio-capture-src`
- `vendor/image-processor-src`
- `vendor/modifiers-napi-src`
- `vendor/url-handler-src`

### 共同特征

- 都是懒加载 wrapper
- 尽量避免在 module eval 阶段 dlopen
- 只保留不可替代的系统能力

### 分工含义

- `vendor/*` 负责真正需要 native 的部分
- `native-ts/*` 负责把重逻辑迁回 TS

这是非常清晰的工程演化方向。

## 这一层最反常的地方

1. `utils` 不是工具箱，而是平台底座。
2. 权限系统引入 transcript-aware classifier。
3. computer use 被包装成动态 MCP，只为满足后端能力命名兼容。
4. 插件系统已经接近包管理器。
5. `memdir` 把长期记忆做成文件协议。
6. `native-ts` 代表“主动摆脱 native 依赖”的工程路线。
7. `outputStyles` 说明 prompt 风格也被产品化了。
