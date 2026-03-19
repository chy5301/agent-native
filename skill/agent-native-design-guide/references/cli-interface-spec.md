# Agent-Native CLI 接口设计规范

> 本文档是 Agent-Native 设计指南 Skill 的 reference 文件。
> 提供面向 Agent 的 CLI 接口设计具体规范，涵盖命令结构、参数、输出、可发现性、错误处理五个维度。
> 关联原则：C1 文本即界面、P1 可发现性、P2 确定性、P3 输出即产品（见 design-principles.md）

---

## 核心认知

Human DX 和 Agent DX 的优化方向正交：

| 维度 | Human DX | Agent DX |
|------|----------|----------|
| 输入偏好 | 扁平 flag、短选项 | JSON payload、结构化输入 |
| 输出偏好 | 格式化表格、彩色文本 | 机器可读 JSON |
| 文档形式 | `--help`、man page | Schema 自省、SKILL.md |
| 安全模型 | 信任用户输入 | 假设输入始终对抗性 |

**量化证据**：CLI token 效率比 MCP 高 33%；CLI `--help` 仅 ~200 tokens vs MCP 连接注入 ~55,000 tokens；结构化输出优化可将 Agent 解读时间降低 5-9 倍。

**结论**：设计良好的 CLI 接口是 Agent-Native 的基础。好的 CLI 同时服务人类和 Agent——通过双模策略（默认人类友好，`--json` 切换 Agent 友好）。

---

## 1. 命令结构

### 1.1 命名约定

`<tool> <resource> <action>` 三段式，与 REST 语义对齐：

```bash
# 好：资源-动作分离，可预测
gws sheets create
docker container list

# 差：动词在前，资源不明
do-spreadsheet --action create
manage --type sheets
```

规则：
- 小写 kebab-case（`my-tool`，非 `myTool`）
- 动词用 CRUD 语义：`create`、`list`、`get`、`update`、`delete`
- 避免缩写，除非行业通用（`ls`、`rm`）

### 1.2 子命令分组

CLI-Anything 在 11 个 GUI 软件改造中沉淀出标准五组：

| 组名 | 职责 | 典型命令 |
|------|------|---------|
| **Project** | 项目/资源管理 | `open`、`close`、`save`、`info` |
| **Core** | 核心领域操作 | 因领域而异 |
| **IO** | 格式转换 | `import`、`export`、`render` |
| **Config** | 配置管理 | `config get`、`config set` |
| **Session** | 会话与撤销 | `undo`、`redo`、`history` |

简单工具可能只需 Core + IO，但超过 10 个子命令时应考虑分组。

### 1.3 交互模型

推荐 Subcommand CLI 为主、REPL 为辅的双模式：

```bash
# Agent 首选：标准 CLI
my-tool project open ./data.json --json
my-tool core transform --params '{"scale": 2}'

# 人类补充：REPL
my-tool          # 无参数进入 REPL
> open data.json
> transform --scale 2
```

Agent 不需要 REPL——它们通过独立命令调用更高效。始终优先确保 Subcommand CLI 模式可用。

### 1.4 全局选项

每个 Agent-Native CLI 应支持的标准全局选项：

| 选项 | 作用 | 优先级 |
|------|------|--------|
| `--json` | 机器可读 JSON 输出 | **P0** |
| `--quiet` | 抑制非关键输出 | P1 |
| `--verbose` | 调试级别输出 | P1 |
| `--dry-run` | 仅验证，不执行 | P1 |
| `--fields <mask>` | 字段掩码，控制输出字段 | P1 |
| `--version` | 版本信息 | P2 |

---

## 2. 参数设计

### 2.1 双路径策略

人类不喜欢在终端写嵌套 JSON，Agent 偏好它。解决方案是双路径：

```bash
# Agent 路径：JSON payload
my-tool create --params '{"title": "Q1 Report", "format": "pdf"}'

# 人类路径：扁平 flag
my-tool create --title "Q1 Report" --format pdf
```

当 `--params` 存在时，忽略单独的 flag，避免合并歧义。

### 2.2 Schema 自省

Agent 需要在运行时查询 CLI 的准确规格：

```bash
# 理想方式：专门的 schema 命令
my-tool schema create
# 输出 JSON Schema：参数类型、必填/可选、默认值、枚举值

# 轻量替代：--help --json
my-tool create --help --json
```

最小实现示例：

```json
{
  "command": "create",
  "parameters": [
    {"name": "title", "type": "string", "required": true},
    {"name": "format", "type": "string", "enum": ["pdf", "html", "json"], "default": "json"}
  ]
}
```

### 2.3 输入加固

Agent 不是可信操作者。四类攻击面及防御：

| 攻击类型 | Agent 典型错误 | 防御策略 |
|----------|---------------|----------|
| 文件路径穿越 | 生成 `../../.ssh` | 规范化路径，沙箱化到工作目录 |
| 控制字符 | 产生不可见字符 | 拒绝 ASCII 0x20 以下字符 |
| 资源 ID 注入 | ID 中嵌入 `?fields=name` | 拒绝含 `?` 和 `#` 的输入 |
| 双重编码 | 预编码 `%2e%2e` | 拒绝含 `%` 的输入 |

在 CLI 入口层统一校验，校验失败时返回结构化错误（见第 3 节）。

---

## 3. 输出设计

> "最快的 Agent 不是推理能力最强的那个，而是不需要对数据格式进行推理的那个。"

输出设计是被严重低估的关键维度（原则 P3：输出即产品）。实测数据：一个测试套件运行 96 秒，Agent 解读结果花了 608 秒——优化输出后降至 66 秒（9.2 倍改善）。

### 3.1 标准 JSON 信封结构

所有 `--json` 输出遵循统一信封：

**成功响应**：

```json
{
  "success": true,
  "data": { "id": 1, "name": "Project A", "status": "active" },
  "metadata": {
    "command": "my-tool project get",
    "timestamp": "2026-03-18T10:30:00Z",
    "version": "1.2.0"
  }
}
```

**错误响应**：

```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Parameter 'format' must be one of: pdf, html, json",
    "parameter": "format",
    "provided": "docx",
    "allowed": ["pdf", "html", "json"]
  }
}
```

关键规则：
- 错误信息必须包含**足够 Agent 自行修正**的信息（提供值、允许值）
- 使用机器可读的 `code` 字段，不依赖 `message` 文本解析
- 列表操作 `data` 返回数组，单项操作返回对象

完整代码示例见 `examples/cli-json-output.py`。

### 3.2 字段掩码

大型 JSON 响应会吞噬 Agent 的上下文窗口。`--fields` 让 Agent 只获取需要的字段：

```bash
# 完整输出（可能数千 token）
my-tool project get --id 123 --json

# 按需裁剪
my-tool project get --id 123 --json --fields "id,name,status"
```

### 3.3 流式输出

大数据集使用 NDJSON（Newline Delimited JSON）：

```bash
my-tool records list --all --ndjson
{"id": 1, "name": "record_1", "status": "active"}
{"id": 2, "name": "record_2", "status": "archived"}
```

优势：可逐行处理、与 `jq`/`grep` 兼容、Agent 可中途停止。

### 3.4 双模输出

默认人类可读，`--json` 切换 Agent 可读：

```bash
# 人类模式（默认）
$ my-tool project list
 ID  | Name      | Status
-----|-----------|--------
 1   | Project A | active
 2   | Project B | done

# Agent 模式
$ my-tool project list --json
{"success":true,"data":[...]}
```

### 3.5 进度报告

长时间操作用 NDJSON 格式报告进度（输出到 stderr），最终结果输出到 stdout：

```json
{"type": "progress", "stage": "downloading", "percent": 45}
{"type": "result", "success": true, "data": {"output": "report.pdf"}}
```

---

## 4. 可发现性

### 4.1 --help 设计规范

`--help` 是 Agent 发现工具能力的第一入口。好的 `--help` 输出：

```
my-tool - Agent-native data processing tool

USAGE:
  my-tool <command> [options]

COMMANDS:
  project     Project management (open, close, save, info)
  transform   Data transformation (resize, convert, filter)
  export      Export to various formats (pdf, html, csv)
  config      Configuration management
  session     Session and undo/redo

GLOBAL OPTIONS:
  --json          Output in JSON format
  --quiet         Suppress non-critical output
  --dry-run       Validate without executing
  --fields MASK   Select output fields

EXAMPLES:
  my-tool project open ./data.json --json
  my-tool transform resize --params '{"width": 800}' --json
```

差的 `--help`：过长描述段落、缺少 EXAMPLES、选项未分组、未展示 `--json`。

完整代码示例见 `examples/cli-help-design.py`。

### 4.2 SKILL.md 推荐结构

SKILL.md 是 Agent 平台发现和使用工具的入口：

```yaml
---
name: my-tool
description: >
  数据处理工具的 Agent 接口。触发词：数据转换、格式导出、
  批量处理。当需要处理结构化数据或生成报告时使用。
---
```

正文依次包含：安装说明、快速开始（典型工作流）、命令速查表、Agent 使用规则（5 条）、Gotchas。

关键设计原则：
1. **description 给模型看**——重点是触发条件。差："A comprehensive data processing tool"。好："数据转换、格式导出。当需要处理结构化数据时使用"
2. **别说废话**——只写能推动 Agent 偏离默认行为的信息
3. **Gotchas 是灵魂**——从实际 Agent 踩坑中积累，每次犯错加一条
4. **文件夹做信息分层**——SKILL.md 精简索引，详细内容放 `references/` 按需加载

### 4.3 渐进式信息分层

复杂工具用文件夹结构做 context engineering（原则 P5）：

```
my-tool/
├── SKILL.md              # ~30 行索引
├── references/
│   ├── api.md            # 完整 API 参考
│   └── gotchas.md        # 踩坑记录（持续增长）
└── lib/
    └── helpers.py        # 可复用辅助函数
```

Agent 读 SKILL.md 获取概览，按需深入。用文件系统做上下文窗口管理。

---

## 5. 错误处理与安全护栏

### 5.1 退出码约定

| 退出码 | 含义 |
|--------|------|
| 0 | 成功 |
| 1 | 一般错误 |
| 2 | 参数/用法错误 |
| 3 | 权限不足 |
| 4 | 依赖缺失 |
| 126 | 命令不可执行 |
| 127 | 命令未找到 |

### 5.2 --dry-run 预览

对变更操作（create/update/delete），支持 `--dry-run`：

```bash
$ my-tool project delete --id 123 --dry-run --json
{
  "dry_run": true,
  "action": "delete",
  "target": {"id": 123, "name": "Project A"},
  "effects": ["Delete project and 15 associated records"],
  "reversible": false
}
```

Agent 先 dry-run 查看影响范围，确认后再执行。SKILL.md 中应明确指导"不可逆操作始终先 dry-run"。

### 5.3 --sanitize 防御间接注入

CLI 输出含用户生成内容时，为不受信任内容添加边界标记：

```json
{
  "subject": "Meeting Notes",
  "body": "<!-- BEGIN_USER_CONTENT -->\n内容...\n<!-- END_USER_CONTENT -->"
}
```

让 Agent 平台识别并隔离不受信任的内容（详细安全规范见 security-model.md）。

---

## 6. 实施路线图

渐进式实施，P0 改造就能覆盖 80% 的 Agent 使用场景：

| 优先级 | 改造项 | 收益 |
|--------|--------|------|
| **P0** | `--json` 输出 | Agent 可用的基线 |
| **P0** | 输入验证/加固 | 安全基线 |
| **P0** | 结构化错误输出 | Agent 可自行修正错误 |
| **P1** | `--fields` 字段掩码 | 上下文窗口保护 |
| **P1** | Schema 自省 | 运行时可发现性 |
| **P1** | `--dry-run` | 变更前验证 |
| **P2** | SKILL.md | Agent 平台集成 |
| **P2** | NDJSON 流式输出 | 大数据集支持 |
| **P3** | `--sanitize` | 间接注入防护 |

**不需要从头重写——大多数模式可以增量添加到现有 CLI。**

---

## 7. 设计对比示例

### 对比 1：输出设计

```bash
# 差：人类可读但 Agent 无法可靠解析
$ bad-tool list
Found 3 items:
  1. Project A (active) - created 2026-01-15
  2. Project B (done) - created 2026-02-20
Total: 3 items

# 好：结构化 JSON，Agent 零解析成本
$ good-tool list --json
{"success":true,"data":[
  {"id":1,"name":"Project A","status":"active","created":"2026-01-15"},
  {"id":2,"name":"Project B","status":"done","created":"2026-02-20"}
]}
```

### 对比 2：错误信息

```bash
# 差：Agent 无法自行修正
$ bad-tool create --format docx
Error: invalid format

# 好：包含修正所需的全部信息
$ good-tool create --format docx --json
{"success":false,"error":{
  "code":"INVALID_PARAMETER",
  "message":"Unsupported format 'docx'",
  "parameter":"format",
  "provided":"docx",
  "allowed":["pdf","html","json","csv"]
}}
```

### 对比 3：可发现性

```bash
# 差：无结构化 schema
$ bad-tool create --help
Usage: bad-tool create [OPTIONS]
Create a new resource.

# 好：结构化 schema，Agent 精确调用
$ good-tool schema create
{"command":"create","parameters":[
  {"name":"title","type":"string","required":true},
  {"name":"format","type":"string","enum":["pdf","html","json"],"default":"json"}
]}
```

---

## 附录：与设计原则的映射

| 本文档规范 | 对应原则 | 关系 |
|-----------|----------|------|
| 命令结构三段式 | C1 文本即界面 | CLI 命令是 Agent 的感知通道 |
| 双路径参数 | C3 对等性 | Agent 和人类用不同方式达到同一目的 |
| JSON 信封结构 | P3 输出即产品 | 输出设计比功能设计更影响 Agent 效率 |
| Schema 自省 | P1 可发现性 | Agent 运行时查询而非依赖静态文档 |
| 字段掩码 | P5 渐进式信息披露 | 只加载需要的上下文 |
| --dry-run | C4 安全边界优先 | 变更操作的安全预览 |
| Gotchas 积累 | P4 Gotchas 驱动迭代 | Skill 通过实践反馈持续演化 |
| SKILL.md 结构 | P6 约束意图而非步骤 | 说清楚意图，把方法交给 Agent |

---

## 平台适配速查

| 平台 | 发现机制 | 适配要点 |
|------|----------|----------|
| Claude Code | SKILL.md | frontmatter + description triggers |
| OpenCode | `--help` + `--json` | `--help` 输出需结构化，无 SKILL.md 支持 |
| OpenClaw | SKILL.md（skill 市场） | 与 Claude Code 兼容，增加 openclaw 元数据 |

最小覆盖：良好的 `--help` + `--json`（覆盖所有平台）→ SKILL.md（覆盖 Claude Code + OpenClaw）。

---

## Gotchas（来自实战经验）

以下来自 CLI-Anything 对 11 个 GUI 软件改造的血泪教训：

1. **包装而非替代**——CLI 必须调用真实软件进行渲染/导出，用 Python 重新实现会丢失滤镜效果、色彩管理等专业能力
2. **不要静默降级**——真实软件是硬依赖，缺少时明确报错（退出码 4），不要用简化方法完成任务
3. **`open("w")` 陷阱**——写文件时 `open("w")` 会在获取文件锁之前截断内容，应先 `open("r+")` 获取锁再 `truncate()`
4. **参数空间差异**——同一效果在不同格式中的参数范围不同（如 MLT `brightness 1.15` 对应 ffmpeg `eq=brightness=0.06`），不能简单映射
