# Karpathy/Levie：一切软件，都将为 Agent 重写

> 来源：微信公众号文章（中文综述）+ 原始来源独立调研
> 调研日期：2026-03-18

---

## 一、原始来源

| 来源 | 作者 | 链接 |
|------|------|------|
| X 长文《Building for trillions of agents》 | Aaron Levie（Box CEO） | https://x.com/levie/article/2030714592238956960 |
| X 回复 | Andrej Karpathy | https://x.com/karpathy/status/2030722108322717778 |
| 微信公众号综述文章 | 数字生命卡兹克 | 微信公众号 |

**Karpathy 原话**：
> 💯 "If you build it, they will come." 现在你去任何一家公司，他们还在用传统界面给你下指令。让你导航到某个网页，点某个按钮，在某个输入框里填某个东西。这突然让人觉得很粗鲁。你为什么要告诉我该怎么操作？请直接给我一个能复制粘贴给 Agent 的东西。

---

## 二、核心论点

### 2.1 万亿 Agent 时代

Levie 的核心判断：**未来每个员工都会有大量 Agent 替他干活，一家企业的 Agent 数量将是员工的 100 倍甚至 1000 倍**。

- 一家 1 万人的公司，可能跑着 100 万到 1000 万个 Agent
- 编码 Agent（Claude Code、Devin、Codex、Cursor）已完成质变——独立沙箱计算环境、长期记忆、自主执行长时间任务
- 这种能力正从编码领域蔓延到所有知识工作（Claude Cowork、Perplexity Computer、Manus、OpenClaw）

**数据支撑**：Imperva 2025 Bad Bot Report（已验证）
- 2024 年自动化流量**首次超过人类流量**，占全部网络流量的 **51%**
- 其中恶意 bot 流量占 37%（同比增长 5%）
- 44% 的高级 bot 流量**专门针对 API**（vs 仅 10% 针对传统应用）

> Agent 取代人类成为软件主要用户，不是未来式，是进行时。

### 2.2 "Make something agents want"

Paul Graham 的经典名言 "Make something people want" 催生了 21 世纪最成功的软件公司。

Levie 提出新口号：**Make something agents want.**

Agent 选软件的逻辑与人类根本不同：

| 维度 | 人类选软件 | Agent 选软件 |
|------|----------|-------------|
| 品牌 | 重要 | 无关 |
| UI 颜值 | 重要 | 无关 |
| 社交推荐 | 重要 | 无关 |
| 广告 | 有效 | 无效 |
| 使用习惯 | 强（切换成本高） | 无（无惯性） |
| 选择标准 | 非理性因素占比大 | 纯理性：API 质量、稳定性、价格、文档 |

**经济学视角**：Agent 市场可能是人类历史上最接近**"完全竞争市场"**的市场——所有参与者拥有完全信息且完全理性，没有噪音、没有偏见、只有适应度。

这意味着**传统护城河（品牌、用户惯性、渠道优势）在 Agent 面前一文不值**。剩下的只有：API 质量、数据独占性、性价比。

### 2.3 API-first 是生存条件

**Levie 原话**：
> 如果你的某个功能没有 API，那它等于不存在。如果它不能通过 CLI 或 MCP Server 暴露出来，你就处于劣势。如果你的 API 混乱、路径冲突，你就是在主动降低自己对 Agent 的价值。

**YC Jared Friedman**：
> 现在最好的开发者工具，大多数连注册账号都不能通过 API 完成。在 Claude Code 时代，这是个大失误。因为 Claude 没法自己注册。把所有账户管理功能放进 API，现在应该是最基本的要求。

**40 年翻译层的讽刺**：
- 1980 年代 Xerox PARC 发明 GUI → 40 年行业都在做"把计算机翻译成人类能理解的视觉语言"
- GUI 本质是翻译层：底下跑的还是 API 调用、CLI 指令、HTTP 请求
- Agent 天生会写代码、调 API、发 HTTP 请求——它不需要翻译层
- 强迫 Agent 用 GUI 就像"请了一位精通英语的翻译，却非要跟他说中文"

### 2.4 Agent 基础设施图谱

Levie 梳理了 Agent 时代全新的基础设施需求：

| 基础设施层 | 需求 | 代表项目 |
|-----------|------|---------|
| **计算环境** | Agent 专属运行沙箱 | E2B, Daytona, Modal, Cloudflare |
| **数据访问** | 企业文件的 Agent 接口 + Agent 自身的记忆存储 | Box |
| **身份与通信** | Agent 专属身份、持久邮箱 | Agentmail |
| **搜索** | 为 Agent 重建的网络搜索 | Parallel, Exa |
| **支付** | Agent 钱包、预算、微支付 | Stripe, Coinbase |
| **安全与合规** | Agent 工作记录的治理和留存 | 待建设 |

> 下一个超大规模数据中心，可能不再为我们的应用服务，而是为我们的 Agent 服务。

### 2.5 商业模式变迁

```
许可证时代 → 按席位订阅 → 按量计费（Agent 时代）
```

- Agent 不归属于特定用户，或用几行指令完成人类几小时的工作——按席位收费失效
- **微支付终于有了真正用武之地**：Agent 按需付费访问工具和信息
- Agent 需要自己付款的能力

### 2.6 委托代理问题（Principal-Agent Problem）

当 Agent 替人类做决定时，利益是否一致？

- 训练数据偏见：如果 Stripe 的文档质量远高于竞品，Agent 会不自觉偏向 Stripe
- 透明度：Agent 可能选了一个对自己更友好但对人类更不透明的服务
- 消费决策权交给了一个不透明的中间层

> Levie 对此一笔带过，但这可能是整个 Agent 时代最值得警惕的暗流。

### 2.7 第四次迁移

软件用户的四次身份转换：

| 时代 | "用户"是谁 | 界面 | 死掉的巨头 |
|------|-----------|------|-----------|
| 大型机 | 穿白大褂的操作员 | 打孔卡、命令行 | — |
| PC | 办公室白领 | 窗口、鼠标 | 不适应的大型机厂商 |
| 移动 | 所有人 | 触屏、手指 | BlackBerry, Nokia |
| **Agent** | **AI Agent** | **API/CLI/MCP** | **?** |

规律：**旧时代的巨头，恰恰因为在旧界面上做得太好，而无法适应新界面。**

- Nokia 实体键盘体验全球第一，但触屏时代成了累赘
- Salesforce UI 设计了 20 年，但对 Agent 来说全是障碍

> 历史的教训只有一条：每次用户变了，不变的软件就会消失。没有例外。

---

## 三、与已有调研的关系

### 3.1 与《AI，正在吞噬所有软件》的互补

| 维度 | 《AI，正在吞噬所有软件》 | Levie/Karpathy |
|------|----------------------|----------------|
| 视角 | 设计师/创始人 | 企业 CEO/AI 研究者 |
| 侧重 | **为什么**（趋势分析） | **怎么做**（基础设施和行动） |
| 护城河 | 软件从资产变耗材 | 传统护城河失效，API 质量成唯一壁垒 |
| 中间层 | GUI 是翻译层，中间层消亡 | 40 年翻译层的讽刺 |
| 商业模式 | Agent 改变产品形态 | 按席位→按量，微支付兴起 |

### 3.2 对已有调研的补充

- **Levie 的 Agent 基础设施图谱**补充了 `../synthesis/landscape.md` 中此前空白的"基础设施"维度
- **"API-first 是生存条件"**是 Justin Poehnelt P0 优先级（`--json` 输出）的**宏观战略背书**
- **"完全竞争市场"视角**是全新贡献——此前调研主要关注技术实现，未涉及市场经济学分析
- **委托代理问题**是此前调研未触及的风险维度
- **Imperva 51% 数据**为"Agent 成为主要用户"提供了**定量证据**

### 3.3 与设计原则的关联

- 验证了 Every.to 的 **Parity 原则**（Agent 和人类应获得同等能力访问）
- 强化了 **"Terminal Is All You Need"** 的核心论点（文本 ACI > GUI）
- 为 Justin Poehnelt 的 **P0: --json** 提供了产业级紧迫性论证

---

## 四、补充调研发现

### 4.1 Karpathy autoresearch

- **GitHub**：https://github.com/karpathy/autoresearch
- ~630 行 Python 工具，让 AI Agent 自主运行 ML 实验
- 2 天内跑了 700 个实验，找到 20 个可叠加的优化
- Karpathy 计划扩展为"SETI@home 式"异步大规模协作
- **意义**：Agent 作为软件主要用户的具体实践案例

### 4.2 Box Automate

- Box 推出的"AI Agent 操作系统"
- 将工作流拆解为可被 AI 增强的片段
- Levie 在 Latent Space 播客 "Every Agent Needs a Box" 中深入讨论

### 4.3 Levie 的延伸思考

- 后续推文探讨 Agent 拥有预算/钱包的下游效应
- 对人类用户失败的商业模式（微支付）可能在 Agent 用户上成功
- **文档即竞争优势**：数字化机构知识的公司获得复利回报

---

## 参考来源

- [Aaron Levie 原文](https://x.com/levie/article/2030714592238956960)
- [Karpathy 回复](https://x.com/karpathy/status/2030722108322717778)
- [Imperva 2025 Bad Bot Report](https://www.imperva.com/resources/resource-library/reports/2025-bad-bot-report/)
- [Latent Space 播客 "Every Agent Needs a Box"](https://www.latent.space/p/box)
- [Karpathy autoresearch](https://github.com/karpathy/autoresearch)
- [Levie 后续推文](https://x.com/levie/status/2031575412967616632)
