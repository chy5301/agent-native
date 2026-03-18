# You Need to Rewrite Your CLI for AI Agents

> 来源：Justin Poehnelt（Google Developer Relations Engineer）
> 链接：https://justin.poehnelt.com/posts/rewrite-your-cli-for-ai-agents/
> 参考实现：[Google Workspace CLI](https://github.com/googleworkspace/cli)

---

## 一、核心论点

**"Human DX optimizes for discoverability and forgiveness. Agent DX optimizes for predictability and defense-in-depth."**

人类 DX（开发者体验）和 Agent DX 的优化方向是正交的：

| 维度 | Human DX | Agent DX |
|------|----------|----------|
| 优化目标 | 可发现性、容错性 | 可预测性、纵深防御 |
| 输入偏好 | 扁平 flag、短选项 | 原始 JSON payload、结构化输入 |
| 输出偏好 | 格式化表格、彩色文本 | 机器可读 JSON |
| 文档形式 | `--help`、man page | Schema 自省、SKILL 文件 |
| 安全模型 | 信任用户输入 | 假设输入始终是对抗性的 |

核心判断：**给现有人类优先的 CLI 打补丁来适配 Agent 是低效的**。作者在构建 Google Workspace CLI 时，从一开始就将 Agent 作为主要消费者。

关键收尾语：**"The agent is not a trusted operator. Build like it."**

---

## 二、具体 CLI 改造建议（七大模式）

### 模式 1：原始 JSON Payload 优先于定制 Flag

**设计张力**：人类不喜欢在终端里写嵌套 JSON，Agent 却偏好它——因为 JSON 更具表达力且与 API schema 对齐。

人类优先的写法（扁平命名空间，有限嵌套）：

```bash
my-cli spreadsheet create \
  --title "Q1 Budget" \
  --locale "en_US" \
  --timezone "America/Denver" \
  --sheet-title "January" \
  --sheet-type GRID \
  --frozen-rows 1 \
  --frozen-cols 2 \
  --row-count 100 \
  --col-count 10 \
  --hidden false
```

Agent 优先的写法（完整 API payload）：

```bash
gws sheets spreadsheets create --json '{
  "properties": {
    "title": "Q1 Budget",
    "locale": "en_US",
    "timeZone": "America/Denver"
  },
  "sheets": [{
    "properties": {
      "title": "January",
      "sheetType": "GRID",
      "gridProperties": {
        "frozenRowCount": 1,
        "frozenColumnCount": 2,
        "rowCount": 100,
        "columnCount": 10
      },
      "hidden": false
    }
  }]
}'
```

**实践方案**：Google Workspace CLI 通过 `--params` 和 `--json` flag 接受完整 API payload，同时保留人类友好的 flag。在同一个二进制中通过环境变量或输出格式 flag 支持两种路径。

---

### 模式 2：Schema 自省替代静态文档

Agent 不读文档页面，它需要在运行时查询 CLI 自身来获取当前 API 的准确规格。

```bash
gws schema drive.files.list
gws schema sheets.spreadsheets.create
```

每次调用会 dump 出方法签名，包含：
- 参数列表及类型
- 请求体结构
- 响应类型
- 所需的 OAuth 作用域

输出格式为**机器可读 JSON**。

**技术实现**：利用 Google Discovery Document，动态解析 `$ref` 引用，使 CLI 本身成为当前 API 规格的权威来源（canonical source of truth），而非依赖可能过时的静态文档。

对于非 API 型 CLI，可以用 `--describe` 或 `--help --json` 提供类似能力。

---

### 模式 3：上下文窗口纪律（Context Window Discipline）

Workspace API 返回的 JSON blob 可能非常庞大，会吞噬 Agent 的上下文窗口。

**字段掩码（Field Masks）**：

```bash
gws drive files list --params '{"fields": "files(id,name,mimeType)"}'
```

**NDJSON 分页流式输出**：

`--page-all` flag 以 NDJSON（Newline Delimited JSON）格式输出，每页一个 JSON 对象。这允许流式处理，无需缓冲顶层数组。

关键指导（来自 CONTEXT.md）：

> "Workspace APIs return massive JSON blobs. ALWAYS use field masks when listing or getting resources by appending `--params '{"fields": "id,name"}'` to avoid overwhelming your context window."

---

### 模式 4：输入加固——防御 Agent 幻觉

**核心原则：Agent 不仅仅是打错字——它们会系统性地幻觉。**

四类攻击面及防御手段：

| 攻击类型 | Agent 的典型错误 | 防御函数 | 防御策略 |
|----------|-----------------|---------|---------|
| 文件路径穿越 | 生成 `../../.ssh` | `validate_safe_output_dir` | 规范化路径并沙箱化到当前工作目录 |
| 控制字符 | 产生不可见字符 | `reject_control_chars` | 拒绝所有 ASCII 0x20 以下的字符 |
| 资源 ID 注入 | 在 ID 中嵌入查询参数，如 `fileId?fields=name` | `validate_resource_name` | 拒绝包含 `?` 和 `#` 的输入 |
| 双重编码 | 预编码字符串导致 `%2e%2e` 代替 `..` | `validate_resource_name` | 拒绝包含 `%` 的输入 |
| URL 路径段 | 路径段中的特殊字符 | `encode_path_segment` | 在 HTTP 层进行百分号编码 |

**设计态度**："This CLI is frequently invoked by AI/LLM agents. Always assume inputs can be adversarial."

文章没有给出完整的函数实现代码，但给出了每个验证函数的核心行为描述。这些都是零信任输入验证的具体实践。

---

### 模式 5：Safety Rails —— `--dry-run` 与 `--sanitize`

**`--dry-run`**：
- 在本地验证请求，不发送 API 调用
- 让 Agent 能够"先思考再行动"
- 对变更操作（mutating operations）尤为关键
- Agent 可以先 dry-run，确认无误后再真正执行

**`--sanitize <TEMPLATE>`**：
- 将 API 响应通过 Google Cloud Model Armor 管道处理后再返回
- 防御数据中嵌入的 prompt 注入（例如：恶意邮件正文中包含覆盖指令的文本）
- 这是防御**间接 prompt 注入**的关键机制——Agent 读取的数据本身可能是攻击载体

---

### 模式 6：发布 Agent Skills，而非仅仅是命令

Skills 是结构化 Markdown 文件，带有 YAML frontmatter，每个 API 表面和工作流各一个。

示例结构：

```yaml
---
name: gws-drive-upload
version: 1.0.0
metadata:
  openclaw:
    requires:
      bins: ["gws"]
---
```

Skills 编码了 `--help` 无法表达的 Agent 特定指导：
- "Always use `--dry-run` for mutating operations"（变更操作始终先 dry-run）
- "Always confirm with user before executing write/delete commands"（写入/删除前始终确认）
- "Add `--fields` to every list call"（每个 list 调用都加字段掩码）

**规模**：Google Workspace CLI 附带了 **100+ 个 `SKILL.md` 文件**。

这与 CONTEXT.md 配合使用——CONTEXT.md 提供全局上下文（如"始终使用字段掩码"），SKILL.md 提供特定操作的详细指导。

---

### 模式 7：多表面架构——CLI + MCP + 扩展 + 环境变量

架构设计——一个事实来源，多个界面：

```
         Discovery Doc（事实来源）
                    ↓
              核心二进制 (gws)
         ↙        ↓        ↓        ↘
       CLI       MCP    Gemini      Env
     (人类)   (stdio)   扩展       变量
```

**MCP（Model Context Protocol）**：

```bash
gws mcp --services drive,gmail
```

将命令暴露为 JSON-RPC 工具（通过 stdio），实现类型化结构调用，消除 shell 转义和参数解析歧义。

**Gemini CLI Extension**：

```bash
gemini extensions install https://github.com/googleworkspace/cli
```

将二进制安装为原生 Agent 能力。

**环境变量认证**：
- `GOOGLE_WORKSPACE_CLI_TOKEN`——直接注入 token
- `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE`——指定凭据文件路径
- 避免需要浏览器重定向的 OAuth 流程，支持无头环境（headless environments）
- 尽可能使用 Service Account

---

## 三、实施优先级排序

文章给出了明确的渐进式实施路径：

| 优先级 | 改造项 | 说明 |
|--------|--------|------|
| **P0** | `--output json` | 机器可读输出基线，最小改动最大收益 |
| **P0** | 输入验证 | 拒绝控制字符、路径穿越、嵌入查询参数、双重编码 |
| **P1** | Schema / `--describe` | 运行时自省能力 |
| **P1** | 字段掩码 / `--fields` | 上下文窗口保护 |
| **P2** | `--dry-run` | 变更前验证 |
| **P2** | CONTEXT.md / SKILL 文件 | 编码 Agent 无法自行推断的不变量 |
| **P3** | MCP 表面 | 适用于 API 型 CLI |

文章强调：**不需要从头重写**。大多数模式可以增量添加。

---

## 四、与 MCP 的关系

文章将 MCP 定位为 CLI 的**补充表面**而非替代品：

1. **CLI 和 MCP 共享同一个核心二进制**——从同一个 Discovery Doc 生成，逻辑一致
2. **MCP 的优势**：消除 shell 转义、参数解析歧义和输出解析问题；Agent 调用类型化函数而非构造字符串
3. **MCP 的适用场景**："If your CLI wraps a structured API, yes"——对于包装结构化 API 的 CLI，MCP 值得投资
4. **MCP 不是唯一路径**——即使有 MCP，CLI 仍然需要，因为人类仍会直接使用

关键洞察：MCP 在优先级排序中排在最后（P3），说明作者认为**做好 CLI 本身的 Agent 适配**比暴露 MCP 接口更重要、更基础。CLI 做好了，MCP 只是多一个 transport 层。

---

## 五、对现有 CLI 工具的适用性分析

文章没有直接提及 gcloud、aws cli 等工具，但其 FAQ 部分提供了适用性指导：

### 对 API 型 CLI（如 gcloud、aws cli、az cli）

这类工具最适合文章的全部七个模式：

| 模式 | 适用性 | 现状评估 |
|------|--------|---------|
| JSON payload | 高 | aws cli 已支持 `--cli-input-json`；gcloud 部分支持 |
| Schema 自省 | 高 | aws cli 有 `aws help` 但非机器可读；gcloud 有 Discovery Doc 但未暴露为 CLI 命令 |
| 字段掩码 | 高 | 两者均支持 `--fields` 或 `--query`，但不是 Agent 友好的默认行为 |
| 输入加固 | 关键 | 目前几乎没有针对 Agent 幻觉的防御 |
| `--dry-run` | 高 | aws cli 有 `--dry-run`（EC2）；gcloud 部分支持 |
| SKILL 文件 | 缺失 | 均未提供 Agent 特定的行为指导文件 |
| MCP | 高 | 两者都包装结构化 API，非常适合 MCP 化 |

### 对非 API 型 CLI（如 git、docker、ffmpeg）

文章 FAQ 明确回答：原则仍然适用。

- **必须做**：机器可读输出、输入加固、明确的不变量文档
- **最有价值**：`--describe` 或 `--help --json` 形式的自省
- **较低优先级**：Schema 自省（因为没有后端 API schema 可映射）

### 对 aws cli 的具体观察

aws cli 是现有工具中最接近文章理念的：
- 已有 `--output json`
- 已有 `--cli-input-json` / `--cli-input-yaml`（对应 JSON payload 模式）
- 已有 `--generate-cli-skeleton`（对应 Schema 自省的雏形）
- 缺失：输入加固、SKILL 文件、MCP 表面、`--sanitize`

### 对 gcloud 的具体观察

gcloud 作为 Google 自己的工具，与文章描述的 Google Workspace CLI 同源但设计哲学更偏人类优先：
- 有 `--format=json` 但默认是人类可读格式
- 有 Discovery Doc 基础但未暴露运行时 schema 查询
- 缺失：Agent 输入加固、SKILL 文件、`--sanitize`

---

## 六、关键启示

1. **Agent 安全是全新的安全领域**：不是传统的用户输入校验，而是防御系统性幻觉——Agent 会以自信的方式生成完全错误的路径、ID 和编码。
2. **SKILL 文件是 Agent 时代的 README**：100+ 个 SKILL.md 说明这不是点缀，而是核心产品组件。
3. **`--sanitize` 是前沿创新**：利用 Model Armor 防御间接 prompt 注入，这在其他 CLI 工具中几乎没有先例。
4. **"One source of truth, two interfaces" 是正确的架构**：避免 CLI 和 MCP 逻辑分叉，从同一个 spec 生成两个表面。
5. **优先级排序极其务实**：先做 JSON 输出和输入验证就能覆盖 80% 的 Agent 使用场景。

---

## 七、与本工作空间核心命题的关联

| 本工作空间命题 | 文章验证 |
|---------------|---------|
| GUI 是翻译层，Agent 不需要 | 文章完全绕过 GUI，CLI 是唯一界面 |
| 产品从 App 变成 Skill | 100+ SKILL.md 文件就是 "产品即 Skill" 的实践 |
| 软件从资产变成耗材 | Discovery Doc → 动态生成 schema → Agent 按需查询，无需安装文档 |
| 中间层消亡 | MCP 消除了 shell 转义/解析这个"中间层"；`--json` 消除了输出格式化这个"中间层" |

这篇文章是目前最具体的 "如何构建 Agent-Native CLI" 实践指南，且来自 Google 内部的一手经验。
