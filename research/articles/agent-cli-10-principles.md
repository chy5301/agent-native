# 给 Agent 设计 CLI 的十个原则

> 来源：Johnixr（公众号文章 + 开源项目）
> 项目：https://github.com/Johnixr/agent-cli-guide
> 性质：面向 Agent 的 CLI 设计原则总结 + 可直接引用的开源规范
> 许可：CC BY 4.0

---

## 一、文章定位与价值

这篇文章是对「钉钉飞书集体 CLI 化」趋势的设计层跟进——不再讨论「要不要做 CLI」，而是直接回答「怎么做」。作者综合了 POSIX 标准、GNU 编码规范、clig.dev、Anthropic tool use 文档、Berkeley BFCL V4、Lightning Labs 的 agent-CLI 设计轴，以及钉钉/飞书 CLI 的实战分析，提炼出 10 条面向 Agent 的 CLI 设计原则。

**与我们已有调研的关系**：

| 已有调研 | 本文新增价值 |
|---------|------------|
| `cli-for-agents.md`（Justin Poehnelt 七模式） | 七模式偏「改造视角」，本文偏「新建规范」；本文增加了幂等设计、schema 自省、退出码语义 |
| `cli-design-spec.md`（G-05 设计规范） | G-05 是综合产出，本文是独立信源；本文的钉钉/飞书实战评分可补充 G-05 的案例层 |
| `cli-anything.md`（CLI-Anything 案例） | CLI-Anything 侧重「用 CLI 做一切」的可能性，本文侧重「怎么设计好的 CLI」|

**核心价值**：本文提供了一个可直接丢给 Coding Agent 使用的开源项目（agent-cli-guide），包含 GUIDE.md 和 CHECKLIST.md 两个文件，Agent 可以在开发 CLI 时直接引用。

---

## 二、Agent 认知模型分析

文章先建立了 Agent 使用 CLI 的认知框架：

### 2.1 Agent 的 CLI 直觉来源

LLM 训练数据中包含大量 shell 命令和 bash 脚本：
- GitHub 代码（约占训练数据 5%）
- Stack Overflow 问答（约 2%）
- 技术博客和文档

因此 Agent 天生对命令行有相当的「直觉」，但这个直觉有边界。

### 2.2 Agent 的天然弱点

| 弱点 | 说明 | 典型案例 |
|------|------|---------|
| **大小写敏感短参数** | 概率选择时易混淆 | `grep -a`（文本处理）vs `grep -A 5`（显示上下文）；`ssh -v`（verbose）vs `ssh -V`（version） |
| **交互式提示** | Agent 无法回答 `[y/N]` | AWS CLI v2 默认 pager 改为 less，导致全球 CI 任务挂起 |
| **幻觉参数** | 自信使用不存在的参数 | 前沿模型已改善，但开源/小模型幻觉率仍高 |
| **非结构化输出** | 无法可靠解析人类可读表格 | JSON 可直接提取字段，table 输出需要猜测 |

### 2.3 好 Agent vs 差 Agent 的区分

文章引用 Surge AI 案例：
- **差 Agent**：幻觉出不存在的类名 → 在错误基础上连续 22 轮坚持 → 39 轮对话改了 693 行代码 → 失败
- **好 Agent**：发现文件截断 → 主动重新读取完整内容 → 一次解决

**设计启示**：好的 CLI 应帮 Agent 区分「观察到的事实」和「猜测」，减少需要猜的东西，增加可验证的东西。

---

## 三、十条设计原则

### 原则 1：名词在前（Noun-Verb）

```bash
# 好：noun-verb，支持树搜索
docker container ls
gh pr create

# 坏：verb-noun，平铺无层级
create-pr
delete-image
```

**机制**：Agent 通过树搜索发现命令——先 `mytool --help` 看名词，再 `mytool user --help` 看动词。每一步都有 `--help` 可查，是确定性的逐层缩小过程。

### 原则 2：长参数优先（Long Flags First）

所有参数必须有长格式（`--verbose`），短格式（`-v`）可选。

**三个理由**：
1. **语义自描述**：`--dry-run` 自描述功能，`-n` 在不同工具中含义各异
2. **消除歧义**：`--verbose` vs `--version` 无混淆，`-v` vs `-V` 一字之差含义相反
3. **LLM 概率优势**：`--output` 在训练数据中与「指定输出」的语义绑定经过海量强化，`-o` 的绑定弱得多

### 原则 3：输出是契约（Structured Output as API Contract）

- stdout 输出 JSON 数据，stderr 输出状态信息，严格分流
- 结构化输出一旦发布就是 API：新增可选字段安全，改名/删除是破坏性变更
- 参考：GitHub CLI 检测管道时自动切换 tab 分隔格式

### 原则 4：感知环境（TTY-Aware Behavior）

- 终端中：彩色、表格、进度条、交互
- 管道中：JSON、无颜色、无交互、无分页
- 非 TTY 下应默认输出 JSON，不应要求额外加 `--json`

### 原则 5：干跑优先（Dry-Run by Default）

每个有副作用的命令都支持 `--dry-run`：
- 输出必须是结构化 JSON diff，不是一句文字提示
- 使用专门退出码（如 exit 10）区分干跑成功和真正执行成功
- 给 Agent 提供零成本的探索-验证循环

### 原则 6：退出码控制（Semantic Exit Codes）

| 退出码 | 含义 | Agent 应对 |
|--------|------|-----------|
| 0 | 成功 | 继续执行 |
| 1 | 一般错误 | 读 stderr 诊断 |
| 2 | 参数错误 | 修正参数重试 |
| 3 | 资源不存在 | 跳过或创建 |
| 4 | 权限不足 | 提示用户授权 |
| 5 | 冲突/已存在 | 跳过或更新 |
| 10 | 干跑通过 | 可安全执行 |

退出码必须跨版本稳定，是契约的一部分。

### 原则 7：防住幻觉（Input Validation & Hallucination Defense）

- 枚举约束：`--format json|table|csv` 而非 `--format <string>`
- 严格验证 URL、路径、域名
- 提供 **schema 自省**：`mytool schema user create` 输出参数定义的 JSON
- schema 应按需查询，而非一次性注入上下文（这是 CLI 相对 MCP 的核心优势）

### 原则 8：幂等设计（Idempotent Operations）

```bash
# 声明式：无论调用多少次结果一致
mytool user ensure --name "john"
mytool user create --name "john" --if-not-exists
```

- Agent 会重试，非幂等命令导致重复资源/消息/扣费
- 支持 `--idempotency-key` 处理天然非幂等操作
- kubectl apply 是黄金标准

### 原则 9：错误即指南（Actionable Error Messages）

好的错误包含四要素：
1. **错误类型**（机器可读）：`permission_denied`
2. **描述**（具体发生了什么）
3. **修复建议**（可执行的命令）
4. **是否可重试**：`retryable: true/false`

### 原则 10：帮助即大脑（Help Text is the Agent's Brain）

Anthropic 发现：**描述是影响工具使用准确率的最关键因素**。映射到 CLI：
- 以 2-3 个示例开头
- 明确标注必需/可选
- 参数描述包含值域
- 50 行以内

---

## 四、钉钉与飞书 CLI 实战评分

### 飞书 CLI 优势
- noun-verb 结构清晰，三层架构
- `lark-cli schema calendar.events.list` 提供完整 API 自省
- 五种输出格式（JSON、NDJSON、table、CSV、pretty）
- 按域申请权限（最小权限原则）
- 权限不足时自动提示修复方案

### 飞书 CLI 待改进
- 非 TTY 下未自动切 JSON
- 退出码缺少文档化的细粒度语义
- `--idempotency-key` 应更广泛使用

### 钉钉 CLI 优势
- `--yes`（AI Agent 模式）语义自描述
- `--mock` 参数方便调试
- 帮助文本全中文（对中文 LLM 更友好）
- 批量熔断防 Agent 失控

### 钉钉 CLI 待改进
- 命令只有两层缺少 shortcut 快捷层
- 没有 schema 自省命令
- 输出格式只有三种
- 帮助文本缺少使用示例

---

## 五、开源项目：agent-cli-guide

### 5.1 项目概况

| 项目 | 信息 |
|------|------|
| 仓库 | https://github.com/Johnixr/agent-cli-guide |
| 许可 | CC BY 4.0 |
| 核心文件 | GUIDE.md（设计指南）、CHECKLIST.md（开发清单） |
| 用途 | 丢给 Coding Agent 作为 CLI 开发参考 |

### 5.2 使用方式

在 CLAUDE.md 或项目指令中添加：
```
CLI design follows the Agent CLI Guide: https://github.com/Johnixr/agent-cli-guide/blob/main/GUIDE.md
```

或在提示中引用：
```
Reference https://github.com/Johnixr/agent-cli-guide for CLI design principles.
Follow the 10 principles and checklist in GUIDE.md.
```

### 5.3 GUIDE.md 内容摘要

GUIDE.md 是文章十条原则的规范化版本，每条原则包含：
- **原则说明**与代码示例
- **Why**：设计理由
- **Rules**：具体实施规则（比文章更详细）

相比文章，GUIDE.md 增加了以下细节：
- 原则 1 增加：禁止 catch-all 子命令、禁止前缀缩写
- 原则 2 增加：密码不通过 flag 传递，使用 `--password-file` 或 stdin
- 原则 3 增加：分页支持（`--page-size`、`--page-all`、NDJSON）
- 原则 7 增加：schema 自省应返回命令名、描述、flag 类型/默认值/枚举、必需/可选、示例
- 原则 8 增加：冲突应返回退出码 5 而非通用错误

### 5.4 CHECKLIST.md 内容摘要

按类别组织的 checklist，共 11 个类别、约 30 个检查项：

| 类别 | 检查项数 |
|------|---------|
| Command Structure | 3 |
| Flags & Parameters | 4 |
| Output | 4 |
| TTY Awareness | 4 |
| Safety & Dry-Run | 3 |
| Exit Codes | 3 |
| Input Validation | 3 |
| Idempotency | 3 |
| Error Messages | 3 |
| Help Text | 5 |

---

## 六、信源交叉分析

### 6.1 与 Justin Poehnelt 七模式的对比

| 维度 | Poehnelt 七模式 | 本文十原则 |
|------|----------------|-----------|
| 视角 | 改造现有 CLI | 设计新 CLI 或规范化 |
| 命令结构 | 提到但非核心 | 原则 1，作为首要原则 |
| Schema 自省 | 强调 `--schema` | 原则 7，作为防幻觉手段 |
| 幂等性 | 未专门讨论 | 原则 8，专门原则 |
| 退出码 | 简要提及 | 原则 6，细粒度定义 |
| 错误处理 | 提到结构化错误 | 原则 9，四要素框架 |
| 安全模型 | 「Agent 不是可信操作者」| 原则 7，具体验证规则 |
| 实战案例 | Google Workspace CLI | 钉钉/飞书 CLI |

**互补关系**：Poehnelt 更偏底层哲学（「Agent DX vs Human DX」），本文更偏实操规范。两者结合可形成完整的设计指导。

### 6.2 与 Lightning Labs 设计轴的关系

文章明确引用了 Lightning Labs 在 lnget 项目中提出的 agent-CLI 设计轴，其中几个具体设计直接进入了本文原则：
- 干跑退出码 10（原则 5/6）
- URL/路径/域名验证规则（原则 7）

### 6.3 与 Anthropic Tool Use 文档的关系

文章多次引用 Anthropic 的发现：
- 「描述是影响工具使用准确率的最关键因素」→ 原则 10
- 枚举约束策略 → 原则 7
- 按需查询 vs 全量注入 → 原则 7（CLI 优于 MCP 的论据）

---

## 七、对我们项目的启示

### 7.1 可直接整合到设计指南的内容

1. **十原则 checklist** 可作为 SKILL.md 中 CLI 设计部分的直接参考
2. **钉钉/飞书评分矩阵** 补充 G-05 设计规范的案例分析
3. **Agent 认知弱点框架** 补充设计原则的理论基础
4. **agent-cli-guide 项目** 可作为 SKILL.md 的外部引用资源

### 7.2 关键洞察

1. **CLI > MCP 的又一论据**：schema 按需查询（`--help` ~200 tokens）vs MCP 全量注入（~55,000 tokens），本文从设计原则层面再次确认
2. **幂等性是 Agent 场景的刚需**：人类很少重试，Agent 经常重试，这改变了 CLI 的设计优先级
3. **退出码从「可忽略细节」变成「控制流本身」**：这是 Agent 作为 CLI 消费者带来的根本性变化
4. **错误信息的角色转变**：从「给人看的提示」变成「给 Agent 的修复指南」，需要机器可读 + 可操作建议

### 7.3 注意事项

- 文章的来源整合性很强但缺乏独立实验验证，十条原则主要基于经验归纳而非受控实验
- agent-cli-guide 项目刚发布，社区采纳度有待观察
- 部分原则（如退出码 10 表示干跑成功）是新提议，尚未形成社区共识
