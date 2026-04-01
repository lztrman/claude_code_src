# 最有意思、最反常的部分

这份列表不是按目录排，而是按“研究价值”和“反常程度”排。

## 1. build-time feature gating 在塑造源码形状

最值得先记住的一点：这里的 `feature(...)` 不是普通开关。

它实际上在做：

- ant/internal vs external 构建裁剪
- dead-code elimination
- 字符串泄漏规避
- 导入路径重写
- overlay/stub 切换

### 为什么反常

很多仓库把 feature flag 写成运行时布尔值；Claude Code 则明显把 `bun:bundle` 当成“源码雕刻器”。

## 2. `main.tsx` + `REPL.tsx` 形成“双巨石内核”

- `main.tsx` 像进程级总控路由器
- `REPL.tsx` 像终端前台应用内核

### 为什么有意思

现代前端/CLI 项目通常会把复杂度拆成很多边界清晰的模块；这里则能看到长期产品化后，为保证整体一致性，复杂度重新汇聚回两个中心化文件。

## 3. `src/ink/` 基本是一台私有终端引擎

它自己维护：

- reconciler
- DOM
- layout
- screen buffer
- diff/render
- termio parser
- selection/search/cursor 语义

### 为什么反常

普通终端 UI 项目不会把这么多层都收回自己维护。

## 4. `src/native-ts/yoga-layout` 是纯 TypeScript 版 Yoga

这是整个仓库最不寻常的单点实现之一。

### 为什么有意思

- 说明团队在主动摆脱 native 依赖
- 也说明他们需要一个可控、可调试、可按自己需求裁剪的布局引擎

## 5. commands / skills / plugins / MCP 在能力池里被故意打平

最终模型看到的是统一能力宇宙，而不是四套孤立系统。

### 具体表现

- `commands.ts` 合并 built-in、skills、plugins、workflows、MCP skill commands
- `SkillTool` 再把一部分 prompt command 暴露给模型
- `services/mcp/client.ts` 把远端 MCP 工具包装成 `Tool`

### 为什么有意思

它暴露出一个核心设计选择：Claude Code 优先统一“能力面”，而不是维护模块边界的纯洁性。

## 6. `Tool` 协议比命令协议还重

`Tool.ts` 不是简简单单的 schema + execute。

它还包括：

- permission
- UI rendering
- destructive/read-only
- progress
- result collapsing
- auto-classifier input
- MCP 元信息

### 研究结论

工具在这里是一级运行时实体，而不是命令的下层实现。

## 7. 权限系统引入了 transcript-aware classifier

`utils/permissions/yoloClassifier.ts` 很反常，因为它不是按命令黑白名单做判断，而是：

- 抽会话 transcript
- 投喂 classifier
- 按上下文裁决危险操作

### 为什么有意思

这是把“安全策略”做成上下文感知代理，而不是 ACL。

## 8. computer use 被包装成动态 MCP，只为命名兼容

`utils/computerUse/setup.ts` 里最关键的事实是：

- 之所以包成 `mcp__computer-use__*`
- 不是因为架构洁癖
- 而是因为后端 system prompt / availability hint 就是按这个名字识别能力

### 为什么反常

这是一种非常现实主义的设计：协议不是抽象边界，而是后端兼容层。

## 9. 远程控制子系统不止一套

并存的有：

- `bridge/`
- `remote/`
- `server/directConnect`
- `upstreamproxy/`

### 为什么有意思

这说明远程控制不是单一路径，而是不同宿主/网络/成本模型下的多套方案并存。

## 10. `upstreamproxy/` 里居然有安全基础设施代码

它做的事包括：

- session token
- CA bundle 拼接
- CONNECT-over-WebSocket relay
- 代理环境注入
- `prctl(PR_SET_DUMPABLE, 0)`

### 为什么反常

这已经不是“AI 编码工具”给人的第一直觉，而是云端安全和网络控制面代码。

## 11. 任务模型有四层并存

- 本地 shell 任务
- 本地子代理任务
- 云端 remote agent
- 同进程 teammate

### 为什么有意思

团队没有把 agent 统一简化成单一执行器，而是把多种隔离级别都产品化了。

## 12. `memdir/` 把长期记忆做成了文件协议

不是向量数据库，不是 opaque state，而是：

- `MEMORY.md` 索引
- topic 文件正文
- 类型学
- 大小限制
- team memory

### 为什么有意思

它把记忆变成了可审计、可版本管理、可人为理解的资产。

## 13. `coordinatorMode.ts` 直接把总控代理工作流写进源码

包括：

- worker 能力边界
- coordinator system prompt
- 继续/终止 worker 的规则
- resume 时切 mode 的规则

### 为什么反常

很多系统把这种东西留在 prompt 模板或文档里；这里把它写成了运行时规则。

## 14. `buddy/` 在严肃平台代码里塞了 companion 系统

它不是简单吉祥物：

- 有 rarity
- 有 species
- 有 stats
- 按用户稳定生成

### 为什么有意思

这说明 Claude Code 想做的不只是工程代理，也包括长期陪伴式终端体验。

## 15. `moreright` 和一批 stub 暴露了 recovered 源码的“缺失层”

这些文件提醒我们：

- 现在看到的不是完整内部源树
- 而是 external build 可见层 + source map 恢复层 + overlay 缺失层

### 研究上的意义

后续如果要继续深挖，必须把“真实实现不存在”和“当前恢复代码真的就是这样”区分开。

## 一个总判断

Claude Code 最有研究价值的地方，不是它“功能很多”，而是它把下面这些通常分属不同产品的东西强行压进了同一棵运行时树里：

- 终端渲染引擎
- AI agent turn engine
- 权限与沙箱策略引擎
- 插件/技能/MCP 平台
- 多代理任务系统
- 远程控制与云端桥接
- 持久记忆协议
- 长期产品实验层
