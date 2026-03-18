# Agent-Native 生态图谱

> 追踪正在实践 Agent-Native 理念的项目、协议和工具。
> 最后更新：2026-03-18

## Agent 运行时

| 项目 | 类型 | 交互方式 | 备注 |
|------|------|---------|------|
| Claude Code | CLI Agent | CLI | Anthropic 官方，Skill 生态最成熟 |
| Codex | CLI Agent | CLI | OpenAI 官方 |
| OpenClaw | CLI Agent | CLI | 开源，让大众认知了 Agent，大量扩展生态 |
| Aider | CLI Agent | CLI | 专注代码编辑 |
| OpenCode | CLI Agent | CLI | Claude Code 的开源替代 |
| Gemini CLI | CLI Agent | CLI | Google 官方，原生支持 Extension 和 MCP |

## 能力封装协议

| 协议/规范 | 思路 | 优劣 | 生态成熟度 |
|-----------|------|------|-----------|
| **MCP** (Model Context Protocol) | 工具描述塞进 context，JSON-RPC/stdio 通信 | 灵活、类型化调用；token 开销大，43% 早期 Server 有注入漏洞 | 高——SDK(Python/TS)、FastMCP v3.0、多个脚手架 |
| **CLI + Skill** | Agent 读 SKILL.md 调 CLI 命令 | 轻量、可组合、确定性强；需手写 Skill 或用 CLI-Anything 生成 | 中——Claude Code/OpenClaw 原生支持 |
| **A2A** (Agent-to-Agent) v0.3.0 RC1 | Agent 间直接通信，Agent Card 发现机制 | Agent 协作标准；生态尚早期 | 低→中——已移交 Linux Foundation，150+ 组织，Python/JS SDK |
| **OpenAPI / REST** | 传统 API 描述 | 成熟生态；为人类开发者设计，非 Agent 原生 | 高——但 Agent 适配度低 |

### MCP 生态详情

| 组件 | 说明 | 状态 |
|------|------|------|
| **FastMCP v3.0** | Python MCP 开发框架，装饰器即工具，自动 schema 生成，中间件，OpenTelemetry 观测 | 成熟，推荐 |
| MCP Python SDK | 官方 Python SDK，内置 FastMCP | 稳定 |
| MCP TypeScript SDK | `@modelcontextprotocol/sdk`，配合 Zod 做 schema 验证 | 稳定 |
| Smithery | TypeScript MCP Server 脚手架（`create-smithery`） | 可用 |
| fastmcp-boilerplate | Python 快速启动模板 | 可用 |
| MCP 安全规范 | OAuth 2.1 强制认证（2025-03 更新），Streamable HTTP 替代 SSE | 规范已定 |

## GUI → CLI 桥接

| 项目 | 做什么 | 状态 |
|------|--------|------|
| **CLI-Anything** (HKUDS) | 为任意桌面软件自动生成 CLI 接口，7 阶段全自动流水线 | ~18k stars，Phase 6.5 新增 SKILL.md 自动生成 |
| 各云厂商 CLI | aliyun / gcloud / aws cli — 天然的 Agent 接口 | 成熟但缺少 Agent 适配（输入加固、SKILL 文件） |
| Google Workspace CLI | Justin Poehnelt 主导，Agent-first CLI 设计范例，100+ SKILL.md | 参考实现级，七大 Agent 适配模式 |

## 双模架构（Agent 输入 + 人类可视化）

| 项目 | 定位 | 状态 |
|------|------|------|
| **A2UI** (Google) | Agent 生成声明式 UI 组件树，客户端映射到原生 widget | v0.8 公测，向 v1.0 推进 |
| **AG-UI** (CopilotKit) | Agent-User 双向交互协议，运行时通信管道，共享状态同步 | 与 A2UI 互补，CopilotKit 是 A2UI 发布合作伙伴 |
| **LibTV** (LiblibAI) | AI 视频创作平台：无限画布（人类）+ OpenClaw Skill（Agent），共享后端 Agent | 生产可用，GitHub 58 stars。画布编排型场景，后端模式（薄中继 Skill + IM 会话）可迁移 |

**新兴标准架构模式**：
- 控制层：CLI / MCP / A2A（面向 Agent，结构化 JSON）
- 展示层：A2UI / AG-UI（Agent 生成声明式 UI，客户端渲染）
- 中继模式：Skill 仅做意图转发，后端 Agent 完成实际工作（LibTV 实践）
- 数据层：文件系统 / API（Agent 和人类共享工作空间）

## Agent 基础设施

> 来源：Aaron Levie《Building for trillions of agents》

| 类别 | 代表项目 | 说明 |
|------|---------|------|
| 计算沙箱 | E2B, Daytona, Modal, Cloudflare | Agent 专属运行环境，下一代超大规模数据中心可能为 Agent 服务 |
| 数据访问 | Box (API-first) | 企业文件的 Agent 接口 + Agent 自身的记忆存储 |
| 身份通信 | Agentmail | Agent 专属邮箱和持久身份 |
| 搜索 | Parallel, Exa | 为 Agent 重建的网络搜索（爬取网页的主要用户已是 Agent） |
| 支付 | Stripe, Coinbase | Agent 钱包、预算和微支付 |
| 安全合规 | 待建设 | Agent 工作记录的治理和留存 |

## Agent 工具开发框架

| 框架 | 定位 | 关键特点 |
|------|------|---------|
| **OpenAI Agents SDK** | 轻量级多 Agent 框架 | 函数即工具、结构化输出、Guardrails、Handoffs、内置 MCP |
| **Google ADK** | 企业级 Agent 开发 | 层级 Agent 树、原生 A2A、多模态、Vertex AI 集成 |
| **LangGraph** | 精细控制的有状态工作流 | 图结构定义流程、检查点持久化、LangSmith 观测 |
| **CrewAI** | 快速原型的角色团队 | 低代码、角色分工直觉化 |

## 设计理论与参考文献

| 资源 | 类型 | 核心贡献 |
|------|------|---------|
| Every.to《Agent-Native Architectures》 | 实践指南 | 五原则：Parity、Granularity、Composability、Emergent Capability、Improvement Over Time |
| Sam Keen《Agent Native Architecture》 | 架构理论 | 语义优先架构、确定性光谱四象限 |
| 《Terminal Is All You Need》(arXiv) | 学术论文 | 三设计属性：表征兼容性、交互媒介透明性、低参与门槛 |
| Justin Poehnelt CLI 改造指南 | 工程实践 | 面向 Agent 的 CLI 七大改造模式，P0-P3 优先级 |
| Google 八种多 Agent 设计模式 | 架构模式 | 顺序管道、并行扇出、监督者模式、人类在环等 |
| Levie《Building for trillions of agents》 | 产业分析 | API-first 生存条件、Agent 基础设施图谱、"Make something agents want" |
| Karpathy（X 回复） | 趋势判断 | "请直接给我一个能复制粘贴给 Agent 的东西"，传统 UI 对 Agent 是障碍 |

## 待调研

- [x] Claude Code Skill 生态的设计模式梳理 → 已在 ref-cli-for-agents.md 中覆盖
- [x] CLI-Anything 的实现原理与局限性 → 已在 TASK_ANALYSIS.md 和本文中覆盖
- [ ] MCP vs CLI+Skill 的实际对比测试 → G-03 任务
- [ ] Agent 权限模型的现有方案调研 → G-06 任务
- [ ] 面向 Agent 的"用户体验"评估框架
- [x] A2UI + AG-UI 的详细架构调研 → 已在 dual-mode-architecture.md 中覆盖
- [x] 各 Agent 平台（Claude Code/OpenCode/OpenClaw）的协议支持矩阵 → 已在 protocol-comparison.md 中覆盖
- [x] LibTV 双接口案例调研 → 已在 case-libtv-dual-interface.md 中覆盖
- [x] Karpathy/Levie 万亿 Agent 文章整理 → 已在 article-karpathy-levie-agents.md 中覆盖
