# 任务计划

## 总体策略

**整合优先 + 原型验证**

不从零造轮子，整合现有成熟资源（Every.to 五原则、Justin Poehnelt CLI 规范、A2UI/AG-UI 双模方案等），在此基础上补充针对性调研、做适配性分析，最终通过一个最小原型验证关键设计假设。

**选择理由**:
- 外部已有高质量的 Agent-Native 设计原则和 CLI 规范，重新发明意义不大
- 真正需求是"指导后续工具设计"，整合比原创更高效
- 原型验证能暴露纯文档分析发现不了的问题

**目标 Agent 平台**: Claude Code、OpenCode、OpenClaw 及其扩展
**设计指南受众**: 自用参考
**设计指南定位**: 全面的范式指南，不限于特定领域

## 阶段里程碑

| 阶段 | 名称 | 退出标准 |
|------|------|----------|
| Phase 0 | 调研准备 | 外部设计规范已整理归档，生态图谱更新至最新状态 |
| Phase 1 | 核心调研 | 协议对比、双模架构、CLI 规范、安全模型四个方向的调研报告完成 |
| Phase 2 | 设计指南 | 设计指南四大章节（原则、接口、架构、安全）完成 |
| Phase 3 | 验证与交付 | 最小原型验证通过，所有产出文档定稿整合 |

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
- **背景信息**: Justin Poehnelt 的《You Need to Rewrite Your CLI for AI Agents》提供了操作性强的 CLI 改造指南（`--output json`、schema 自省、输入验证等），CLI-Anything 项目展示了自动生成 Agent 友好 CLI 的实践。需要在此基础上，结合 Claude Code Skill 生态的设计模式（.claude/commands/ 下的 Markdown + YAML frontmatter）、`--help` 的可发现性设计，以及目标平台（Claude Code/OpenCode/OpenClaw）的实际调用方式，整合出一份完整的 CLI 接口设计规范。
- **涉及文件**:
  - research/cli-design-spec.md（新建）
- **具体步骤**:
  1. 以 Justin Poehnelt 规范为基础框架，按实施优先级组织
  2. 补充 CLI-Anything 的设计选择（Click 框架、命令分组、状态管理）
  3. 调研 Claude Code Skill 的设计模式：SKILL.md 的结构、命令描述方式、参数传递约定
  4. 研究目标 Agent 平台实际如何发现和调用 CLI 工具（读 --help？读 SKILL.md？读 schema？）
  5. 整合为统一规范：命令结构、参数约定、输出格式、可发现性、错误处理
- **验收标准**:
  - [ ] CLI 设计规范覆盖：命令结构、参数、输出格式、可发现性、错误处理 5 个维度
  - [ ] 包含具体的命令设计示例（好的 vs 差的对比）
  - [ ] 包含 SKILL.md 的推荐结构
  - [ ] 规范内容可直接用于后续设计指南
- **自测方法**: 检查 cli-design-spec.md 的完整性和示例覆盖度
- **回滚方案**: 删除 research/cli-design-spec.md
- **预估工作量**: L (约 2 小时)

#### [G-06] 调研-权限与安全模型

- **阶段**: Phase 1 - 核心调研
- **依赖**: G-03
- **目标**: 产出面向 Agent 的权限边界和安全防护方案调研报告
- **背景信息**: Agent 自主调用工具时的权限和安全问题是 Agent-Native 设计的核心关切之一。已知的安全风险包括：43% 早期 MCP Server 存在命令注入漏洞（OWASP 报告）、prompt 注入攻击、路径穿越、权限过度授予等。现有的安全方案包括：MCP 的 OAuth 2.1 强制认证（2025年3月规范更新）、Claude Code 的权限确认机制、最小权限原则的实际实现方式。需要调研这些方案并整理出面向 Agent 工具开发者的安全设计指南。
- **涉及文件**:
  - research/security-model.md（新建）
- **具体步骤**:
  1. 调研 MCP 安全规范：OAuth 2.1 要求、参数 JSON Schema 验证、传输安全
  2. 调研 OWASP MCP 安全指南的具体建议
  3. 分析 Claude Code 的权限模型：工具调用确认、权限白名单、沙箱机制
  4. 整理 Agent 工具的常见安全威胁和防护措施
  5. 设计面向 Agent 工具开发者的安全检查清单
- **验收标准**:
  - [ ] 安全威胁清单：至少 5 种常见威胁及防护措施
  - [ ] 权限模型设计建议：最小权限、分级授权、确认机制
  - [ ] 安全检查清单：可直接用于工具开发的 checklist
  - [ ] MCP OAuth 2.1 和 Claude Code 权限机制已描述
- **自测方法**: 检查 security-model.md 是否包含威胁清单和安全检查清单
- **回滚方案**: 删除 research/security-model.md
- **预估工作量**: M (约 1.5 小时)

---

### Phase 2: 设计指南

#### [G-07] 编写-设计原则章节

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-01, G-03, G-04
- **目标**: 编写设计指南的"设计原则"章节，整合外部五原则与自有洞察为统一的原则体系
- **背景信息**: 当前 docs/design-principles.md 有 7 条自拟原则（文本优先、可发现性、确定性、可组合性、最小权限、可逆性、文档即接口），外部有 Every.to 的五原则（Parity、Granularity、Composability、Emergent Capability、Improvement Over Time）。两套原则有重叠（可组合性）也有互补。需要整合为一套统一的、层次清晰的设计原则体系，并用调研中的实际案例做支撑。最终替换现有的 design-principles.md。
- **涉及文件**:
  - docs/design-principles.md（重写）
  - docs/design-guide.md（新建，设计指南主文档）
- **具体步骤**:
  1. 对比两套原则，识别重叠和互补关系
  2. 设计统一的原则层次：核心原则（3-5 条）+ 实践原则（5-8 条）
  3. 每条原则附带：定义、为什么重要、实际示例、反面案例
  4. 创建 design-guide.md 作为设计指南主文档，设计原则作为第一章
  5. 更新 design-principles.md 为设计指南的原则章节
- **验收标准**:
  - [ ] 统一原则体系包含核心原则和实践原则两个层次
  - [ ] 每条原则有定义、理由、正面示例
  - [ ] design-guide.md 主文档框架已建立
  - [ ] 与 Every.to 五原则的对应关系已说明
- **自测方法**: 检查 design-guide.md 和 design-principles.md 的内容一致性和完整性
- **回滚方案**: `git checkout -- docs/design-principles.md`，删除 docs/design-guide.md
- **预估工作量**: L (约 2 小时)

#### [G-08] 编写-接口设计规范章节

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-05, G-07
- **目标**: 编写设计指南的"接口设计规范"章节，提供 CLI 命令结构、参数约定、输出格式的具体规范
- **背景信息**: Phase 1 的 G-05 产出了 CLI 接口设计调研报告（整合 Justin Poehnelt 规范、CLI-Anything 实践、Skill 设计模式）。本任务将调研成果转化为设计指南中的可操作规范章节，包含命令设计模板、参数约定、输出格式标准、可发现性设计（--help、schema、SKILL.md）和错误处理规范。规范需要足够具体，让开发者可以直接参照实现。
- **涉及文件**:
  - docs/design-guide.md（追加接口规范章节）
- **具体步骤**:
  1. 从 G-05 调研报告中提取核心规范条目
  2. 编写命令结构规范：命名约定、子命令组织、全局选项
  3. 编写输出格式规范：`--json` 标准输出结构、错误输出格式、流式输出
  4. 编写可发现性规范：`--help` 格式、SKILL.md 模板、schema 自省
  5. 附带命令设计的好/坏对比示例
- **验收标准**:
  - [ ] 接口规范章节已追加到 design-guide.md
  - [ ] 覆盖命令结构、参数、输出格式、可发现性、错误处理
  - [ ] 包含 SKILL.md 模板
  - [ ] 包含至少 2 组好/坏对比示例
- **自测方法**: 检查 design-guide.md 中接口规范章节的完整性
- **回滚方案**: `git checkout -- docs/design-guide.md`
- **预估工作量**: M (约 1.5 小时)

#### [G-09] 编写-架构模式章节

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-03, G-04, G-07
- **目标**: 编写设计指南的"架构模式"章节，提供 Agent-first 双模架构的具体方案
- **背景信息**: Phase 1 的 G-03（协议对比）和 G-04（双模架构）产出了调研报告。本任务将调研成果转化为可落地的架构模式指南，核心是三层架构：控制层（CLI/MCP/A2A 面向 Agent）、展示层（A2UI/AG-UI/Web Dashboard 面向人类）、数据层（文件系统/API 共享工作空间）。需要为不同复杂度的工具提供不同的架构选择建议。
- **涉及文件**:
  - docs/design-guide.md（追加架构模式章节）
- **具体步骤**:
  1. 定义三层架构模式：控制层、展示层、数据层
  2. 为不同复杂度的工具提供架构选择矩阵（简单 CLI 工具 / 中等复杂度服务 / 复杂平台）
  3. 描述协议选择建议：什么场景用 CLI+Skill、什么场景加 MCP、什么场景引入 A2A
  4. 描述展示层选择建议：纯 CLI 输出 / CLI + Streamlit 快速原型 / A2UI 声明式 UI
  5. 附带概念架构图（文本描述）和技术栈推荐
- **验收标准**:
  - [ ] 架构模式章节已追加到 design-guide.md
  - [ ] 三层架构定义清晰，各层职责明确
  - [ ] 包含不同复杂度的架构选择矩阵
  - [ ] 包含概念架构图
- **自测方法**: 检查 design-guide.md 中架构模式章节的完整性
- **回滚方案**: `git checkout -- docs/design-guide.md`
- **预估工作量**: L (约 2 小时)

#### [G-10] 编写-安全与权限章节

- **阶段**: Phase 2 - 设计指南
- **依赖**: G-06, G-07
- **目标**: 编写设计指南的"安全与权限"章节，提供 Agent 工具的安全设计规范
- **背景信息**: Phase 1 的 G-06 产出了权限与安全模型调研报告（MCP OAuth 2.1、OWASP 安全指南、Claude Code 权限机制、常见威胁和防护）。本任务将调研成果转化为设计指南中的安全规范章节，包含权限模型设计、输入校验规范、安全检查清单。重点是让工具开发者能直接按照清单检查自己的实现。
- **涉及文件**:
  - docs/design-guide.md（追加安全与权限章节）
- **具体步骤**:
  1. 从 G-06 调研报告中提取安全设计规范
  2. 编写权限模型设计指南：最小权限、分级授权、显式确认
  3. 编写输入校验规范：参数验证、路径安全、prompt 注入防护
  4. 编写安全检查清单：开发阶段 checklist、发布前 checklist
  5. 附带常见安全漏洞的代码示例（坏的 → 好的修复）
- **验收标准**:
  - [ ] 安全章节已追加到 design-guide.md
  - [ ] 权限模型设计指南完整
  - [ ] 安全检查清单可直接使用
  - [ ] 包含至少 2 个安全漏洞修复示例
- **自测方法**: 检查 design-guide.md 中安全章节的完整性和检查清单可用性
- **回滚方案**: `git checkout -- docs/design-guide.md`
- **预估工作量**: M (约 1.5 小时)

---

### Phase 3: 验证与交付

#### [G-11] 验证-原型实现

- **阶段**: Phase 3 - 验证与交付
- **依赖**: G-08, G-09
- **目标**: 用一个最小 CLI 工具原型验证设计指南中的接口规范和架构模式是否可行
- **背景信息**: 设计指南在 Phase 2 中完成了原则、接口规范、架构模式、安全四个章节。但纯文档的指南可能存在理论和实践的脱节。本任务将按照设计指南的规范，实现一个最小的 CLI 工具原型（如一个简单的文件处理或数据转换工具），验证：命令结构规范是否合理、`--json` 输出格式是否好用、SKILL.md 的可发现性是否有效、双模输出（CLI 结构化 + 简单 Web 可视化）是否可行。验证过程中发现的问题用于反馈修正设计指南。
- **涉及文件**:
  - examples/minimal-tool/（新建目录）
  - examples/minimal-tool/cli.py（新建）
  - examples/minimal-tool/SKILL.md（新建）
  - examples/minimal-tool/README.md（新建）
- **具体步骤**:
  1. 选择一个简单的工具场景（如文本分析、文件格式转换等）
  2. 按照设计指南的接口规范实现 CLI（Click 框架、`--json` 输出、`--help` 可发现性）
  3. 编写 SKILL.md，按照设计指南的模板
  4. 实现最简的双模输出：CLI 结构化 JSON + 可选的简单 HTML 报告
  5. 记录验证过程中发现的问题和设计指南需修正之处
- **验收标准**:
  - [ ] 原型工具可运行，CLI 命令可正常执行
  - [ ] `--json` 输出符合设计指南的格式规范
  - [ ] SKILL.md 内容完整，Agent 可据此理解和调用
  - [ ] 验证发现的问题已记录
- **自测方法**: `uv run python examples/minimal-tool/cli.py --help` 和 `--json` 输出验证
- **回滚方案**: 删除 examples/minimal-tool/ 目录
- **预估工作量**: L (约 2.5 小时)

#### [G-12] 整合-最终交付

- **阶段**: Phase 3 - 验证与交付
- **依赖**: G-10, G-11
- **目标**: 根据原型验证反馈修正设计指南，整合所有产出文档，更新 README
- **背景信息**: 经过 Phase 0-2 的调研和编写，以及 Phase 3 G-11 的原型验证，所有内容已经产出但可能存在不一致或需要修正之处。本任务负责：根据 G-11 验证反馈修正设计指南、确保所有文档的交叉引用正确、更新 README.md 反映最终的项目结构和产出物、确保 design-guide.md 作为一个完整的独立文档可以被阅读。
- **涉及文件**:
  - docs/design-guide.md（修正）
  - docs/design-principles.md（同步修正）
  - README.md（更新）
  - research/landscape.md（最终更新）
- **具体步骤**:
  1. 根据 G-11 验证记录修正设计指南中的问题
  2. 通读 design-guide.md 全文，确保章节间逻辑连贯、术语一致
  3. 确保 design-principles.md 与 design-guide.md 中的原则章节同步
  4. 更新 README.md：反映最终目录结构、产出物清单、设计指南摘要
  5. 最终检查所有文档的交叉引用和链接
- **验收标准**:
  - [ ] design-guide.md 为完整可独立阅读的文档
  - [ ] README.md 反映最新的项目状态和产出物
  - [ ] 所有文档无明显的不一致或断链
  - [ ] G-11 验证反馈已体现在设计指南中
- **自测方法**: 通读所有 docs/ 和 research/ 下的文档，检查一致性
- **回滚方案**: `git checkout -- docs/ README.md research/landscape.md`
- **预估工作量**: M (约 1.5 小时)
