**[English](DOCUMENTATION.md)** | **[中文](DOCUMENTATION.zh-CN.md)**

# Claude Code — 源码文档与架构分析

> **Claude Code** 是 Anthropic 官方的 Claude CLI 工具。本文档提供了对其源码架构、模块和内部设计模式的全面逆向工程分析。

---

## 如何阅读本文档

本文档是对 Claude Code 源码树进行深度分析的成果。它面向希望了解 Claude Code 底层工作原理的**开发者和工程师**——包括其架构、模块边界、数据流和设计决策。

**结构：** 文档采用自顶向下的组织方式，从项目概览和技术栈开始，然后深入到各个主要子系统（工具、命令、状态、服务、UI）。每个章节都是独立的；你可以通过目录跳转到任意章节。

**受众：** 熟悉 TypeScript、React 和 CLI 工具的中高级开发者。有 AI/LLM 工具开发经验会有帮助，但不是必需的。

**范围：** 本文档涵盖源码树结构、模块清单和架构模式。不深入涉及运行时行为，也不包含性能基准测试或安全审计。

---

## 目录

1. [项目概览](#1-项目概览)
2. [技术栈](#2-技术栈)
3. [目录结构](#3-目录结构)
4. [入口点](#4-入口点)
5. [核心架构](#5-核心架构)
6. [工具系统](#6-工具系统)
7. [命令系统](#7-命令系统)
8. [状态管理](#8-状态管理)
9. [任务系统](#9-任务系统)
10. [服务与集成](#10-服务与集成)
11. [UI 层](#11-ui-层)
12. [工具函数](#12-工具函数)
13. [特殊模式](#13-特殊模式)
14. [插件与技能](#14-插件与技能)
15. [钩子与可扩展性](#15-钩子与可扩展性)
16. [文件统计](#16-文件统计)
17. [架构模式](#17-架构模式)

---

## 1. 项目概览

Claude Code 是一个功能丰富的交互式终端应用，允许直接在命令行中进行 AI 辅助软件工程。它提供：

- **交互式 REPL**，用于与 Claude 进行代码相关的对话
- **40+ 工具**，用于文件操作、Shell 执行、网络搜索等
- **100+ 斜杠命令**，用于提交、审查、调试等工作流
- **Agent/任务系统**，用于通过子 Agent 并行处理复杂工作
- **计划模式**，用于在编码前设计实现策略
- **MCP（Model Context Protocol）** 集成，提供可扩展的服务端工具
- **插件与技能系统**，用于用户自定义扩展
- **语音模式**、**桌面/移动端桥接**和**远程会话**

---

## 2. 技术栈

| 层级           | 技术                                                         |
|----------------|--------------------------------------------------------------|
| 语言           | TypeScript (`.ts` / `.tsx`)                                  |
| 运行时         | Bun（打包工具，通过 `bun:bundle` 实现特性标志）                |
| UI 框架        | React + [Ink](https://github.com/vadimdemedes/ink)（终端 React 渲染器）|
| API 客户端     | `@anthropic-ai/sdk`（Anthropic SDK）                          |
| MCP            | `@modelcontextprotocol/sdk`                                  |
| CLI 框架       | `@commander-js/extra-typings`                                |
| 数据验证       | Zod v4                                                       |
| 样式           | Chalk（终端颜色）                                             |
| 状态管理       | Zustand 风格 Store + React Context                           |

---

## 3. 目录结构

```text
claude-code-analysis/
└── src/                          # 所有源码（单一顶级目录）
    ├── main.tsx                  # 主引导与初始化
    ├── QueryEngine.ts            # 对话循环编排器
    ├── Tool.ts                   # 工具类型定义与接口
    ├── Task.ts                   # 任务类型定义与生命周期
    ├── commands.ts               # 命令注册表
    ├── tools.ts                  # 工具注册表与工厂
    ├── context.ts                # 系统/用户上下文构建器
    ├── query.ts                  # 查询上下文准备
    ├── setup.ts                  # 启动阶段编排
    ├── history.ts                # 聊天会话历史
    ├── cost-tracker.ts           # Token 用量与定价
    ├── ink.ts                    # Ink 渲染封装
    ├── replLauncher.tsx          # REPL React 组件启动器
    ├── tasks.ts                  # 任务执行管理器
    │
    ├── commands/                 # 101 个命令模块（斜杠命令）
    ├── tools/                    # 41 个工具实现
    ├── services/                 # 核心服务（API、MCP、分析等）
    ├── components/               # React/Ink UI 组件（130+ 文件）
    ├── utils/                    # 工具函数（300+ 文件）
    ├── state/                    # 应用状态管理
    ├── types/                    # TypeScript 类型定义
    ├── hooks/                    # React Hooks
    ├── schemas/                  # Zod 验证模式
    ├── tasks/                    # 任务类型实现
    ├── entrypoints/              # 入口点定义（CLI、SDK、MCP）
    ├── bootstrap/                # 应用启动与全局状态
    ├── screens/                  # 全屏 UI 布局
    ├── plugins/                  # 插件系统（内置插件）
    ├── skills/                   # 自定义技能系统（内置技能）
    ├── memdir/                   # 内存目录自动发现
    ├── constants/                # 应用常量
    ├── migrations/               # 数据/模式迁移
    ├── ink/                      # Ink 终端自定义
    ├── keybindings/              # 键盘绑定系统
    ├── context/                  # React Context（信箱、通知）
    ├── query/                    # 查询处理模块
    ├── outputStyles/             # 输出格式化样式
    ├── vim/                      # Vim 模式集成
    ├── voice/                    # 语音输入/输出
    ├── native-ts/                # 原生 TypeScript 绑定
    ├── assistant/                # Kairos（助手）模式
    ├── bridge/                   # Bridge 模式（常驻远程连接）
    ├── buddy/                    # Buddy/队友系统
    ├── coordinator/              # 多 Agent 协调
    ├── remote/                   # 远程会话处理
    ├── server/                   # 服务器实现
    ├── cli/                      # CLI 参数解析
    └── upstreamproxy/            # 上游代理设置
```

---

## 4. 入口点

### 主入口：`src/main.tsx`

主引导文件（约 1,400 行）。执行以下操作：

1. 通过 `startupProfiler.ts` 进行**启动性能分析**
2. **MDM（移动设备管理）** 原始数据预读取
3. **钥匙串预取**（macOS OAuth + API 密钥并行读取）
4. 通过 Bun 的 `feature()` 进行**特性标志初始化**，实现死代码消除
5. 通过 Commander.js 进行 **CLI 参数解析**
6. **身份认证**（API 密钥、OAuth、AWS Bedrock、GCP Vertex、Azure）
7. **GrowthBook** 初始化（A/B 测试与特性标志）
8. **策略限制**和**远程托管设置**加载
9. **工具、命令、技能和 MCP 服务器**注册
10. 通过 `replLauncher.tsx` **启动 REPL**

### 其他入口点（`src/entrypoints/`）

| 文件                | 用途                                         |
|---------------------|----------------------------------------------|
| `cli.tsx`           | CLI 入口点，带 React/Ink UI 渲染             |
| `init.ts`           | 引导初始化、版本检查                         |
| `mcp.ts`            | Model Context Protocol 集成                  |
| `agentSdkTypes.ts`  | Agent SDK 的类型定义                         |
| `sandboxTypes.ts`   | 沙箱执行环境类型                             |
| `sdk/`              | SDK 相关实现                                 |

### 启动阶段：`src/setup.ts`

负责编排：
- Node.js 版本验证
- 工作树初始化
- 会话与权限模式设置
- Git 根目录检测
- UDS 消息服务器启动

---

## 5. 核心架构

### 5.1 查询引擎（`src/QueryEngine.ts`）

应用的核心（约 46KB）。管理用户与 Claude 之间的对话循环：

- **消息管理** — 维护包含用户、助手、系统和工具消息的对话历史
- **流式传输** — 实时 Token 流式传输与工具调用执行
- **自动压缩** — 当接近上下文窗口限制时自动压缩上下文
- **提示词缓存** — 通过缓存感知策略优化重复上下文
- **重试逻辑** — 处理 API 错误、速率限制和过载，支持退避策略
- **用量追踪** — 统计 Token 数量（输入/输出/缓存读取/缓存写入）和成本
- **工具编排** — 分发工具调用、收集结果、管理权限

### 5.2 上下文构建器（`src/context.ts`、`src/query.ts`）

准备系统提示词和用户上下文：

- 发现并合并 `CLAUDE.md` 文件（项目级、用户级、全局级）
- 构建系统上下文（操作系统、Shell、平台、Git 状态）
- 集成用户上下文（权限、工作目录）
- 跨查询缓存上下文以提升性能

### 5.3 成本追踪（`src/cost-tracker.ts`）

追踪每次会话的成本：
- 按模型统计 Token 数量（输入、输出、缓存读取/写入）
- 通过定价表计算美元成本
- 会话持续时间追踪
- 网络搜索请求计数
- 文件变更指标

---

## 6. 工具系统

### 架构（`src/Tool.ts`、`src/tools.ts`）

每个工具都是一个自包含的模块，包含：
- **JSON Schema** 输入验证
- **权限模型**（询问/允许/拒绝模式）
- **进度追踪**类型
- **错误处理**和用户提示

### 完整工具清单（41 个工具）

#### 文件操作
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `FileReadTool`    | 读取文件内容，支持行范围                    |
| `FileWriteTool`   | 创建或覆盖文件                             |
| `FileEditTool`    | 精确字符串替换编辑                         |
| `GlobTool`        | 基于模式的文件匹配                         |
| `GrepTool`        | 基于正则的内容搜索（基于 ripgrep）          |

#### 代码执行
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `BashTool`        | 执行 Shell 命令，支持超时                   |
| `PowerShellTool`  | 执行 PowerShell 命令（Windows）             |
| `REPLTool`        | 执行 Python 代码（仅内部使用）              |
| `NotebookEditTool`| Jupyter Notebook 单元格操作                |

#### 网络与搜索
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `WebFetchTool`    | 获取并解析网页内容                         |
| `WebSearchTool`   | 互联网搜索                                 |
| `ToolSearchTool`  | 搜索可用的延迟加载工具                     |

#### Agent 与任务管理
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `AgentTool`       | 生成子 Agent 进行并行工作                   |
| `TaskCreateTool`  | 创建新的后台任务                           |
| `TaskGetTool`     | 获取任务状态和结果                         |
| `TaskUpdateTool`  | 更新任务状态或描述                         |
| `TaskListTool`    | 列出所有任务及其状态                       |
| `TaskStopTool`    | 终止正在运行的任务                         |
| `TaskOutputTool`  | 流式读取任务输出                           |
| `SendMessageTool` | 向运行中的 Agent 发送消息                   |

#### 计划与工作流
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `EnterPlanModeTool` | 进入只读计划模式                         |
| `ExitPlanModeTool`  | 退出计划模式并获得批准                   |
| `EnterWorktreeTool` | 创建隔离的 Git 工作树                    |
| `ExitWorktreeTool`  | 从工作树返回并携带变更                   |

#### MCP（Model Context Protocol）
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `MCPTool`         | 调用 MCP 服务器上的工具                     |
| `McpAuthTool`     | MCP 服务器认证                             |
| `ListMcpResourcesTool` | 列出 MCP 服务器资源                  |
| `ReadMcpResourceTool`  | 读取特定 MCP 资源                     |

#### 配置与系统
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `ConfigTool`      | 读取/修改设置                              |
| `SkillTool`       | 执行用户自定义技能                         |
| `AskUserQuestionTool` | 提示用户输入/确认                      |
| `BriefTool`       | 生成会话摘要                               |
| `TodoWriteTool`   | 管理待办事项列表                           |
| `SleepTool`       | 暂停执行指定时长                           |

#### 团队与远程
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `TeamCreateTool`  | 创建 Agent 团队                             |
| `TeamDeleteTool`  | 删除 Agent 团队                             |
| `RemoteTriggerTool` | 触发远程任务执行                         |
| `ScheduleCronTool`  | 调度定时任务                             |
| `LSPTool`         | Language Server Protocol 集成              |

#### 内部工具
| 工具              | 用途                                       |
|-------------------|--------------------------------------------|
| `SyntheticOutputTool` | 用于结构化响应的合成输出               |

---

## 7. 命令系统

### 注册表（`src/commands.ts`）

命令是 `src/commands/` 下的模块化目录，每个目录包含一个 `index.ts`（或类似文件），导出一个 `Command` 定义，包含名称、描述、处理函数和可选的别名。

### 完整命令清单（101 个模块）

#### Git 与版本控制
`commit`、`commit-push-pr`、`diff`、`branch`、`review`、`autofix-pr`、`pr_comments`、`teleport`、`rewind`、`tag`

#### 会话与历史
`session`、`resume`、`clear`、`compact`、`export`、`share`、`summary`、`context`

#### 配置与设置
`config`、`permissions`、`privacy-settings`、`theme`、`color`、`keybindings`、`vim`、`output-style`、`statusline`、`env`

#### Agent 与任务管理
`agents`、`tasks`、`brief`

#### 文件与代码操作
`files`、`add-dir`、`diff`、`debug-tool-call`、`copy`

#### 开发与调试
`doctor`、`heapdump`、`perf-issue`、`stats`、`bughunter`、`ctx_viz`、`ant-trace`

#### 身份认证
`login`、`logout`、`oauth-refresh`

#### 扩展与插件
`mcp`、`plugin`、`reload-plugins`、`skills`

#### 工作区
`plan`、`sandbox-toggle`、`init`

#### 信息与帮助
`help`、`version`、`cost`、`usage`、`extra-usage`、`release-notes`、`status`、`insights`

#### 平台集成
`desktop`、`mobile`、`chrome`、`ide`、`install`、`install-github-app`、`install-slack-app`

#### 记忆与知识
`memory`、`good-claude`

#### 模型与性能
`model`、`effort`、`fast`、`thinkback`、`thinkback-play`、`advisor`

#### 特殊操作
`bridge`、`voice`、`remote-setup`、`remote-env`、`stickers`、`feedback`、`onboarding`、`passes`、`ultraplan`、`rename`、`exit`

---

## 8. 状态管理

### Store 架构（`src/state/`）

| 文件              | 用途                                           |
|-------------------|------------------------------------------------|
| `AppState.tsx`    | React Context 提供者，带 `useAppState(selector)` Hook |
| `AppStateStore.ts`| 中心状态结构定义                               |
| `store.ts`        | Zustand 风格 Store 实现                        |

### 关键状态字段

```typescript
{
  settings: UserSettings           // 来自 settings.json 的用户配置
  mainLoopModel: string            // 当前活跃的 Claude 模型
  messages: Message[]              // 对话历史
  tasks: TaskState[]               // 运行中/已完成的任务
  toolPermissionContext: {         // 每个工具的权限规则
    rules: PermissionRule[]
    bypassMode: 'auto' | 'block' | 'ask'
    denialTracking: DenialTrackingState
  }
  kairosEnabled: boolean           // 助手模式标志
  remoteConnectionStatus: Status   // 远程会话连接状态
  replBridgeEnabled: boolean       // 常驻桥接（CCR）状态
  speculationState: Cache          // 内联推测缓存/预览
}
```

---

## 9. 任务系统

### 任务类型（`src/Task.ts`）

| 类型                  | 描述                                            |
|-----------------------|-------------------------------------------------|
| `local_bash`          | 本地 Shell 命令执行                              |
| `local_agent`         | 本地子 Agent（通过 AgentTool 生成）               |
| `remote_agent`        | 远程 Agent 执行                                  |
| `in_process_teammate` | 进程内队友（共享内存空间）                       |
| `local_workflow`      | 本地多步骤工作流                                 |
| `monitor_mcp`         | MCP 服务器监控任务                               |
| `dream`               | 自动梦境后台任务                                 |

### 任务生命周期

```text
pending -> running -> completed
                   -> failed
                   -> killed
```

### 任务状态结构

```typescript
{
  id: string           // 带类型前缀的唯一 ID（例如 "b-xxx" 表示 bash）
  type: TaskType
  status: TaskStatus
  description: string
  startTime: number
  endTime?: number
  outputFile: string   // 磁盘持久化输出
  outputOffset: number // 当前读取位置
  notified: boolean    // 是否已报告完成
}
```

---

## 10. 服务与集成

### 10.1 API 客户端（`src/services/api/`）

| 文件                        | 用途                                         |
|-----------------------------|----------------------------------------------|
| `client.ts`                 | Anthropic SDK 客户端，支持多提供商             |
| `claude.ts`                 | 消息流式传输与工具调用处理                    |
| `bootstrap.ts`              | 启动时获取引导数据                           |
| `usage.ts`                  | Token 用量记录                               |
| `errors.ts` / `errorUtils.ts` | 错误分类与处理                            |
| `logging.ts`                | API 请求/响应日志                            |
| `withRetry.ts`              | 指数退避重试逻辑                             |
| `filesApi.ts`               | 文件上传/下载                                |
| `sessionIngress.ts`         | 远程会话桥接                                 |
| `grove.ts`                  | Grove 集成                                   |
| `referral.ts`               | 推荐/通行证系统                              |

**支持的提供商：**
- Anthropic 直连 API
- AWS Bedrock
- Google Cloud Vertex AI
- Azure Foundry

### 10.2 MCP 集成（`src/services/mcp/`）

| 文件                    | 用途                                           |
|-------------------------|------------------------------------------------|
| `client.ts`             | MCP 客户端实现                                 |
| `types.ts`              | 服务器定义与连接类型                           |
| `config.ts`             | 配置加载与验证                                 |
| `auth.ts`               | MCP 服务器的 OAuth/认证                        |
| `officialRegistry.ts`   | 官方 MCP 服务器注册表                          |
| `InProcessTransport.ts` | 进程内 MCP 传输                                |
| `normalization.ts`      | URL/配置规范化                                 |
| `elicitationHandler.ts` | 通过 MCP 提示用户                              |

### 10.3 分析与遥测（`src/services/analytics/`）

| 文件                        | 用途                                       |
|-----------------------------|--------------------------------------------|
| `index.ts`                  | 事件日志 API                               |
| `growthbook.ts`             | 特性标志与 A/B 测试                        |
| `sink.ts`                   | 分析接收器配置                             |
| `datadog.ts`                | Datadog 集成                               |
| `firstPartyEventLogger.ts`  | 第一方分析                                 |
| `metadata.ts`               | 事件元数据增强                             |

### 10.4 上下文压缩（`src/services/compact/`）

| 文件                       | 用途                                        |
|----------------------------|---------------------------------------------|
| `compact.ts`               | 完整上下文窗口压缩                         |
| `autoCompact.ts`           | 自动压缩触发器                             |
| `microCompact.ts`          | 选择性消息修剪                             |
| `compactWarning.ts`        | 压缩用户警告                               |
| `sessionMemoryCompact.ts`  | 跨压缩的记忆持久化                         |

### 10.5 其他服务

| 目录/文件                     | 用途                                      |
|-------------------------------|-------------------------------------------|
| `SessionMemory/`              | 会话记忆持久化与转录                      |
| `MagicDocs/`                  | 智能文档生成                              |
| `AgentSummary/`               | Agent 执行摘要                            |
| `PromptSuggestion/`           | 建议的后续提示词                          |
| `extractMemories/`            | 从对话中提取学习内容                      |
| `plugins/`                    | 插件生命周期管理                          |
| `oauth/`                      | OAuth 客户端流程                          |
| `lsp/`                        | Language Server Protocol 客户端           |
| `remoteManagedSettings/`      | 远程配置同步                              |
| `settingsSync/`               | 设置同步                                  |
| `teamMemorySync/`             | 团队记忆同步                              |
| `policyLimits/`               | 速率限制与配额                            |
| `autoDream/`                  | 自动梦境后台功能                          |
| `tips/`                       | 上下文提示系统                            |
| `toolUseSummary/`             | 工具使用分析                              |
| `voice.ts` / `voiceStreamSTT.ts` | 语音输入处理                          |

---

## 11. UI 层

### 框架

UI 使用 **React** 构建，通过 **Ink** 渲染到终端。组件使用标准 React 模式（Hooks、Context、Props），但渲染为终端 ANSI 输出而非 DOM。

### 核心应用组件

| 组件               | 文件                          | 用途                          |
|--------------------|-------------------------------|-------------------------------|
| `App`              | `components/App.tsx`          | 根应用组件                    |
| `REPL`             | `screens/REPL.tsx`            | 主 REPL 界面                  |
| `Messages`         | `components/Messages.tsx`     | 对话消息列表                  |
| `PromptInput`      | `components/PromptInput/`     | 带自动补全的用户输入          |
| `StatusLine`       | `components/StatusLine.tsx`   | 底部状态栏                    |

### 组件分类

#### 消息展示
`Message.tsx`、`MessageRow.tsx`、`MessageResponse.tsx`、`MessageModel.tsx`、`MessageTimestamp.tsx`、`MessageSelector.tsx`、`messages/`（专用消息类型子目录）

#### 对话框与模态组件
`TrustDialog/`、`AutoModeOptInDialog.tsx`、`BypassPermissionsModeDialog.tsx`、`CostThresholdDialog.tsx`、`BridgeDialog.tsx`、`ExportDialog.tsx`、`InvalidConfigDialog.tsx`、`InvalidSettingsDialog.tsx`、`ManagedSettingsSecurityDialog/`、`IdeAutoConnectDialog.tsx`、`IdleReturnDialog.tsx`、`WorktreeExitDialog.tsx`、`RemoteEnvironmentDialog.tsx`

#### 代码展示
`HighlightedCode/`、`StructuredDiff/`、`FileEditToolDiff.tsx`

#### 设置与配置
`Settings/`、`ThemePicker.tsx`、`OutputStylePicker.tsx`、`ModelPicker.tsx`、`LanguagePicker.tsx`

#### 任务与 Agent UI
`tasks/`、`teams/`、`agents/`、`CoordinatorAgentStatus.tsx`、`TaskListV2.tsx`、`TeammateViewHeader.tsx`

#### 导航与搜索
`GlobalSearchDialog.tsx`、`HistorySearchDialog.tsx`、`QuickOpenDialog.tsx`、`SearchBox.tsx`

#### 设计系统
`design-system/`、`Spinner/`、`CustomSelect/`、`LogoV2/`、`HelpV2/`

#### 权限
`permissions/`（基于角色的访问对话框和提示）

---

## 12. 工具函数

`src/utils/` 目录包含 **300+ 文件**，提供底层功能。主要分类：

### Git 与版本控制
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `git.ts`        | Git 命令封装                                   |
| `git/`          | 扩展 Git 工具                                  |
| `gitDiff.ts`    | Diff 生成与解析                                |
| `gitSettings.ts`| Git 指令开关                                   |
| `github/`       | GitHub API 辅助工具                            |
| `worktree.ts`   | Git 工作树自动化                               |

### Shell 与进程
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `Shell.ts`      | Shell 执行封装                                 |
| `shell/`        | Shell 配置与辅助工具                           |
| `bash/`         | Bash 专用工具                                  |
| `powershell/`   | PowerShell 工具                                |
| `execFileNoThrow.ts` | 安全进程启动                              |
| `process.ts`    | 进程管理                                       |

### 认证与安全
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `auth.ts`       | API 密钥、OAuth、AWS/GCP 凭证管理              |
| `secureStorage/`| 钥匙串集成（macOS）                            |
| `permissions/`  | 权限规则、文件系统沙箱                         |
| `crypto.ts`     | 加密工具                                       |
| `sandbox/`      | 沙箱环境                                       |

### 配置
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `config.ts`     | `.claude/config.json` 管理                     |
| `settings/`     | `settings.json` 验证与应用                     |
| `env.ts`        | 静态环境变量                                   |
| `envDynamic.ts` | 动态环境检测                                   |
| `envUtils.ts`   | 环境变量解析                                   |
| `managedEnv.ts` | 托管环境配置                                   |

### 文件系统
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `claudemd.ts`   | CLAUDE.md 自动发现与解析                       |
| `fileStateCache.ts` | 文件变更追踪                               |
| `fileHistory.ts`| 文件快照（用于撤销）                           |
| `filePersistence/` | 持久化文件存储                              |
| `glob.ts`       | Glob 模式匹配                                  |
| `ripgrep.ts`    | Ripgrep 集成                                   |

### AI 与模型
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `model/`        | 模型选择与上下文窗口管理                       |
| `modelCost.ts`  | Token 定价表                                   |
| `thinking.ts`   | 扩展思考模式配置                               |
| `effort.ts`     | 任务工作量级别管理                             |
| `fastMode.ts`   | 速度优化模式                                   |
| `advisor.ts`    | AI 顾问集成                                    |
| `tokens.ts`     | Token 计数与估算                               |

### Agent 与集群
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `swarm/`        | 多 Agent 集群协调                              |
| `teammate.ts`   | 队友/Agent 模式工具                            |
| `forkedAgent.ts`| 分叉 Agent 进程管理                            |
| `agentContext.ts`| Agent 执行上下文                               |

### 性能与诊断
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `startupProfiler.ts` | 启动性能监控                              |
| `headlessProfiler.ts`| 运行时性能分析                             |
| `fpsTracker.ts` | 帧率指标                                       |
| `diagLogs.ts`   | 诊断日志（无 PII）                             |
| `debug.ts`      | 调试工具                                       |

### UI 辅助工具
| 文件/目录        | 用途                                           |
|-----------------|------------------------------------------------|
| `theme.ts`      | 主题管理                                       |
| `renderOptions.ts` | Ink 渲染配置                                |
| `format.ts`     | 数字/时长格式化                                |
| `markdown.ts`   | Markdown 处理                                  |
| `cliHighlight.ts` | CLI 语法高亮                                 |

---

## 13. 特殊模式

### 13.1 Bridge 模式（`src/bridge/`）
通过基于 WebSocket 的会话入口与 Claude.ai 保持常驻连接。支持持久化后台会话和远程访问。

### 13.2 Kairos / 助手模式（`src/assistant/`）
企业助手功能：
- 后台任务处理
- 推送通知
- GitHub Webhook 订阅
- 远程任务监控
- 通过 `KAIROS` 标志进行功能门控

### 13.3 协调器模式（`src/coordinator/`）
多 Agent 编排：
- 任务面板管理
- Agent 交互协调
- 通过 `COORDINATOR_MODE` 标志进行功能门控

### 13.4 语音模式（`src/voice/`）
语音输入/输出支持：
- 语音转文字（STT）集成
- 文字转语音
- 语音转录
- 语音关键词检测

### 13.5 计划模式
用于在编码前设计实现策略的只读模式：
- 在 `.claude/plans/` 中创建计划文件
- 限制工具为只读操作
- 执行前需要用户明确批准
- 由 `EnterPlanModeTool` / `ExitPlanModeTool` 管理

### 13.6 工作树模式
用于安全实验的 Git 工作树隔离：
- 创建临时 Git 工作树
- 支持 tmux 会话管理
- 临时分支创建
- 变更可以合并或丢弃

### 13.7 Vim 模式（`src/vim/`）
终端输入的 Vim 键绑定集成。

---

## 14. 插件与技能

### 插件系统（`src/plugins/`）

- `plugins/bundled/` 中的**内置插件**（键盘快捷键、主题等）
- 通过 `PluginInstallationManager` 管理插件生命周期
- 用于插件管理的 CLI 命令
- 通过 `reload-plugins` 命令支持热重载

### 技能系统（`src/skills/`）

- `skills/bundled/` 中的**内置技能**（提交、审查、简化等）
- 技能是可通过 `/skill-name` 调用的命名提示词
- 技能发现与执行引擎
- 变更检测支持实时更新

---

## 15. 钩子与可扩展性

### 钩子模式（`src/schemas/hooks.ts`）

通过 Zod 验证定义：
- `HookEvent` — 执行前/执行后生命周期钩子
- `PromptRequest` / `PromptResponse` — 用户提示协议
- 同步和异步钩子响应模式
- 权限决策钩子

### React Hooks（`src/hooks/`）

| Hook                | 用途                                       |
|---------------------|--------------------------------------------|
| `useSettings`       | 设置变更检测                               |
| `useTerminalSize`   | 终端尺寸追踪                               |
| `useExitOnCtrlC`    | 信号处理                                   |
| `useBlink`          | 光标闪烁动画                               |
| `useDoublePress`    | 双按键检测                                 |
| `useCanUseTool`     | 工具权限验证                               |

### 工具 Hooks（`src/utils/hooks/`）

用于 Shell 配置、权限状态和工具行为的额外 Hooks。

---

## 16. 文件统计

| 分类                        | 数量    |
|-----------------------------|---------|
| **TypeScript 文件总数**     | 1,884   |
| **命令模块**                | 101     |
| **工具实现**                | 41      |
| **UI 组件**                 | 130+    |
| **工具函数文件**            | 300+    |
| **服务模块**                | 35+     |
| **顶层源文件**              | 18      |
| **入口点**                  | 6       |
| **`src/` 中的子目录**       | 37      |

---

## 17. 架构模式

### 延迟加载与死代码消除
通过 `require()` 条件导入，由 Bun 的 `feature()` 标志门控。这使得在打包时可以对整个子系统（例如 `KAIROS`、`COORDINATOR_MODE`）进行 Tree-Shaking。

```typescript
const assistantModule = feature('KAIROS')
  ? require('./assistant/index.js')
  : null;
```

### 基于工具的执行模型
与外部世界的每次交互都通过已注册的 `Tool` 进行。工具具有：
- 声明式 JSON Schema 输入
- 执行前的权限检查
- 执行期间的进度报告
- 结构化结果输出

### 命令模式
斜杠命令是模块化目录，每个目录导出一个 `Command` 对象。`commands.ts` 中的注册表将它们聚合在一起，支持可选的特性门控条件。

### 消息驱动架构
对话是一系列类型化消息（`UserMessage`、`AssistantMessage`、`SystemMessage`、`ProgressMessage` 等），由 `QueryEngine` 管理。工具结果作为 `ToolResultBlockParam` 消息注入。

### 上下文压缩
当对话上下文接近模型窗口限制时，系统自动压缩旧消息，同时保留关键上下文。提供多种策略：完整压缩、微压缩（选择性修剪）和会话记忆持久化。

### 权限优先安全模型
每次工具执行都经过权限检查。模式包括：
- **询问** — 每次操作都提示用户
- **自动允许** — 基于信任规则允许
- **拒绝** — 阻止特定操作
- **文件系统沙箱** — 将文件访问限制在项目目录内

### React 终端 UI
整个 UI 是通过 Ink 渲染的 React 组件树。这使得以下特性成为可能：
- 声明式 UI 更新
- 组件组合与复用
- 状态驱动渲染
- 针对终端特定行为的 Hooks（光标、窗口调整、键盘）

---

---

## 贡献

欢迎贡献！如果你发现了额外的架构细节、发现了不准确之处，或想扩展特定子系统的覆盖范围：

1. **Fork** 本仓库
2. 为你的更改**创建分支**（`git checkout -b fix/tool-system-details`）
3. **提交 Pull Request**，并附上清晰的描述说明你添加或修正了什么

请保持贡献的事实性和技术准确性。这是一份参考文档——推测内容应明确标注。

---

> **免责声明：** 这是一份非官方的独立分析。Claude Code 是 [Anthropic](https://www.anthropic.com/) 的产品。所有商标归其各自所有者所有。

*生成于 2025-03-31。基于对 Claude Code 源码树的分析。*
