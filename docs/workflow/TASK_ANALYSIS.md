# 任务分析报告

## 项目概述

**项目名称**: agent-native-design-guide
**项目性质**: 调研与方法论工作空间，非代码项目
**最终目标**: 产出 Agent-Native 软件设计指南，为后续实现两个面向 Agent 的软件工具提供设计参考

**用户后续计划**:
- 实现两个为 OpenClaw 类 Agent 定制的软件工具
- 工具需支持独立工作，但主要面向 Agent 调用
- 输入面向 Agent（CLI/结构化），输出支持 GUI 可视化 + 结构化数据双模
- GUI 管理界面是否需要尚未决定

## 现状分析

### 已有资料

| 文件 | 内容 | 完成度 |
|------|------|--------|
| `README.md` | 项目定位、核心命题、研究范围 | 框架完整 |
| `docs/design-principles.md` | 7 条设计原则草案 | 初稿，待深化验证 |
| `research/landscape.md` | Agent 运行时、能力封装协议、GUI→CLI 桥接生态图谱 | 框架级，待补充 |
| `research/article-software-eaten-by-ai.md` | 全文摘录 + 核心论点 | 完整 |
| `research/article-gui-will-die-cli-is-everything.md` | 全文摘录 + 核心论点 | 完整 |

### 已建立的认知框架

1. **交互范式**: CLI+Skill 是 Agent 时代的原生交互方式，GUI 退化为结果可视化层
2. **产品形态**: App → Skill，用户从人变为 Agent
3. **软件性质**: 资产 → 耗材，生产成本趋零
4. **设计重心**: 从人类操作路径优化转向 Agent 决策路径优化
5. **核心评估维度**: 可调用性、可组合性、可靠性、信任度、权限边界

## 外部资源调研：可直接复用的工具与规范

### 已有的 Agent-Native 设计原则/指南

| 资源 | 内容 | 可复用性 |
|------|------|----------|
| **Every.to《Agent-Native Architectures》** | 五原则体系：Parity（对等）、Granularity（原子性）、Composability（可组合）、Emergent Capability（涌现）、Improvement Over Time（持续改进） | ⭐⭐⭐ 可直接作为设计指南的理论基础 |
| **Sam Keen《Agent Native Architecture》** | 语义优先架构——让 Agent 的语言理解能力成为核心，结构化数据由 Agent 运行时推理产生 | ⭐⭐ 提供了一种不同于传统 schema-first 的设计思路 |
| **Simon Willison《Agentic Engineering Patterns》** | 实际工程中的 Agentic 模式整理 | ⭐⭐ 工程实践参考 |
| **Google 八种多 Agent 设计模式** | 顺序管道、并行扇出、监督者模式、人类在环等 | ⭐⭐ 多 Agent 协作场景参考 |
| **《Terminal Is All You Need》(arXiv)** | 人机协作三属性：表征兼容性、交互媒介透明性、低参与门槛；结论：终端/CLI 天然具备 | ⭐⭐⭐ 为 CLI-first 提供学术论据 |

### 面向 Agent 的 CLI 设计规范

**Justin Poehnelt《You Need to Rewrite Your CLI for AI Agents》** 提供了最具操作性的 CLI 改造指南：
- 支持 `--output json` / `--json`
- 运行时 schema 自省（`cli schema method.name`）
- 输入验证：拒绝控制字符、防路径穿越、阻止双重编码
- `--dry-run` 验证请求、`--sanitize` 防 prompt 注入
- NDJSON 分页流式输出
- `CONTEXT.md` 或 SKILL 文件编码约束
- **实施优先级**: `--output json` → 输入验证 → schema 自省 → 字段掩码 → `--dry-run` → SKILL 文件 → MCP 接口

### 能力封装协议与开发工具

| 协议/工具 | 成熟度 | 开发体验 | 关键信息 |
|-----------|--------|----------|----------|
| **MCP + FastMCP v3.0** | 高 | Python 装饰器即工具，自动 schema 生成，中间件，OpenTelemetry 观测 | 生态最成熟；但 43% 早期 MCP Server 存在命令注入漏洞，安全需重视 |
| **CLI + Skill** | 中 | CLI-Anything 可自动生成；手写 SKILL.md 成本低 | 轻量、确定性强；CLI-Anything 刚上线 SKILL.md 自动生成（Phase 6.5） |
| **A2A v0.3.0 RC1** | 低→中 | Python/JS SDK；已移交 Linux Foundation，150+ 组织 | Agent 间通信标准；生态尚早期 |
| **MCP 脚手架**: Smithery / fastmcp-boilerplate / MCP Tools | 中 | 项目模板 + CLI 初始化器 | 可快速搭建 MCP Server |

### 双模架构方案（Agent 输入 + 人类可视化）

| 方案 | 定位 | 状态 |
|------|------|------|
| **A2UI (Google)** | Agent 生成声明式 UI 组件树，客户端映射到原生 widget | v0.8 公测，向 v1.0 推进 |
| **AG-UI (CopilotKit)** | Agent-User 双向交互协议，运行时通信管道 | 与 A2UI 互补；CopilotKit 是 A2UI 发布合作伙伴 |

**新兴标准架构模式**:
- **控制层**: CLI / MCP / A2A（面向 Agent，结构化 JSON）
- **展示层**: A2UI / AG-UI（Agent 生成声明式 UI，客户端渲染）
- **数据层**: 文件系统 / API（Agent 和人类共享工作空间）

### Agent 工具开发框架

| 框架 | 定位 | 与本项目的关联 |
|------|------|---------------|
| **OpenAI Agents SDK** | 轻量级多 Agent 框架 | 函数即工具、内置 MCP 支持 |
| **Google ADK** | 企业级 Agent 开发 | 原生 A2A、层级 Agent 树 |
| **FastMCP** | Python MCP Server | 装饰器即工具，最适合快速搭建 Agent 工具 |
| **CLI-Anything** | 软件 Agent 化 | 自动生成 CLI + SKILL.md |

## 目标定义

### 预期产出物

1. **生态调研报告**: 对 Agent-Native 生态、协议对比、已有设计规范的系统梳理
2. **设计指南**: 可操作的 Agent-Native 软件设计最佳实践，包含：
   - 接口设计规范（CLI 命令结构、参数约定、输出格式）
   - 双模输入输出协议（Agent 结构化输入 + 人类可视化输出）
   - 架构模式（Agent-first 双模架构：控制层 + 展示层 + 数据层）
   - 权限与安全模型
   - 可发现性设计（`--help`、schema、SKILL.md）
3. **原型验证**: 用一个最小工具原型验证设计指南的可行性

### 成功标准

- 调研覆盖 MCP、CLI+Skill、A2A、A2UI/AG-UI 的对比分析
- 整合 Every.to 五原则、CLI 设计规范等已有资源，形成统一的设计指南
- 设计指南可直接指导后续两个工具的架构决策
- 明确回答：Agent 输入 + 双模输出的具体实现方案

## 差距分析

| 维度 | 现状 | 目标 | 差距 |
|------|------|------|------|
| 生态调研 | 表格级概览 | 深度对比 + 已有规范整合 | 需深入 Every.to/CLI 规范/A2UI 等外部资源 |
| 设计原则 | 7 条自拟原则 | 整合外部五原则 + 自有洞察的统一框架 | 需对齐、整合、补充 |
| 双模架构 | 概念层 | A2UI/AG-UI + CLI/MCP 的具体方案 | 需调研 A2UI/AG-UI 细节并设计方案 |
| CLI 规范 | 无 | Justin Poehnelt 规范 + 自有补充 | 可直接引入已有规范 |
| 权限模型 | 仅列为待调研 | MCP OAuth 2.1 + 最小权限的具体方案 | 需从 MCP 安全规范入手 |
| 实践验证 | 无 | 最小工具原型 | examples/ 目录为空 |

## 风险清单

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| Agent 生态变化快，调研结果快速过时 | 中 | 聚焦不变的原则而非具体版本；标注调研时间 |
| 调研范围过广导致产出不够聚焦 | 高 | 始终以"指导后续两个工具设计"为筛选标准 |
| 外部规范照搬不适配自身场景 | 中 | 每个引入的规范都要结合后续工具场景做适配验证 |
| A2UI/AG-UI 尚处早期，可能不稳定 | 中 | 作为参考方向而非硬依赖；设计指南保留灵活性 |

## feature 补充检查项

- [ ] 后续工具的目标集成位置（OpenClaw 生态 / 独立运行 / 两者兼顾）需用户确认
- [ ] 双模架构参考实现：A2UI + AG-UI 的组合方案需详细调研

## 待确认问题

1. **后续两个工具的大致方向？** 具体领域（如代码辅助、数据处理、部署管理等）会影响调研重点
2. **目标 Agent 平台？** 主要面向 Claude Code / OpenClaw，还是需要更广泛的兼容性？
3. **设计指南的受众？** 仅供自己参考，还是可能对外发布？
4. **原型验证的范围？** 最小 CLI 工具原型 vs 架构设计文档验证？

## 分析过程中的假设

- 假设 CLI+Skill 是首选交互方式（基于用户在 CLAUDE.md 中的判断），MCP 作为补充接入方式
- 假设设计指南的首要目标是指导自己的后续开发
- 假设后续工具以 Python 为主要开发语言（基于项目配置中使用 uv）
