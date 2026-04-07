# Claude Code 源代码快照（安全研究专用）

> 本仓库镜像了一份**公开暴露的 Claude Code 源代码快照**，该快照于**2026年3月31日**通过 npm 分发包中的 source map 泄露事件被公开获取。本仓库的维护目的为**教育研究、防御性安全分析、软件供应链研究**。

---

## 研究背景

本仓库由一名**在校大学生**维护，研究方向包括：

- 软件供应链泄露与构建产物暴露问题
- 安全的软件工程实践
- 智能体化开发者工具架构
- 真实世界 CLI 系统的防御性分析

本归档库旨在支持：

- 教学研究
- 安全研究实践
- 架构评审
- 打包与发布流程缺陷的讨论

本仓库**不主张**对原始代码的所有权，也不应被解读为 Anthropic 官方仓库。

---

## 公开快照的泄露过程

Chaofan Shou（@Fried_rice）公开指出，Claude Code 源代码可通过 npm 包中暴露的 `.map` 文件获取：

> **"Claude code 源代码通过其 npm 仓库中的 map 文件泄露了！"**
>
> —— [@Fried_rice，2026年3月31日](https://x.com/Fried_rice/status/2038894956459290963)

该已发布的 source map 引用了存储在 Anthropic R2 存储桶中的未混淆 TypeScript 源码，导致 `src/` 目录快照可被公开下载。

---

## 仓库范围

Claude Code 是 Anthropic 推出的 CLI 工具，用于在终端中与 Claude 交互，完成软件工程相关任务，例如编辑文件、运行命令、搜索代码库、协调工作流等。

本仓库包含了 `src/` 目录的镜像快照，用于研究与分析。

- **公开暴露时间**：2026-03-31
- **开发语言**：TypeScript
- **运行时**：Bun
- **终端 UI**：React + [Ink](https://github.com/vadimdemedes/ink)
- **代码规模**：约 1900 个文件，51.2 万+ 行代码

---

## 目录结构

```text
src/
├── main.tsx                 # 入口点编排（基于 Commander.js 的 CLI 路径）
├── commands.ts              # 命令注册表
├── tools.ts                 # 工具注册表
├── Tool.ts                  # 工具类型定义
├── QueryEngine.ts           # LLM 查询引擎
├── context.ts               # 系统/用户上下文收集
├── cost-tracker.ts          # Token 成本追踪
│
├── commands/                # 斜杠命令实现（约50个）
├── tools/                   # 智能体工具实现（约40个）
├── components/              # Ink UI 组件（约140个）
├── hooks/                   # React 钩子
├── services/                # 外部服务集成
├── screens/                 # 全屏 UI（诊断工具、REPL、会话恢复）
├── types/                   # TypeScript 类型定义
├── utils/                   # 工具函数
│
├── bridge/                  # IDE 与远程控制桥接层
├── coordinator/             # 多智能体协调器
├── plugins/                 # 插件系统
├── skills/                  # 技能系统
├── keybindings/             # 快捷键配置
├── vim/                     # Vim 模式
├── voice/                   # 语音输入
├── remote/                  # 远程会话
├── server/                  # 服务端模式
├── memdir/                  # 持久化内存目录
├── tasks/                   # 任务管理
├── state/                   # 状态管理
├── migrations/              # 配置迁移
├── schemas/                 # 配置模式（Zod）
├── entrypoints/             # 初始化逻辑
├── ink/                     # Ink 渲染器封装
├── buddy/                   # 伴生精灵
├── native-ts/               # 原生 TypeScript 工具集
├── outputStyles/            # 输出样式
├── query/                   # 查询流水线
└── upstreamproxy/           # 代理配置
```



## 架构概述

### 1. 工具系统（`src/tools/`）

Claude Code 可调用的每一个工具均实现为独立模块。每个工具定义了输入模式、权限模型和执行逻辑。

|                   工具                   |             描述              |
| :--------------------------------------: | :---------------------------: |
|                `BashTool`                |        Shell 命令执行         |
|              `FileReadTool`              | 文件读取（图片、PDF、笔记本） |
|             `FileWriteTool`              |        文件创建 / 覆盖        |
|              `FileEditTool`              |  部分文件修改（字符串替换）   |
|                `GlobTool`                |       文件模式匹配搜索        |
|                `GrepTool`                |    基于 ripgrep 的内容搜索    |
|              `WebFetchTool`              |         URL 内容获取          |
|             `WebSearchTool`              |           网络搜索            |
|               `AgentTool`                |         子智能体生成          |
|               `SkillTool`                |           技能执行            |
|                `MCPTool`                 |      MCP 服务端工具调用       |
|                `LSPTool`                 |   语言服务器协议（LSP）集成   |
|            `NotebookEditTool`            |      Jupyter 笔记本编辑       |
|   `TaskCreateTool` / `TaskUpdateTool`    |        任务创建与管理         |
|            `SendMessageTool`             |       智能体间消息传递        |
|   `TeamCreateTool` / `TeamDeleteTool`    |        团队智能体管理         |
| `EnterPlanModeTool` / `ExitPlanModeTool` |         计划模式切换          |
| `EnterWorktreeTool` / `ExitWorktreeTool` |       Git worktree 隔离       |
|             `ToolSearchTool`             |         延迟工具发现          |
|             `CronCreateTool`             |        定时触发器创建         |
|           `RemoteTriggerTool`            |          远程触发器           |
|               `SleepTool`                |         主动模式等待          |
|          `SyntheticOutputTool`           |        结构化输出生成         |

### 2. 命令系统（`src/commands/`）

用户通过 `/` 前缀调用的面向用户的斜杠命令。

|         命令         |      描述      |
| :------------------: | :------------: |
|      `/commit`       | 创建 git 提交  |
|      `/review`       |    代码评审    |
|      `/compact`      |   上下文压缩   |
|        `/mcp`        | MCP 服务端管理 |
|      `/config`       |    设置管理    |
|      `/doctor`       |    环境诊断    |
| `/login` / `/logout` |    身份认证    |
|      `/memory`       | 持久化内存管理 |
|      `/skills`       |    技能管理    |
|       `/tasks`       |    任务管理    |
|        `/vim`        |  Vim 模式切换  |
|       `/diff`        |    查看变更    |
|       `/cost`        |  查看使用成本  |
|       `/theme`       |    切换主题    |
|      `/context`      |  上下文可视化  |
|    `/pr_comments`    |  查看 PR 评论  |
|      `/resume`       | 恢复之前的会话 |
|       `/share`       |    分享会话    |
|      `/desktop`      |  桌面应用切换  |
|      `/mobile`       |  移动应用切换  |

### 3. 服务层（`src/services/`）

|           服务           |                   描述                   |
| :----------------------: | :--------------------------------------: |
|          `api/`          | Anthropic API 客户端、文件 API、引导程序 |
|          `mcp/`          |  模型上下文协议（MCP）服务端连接与管理   |
|         `oauth/`         |            OAuth 2.0 认证流程            |
|          `lsp/`          |       语言服务器协议（LSP）管理器        |
|       `analytics/`       |   基于 GrowthBook 的功能开关与数据分析   |
|        `plugins/`        |                插件加载器                |
|        `compact/`        |              对话上下文压缩              |
|     `policyLimits/`      |               组织策略限制               |
| `remoteManagedSettings/` |               远程托管设置               |
|    `extractMemories/`    |               自动内存提取               |
|   `tokenEstimation.ts`   |              Token 数量估算              |
|    `teamMemorySync/`     |               团队内存同步               |

### 4. 桥接系统（`src/bridge/`）

双向通信层，用于连接 IDE 扩展（VS Code、JetBrains）与 Claude Code CLI。

- `bridgeMain.ts` — 桥接主循环
- `bridgeMessaging.ts` — 消息协议
- `bridgePermissionCallbacks.ts` — 权限回调
- `replBridge.ts` — REPL 会话桥接
- `jwtUtils.ts` — 基于 JWT 的认证
- `sessionRunner.ts` — 会话执行管理

### 5. 权限系统（`src/hooks/toolPermission/`）

对每一次工具调用进行权限检查。根据配置的权限模式（`default`、`plan`、`bypassPermissions`、`auto` 等），要么提示用户批准 / 拒绝，要么自动解析权限。

### 6. 功能开关

通过 Bun 的 `bun:bundle` 功能开关实现无效代码剔除：

```typescript
import { feature } from 'bun:bundle'

// 未激活的代码会在构建时被完全剥离
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

关键开关：`PROACTIVE`、`KAIROS`、`BRIDGE_MODE`、`DAEMON`、`VOICE_MODE`、`AGENT_TRIGGERS`、`MONITOR_TOOL`

------

## 核心文件详情

### `QueryEngine.ts`（约 4.6 万行）

LLM API 调用的核心引擎。处理流式响应、工具调用循环、思考模式、重试逻辑和 Token 计数。

### `Tool.ts`（约 2.9 万行）

定义所有工具的基础类型和接口 —— 输入模式、权限模型和进度状态类型。

### `commands.ts`（约 2.5 万行）

管理所有斜杠命令的注册与执行。使用条件导入根据环境加载不同的命令集。

### `main.tsx`

基于 Commander.js 的 CLI 解析器和 React/Ink 渲染器初始化。启动时，会叠加 MDM 设置、钥匙串预取和 GrowthBook 初始化，以加快启动速度。

------

## 技术栈

|   类别   |                           技术选型                           |
| :------: | :----------------------------------------------------------: |
|  运行时  |                    [Bun](https://bun.sh)                     |
| 开发语言 |                    TypeScript（严格模式）                    |
| 终端 UI  | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI 解析 | [Commander.js](https://github.com/tj/commander.js)（extra-typings） |
| 模式验证 |                  [Zod v4](https://zod.dev)                   |
| 代码搜索 |       [ripgrep](https://github.com/BurntSushi/ripgrep)       |
|   协议   |       [MCP SDK](https://modelcontextprotocol.io)、LSP        |
|   API    |         [Anthropic SDK](https://docs.anthropic.com)          |
| 可观测性 |                     OpenTelemetry + gRPC                     |
| 功能开关 |                          GrowthBook                          |
|   认证   |                OAuth 2.0、JWT、macOS Keychain                |

------

## 关键设计模式

### 并行预取

通过在重量级模块评估开始前并行预取 MDM 设置、钥匙串读取和 API 预连接，优化启动时间。

```typescript
// main.tsx — 在其他导入前作为副作用触发
startMdmRawRead()
startKeychainPrefetch()
```

### 懒加载

重量级模块（OpenTelemetry、gRPC、数据分析、部分功能门控子系统）通过动态 `import()` 延迟加载，直到实际需要时才引入。

### 智能体集群

通过 `AgentTool` 生成子智能体，由 `coordinator/` 处理多智能体编排。`TeamCreateTool` 支持团队级并行工作。

### 技能系统

`skills/` 中定义的可复用工作流通过 `SkillTool` 执行。用户可添加自定义技能。

### 插件架构

内置和第三方插件通过 `plugins/` 子系统加载。

------

## 研究 / 所有权声明

- 本仓库是**教育和防御性安全研究归档库**，由在校大学生维护。
- 本仓库的存在旨在研究源代码暴露、打包缺陷以及现代智能体化 CLI 系统的架构。
- Claude Code 原始源代码的所有权归 **Anthropic** 所有。
- 本仓库**与 Anthropic 无关联、未获其认可、亦非其维护**。

