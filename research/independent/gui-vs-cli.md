# GUI vs CLI for Agents：正反证据综合调研

> 调研日期：2026-03-18 | 独立调研，非基于用户提供材料
> 核心问题：检验"GUI 将死，CLI 是 Agent 的原生交互方式"这一命题

---

## 一、GUI Agent 的最新进展（2025-2026）

### 1.1 主要玩家

| 产品 | 厂商 | 发布时间 | 关键特点 |
|------|------|----------|----------|
| Computer Use | Anthropic | 2024-10 beta | 桌面全局控制，模拟人类鼠标键盘 |
| Operator (CUA) | OpenAI | 2025-01 | GPT-4o 视觉 + RL，仅浏览器内 |
| Project Mariner | Google | 2025 I/O | 基于 Gemini，支持 10 个并发任务 |
| Agent S2 | Simular | 2025 | OSWorld 当前 SOTA |
| UI-Venus 1.5 | inclusionAI | 2026-02 | ScreenSpot-Pro SOTA |

### 1.2 基准测试成绩

**OSWorld（桌面 OS 任务，50-step）：**
- 人类：72.36%
- Simular Agent S2：34.5%（当前 SOTA）
- OpenAI CUA：~32.6%
- 最佳 Agent 仅达人类水平的 **47.7%**

**WebArena（Web 交互）：**
- IBM CUGA：61.7%（单 Agent 记录）
- OpenAI Operator：58%
- Gemini 2.5 Pro：54.8%

**WebVoyager（浏览器导航）：**
- OpenAI CUA：87%
- Google Mariner：83.5%
- Anthropic Computer Use：56%

**ScreenSpot-Pro（高分辨率 GUI 元素定位）：**
- 最佳基线模型：18.9%
- MEGA-GUI：73.18%（近 4 倍提升）

**CUB（Computer Use Benchmark，最综合）：**
- Writer's Action Agent：10.4%（最高分）
- 其他主流 Agent：个位数

**趋势判断**：GUI Agent 确实在快速改进（ScreenSpot-Pro 从 18.9% 到 73.18%），但在最困难的综合基准上仍停留在个位数，与人类差距巨大。

---

## 二、GUI vs CLI 效率对比

### 2.1 直接对比数据

目前没有找到专门的 GUI Agent vs CLI Agent 对照实验论文，但以下间接数据很有说服力：

**"Terminal Is All You Need"论文（CHI 2026 Workshop, arXiv:2603.10664）：**
- OSWorld 上 GUI Agent 成功率仅 **12.24%**，人类 72.36%
- 精心设计的文本界面在 SWE-bench 上比默认 Linux shell **高 10.7 个百分点**
- 可执行 Python 代码作为动作空间比 JSON 函数调用**高 20% 成功率**

**CLI vs MCP Token 效率（CircleCI 报告）：**
- CLI 的 token 效率比 MCP **高 33%**
- 任务完成分：CLI 77 vs MCP 60（浏览器自动化基准）
- MCP 连接 GitHub 服务器注入 ~55,000 tokens 上下文，CLI `help` 仅 ~200 tokens

### 2.2 GUI Agent 的核心失败模式

1. **视觉定位的"大海捞针"问题**：高分辨率屏幕上目标控件可能仅占屏幕面积 < 0.1%
2. **规划错误是主因**：错误分析表明规划错误是主要失败原因，甚至超过视觉定位错误
3. **错误级联不可逆**：早期错误在密集 UI 中会不可逆传播
4. **延迟问题**：推理延迟以秒计，远超人类预期的 200ms
5. **步骤效率**：当前 Agent 比人类**低 40-170%**

---

## 三、"GUI 不会死"的论点

### 3.1 Google A2UI：GUI 进化而非消亡

Google 的 A2UI（Agent-to-User Interface）项目代表中间立场：
- Agent 可以**动态生成**上下文相关的 UI，而非使用固定 GUI
- "Interfaces are no longer designed. They are compiled from intent."
- 声明式 JSON 描述，非可执行代码，确保安全

**核心论点**：即使 Agent 用 CLI 完成所有任务，人类仍需要视觉界面来确认和监督。

### 3.2 CLI-Anything 与 OpenCLI：CLI 化覆盖面的快速扩展

CLI-Anything（港大 HKUDS）主张"Making ALL Software Agent-Native"，但其方案本身揭示了一个现实：
- 需要为 13 个主要应用（GIMP、Blender、LibreOffice、OBS 等）**逐一生成 CLI 包装器**
- 本质是将 **GUI 应用的能力通过 CLI 暴露给 Agent**，说明大量专业软件的核心能力仍绑定在 GUI 应用中

**但 OpenCLI（2026-03）的出现显著改变了这个论述的力度**。OpenCLI 走浏览器路线（Chrome 扩展 + 本地 Daemon + CDP），将覆盖面扩展到所有 Web 应用和 Electron 应用，不需要源码、不需要逐一改造。其 AI 自发现管线（explore/synthesize/cascade）甚至让 Agent 可以自动发现陌生网站的 API 并生成适配器。

两条路线合在一起：有源码桌面软件（CLI-Anything）+ 网站和 Electron 应用（OpenCLI）≈ "一切软件"的 CLI 化。

**但需要注意**：浏览器路线依赖 DOM 结构和网络请求模式，稳定性不如源码路线。网站改版、应用更新都可能导致适配器失效。OpenCLI 本质上仍是一种 GUI 自动化的变体（通过 DOM 操作而非像素识别），在可靠性光谱上处于源码路线和 GUI Agent 之间

### 3.3 GUI Agent 支持者的核心论据

1. **普适性**：GUI Agent 可操作**任何现有软件**，无需适配。数百万应用没有也永远不会有 CLI
2. **视觉任务不可约化**：判断设计"看起来对不对"不能通过 CLI 完成
3. **基准快速提升**：ScreenSpot-Pro 从 18.9% 到 73.18%，暗示可能很快突破实用阈值
4. **人类监督需要 UI**：Agent 通过 CLI 执行，人类仍需视觉界面审查结果
5. **企业遗留系统**：大量关键系统只有 GUI，短期内不会改变

---

## 四、综合分析

### 4.1 效率维度对比

| 维度 | CLI/工具调用 | GUI Agent | 优势方 |
|------|-------------|-----------|--------|
| Token 效率 | 高（~200 token 启动） | 低（截图+坐标） | CLI |
| 任务完成率 | 高 | 低（OSWorld 34.5% vs 人类 72%） | CLI |
| 步骤效率 | 接近人类 | 比人类低 40-170% | CLI |
| 延迟 | 毫秒级 | 秒级 | CLI |
| 软件覆盖范围 | 快速扩展中（CLI-Anything 源码路线 + OpenCLI 浏览器路线覆盖桌面/Web/Electron） | 理论上覆盖所有 GUI 软件 | GUI（但差距在缩小） |
| 视觉/空间任务 | 无法处理 | 可以处理 | GUI |
| 遗留系统兼容 | 差 | 好 | GUI |
| 人类可监督性 | 需学习或信任输出 | 直观可视 | GUI |
| 改进潜力 | 已较成熟 | 快速提升中 | GUI |

### 4.2 修正后的命题

原命题："GUI 将死，CLI 是 Agent 的原生交互方式"

**修正为**：

> CLI/工具调用是 Agent-to-Software 交互的最优路径（效率高出一个数量级），但 GUI 不会死——它会分化为两个新角色：(1) Agent-to-Legacy-Software 的桥接层（源码路线 CLI 包装 + 浏览器路线 DOM 操作，两条路线合在一起已接近覆盖"一切软件"），(2) Agent-to-Human 的展示层（动态生成的临时 UI）。

---

## 参考来源

- [Terminal Is All You Need (arXiv:2603.10664)](https://arxiv.org/html/2603.10664)
- [GUI Agents: A Survey (arXiv:2412.13501)](https://arxiv.org/html/2412.13501v1)
- [GUI Agents with Foundation Models (arXiv:2411.04890)](https://arxiv.org/html/2411.04890v2)
- [ScreenSpot-Pro (arXiv:2504.07981)](https://arxiv.org/html/2504.07981v1)
- [MEGA-GUI (arXiv:2511.13087)](https://arxiv.org/html/2511.13087)
- [CircleCI: MCP vs CLI](https://circleci.com/blog/mcp-vs-cli/)
- [A2UI (Google)](https://developers.googleblog.com/introducing-a2ui-an-open-project-for-agent-driven-interfaces/)
- [CLI-Anything (GitHub)](https://github.com/HKUDS/CLI-Anything)
- [OpenCLI (GitHub)](https://github.com/jackwener/opencli)
- [o-mega: 2025-2026 Benchmarks Guide](https://o-mega.ai/articles/the-2025-2026-guide-to-ai-computer-use-benchmarks-and-top-ai-agents)
