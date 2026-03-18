# 能力封装协议深度对比

> 目标：为后续工具的接入方式选择提供依据
> 目标平台：Claude Code、OpenCode、OpenClaw

---

## 一、协议概述

### MCP (Model Context Protocol)

- **思路**：工具描述通过 JSON Schema 注入 Agent 上下文，Agent 通过 JSON-RPC 调用工具
- **传输方式**：stdio（本地进程）、Streamable HTTP（远程）、SSE（已弃用）
- **核心优势**：类型化调用消除 shell 转义歧义，自动 schema 发现，跨平台支持
- **核心劣势**：token 开销大（所有工具描述塞进 context），43% 早期 Server 有注入漏洞（OWASP），需要额外的 Server 进程
- **安全规范**：OAuth 2.1 强制认证（2025-03 更新），参数需 JSON Schema 验证

### CLI + Skill

- **思路**：Agent 读 SKILL.md 了解能力，通过 Bash 执行 CLI 命令
- **传输方式**：shell 标准输入/输出
- **核心优势**：轻量（SKILL.md 仅几百 tokens）、确定性强、可组合、无额外进程
- **核心劣势**：需要 shell 转义处理、输出解析可能不稳定、需手写 SKILL.md 或用 CLI-Anything 生成
- **安全考量**：需自行实现输入验证（路径穿越、控制字符、双重编码防护）

### A2A (Agent-to-Agent) v0.3.0 RC1

- **思路**：Agent 间直接通信，通过 Agent Card 发现机制互相发现能力
- **传输方式**：HTTP/gRPC
- **核心优势**：标准化的 Agent 间协作协议，支持安全卡签名
- **核心劣势**：生态尚早期，面向 Agent 间通信而非工具调用，目标平台暂无原生支持
- **现状**：已移交 Linux Foundation，150+ 组织参与，Python/JS SDK 可用

### OpenAPI / REST

- **思路**：传统 API 描述规范，通过 HTTP 调用
- **传输方式**：HTTP
- **核心优势**：生态最成熟，工具链完善，几乎所有服务都有 REST API
- **核心劣势**：为人类开发者设计（非 Agent 原生），缺少 Agent 特定的可发现性和安全防护
- **适用性**：Agent 可通过 Bash 调用 curl/httpie，或通过 MCP 包装后调用

---

## 二、多维度对比矩阵

| 维度 | MCP | CLI + Skill | A2A | OpenAPI/REST |
|------|-----|-------------|-----|-------------|
| **Token 成本** | 高（工具定义塞进 context） | 低（SKILL.md 几百 tokens） | 中（Agent Card 描述） | 低（不注入 context） |
| **确定性** | 高（类型化 JSON-RPC） | 中（依赖 shell 解析） | 高（结构化协议） | 中（依赖 HTTP 解析） |
| **可发现性** | 高（自动 schema 注入） | 中（需读 SKILL.md 或 --help） | 高（Agent Card 发现） | 中（需读 OpenAPI spec） |
| **可组合性** | 中（工具间无标准编排） | 高（shell 管道天然可组合） | 高（Agent 间可协作编排） | 低（需手动编排） |
| **开发复杂度** | 中（需实现 MCP Server） | 低（CLI + SKILL.md） | 高（需实现 A2A Agent） | 低（标准 REST API） |
| **生态成熟度** | 高（FastMCP、SDK、脚手架） | 中（CLI-Anything、ClawHub） | 低→中（刚移交 LF） | 高（最成熟） |
| **安全机制** | 高（OAuth 2.1、Schema 验证） | 低→中（需自行实现） | 中（安全卡签名） | 中（标准 HTTP 安全） |
| **额外进程** | 需要（MCP Server 进程） | 不需要（直接调 CLI） | 需要（A2A Agent 进程） | 不需要（HTTP 调用） |

---

## 三、目标平台支持评估

| 协议 | Claude Code | OpenCode | OpenClaw |
|------|------------|----------|----------|
| **MCP** | 完整支持（HTTP/SSE/stdio） | 支持（stdio） | 支持（MCP Registry） |
| **CLI + Skill** | 完整支持（Agent Skills 标准 + .claude/skills/） | 仅 Bash 工具（无 SKILL.md） | 完整支持（ClawHub + SKILL.md） |
| **A2A** | 不支持 | 不支持 | 不支持 |
| **OpenAPI/REST** | 通过 Bash curl 或 MCP 包装 | 通过 Bash curl 或 MCP 包装 | 通过 system.run curl 或 MCP 包装 |

### 平台架构差异

| 维度 | Claude Code | OpenCode | OpenClaw |
|------|------------|----------|----------|
| **定位** | 最成熟的 CLI Agent | 精简的代码编辑 Agent | 全平台个人助手/"Agent OS" |
| **架构** | 单体 CLI | 单体 CLI + TUI | Gateway-Agent 分布式 |
| **工具调用** | MCP JSON-RPC + 内置工具 | MCP stdio + 内置工具 | WebSocket RPC + MCP + system.run |
| **Skill 生态** | Agent Skills 标准 | 无 | ClawHub（5400+ 社区技能） |
| **多渠道** | Terminal/IDE/Web/Slack | Terminal + TUI | Telegram/Discord/Slack/Signal/iMessage/Web/... |
| **插件系统** | Plugins（MCP+Skills+Hooks 打包） | 无 | Extensions（workspace packages） |

---

## 四、场景推荐

### 场景 A：简单工具调用（查询、格式转换等）

**推荐**：CLI + Skill

- Token 开销最低，开发最快
- 3 个目标平台中 2 个原生支持（Claude Code、OpenClaw），OpenCode 可通过 Bash 工具执行
- 示例：文件处理工具、数据转换工具、系统管理脚本

### 场景 B：复杂 API 包装（多端点、多参数、需类型安全）

**推荐**：MCP（主） + CLI + Skill（辅）

- MCP 提供类型化调用，消除 shell 转义和参数解析歧义
- CLI 作为人类直接使用的备选入口
- 参考 Google Workspace CLI 的"一个事实来源，两个界面"架构
- 3 个平台均支持 MCP

### 场景 C：Agent 间协作（多 Agent 工作流）

**推荐**：暂用 MCP，关注 A2A 演进

- A2A 是 Agent 间通信的正确抽象，但目标平台暂无原生支持
- 短期内可用 MCP 模拟 Agent 间工具共享
- OpenClaw 的 Gateway 架构和跨会话通信（`sessions_send`）已部分实现 Agent 协作

### 场景 D：面向最广泛 Agent 平台的通用工具

**推荐**：CLI + Skill（底层） + MCP（可选表面）

- CLI 是最通用的——任何能执行 shell 命令的 Agent 都能调用
- SKILL.md 为支持 Skill 的平台提供增强可发现性
- MCP 作为可选表面为支持 MCP 的平台提供类型安全
- 优先级参照 Justin Poehnelt 建议：`--json` → 输入验证 → schema 自省 → SKILL.md → MCP

---

## 五、推荐策略

### 主方案：CLI + Skill

**理由**：
1. **覆盖面最广**：CLI 是所有 Agent 平台的最大公约数（Claude Code/OpenCode/OpenClaw 均可通过 Bash/system.run 执行）
2. **开发成本最低**：写 CLI + SKILL.md 比实现 MCP Server 简单
3. **Token 效率最高**：SKILL.md 仅几百 tokens vs MCP 的完整 schema 注入
4. **确定性设计**：`--json` 输出 + 输入验证 = 高确定性
5. **与用户现有判断一致**：CLI+Skill 是 Agent 时代的原生交互方式

### 补充方案：MCP 表面

**何时加**：当工具包装的是复杂结构化 API（多端点、嵌套参数），CLI 的 shell 转义和参数解析成为瓶颈时。

**怎么加**：参照"一个事实来源，两个界面"模式——核心逻辑实现一次，CLI 和 MCP 共享同一份 schema/逻辑。

### 实施优先级

| 优先级 | 动作 | 覆盖平台 |
|--------|------|---------|
| P0 | CLI + `--json` 输出 + 输入验证 | 全部 |
| P1 | SKILL.md | Claude Code, OpenClaw |
| P1 | `--help --json` / schema 自省 | 全部 |
| P2 | `--dry-run` + `--fields` | 全部 |
| P3 | MCP Server 表面 | 全部（支持 MCP 的平台体验更优） |

---

## 六、关键结论

1. **CLI 是 Agent 世界的 USB 接口**——最通用、最低门槛的工具接入方式
2. **MCP 是 CLI 的补充而非替代**——优先做好 CLI，MCP 作为增强层按需添加
3. **A2A 值得关注但暂不投入**——目标平台均无原生支持，等生态成熟
4. **SKILL.md 是高杠杆投入**——几百 tokens 的文件能显著提升 Agent 对工具的理解和使用效率
5. **OpenCode 是短板**——不支持 SKILL.md，但可通过 Bash 工具 + `--help` 覆盖
