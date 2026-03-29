# Agent-Native 安全与权限模型

> 本文档是 Agent-Native 设计指南 Skill 的 reference 文件。
> 提供面向 Agent 工具的安全设计规范：威胁模型、权限设计、输入校验、安全检查清单。
> 关联原则：C4 安全边界优先、P2 确定性（见 design-principles.md）

---

## 核心认知

Agent 架构存在一个根本性矛盾：**Agent 需要读取外部数据来完成任务，但外部数据本身可能包含恶意指令。** 这与 SQL 注入类似，但攻击面更广——Agent 的行为由 prompt（指令）+ context（数据）共同决定，而 context 来自外部。

| 传统 IAM 假设 | Agent 场景现实 |
|--------------|--------------|
| 粗粒度角色权限 | Agent 获得远超单次任务所需的权限 |
| 静态身份绑定 | Agent 身份短暂、动态创建、可被委托 |
| 人类在回路中审批 | Agent 自主决策，无法逐次审批 |
| 单体应用边界 | Agent 跨工具、跨服务、跨 Agent 调用 |
| 审计基于操作者 | 委托链中责任难以归属 |

**生产环境现状**：88% 组织已报告 Agent 安全事件；仅 14.4% 的部署获得完整安全审批；Shadow AI 泄露平均多花费 $670K；MCP 生态 60 天累计 30+ CVE；ClawHub 遭 1,184 个恶意 Skill 供应链攻击。

**结论**：安全必须从第一行代码开始设计，不是发布后再补。工具开发者不能依赖平台防御——不同平台安全能力差异巨大。

---

## 1. 威胁模型速查

基于 OWASP Agentic Top 10（2026），精简为工具开发者需要关注的核心威胁和防护措施。

| 编号 | 威胁 | 工具开发者的防护重点 |
|------|------|-------------------|
| ASI01 | **目标劫持** — 恶意文本篡改 Agent 目标 | 输入边界标记、指令-数据分离（JSON 参数而非自然语言拼接）、输出净化 |
| ASI02 | **工具误用** — 参数注入、路径穿越、权限滥用 | 输入验证加固、`--dry-run`、操作白名单、路径沙箱 |
| ASI03 | **身份滥用** — Agent 以用户身份执行超出意图的操作 | 最小权限声明、分级授权、JIT 短生命周期凭证 |
| ASI04 | **供应链攻击** — Skill/插件被投毒 | Skill 签名、依赖锁定、自动安全扫描 |
| ASI05 | **代码执行** — Agent 生成或运行不安全代码 | 沙箱执行、代码审查 Hook、可执行文件白名单 |
| ASI06 | **上下文投毒** — 污染 Agent 记忆影响长期决策 | 记忆写入验证、来源信任标记、衰减机制 |
| ASI07 | **通信不安全** — Agent 间消息缺乏认证 | 消息签名、身份验证、TLS 加密 |
| ASI08 | **级联失败** — 错误在 Agent 链中放大 | 幂等操作、检查点机制、断路器模式 |
| ASI09 | **信任滥用** — 用户过度信任 Agent 建议 | 透明决策、操作预览（`--dry-run`）、不可逆操作警告 |
| ASI10 | **恶意 Agent** — 被攻破的 Agent 暗中执行有害行为 | 行为监控、输出审计、最小网络权限 |

**工具开发者最直接面对的威胁**：ASI01（目标劫持）、ASI02（工具误用）、ASI03（身份滥用）。这三项的防护主要在工具层实现，不依赖平台。

---

## 2. 权限模型设计

### 2.1 分级授权

| 级别 | 权限范围 | 确认机制 | 典型场景 |
|------|---------|---------|---------|
| **L0** 只读 | 读取指定目录/文件 | 无需确认 | 查询、搜索、信息获取 |
| **L1** 受限写入 | 写入指定目录 | 首次确认 | 文件生成、配置修改 |
| **L2** 系统操作 | 执行 shell 命令 | 逐次确认或白名单 | 构建、测试、部署 |
| **L3** 敏感操作 | 网络请求、凭证访问 | 强制逐次确认 | API 调用、认证操作 |
| **L4** 不可逆操作 | 删除、发布、push | 强制确认 + dry-run | 生产部署、数据删除 |
| **L4+** 网络不可撤回 | 发消息、提交表单 | 强制确认 + 预览 + 日志 | 浏览器代理场景（发出即不可撤回） |

### 2.2 确认机制

```
自动允许：已白名单的 L0 读取、已确认的 L1 写入
    ↓
沙箱内执行：中风险操作在沙箱内自动执行
    ↓
单次确认：新的 L2/L3 操作，确认后可选"总是允许"
    ↓
强制确认：L4 不可逆操作，每次确认，先 dry-run
```

**关键设计原则**：沙箱化减少 84% 的权限提示（Claude Code 实测数据）。**定义边界后在边界内自由操作，比逐次确认更实用**——频繁确认会被用户无脑点"允许"，反而降低安全性。

### 2.3 能力声明

工具/Skill 在元数据中显式声明所需权限，运行时平台强制执行：

```yaml
# SKILL.md frontmatter 或 skill.json 中
permissions:
  filesystem: read          # L0: 只读文件系统
  paths: ["/data/*.csv"]    # 限定路径范围
  network: outbound         # L3: 需要网络请求
  domains: ["api.example.com"]  # 限定域名
  executables: ["python3"]  # 限定可执行文件
```

**原则**：声明不多不少。声明过宽（"需要完整文件系统访问"）会触发用户警觉；声明过窄会在运行时报错。类似 Android 权限模型，但粒度更细。

### 2.4 JIT 权限与委托链

- **即时权限**：凭证有效期 15 分钟（可配置），到期自动收回，需要时重新申请
- **委托链追踪**：用户→Agent A→Agent B→工具调用，整条链通过身份标识可审计，每个节点记录调用者、权限、时间戳、操作

---

## 3. 输入校验规范

### 3.1 路径安全

所有文件路径参数必须经过三步处理：

1. **规范化**：解析 `..`、符号链接、URL 编码
2. **边界检查**：确认路径在允许范围内（工作目录或声明路径）
3. **拒绝穿越**：检测 `../`、`%2e%2e/`、双重编码

### 3.2 参数验证

- **ID/名称**：正则校验格式，拒绝特殊字符和控制字符
- **JSON 参数**：Schema 验证，拒绝未知字段
- **数值参数**：范围检查，防止溢出
- **不可见字符**：过滤 Unicode 控制字符（可携带隐藏指令）

### 3.3 Prompt 注入防护

- **指令-数据分离**：通过结构化 JSON 输入而非自然语言拼接传递参数
- **用户内容边界标记**：输出中的用户生成内容用 `<!-- BEGIN_USER_CONTENT -->` / `<!-- END_USER_CONTENT -->` 隔离
- **`--sanitize` 模式**：对含有用户生成内容的输出自动标记边界

### 3.4 输出安全

- 结构化 JSON 输出（`--json`），避免 Agent 解析自然语言中的注入内容
- 错误信息不泄露内部路径、版本号、堆栈跟踪
- 大输出支持 `--fields` 字段掩码，防止上下文窗口溢出

---

## 4. 安全漏洞修复示例

### 示例 1：路径穿越（ASI02）

来源：Smithery MCP 平台漏洞（3,000+ 服务受影响）

```python
# ❌ 危险：直接使用用户输入的路径
import os

def read_file(user_path: str) -> str:
    # 攻击者传入 "../../etc/passwd" 即可读取系统文件
    with open(user_path) as f:
        return f.read()
```

```python
# ✅ 安全：规范化 + 边界检查
import os

ALLOWED_DIR = os.path.realpath("/data/workspace")

def read_file(user_path: str) -> str:
    # 1. 规范化：解析 .. 和符号链接
    real_path = os.path.realpath(user_path)

    # 2. 边界检查：必须在允许目录内
    if not real_path.startswith(ALLOWED_DIR + os.sep):
        raise PermissionError(
            f"路径 '{user_path}' 超出允许范围 '{ALLOWED_DIR}'"
        )

    # 3. 存在性检查
    if not os.path.exists(real_path):
        raise FileNotFoundError(f"文件不存在: {user_path}")

    with open(real_path) as f:
        return f.read()
```

### 示例 2：Shell 命令注入（ASI02）

来源：mcp-remote 漏洞（437,000+ 安装受影响）

```python
# ❌ 危险：字符串拼接构造 shell 命令
import os

def run_tool(tool_name: str, args: str) -> str:
    # 攻击者传入 tool_name = "legit; rm -rf /" 即可注入
    return os.popen(f"{tool_name} {args}").read()
```

```python
# ✅ 安全：参数化 API + 输入验证
import subprocess
import re

ALLOWED_TOOLS = {"grep", "wc", "sort", "head", "tail"}

def run_tool(tool_name: str, args: list[str]) -> str:
    # 1. 白名单校验
    if tool_name not in ALLOWED_TOOLS:
        raise ValueError(f"工具 '{tool_name}' 不在允许列表中")

    # 2. 参数格式校验（拒绝 shell 元字符）
    for arg in args:
        if re.search(r'[;&|`$(){}]', arg):
            raise ValueError(f"参数包含危险字符: '{arg}'")

    # 3. 参数化调用（列表形式，不经过 shell 解析）
    result = subprocess.run(
        [tool_name] + args,
        capture_output=True, text=True, timeout=30,
        # shell=False 是默认值，显式写出以强调安全意图
    )
    return result.stdout
```

### 示例 3：全权委托（ASI03）

来源：OpenCLI 安全模型分析——Agent 默认获得用户在浏览器中的全部权限

```python
# ❌ 危险：工具继承用户全部权限，无分级
def execute_action(action: str, user_token: str):
    # Agent 能做用户能做的一切——发消息、删数据、转账
    api.call(action, auth=user_token)
```

```python
# ✅ 安全：分级权限 + 操作确认
RISK_LEVELS = {
    "read": 0,    # L0: 自动允许
    "write": 1,   # L1: 首次确认
    "delete": 4,  # L4: 每次确认 + dry-run
    "send": 4,    # L4+: 网络不可撤回，强制预览
}

def execute_action(action: str, user_token: str, dry_run: bool = False):
    risk = RISK_LEVELS.get(action.split("_")[0], 3)

    # L4+ 操作强制 dry-run 预览
    if risk >= 4 and not dry_run:
        preview = api.preview(action, auth=user_token)
        return {"status": "preview", "effect": preview,
                "message": "此操作不可撤回，请确认后以 --confirm 执行"}

    # L3+ 需要限定范围的 token
    scoped_token = api.scope_token(
        user_token,
        permissions=[action],
        ttl_minutes=15  # JIT: 15 分钟后过期
    )

    return api.call(action, auth=scoped_token)
```

---

## 5. 安全检查清单

### 5.1 开发阶段

**输入安全**（防 ASI01/ASI02）：

- [ ] 所有外部输入（CLI 参数、文件内容、API 响应）经过验证和净化
- [ ] 路径参数：规范化 → 边界检查 → 拒绝穿越
- [ ] ID/名称参数：正则校验，拒绝特殊字符
- [ ] JSON 参数：Schema 验证，拒绝未知字段
- [ ] 不可见 Unicode 字符和双重编码检测
- [ ] 用户生成内容有边界标记

**权限设计**（防 ASI03）：

- [ ] 声明最小权限集（不多不少）
- [ ] 只读与写入操作明确分离
- [ ] 不可逆操作支持 `--dry-run` 预览
- [ ] 权限需求在 SKILL.md / `--help` 中透明

**代码安全**（防 ASI05）：

- [ ] 不使用字符串拼接构造 shell 命令——用 `subprocess.run(list)`
- [ ] 不硬编码凭证、API 密钥、密码
- [ ] 错误信息不泄露内部路径和堆栈跟踪
- [ ] 日志中不记录敏感数据

**输出安全**（防 ASI01/ASI06）：

- [ ] 结构化 JSON 输出（`--json`）
- [ ] 包含用户生成内容的输出字段有边界标记
- [ ] 大输出支持 `--fields` 字段掩码

### 5.2 发布前检查

- [ ] 所有依赖已审查，无已知漏洞
- [ ] 锁文件（package-lock.json / uv.lock）已提交
- [ ] CI 中运行安全扫描
- [ ] Skill 权限声明与实际使用一致
- [ ] 在沙箱环境中测试工具行为
- [ ] 验证无网络时报错而非静默失败
- [ ] 操作日志完整：请求参数、操作类型、结果、时间戳
- [ ] 高风险操作有审计追踪

---

## 6. 平台安全机制速查

| 维度 | Claude Code | OpenClaw | OpenCode |
|------|------------|----------|----------|
| **沙箱** | OS 级（Seatbelt/bubblewrap），默认开启 | 容器级，默认关闭 | 无专门沙箱 |
| **权限模型** | 四层纵深防御 | Exec 白名单 + Skill 权限声明 | 基于 MCP 工具确认 |
| **确认机制** | 分层提示 + 白名单 + 5 种模式 | Allow Once/Always/Deny | 简单工具调用确认 |
| **Hooks** | PreToolUse/PostToolUse | 无 | 无 |
| **组织策略** | 托管设置（不可覆盖） | 需手动配置 | 无 |
| **Skill 审查** | Plugin 审查机制 | ClawHub Skill Vetter | 无 |

**对工具开发者的启示**：

三层防御中，**工具层是唯一可控的防线**：

| 防御层 | 责任方 | 可控性 |
|--------|-------|--------|
| 工具层 | 工具开发者 | ✅ 完全可控——输入验证、权限声明、`--dry-run`、结构化输出 |
| 平台层 | Agent 平台 | ⚠️ 不可控——不同平台差异巨大 |
| 生态层 | 社区/标准 | ⚠️ 不可控——仍在演进中 |

**设计为可在沙箱内运行**：不假设对宿主系统的完全访问。文件操作限定在工作目录，网络请求声明所需域名。如果工具无法在沙箱内工作，在 SKILL.md 中清晰说明原因和所需权限。

---

## 附录：与设计原则的映射

| 安全概念 | 对应原则 | 关系说明 |
|----------|---------|---------|
| 分级授权模型 | C4 安全边界优先 | 先定义权限边界，在边界内自由操作 |
| 输入验证加固 | P2 确定性 | 验证过的输入才能产生可预测的输出 |
| `--dry-run` 预览 | C4 + P2 | 安全边界 + 可预测——先看效果再执行 |
| 能力声明 | C1 文本即界面 | 权限通过文本（SKILL.md/YAML）描述，Agent 可感知 |
| 结构化错误输出 | P3 输出即产品 | 安全的错误信息也是产品——不泄露敏感信息，格式可解析 |
| 沙箱优先 | C4 | "沙箱化减少 84% 权限提示"——边界比确认更有效 |
| 三层防御 | C2 原子化 | 各层独立防御，不依赖单一防线 |
