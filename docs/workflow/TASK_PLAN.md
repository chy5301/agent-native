# 任务计划

## 总体策略

**整合优先 + Skill 交付**

不从零造轮子，整合现有成熟资源（Every.to 五原则、Justin Poehnelt CLI 规范、A2UI/AG-UI 双模方案等），在此基础上补充针对性调研、做适配性分析，最终产出一个**可安装的设计指南 Skill**——Agent 在设计 Agent-Native 工具时可以直接调用获取设计决策参考。

**选择理由**:
- 外部已有高质量的 Agent-Native 设计原则和 CLI 规范，重新发明意义不大
- 真正需求是"指导后续工具设计"，整合比原创更高效
- Skill 形态让设计指南本身成为 Agent-Native 的——Agent 可以按需查阅，而非人类翻阅长文档

**目标 Agent 平台**: Claude Code、OpenCode、OpenClaw 及其扩展
**设计指南受众**: 自用参考（Skill 形态，Agent 可直接调用）
**设计指南定位**: 全面的范式指南，不限于特定领域

**Skill 目标结构**:
```
skill/agent-native-design-guide/
├── SKILL.md                        ← 入口（~2000 词，决策框架 + 快速参考）
├── references/                     ← 详细知识库（按需加载）
│   ├── design-principles.md        ← 设计原则体系
│   ├── cli-interface-spec.md       ← CLI 接口设计规范
│   ├── architecture-patterns.md    ← 架构模式
│   ├── security-model.md           ← 安全与权限
│   ├── protocol-comparison.md      ← 协议对比（精炼版）
│   └── dual-mode-architecture.md   ← 双模架构（精炼版）
└── examples/                       ← 可复制的示例
    ├── skill-md-template.md        ← SKILL.md 模板
    ├── cli-json-output.py          ← --json 输出示例
    └── cli-help-design.py          ← --help 可发现性示例
```

## 阶段里程碑

| 阶段 | 名称 | 退出标准 |
|------|------|----------|
| Phase 0 | 调研准备 | 外部设计规范已整理归档，生态图谱更新至最新状态 |
| Phase 1 | 核心调研 | 协议对比、双模架构、CLI 规范、安全模型四个方向的调研报告完成 |
| Phase 2 | 设计指南 | Skill 的四个 reference 文档和 examples 完成 |
| Phase 3 | Skill 整合与交付 | SKILL.md 入口文件完成，Skill 可安装使用 |

## 任务列表

### Phase 0: 调研准备

#### [G-01] 整理-外部设计规范

- **阶段**: Phase 0 - 调研准备
- **依赖**: 无
- **目标**: 将已发现的外部 Agent-Native 设计规范深入阅读并提炼为可引用的参考笔记
- **背景信息**: 调研发现了多个高质量的外部 Agent-Native 设计资源：Every.to《Agent-Native Architectures》（五原则体系）、Sam Keen《Agent Native Architecture》（语义优先架构）、《Terminal Is All You Need》(arXiv 2603.10664)（终端作为人机协作最优媒介的论证）、Justin Poehnelt 的 CLI 改造指南。这些资源需要深入阅读、提炼核心观点，并以结构化笔记的形式归档到 research/ 目录，作为后续设计指南编写的引用基础。
- **涉及文件**:
  - research/ref-agent-native-architectures.md（新建）
  - research/ref-terminal-is-all-you-need.md（新建）
  - research/ref-cli-for-agents.md（新建）
- **具体步骤**:
  1. 访问并深入阅读 Every.to《Agent-Native Architectures》，提炼五原则的具体内容、适用场景和局限性
  2. 阅读 arXiv 论文《Terminal Is All You Need》，提炼三属性模型和关键论证
  3. 阅读 Justin Poehnelt 的 CLI 改造指南，提炼实施优先级和具体规范
  4. 补充阅读 Sam Keen 的语义优先架构文章，提炼与传统 schema-first 的差异
  5. 每篇整理为结构化笔记：来源、核心观点、可引用结论、与本项目的关联
- **验收标准**:
  - [ ] 3 篇参考笔记已创建且内容完整
  - [ ] 每篇包含：来源链接、核心观点摘要、可引用结论、本项目适用性分析
  - [ ] Every.to 五原则的具体定义和示例已提炼
  - [ ] Justin Poehnelt CLI 规范的实施优先级已明确列出
- **自测方法**: 检查 research/ 目录下 3 个 ref-*.md 文件是否存在且内容完整
- **回滚方案**: 删除新建的 3 个文件
- **预估工作量**: L (约 2 小时)

#### [G-02] 更新-生态图谱

- **阶段**: Phase 0 - 调研准备
- **依赖**: 无
- **目标**: 将 landscape.md 从框架级概览更新为包含最新生态信息的完整图谱
- **背景信息**: 当前 research/landscape.md 仅包含表格级的概览信息（5 个 Agent 运行时、4 个能力封装协议、2 个 GUI→CLI 桥接项目），缺少 FastMCP v3.0、A2UI、AG-UI、A2A v0.3.0、CLI-Anything Phase 6.5 等最新进展。此外还缺少 Agent 工具开发框架（OpenAI Agents SDK、Google ADK 等）和安全相关的生态信息。需要补充这些内容，使生态图谱成为后续调研的索引入口。
- **涉及文件**:
  - research/landscape.md（更新）
- **具体步骤**:
  1. 补充能力封装协议部分：MCP 的最新版本和生态（FastMCP v3.0、SDK、脚手架）、A2A v0.3.0 RC1 状态
  2. 新增"双模架构"部分：A2UI (Google)、AG-UI (CopilotKit) 的定位和状态
  3. 新增"Agent 工具开发框架"部分：OpenAI Agents SDK、Google ADK、LangGraph、CrewAI
  4. 补充 CLI-Anything 最新进展（Phase 6.5 SKILL.md 自动生成）
  5. 更新"待调研"清单，标记已完成和新增的调研项
- **验收标准**:
  - [ ] landscape.md 包含至少 6 个主要类别的生态信息
  - [ ] 每个条目包含：项目名、类型、状态、关键特点
  - [ ] A2UI、AG-UI、FastMCP v3.0、A2A v0.3.0 均已收录
  - [ ] 待调研清单已更新
- **自测方法**: 检查 landscape.md 内容完整性和信息准确性
- **回滚方案**: `git checkout -- research/landscape.md`
- **预估工作量**: M (约 1 小时)

---

### Phase 1: 核心调研

#### [G-03] 调研-能力封装协议对比

- **阶段**: Phase 1 - 核心调研
- **依赖**: G-01, G-02
- **目标**: 产出 MCP、CLI+Skill、A2A、OpenAPI 四种协议的深度对比报告，为后续工具的接入方式选择提供依据
- **背景信息**: Agent 调用外部工具/服务目前有多种协议方案：MCP（Model Context Protocol，工具描述塞进 context）、CLI+Skill（Agent 读 SKILL.md 调 CLI 命令）、A2A（Agent-to-Agent，Google 提出的 Agent 间通信协议）、OpenAPI/REST（传统 API 描述）。每种方案在 token 成本、确定性、可发现性、开发复杂度、生态成熟度等维度各有优劣。目标平台为 Claude Code、OpenCode、OpenClaw，需评估各协议在这些平台上的支持情况。
- **涉及文件**:
  - research/protocol-comparison.md（新建）
- **具体步骤**:
  1. 定义对比维度：token 成本、确定性、可发现性、可组合性、开发复杂度、生态成熟度、目标平台支持
  2. 逐协议分析：MCP 的实际 token 开销和安全风险、CLI+Skill 的轻量优势和局限、A2A 的 Agent 间通信场景、OpenAPI 的成熟度和 Agent 适配度
  3. 交叉对比：在不同场景下（简单工具调用、复杂工作流、Agent 间协作）哪种方案最优
  4. 结合目标平台（Claude Code / OpenCode / OpenClaw）评估各协议的实际支持程度
  5. 输出推荐策略：主用方案 + 补充方案
- **验收标准**:
  - [ ] 对比报告包含 4 种协议在 7+ 个维度的对比矩阵
  - [ ] 包含不同场景下的方案推荐
  - [ ] 包含目标平台的协议支持评估
  - [ ] 有明确的推荐结论
- **自测方法**: 检查 protocol-comparison.md 是否包含对比矩阵和推荐结论
- **回滚方案**: 删除 research/protocol-comparison.md
- **预估工作量**: L (约 2.5 小时)

#### [G-04] 调研-双模架构方案

- **阶段**: Phase 1 - 核心调研
- **依赖**: G-01, G-02
- **目标**: 产出 Agent 结构化输入/输出 + 人类 GUI 可视化的双模架构方案调研报告
- **背景信息**: 用户后续要开发的工具需要同时服务两类消费者：Agent（需要结构化 JSON 输入输出）和人类（需要可视化 GUI 查看结果/过程）。当前的新兴方案包括 A2UI（Google，Agent 生成声明式 UI 组件树）和 AG-UI（CopilotKit，Agent-User 双向交互协议）。此外还有传统方案：CLI 工具 + 独立 Web Dashboard、API Server + 前端 SPA 等。需要调研这些方案的具体实现方式、适用场景和取舍。
- **涉及文件**:
  - research/dual-mode-architecture.md（新建）
- **具体步骤**:
  1. 深入调研 A2UI：声明式 UI 组件树的工作原理、组件类型、与后端的交互方式、v0.8 的能力边界
  2. 深入调研 AG-UI：运行时通信协议、共享状态同步机制、与 A2UI 的互补关系
  3. 梳理传统双模方案：CLI + Web Dashboard 分离、API Server + SPA、Streamlit/Gradio 类快速原型
  4. 对比分析：开发成本、维护成本、Agent 体验、人类体验、技术栈要求
  5. 结合用户场景（工具部署后主要面向 Agent 调用，GUI 用于可视化结果/过程）给出推荐方案
- **验收标准**:
  - [ ] A2UI 和 AG-UI 的工作原理已清晰描述
  - [ ] 至少 3 种双模方案的对比分析
  - [ ] 有针对用户具体场景的推荐方案
  - [ ] 包含推荐方案的概念架构图（文本描述）
- **自测方法**: 检查 dual-mode-architecture.md 是否包含方案对比和推荐
- **回滚方案**: 删除 research/dual-mode-architecture.md
- **预估工作量**: L (约 2.5 小时)

#### [G-05] 调研-CLI 接口设计规范

- **阶段**: Phase 1 - 核心调研
- **依赖**: G-01
- **目标**: 整合已有 CLI 设计规范并补充调研，产出面向 Agent 的 CLI 接口设计最佳实践
- **背景信息**: Justin Poehnelt 的《You Need to Rewrite Your CLI for AI Agents》提供了操作性强的 CLI 改造指南（`--output json`、schema 自省、输入验证等），CLI-Anything 项目展示了自动生成 Agent 友好 CLI 的实践。需要在此基础上，结合 Claude Code Skill 生态的设计模式（.claude/commands/ 下的 Markdown + YAML frontmatter）、`--help` 的可发现性设计，以及目标平台（Claude Code/OpenCode/OpenClaw）的实际调用方式，整合出一份完整的 CLI 接口设计规范。**新增输入源**：独立调研 `research/independent-developer-voices.md` 揭示"输出设计优化 > 工具功能"（结构化输出可减少 Agent 解析时间 5-9x），`research/independent-gui-vs-cli.md` 提供了 GUI Agent 基准数据（最佳 Agent 仅达人类水平的 47.7%）作为 CLI 路线的量化支撑，`research/article-anthropic-skill-craft.md` 提供了 Anthropic 内部 Skill 的文件夹结构和信息分层最佳实践。`research/case-cli-anything.md` 提供了 CLI-Anything 项目的 7-Phase SOP 实战案例——包含双模交互（REPL+Subcommand）、命令分组（Project/Core/IO/Config/Session）、后端包装模式（utils/<software>_backend.py）、输出格式设计（--json flag 双模输出）等具体架构决策。
- **涉及文件**:
  - research/cli-design-spec.md（新建）
- **具体步骤**:
  1. 以 Justin Poehnelt 规范为基础框架，按实施优先级组织
  2. 补充 CLI-Anything 的设计选择（Click 框架、命令分组、状态管理）
  3. 调研 Claude Code Skill 的设计模式：SKILL.md 的结构、命令描述方式、参数传递约定
  4. 研究目标 Agent 平台实际如何发现和调用 CLI 工具（读 --help？读 SKILL.md？读 schema？）
  5. **新增**：将"输出设计优化"作为核心维度——整合 developer-voices 调研中的发现（结构化输出格式设计、错误输出规范、进度报告格式），这直接影响 Agent 的 token 效率和任务成功率
  6. **新增**：整合 Anthropic Skill 经验中的文件夹结构最佳实践（渐进式信息披露、context engineering）
  7. 整合为统一规范：命令结构、参数约定、**输出设计**、可发现性、错误处理
- **验收标准**:
  - [ ] CLI 设计规范覆盖：命令结构、参数、**输出设计**、可发现性、错误处理 5 个维度
  - [ ] **输出设计**部分包含结构化输出格式规范和 Agent token 效率考量
  - [ ] 包含具体的命令设计示例（好的 vs 差的对比）
  - [ ] 包含 SKILL.md 的推荐结构（结合 Anthropic 经验）
  - [ ] 规范内容可直接用于后续设计指南
- **自测方法**: 检查 cli-design-spec.md 的完整性和示例覆盖度
- **回滚方案**: 删除 research/cli-design-spec.md
- **预估工作量**: L (约 2.5 小时)

#### [G-06] 调研-权限与安全模型

- **阶段**: Phase 1 - 核心调研
- **依赖**: G-03
- **目标**: 产出面向 Agent 的权限边界和安全防护方案调研报告
- **背景信息**: Agent 自主调用工具时的权限和安全问题是 Agent-Native 设计的核心关切之一。已知的安全风险包括：43% 早期 MCP Server 存在命令注入漏洞（OWASP 报告）、prompt 注入攻击、路径穿越、权限过度授予等。现有的安全方案包括：MCP 的 OAuth 2.1 强制认证（2025年3月规范更新）、Claude Code 的权限确认机制、最小权限原则的实际实现方式。**新增关键输入**：独立调研 `research/independent-security-trust.md` 提供了 OWASP Agentic Top 10（2026 版，10 种 Agent 特有安全风险）、MCP 安全事件时间线（6 起重大事件）、生产环境安全数据（88% 组织已报告安全事件、仅 14.4% 获完整安全审批），以及 Agent 安全的"原罪"——Agent 需要读取外部数据但外部数据可能包含恶意指令。`research/independent-developer-voices.md` 补充了 MCP 生态的真实痛点（60 天 30+ CVE、95% 服务器质量堪忧）。
- **涉及文件**:
  - research/security-model.md（新建）
- **具体步骤**:
  1. **以 OWASP Agentic Top 10 为威胁模型框架**（来自 independent-security-trust.md），覆盖 ASI01-ASI10 的全部 10 种 Agent 特有风险
  2. 调研 MCP 安全规范：OAuth 2.1 要求、参数 JSON Schema 验证、传输安全
  3. **整合 MCP 安全事件时间线**（来自 independent-security-trust.md），提炼真实案例的教训
  4. 分析 Claude Code 的权限模型：工具调用确认、权限白名单、沙箱机制
  5. 整理 Agent 工具的安全威胁和防护措施（整合 OWASP Top 10 + MCP 事件 + 生产数据）
  6. 设计面向 Agent 工具开发者的安全检查清单
- **验收标准**:
  - [ ] 安全威胁清单：覆盖 OWASP Agentic Top 10 的核心威胁及防护措施
  - [ ] 包含 MCP 安全事件案例分析（至少 3 起）
  - [ ] 权限模型设计建议：最小权限、分级授权、确认机制
  - [ ] 安全检查清单：可直接用于工具开发的 checklist
  - [ ] MCP OAuth 2.1 和 Claude Code 权限机制已描述
  - [ ] 包含生产环境安全数据引用
- **自测方法**: 检查 security-model.md 是否包含 OWASP Agentic Top 10 框架、威胁清单和安全检查清单
- **回滚方案**: 删除 research/security-model.md
- **预估工作量**: L (约 2 小时)

---

### Phase 2: 设计指南

#### [G-07] 编写-设计原则参考文档

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-01, G-03, G-04
- **目标**: 编写 Skill 的设计原则 reference 文档，整合外部五原则与自有洞察为统一的原则体系
- **背景信息**: 最终产物为可安装的设计指南 Skill（`skill/agent-native-design-guide/`）。当前有三套原则需要整合：(1) docs/design-principles.md 的 7 条自拟原则（文本优先、可发现性、确定性、可组合性、最小权限、可逆性、文档即接口），(2) Every.to 的五原则（Parity、Granularity、Composability、Emergent Capability、Improvement Over Time），(3) **新增** Anthropic 内部 Skill 打造的 9 条写作原则（`research/article-anthropic-skill-craft.md`：别说废话、Gotchas 是灵魂、文件夹做信息分层、留有灵活性、首次配置、description triggers、跨会话记忆、代码优于指令、optional hooks）。此外，独立调研提供了重要的平衡视角：`research/independent-counter-arguments.md` 指出 Agent 可靠性瓶颈（每步 85% 准确率 → 10 步仅 20% 成功率）、企业合规要求与"用完即弃"冲突等；`research/independent-enterprise-perspective.md` 提供了企业部署的真实数据（171% ROI、但 40% 项目将被取消）；`research/independent-economics-org-theory.md` 提供了 SaaS 经济学变迁和组织理论视角。原则体系需要回应这些质疑，而非回避。
- **涉及文件**:
  - skill/agent-native-design-guide/references/design-principles.md（新建）
- **具体步骤**:
  1. 对比三套原则体系（自拟 7 条 + Every.to 5 条 + Anthropic 9 条），识别重叠和互补
  2. 设计统一的原则层次：核心原则（3-5 条）+ 实践原则（5-8 条）
  3. 每条原则附带：定义、为什么重要、实际示例、反面案例
  4. **新增**：为核心原则添加"边界条件"或"何时不适用"段落，整合 counter-arguments 的质疑（如可靠性门槛、企业合规需求）
  5. **新增**：整合 Anthropic 的 Skill 写作原则，特别是"Gotchas 是灵魂"和"文件夹做信息分层"作为实践原则
  6. 用调研中的 Karpathy/Levie、LibTV、企业案例等做支撑
  7. 确保文件自包含，Agent 无需阅读其他文件即可理解完整原则体系
- **验收标准**:
  - [ ] 统一原则体系包含核心原则和实践原则两个层次
  - [ ] 每条原则有定义、理由、正面示例
  - [ ] 与 Every.to 五原则和 Anthropic 九条原则的对应关系已说明
  - [ ] 核心原则包含"边界条件/何时不适用"段落
  - [ ] 文件自包含，2000-5000 词范围内
- **自测方法**: 检查 skill/agent-native-design-guide/references/design-principles.md 的完整性和自包含性
- **回滚方案**: 删除新建文件
- **预估工作量**: L (约 2 小时)

#### [G-08] 编写-接口规范参考文档

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-05, G-07
- **目标**: 编写 Skill 的 CLI 接口规范 reference 文档和代码示例，提供命令结构、参数约定、输出格式的具体规范
- **背景信息**: Phase 1 的 G-05 产出了 CLI 接口设计调研报告（research/cli-design-spec.md），其中包含"输出设计优化"核心维度（结构化输出减少 Agent 解析时间 5-9x）。本任务将调研成果精炼为 Skill 的 reference 文件，同时创建 examples/ 目录下的可复制代码示例。规范需要足够具体，让 Agent 可以直接参照指导工具开发。**额外输入**：Anthropic Skill 经验（`research/article-anthropic-skill-craft.md`）中的文件夹结构最佳实践和信息分层策略可指导示例设计。CLI-Anything 的 SKILL.md 自动生成策略（`research/case-cli-anything.md`）提供了命令表结构、Agent 使用指南格式等可直接借鉴的模式。
- **涉及文件**:
  - skill/agent-native-design-guide/references/cli-interface-spec.md（新建）
  - skill/agent-native-design-guide/examples/cli-json-output.py（新建）
  - skill/agent-native-design-guide/examples/cli-help-design.py（新建）
- **具体步骤**:
  1. 从 G-05 调研报告中提取核心规范条目
  2. 编写命令结构规范：命名约定、子命令组织、全局选项
  3. 编写输出格式规范：`--json` 标准输出结构、错误输出格式、流式输出
  4. 编写可发现性规范：`--help` 格式、SKILL.md 模板、schema 自省
  5. 创建 examples/ 目录下的代码示例（--json 输出、--help 设计）
- **验收标准**:
  - [ ] reference 文件覆盖命令结构、参数、输出格式、可发现性、错误处理
  - [ ] 包含至少 2 组好/坏对比示例
  - [ ] examples/ 下有可运行的 Python 代码示例
  - [ ] 文件自包含，2000-5000 词范围内
- **自测方法**: 检查 reference 文件完整性，`uv run python` 验证示例可运行
- **回滚方案**: 删除新建文件
- **预估工作量**: L (约 2 小时)

#### [G-09] 编写-架构模式参考文档

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-03, G-04, G-07
- **目标**: 编写 Skill 的架构模式 reference 文档和 SKILL.md 模板示例，提供 Agent-first 双模架构的具体方案
- **背景信息**: Phase 1 的 G-03（research/protocol-comparison.md）和 G-04（research/dual-mode-architecture.md）产出了调研报告。本任务将调研成果精炼为 Skill 的 reference 文件，核心是三层架构：控制层（CLI/MCP/A2A 面向 Agent）、展示层（A2UI/AG-UI/Web Dashboard 面向人类）、数据层（文件系统/API 共享工作空间）。同时提供 SKILL.md 模板示例供 Agent 复制使用。**额外输入**：`research/independent-enterprise-perspective.md` 提供了企业级部署模式（Bank of America、Siemens 等案例），`research/independent-economics-org-theory.md` 提供了 SaaS→Agent 的商业模式变迁视角，这些应融入架构选择矩阵的"企业场景"考量。Anthropic Skill 9 类型分类（`research/article-anthropic-skill-craft.md`）可指导 SKILL.md 模板的类型化设计。CLI-Anything（`research/case-cli-anything.md`）展示了后端包装模式（Agent CLI 包装真实软件而非替代）、PEP 420 namespace packages、SKILL.md 自动生成等架构实践，是"改造已有软件"场景的范本。
- **涉及文件**:
  - skill/agent-native-design-guide/references/architecture-patterns.md（新建）
  - skill/agent-native-design-guide/examples/skill-md-template.md（新建）
- **具体步骤**:
  1. 定义三层架构模式：控制层、展示层、数据层
  2. 为不同复杂度的工具提供架构选择矩阵（简单 CLI 工具 / 中等复杂度服务 / 复杂平台）
  3. 描述协议选择建议：什么场景用 CLI+Skill、什么场景加 MCP、什么场景引入 A2A
  4. 描述展示层选择建议：纯 CLI 输出 / --report HTML / A2UI 声明式 UI
  5. 创建 SKILL.md 模板示例（包含 frontmatter、body 结构、触发描述最佳实践）
- **验收标准**:
  - [ ] 三层架构定义清晰，各层职责明确
  - [ ] 包含不同复杂度的架构选择矩阵
  - [ ] 包含概念架构图（文本描述）
  - [ ] SKILL.md 模板示例可直接复制使用
  - [ ] 文件自包含，2000-5000 词范围内
- **自测方法**: 检查 reference 文件完整性和 SKILL.md 模板的实用性
- **回滚方案**: 删除新建文件
- **预估工作量**: L (约 2 小时)

#### [G-10] 编写-安全与权限参考文档

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-06, G-07
- **目标**: 编写 Skill 的安全与权限 reference 文档，提供 Agent 工具的安全设计规范和检查清单
- **背景信息**: Phase 1 的 G-06 产出了权限与安全模型调研报告（research/security-model.md），其中包含基于 OWASP Agentic Top 10 的完整威胁模型和 MCP 安全事件案例分析。本任务将调研成果精炼为 Skill 的 reference 文件，包含权限模型设计、输入校验规范、安全检查清单。重点是让 Agent 能直接输出可操作的安全建议和检查清单。**额外输入**：生产环境安全数据（88% 组织已报告安全事件、Shadow AI 泄露平均多花费 $670K）可增强安全建议的说服力。
- **涉及文件**:
  - skill/agent-native-design-guide/references/security-model.md（新建）
- **具体步骤**:
  1. 从 G-06 调研报告中提取安全设计规范
  2. 编写权限模型设计指南：最小权限、分级授权、显式确认
  3. 编写输入校验规范：参数验证、路径安全、prompt 注入防护
  4. 编写安全检查清单：开发阶段 checklist、发布前 checklist
  5. 附带常见安全漏洞的代码示例（坏的 → 好的修复）
- **验收标准**:
  - [ ] 权限模型设计指南完整
  - [ ] 安全检查清单可直接使用
  - [ ] 包含至少 2 个安全漏洞修复示例
  - [ ] 文件自包含，2000-5000 词范围内
- **自测方法**: 检查 reference 文件的完整性和检查清单可用性
- **回滚方案**: 删除新建文件
- **预估工作量**: M (约 1.5 小时)

---

### Phase 3: Skill 整合与交付

#### [G-11] ~~验证-原型实现~~ ❌ 已取消

> **取消原因**：最终产物调整为设计指南 Skill，Skill 本身就是产物，不需要额外的原型验证。原 G-11 的部分验证职能（代码示例）已整合到 G-08 的 examples/ 中。

#### [G-12] 编写-SKILL.md 入口与整合交付

- **阶段**: Phase 3 - Skill 整合与交付
- **依赖**: G-08, G-09, G-10
- **目标**: 编写 Skill 的 SKILL.md 入口文件，精炼调研产出为 Skill reference，整合所有产出，更新 README
- **背景信息**: Phase 2 完成了 Skill 的四个 reference 文档（设计原则、接口规范、架构模式、安全权限）和 examples。本任务的核心是编写 SKILL.md 入口文件——这是 Skill 被 Agent 触发时首先加载的内容（~2000 词），需要包含触发描述、决策框架、快速参考表和 reference 导航索引。同时需要将 Phase 1 的 G-03（协议对比）和 G-04（双模架构）调研产出精炼为 Skill reference 版本。最后更新 README.md 反映 Skill 产出。**关键设计参考**：`research/article-anthropic-skill-craft.md` 的 9 条写作原则应直接指导 SKILL.md 的结构——特别是"description triggers"（触发描述需包含足够的关键短语）、"别说废话"（只写能推动 Agent 偏离默认行为的信息）、"Gotchas 是灵魂"（预置常见设计陷阱）、"文件夹做信息分层"（SKILL.md 入口精简，reference 按需加载）。CLI-Anything 的 HARNESS.md（763 行 SOP）是"长篇设计指南 Skill"的实战范本（`research/case-cli-anything.md`），其结构（目标→SOP 阶段→关键教训→规则→目录结构）可参考。
- **涉及文件**:
  - skill/agent-native-design-guide/SKILL.md（新建，核心交付物）
  - skill/agent-native-design-guide/references/protocol-comparison.md（新建，从 research/ 精炼）
  - skill/agent-native-design-guide/references/dual-mode-architecture.md（新建，从 research/ 精炼）
  - README.md（更新）
- **具体步骤**:
  1. 编写 SKILL.md frontmatter：name、description（触发描述，包含"设计 Agent-Native 工具""CLI 接口设计""选择协议""双模架构""安全模型"等触发短语）
  2. 编写 SKILL.md body：决策框架（快速判断协议/架构/安全方案）、核心原则速查、CLI 设计清单、安全检查清单、reference 导航索引
  3. 精炼 research/protocol-comparison.md → skill references 版本（保留核心对比矩阵和推荐策略，去除详细论证）
  4. 精炼 research/dual-mode-architecture.md → skill references 版本（保留方案对比和推荐，去除协议细节）
  5. 通读全部 Skill 文件，确保 reference 间术语一致、交叉引用正确
  6. 更新 README.md：反映最终目录结构、Skill 安装说明、产出物清单
- **验收标准**:
  - [ ] SKILL.md frontmatter description 包含明确的触发短语
  - [ ] SKILL.md body 在 1500-2000 词范围内，包含决策框架和快速参考
  - [ ] 6 个 reference 文件完整且自包含
  - [ ] examples/ 目录有可用的代码示例和模板
  - [ ] README.md 反映最新的项目状态和 Skill 安装方法
  - [ ] Skill 目录结构符合 OpenClaw/Claude Code skill 规范
- **自测方法**: 检查 Skill 目录完整性，模拟 Agent 触发场景验证 SKILL.md 的导航有效性
- **回滚方案**: 删除 skill/ 目录下新建文件，`git checkout -- README.md`
- **预估工作量**: L (约 2.5 小时)
