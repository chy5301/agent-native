# A2UI 与 AG-UI 深度调研

> 调研日期：2026-03-17
> 信息来源：官方文档、GitHub 仓库、技术博客

---

## 一、A2UI（Google）— Agent-to-User Interface

### 1.1 项目概况

- **发起方**：Google，Apache 2.0 开源
- **GitHub**：https://github.com/google/A2UI （13.3k stars, 452 commits, 47 contributors）
- **当前版本**：v0.8（Public Preview, Stable）；v0.9（Draft）
- **代码构成**：TypeScript 86.2%, Python 11.7%
- **定位**：声明式生成式 UI 规范，让 Agent 以结构化数据描述 UI，而非生成可执行代码

### 1.2 核心工作原理

A2UI 是一个 **基于 JSONL 的流式协议**。Agent 不生成 HTML/CSS/JS，而是发送声明式 JSON 描述"我想展示什么"，由客户端负责"如何渲染"。

**交互循环（Emit-Render-Signal-Reason）**：

1. **Emit**：Agent 发送 JSONL，描述 UI 结构和数据绑定
2. **Render**：客户端将抽象组件映射到原生 Widget（React / Flutter / Angular / Lit）
3. **Interact**：用户与渲染后的 UI 交互
4. **Signal**：渲染器发送结构化 `userAction` 事件（不是自由文本）
5. **Reason**：Agent 消费事件，更新 UI，循环重启

### 1.3 协议消息类型（Server → Client）

| 消息类型 | 用途 |
|---|---|
| `surfaceUpdate` | 发送组件定义（扁平列表 + ID 引用） |
| `dataModelUpdate` | 更新应用状态（独立于 UI 结构） |
| `beginRendering` | 通知客户端开始渲染（指定根组件和 Catalog） |
| `deleteSurface` | 删除一个 UI Surface 及其内容 |

**Client → Server**：

| 消息类型 | 用途 |
|---|---|
| `userAction` | 用户操作事件（包含 name, surfaceId, sourceComponentId, timestamp, context） |
| `error` | 客户端错误上报 |

### 1.4 组件体系

**核心设计**：组件使用 **扁平邻接表** 结构（非嵌套树），每个组件有唯一 ID，通过 ID 引用建立父子关系。这种设计对 LLM 增量生成和渐进式渲染非常友好。

**标准组件清单（v0.8）**：

| 类别 | 组件 | 说明 |
|---|---|---|
| **布局** | Row | 水平排列子元素 |
| | Column | 垂直排列子元素 |
| | List | 可滚动列表，支持静态子元素和动态模板 |
| **展示** | Text | 文本显示（支持 h1-h5, caption, body 等 usageHint） |
| | Image | 图片展示（URL source, fit 属性） |
| | Icon | 图标渲染 |
| | Divider | 分隔线 |
| **交互** | Button | 可点击按钮，触发 action |
| | TextField | 文本输入（shortText, longText, number, obscured, date） |
| | CheckBox | 布尔开关 |
| | Slider | 数值范围输入（min/max） |
| | DateTimeInput | 日期/时间选择器 |
| | ChoicePicker / MultipleChoice | 单选/多选 |
| **容器** | Card | 带边框和内边距的容器 |
| | Modal | 弹出对话框 |
| | Tabs | 标签页切换 |

**自定义组件**：客户端可注册自定义组件（图表、地图、专业可视化），Agent 通过 Catalog 系统发现并使用。

### 1.5 数据绑定机制（BoundValue）

```json
{
  "literalString": "静态值",
  "path": "/model/location"
}
```

- **仅 literal**：静态显示
- **仅 path**：从数据模型动态解析
- **两者皆有**：初始化数据模型并绑定，支持双向更新

### 1.6 Catalog 系统

Catalog 定义了 Server 与 Client 之间的 UI 契约：

1. **预定义 Catalog**：通过 catalogId 引用（如标准 Catalog）
2. **自定义 Catalog**：Server 声明支持的 Catalog
3. **内联 Catalog**：运行时定义（需 Server 声明 `acceptsInlineCatalogs`）

协商流程：Server 广播支持的 Catalog → Client 声明自己支持的 Catalog → Server 在 `beginRendering` 中选择

### 1.7 安全模型

A2UI 的安全性建立在 **"数据而非代码"** 的根本原则上：

- **声明式 JSON**：不执行任何代码，消除注入攻击和远程代码执行风险
- **受信组件目录**：Agent 只能请求客户端预批准的组件，无法引入任意 UI 元素
- **开发者控制权**：渲染层由客户端应用控制，Agent 无法绕过安全边界
- **跨信任边界安全**：在多 Agent 场景下，远程 Sub-agent 的 UI 输出仍受宿主应用的组件白名单约束

**对比 MCP Apps 的 iframe 沙箱方案**：A2UI 的声明式方法从根本上避免了需要沙箱的问题，因为根本没有可执行代码需要隔离。

### 1.8 v0.8 能力边界与限制

**能力**：
- 完整的声明式 UI 生成和渲染管线
- JSONL 流式传输，支持渐进式渲染
- 多 Surface 支持（独立的 UI 区域）
- 数据绑定和动态更新
- 跨平台（Web, Mobile, Desktop）
- 通过 A2A Extension 支持多 Agent 协作

**明确限制**：
- **无转换器**：不支持格式化器、条件逻辑等——所有数据转换必须在 Server 端完成
- **无 Client 端逻辑**：纯声明式，客户端不能运行 Agent 定义的任何逻辑
- **规范仍在演进**："the specification and implementations are functional but are still evolving"
- **渲染器支持有限**：目前主要有 Web (Node.js) 和 Flutter，React/Jetpack Compose/SwiftUI 在路线图中
- **组件类型依赖 Catalog**：核心协议本身不硬编码组件类型

### 1.9 传输方式

A2UI 本身不定义传输层，依赖：
- **A2A Protocol**（Google 的 Agent-to-Agent 协议）
- **AG-UI**（CopilotKit 的 Agent-User 交互协议）
- **SSE（Server-Sent Events）** 作为底层流式传输

---

## 二、AG-UI（CopilotKit）— Agent-User Interaction Protocol

### 2.1 项目概况

- **发起方**：CopilotKit 团队，MIT 开源
- **GitHub**：https://github.com/ag-ui-protocol/ag-ui
- **文档**：https://docs.ag-ui.com/
- **定位**：开放、轻量的事件驱动协议，标准化 AI Agent 与用户界面之间的实时双向通信

### 2.2 架构定位

AG-UI 是 Agentic 技术栈中的 **传输层/运行时通道**：

```
Agent ↔ Tools    →  MCP（Model Context Protocol, Anthropic）
Agent ↔ Agent    →  A2A（Agent-to-Agent, Google）
Agent ↔ User     →  AG-UI（本协议）
```

类比：AG-UI 是 **管道**，A2UI 是管道里的 **内容**。

### 2.3 通信机制

**基于事件的架构**，构建在 HTTP 和 WebSocket 之上：

- Agent 后端发射标准化事件（约 16+ 种事件类型）
- Agent 接收简化的 AG-UI 兼容输入
- 中间件层支持多种传输方式（SSE, WebSocket, Webhook）
- 支持松散的事件格式匹配

### 2.4 完整事件类型体系

**1. 生命周期事件（Lifecycle）**

| 事件 | 说明 |
|---|---|
| `RunStarted` | 启动执行上下文（runId, threadId, 可选 parentRunId） |
| `RunFinished` | 标记成功完成，携带可选 result |
| `RunError` | 不可恢复的错误（message + error code） |
| `StepStarted` / `StepFinished` | 可选的子任务粒度跟踪 |

**2. 文本消息事件（Text Message）**

| 事件 | 说明 |
|---|---|
| `TextMessageStart` | 初始化消息（messageId, role） |
| `TextMessageContent` | 增量文本块（delta 字段） |
| `TextMessageEnd` | 消息完成 |
| `TextMessageChunk` | 便捷包装器，自动展开为 Start→Content→End |

**3. 工具调用事件（Tool Call）**

| 事件 | 说明 |
|---|---|
| `ToolCallStart` | 宣告工具调用（toolCallId, toolCallName） |
| `ToolCallArgs` | 流式传输参数数据块 |
| `ToolCallEnd` | 参数传输完成 |
| `ToolCallResult` | 执行结果（content） |
| `ToolCallChunk` | 便捷事件，自动展开 |

**4. 状态管理事件（State Management）**

| 事件 | 说明 |
|---|---|
| `StateSnapshot` | 完整状态快照（替换先前状态） |
| `StateDelta` | 增量更新，使用 JSON Patch (RFC 6902) |
| `MessagesSnapshot` | 完整对话历史 |

**5. 活动事件（Activity）**

| 事件 | 说明 |
|---|---|
| `ActivitySnapshot` | 完整活动状态 |
| `ActivityDelta` | 活动增量更新（RFC 6902 JSON Patch） |

**6. 推理事件（Reasoning）**

| 事件 | 说明 |
|---|---|
| `ReasoningStart` / `ReasoningEnd` | 推理上下文边界 |
| `ReasoningMessageStart/Content/End` | 流式输出可见的推理片段 |
| `ReasoningMessageChunk` | 便捷事件 |
| `ReasoningEncryptedValue` | 加密的 chain-of-thought（客户端不可见，跨轮次保持） |

**7. 特殊事件（Special）**

| 事件 | 说明 |
|---|---|
| `Raw` | 包装外部系统事件 |
| `Custom` | 应用自定义事件（name + value） |

### 2.5 共享状态双向同步机制

AG-UI 的核心创新是 **Snapshot-Delta 模式**：

1. **StateSnapshot**：发送完整 JSON 状态快照（用于初始同步或偶尔的全量刷新）
2. **StateDelta**：发送增量变更，使用 **JSON Patch (RFC 6902)** 格式

**工作流程**：
- Agent 启动时发送 StateSnapshot 初始化前端状态
- 后续变更通过 StateDelta 流式推送（如 `{"op": "add", "path": "/items/5", "value": "hello"}`）
- 偶尔发送 StateSnapshot 重新同步
- 支持 **读写双向共享状态存储**（typed stores），冲突解决通过事件溯源差异实现

**实际效果**：类似协同编辑——Agent 和用户界面共享一个"文档"，通过高效的增量更新保持同步，互不覆盖。

### 2.6 支持的 Agent 框架

**一方支持（内建文档和集成）**：
- LangGraph
- CrewAI
- Microsoft Agent Framework
- Google ADK
- AWS Strands Agents
- Mastra
- Pydantic AI
- Agno
- LlamaIndex
- AG2

**开发中**：
- AWS Bedrock Agents
- OpenAI Agent SDK
- Cloudflare Agents

### 2.7 前端集成方式

**客户端实现**：
- **CopilotKit**：一方客户端，功能最完整
- **Terminal + Agent**：终端客户端
- **React Native**：开发中

**社区 SDK**：Kotlin, Golang, Dart, Java, Rust, Ruby, .NET（开发中）

### 2.8 关键高级能力

- **Human-in-the-loop**：暂停、审批、编辑、重试、升级——执行中途介入
- **Agent 引导（Steering）**：用户实时输入动态重定向 Agent 执行
- **Sub-agent 组合**：嵌套委托，带作用域状态、追踪和取消
- **工具输出流式渲染**：长时间运行效果的实时渲染
- **多模态**：文件、图片、音频、转录文本，带注释支持

---

## 三、A2UI 与 AG-UI 的互补关系

### 3.1 层次定位

```
┌──────────────────────────────────────┐
│         用户界面（React/Flutter/...）  │
├──────────────────────────────────────┤
│  AG-UI：运行时通信通道（传输层）       │  ← 如何传递
├──────────────────────────────────────┤
│  A2UI：声明式 UI 描述（内容层）        │  ← 传递什么
├──────────────────────────────────────┤
│  MCP：工具调用协议（能力层）           │  ← Agent 能做什么
├──────────────────────────────────────┤
│  A2A：Agent 间协作协议（协作层）       │  ← Agent 间如何协作
└──────────────────────────────────────┘
```

### 3.2 具体配合方式

1. Agent 通过 MCP 调用后端工具获取数据
2. Agent 生成 A2UI 声明式 JSON 描述想要展示的 UI
3. A2UI 载荷通过 AG-UI 事件流实时推送到前端
4. 前端根据 A2UI 描述渲染原生组件
5. 用户操作通过 AG-UI 事件回传给 Agent
6. Agent 处理后发送更新的 A2UI 载荷

### 3.3 关键区分

| 维度 | A2UI | AG-UI |
|---|---|---|
| 解决的问题 | **什么** UI | **如何** 传递 |
| 协议类型 | UI 描述规范 | 运行时通信协议 |
| 状态管理 | 无状态（UI 描述） | 有状态（Snapshot-Delta） |
| 安全重点 | 组件白名单 | 类型化操作 + Schema 验证 |
| 发起方 | Google | CopilotKit |
| 许可证 | Apache 2.0 | MIT |

---

## 四、传统双模方案对比

### 4.1 Streamlit / Gradio 作为 Agent UI 层

**Streamlit**：
- 定位：Python 数据应用和仪表盘
- 优势：自定义布局丰富、数据可视化能力强、复杂 UI/UX
- 局限：不是为 Agent 交互设计的，缺乏双向实时通信、无标准化 Agent 事件协议
- Agent 集成：通过 Python API 调用，本质上是"Agent 写 Python 脚本生成 UI"

**Gradio**：
- 定位：ML 模型演示和 LLM 聊天界面
- 优势：快速搭建 AI demo、Hugging Face 生态集成
- 局限：UI 灵活性不如 Streamlit，难以构建复杂应用
- Agent 集成：更自然地支持聊天式交互，但仍非标准化协议

**与 A2UI/AG-UI 的根本差异**：
- Streamlit/Gradio 是 **框架**，绑定 Python 生态，输出是完整 Web 应用
- A2UI/AG-UI 是 **协议**，语言无关，定义的是 Agent 与 UI 之间的通信契约
- Streamlit/Gradio 的 UI 由开发者预定义；A2UI 的 UI 由 Agent 运行时动态生成

### 4.2 CLI + Web Dashboard 分离架构

传统模式中，Agent 工具常见的双模方案：

- CLI 用于开发者/运维人员直接交互
- Web Dashboard 用于可视化监控和管理
- 两者共享后端 API，但前端完全独立

**问题**：
- 维护两套前端的成本高
- CLI 和 Web UI 的功能对等性难以保证
- 缺乏标准化的 Agent 交互协议，每个工具自造轮子

### 4.3 同时提供 CLI 和 Web UI 的 Agent 工具案例

**Chainlit**：
- Python 框架，专注构建对话式 AI 应用
- 提供 Web UI（聊天界面）
- 2025 年 5 月原团队退出活跃开发，由社区维护者接管
- 本质仍是 Web 应用框架，非标准化协议

**Open WebUI**：
- v0.7 引入内建工具（web search, memory, notes, knowledge bases）
- 主要是 Web 界面，通过 mcpo 适配 OpenAPI
- 面向终端用户而非开发者集成

**Open Interpreter**：
- 桌面 Agent，可编辑文档、填写表单
- 提供 CLI 和桌面应用双模式
- 但两种模式更多是独立客户端，非统一协议

**Dify / Flowise / n8n / LangFlow**：
- 都提供 Web UI 用于可视化编排 Agent 工作流
- 同时暴露 API 接口供程序化调用
- 但 CLI 支持有限或不是一等公民
- 本质是"低代码平台 + API"，不是 CLI-native 设计

### 4.4 新范式的优势

A2UI + AG-UI 的组合代表了一种根本不同的思路：

| 传统双模 | A2UI + AG-UI |
|---|---|
| 开发者预定义 UI | Agent 运行时动态生成 UI |
| CLI 和 Web UI 分别构建 | 统一协议，客户端自由实现 |
| 框架绑定（Python/JS） | 协议驱动，语言无关 |
| UI 是应用的一部分 | UI 是 Agent 的输出 |
| 状态管理由应用框架处理 | 标准化的 Snapshot-Delta 同步 |

---

## 五、对 Agent-Native 命题的启示

### 5.1 A2UI 验证了"GUI 是翻译层"

A2UI 的存在本身就证明了：当 Agent 成为主要"用户"时，UI 不再需要人类设计师一像素一像素打磨。Agent 发送声明式数据，客户端按自己的设计系统渲染。**UI 从"产品"降级为"协议的渲染终端"**。

### 5.2 AG-UI 定义了 Agent 时代的交互基础设施

AG-UI 的 16+ 种事件类型覆盖了 Agent 与用户交互的完整生命周期。这不是 REST API 的简单扩展，而是一套为 Agent 实时性、流式输出、中途介入等特征量身定制的通信协议。

### 5.3 协议层而非框架层

最关键的转变：从 Streamlit/Gradio 这样的 **框架** 走向 A2UI/AG-UI 这样的 **协议**。框架绑定生态和语言，协议释放可组合性。这与 "Skill 替代 App" 的命题高度吻合——Skill 需要的是标准化接口，不是包裹它的应用外壳。

### 5.4 安全边界的重新定义

A2UI 的 "数据而非代码" 安全模型代表了 Agent 时代的安全思维：不是隔离不信任的代码（iframe 沙箱），而是从根本上不允许 Agent 提交可执行代码。这是一种更根本的安全范式。

---

## 参考来源

- [A2UI 官方文档](https://a2ui.org/)
- [A2UI v0.8 规范](https://a2ui.org/specification/v0.8-a2ui/)
- [A2UI 组件画廊](https://a2ui.org/reference/components/)
- [A2UI GitHub 仓库](https://github.com/google/A2UI)
- [AG-UI 官方文档](https://docs.ag-ui.com/introduction)
- [AG-UI 事件类型详解](https://docs.ag-ui.com/concepts/events)
- [AG-UI GitHub 仓库](https://github.com/ag-ui-protocol/ag-ui)
- [AG-UI 与 A2UI 的互补关系 — CopilotKit Blog](https://www.copilotkit.ai/blog/ag-ui-and-a2ui-explained-how-the-emerging-agentic-stack-fits-together)
- [Agentic UI 协议对比 — CopilotKit Blog](https://www.copilotkit.ai/blog/the-state-of-agentic-ui-comparing-ag-ui-mcp-ui-and-a2ui-protocols)
- [A2UI vs AG-UI — A2UI.sh](https://a2ui.sh/articles/a2ui-vs-ag-ui)
- [A2UI 2026 完整指南 — DEV Community](https://dev.to/czmilo/the-a2ui-protocol-a-2026-complete-guide-to-agent-driven-interfaces-2l3c)
- [Agent UI 标准化 — The New Stack](https://thenewstack.io/agent-ui-standards-multiply-mcp-apps-and-googles-a2ui/)
- [Chainlit GitHub](https://github.com/Chainlit/chainlit)
- [Open WebUI 文档](https://docs.openwebui.com/features/extensibility/plugin/tools/)

---

## 六、推荐方案

### 用户场景回顾

用户计划开发两个面向 Agent 的工具：
- 输入主要面向 Agent（CLI/结构化）
- 输出需要支持 GUI 可视化（面向人类）和结构化输出（面向 Agent）
- 部署后主要由 Agent 调用，人类查看结果/过程

### 方案推荐：分层递进

| 阶段 | 方案 | 成本 | 适用场景 |
|------|------|------|---------|
| **MVP** | CLI `--json` 输出 + 简单 HTML 报告生成 | 低 | 快速验证，人类通过浏览器查看静态报告 |
| **进阶** | CLI + Streamlit/Gradio 快速 Dashboard | 中 | 需要实时交互式可视化时 |
| **标准化** | CLI + A2UI 声明式 UI | 中→高 | 当 A2UI 稳定到 v1.0，且需要跨平台渲染时 |

### 推荐的 MVP 双模架构

```
用户/Agent 输入
       ↓
   CLI 工具（Python Click）
       ↓
  ┌────┴────┐
  ↓         ↓
--json    --report
结构化     HTML 报告
JSON      （可选）
输出       生成
```

**理由**：
1. `--json` 是 Agent 消费的主路径，也是 P0 必备
2. `--report` 生成静态 HTML 文件，人类用浏览器打开即可，无需额外进程
3. 开发成本极低，不引入前端框架依赖
4. 后续可平滑升级到 A2UI（将 HTML 报告替换为 A2UI JSONL 输出）

### A2UI 的引入时机

当以下条件**同时满足**时考虑引入 A2UI：
- A2UI 达到 v1.0 稳定版
- 工具需要**实时交互式** UI（不只是查看静态报告）
- 需要**跨平台**渲染（Web + Mobile）
- Agent 需要根据用户在 UI 上的操作**动态调整**行为
