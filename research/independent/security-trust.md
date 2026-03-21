# Agent-Native 安全与信任模型调研

> 调研时间：2026-03-18 | 独立调研，非基于用户提供材料
> 核心问题：当 Agent 成为软件的主要用户，安全模型需要怎样的根本性变化

---

## 一、Agent 安全威胁模型

### 1.1 OWASP Agentic Top 10（2026）

OWASP 于 2025 年 12 月发布，100+ 行业专家协作完成：

| 编号 | 风险名称 | 核心问题 |
|------|----------|----------|
| ASI01 | Agent Goal Hijack | 通过恶意文本篡改 Agent 目标 |
| ASI02 | Tool Misuse & Exploitation | 合法工具被不安全使用 |
| ASI03 | Identity & Privilege Abuse | "困惑代理人"问题 |
| ASI04 | Supply Chain Vulnerabilities | 动态获取的组件被攻破 |
| ASI05 | Unexpected Code Execution | Agent 生成或运行不安全代码 |
| ASI06 | Memory & Context Poisoning | 污染记忆系统影响长期决策 |
| ASI07 | Insecure Inter-Agent Communication | 多 Agent 消息缺乏认证加密 |
| ASI08 | Cascading Failures | 错误在 Agent 链中级联传播 |
| ASI09 | Human-Agent Trust Exploitation | 用户过度信任 Agent 建议 |
| ASI10 | Rogue Agents | 被攻破的 Agent 暗中执行有害行为 |

**关键洞察**：ASI01 和 ASI06 揭示本质矛盾——Agent 需要读取外部数据来完成任务，但外部数据本身可能包含恶意指令。这是 Agent 架构的"原罪"。

### 1.2 核心攻击向量

**Prompt Injection——Agent 时代的 SQL Injection**：
- **35% 的真实世界 AI 安全事件仅由简单提示触发**
- 部分案例造成超 10 万美元的实际损失，无需编写一行代码
- 跨 Agent 注入：两个 Agent 通过互相重写配置文件来提权

**Memory Poisoning**：
- Galileo AI 发现，**单个被攻破的 Agent 在 4 小时内可污染 87% 的下游决策**

### 1.3 MCP 安全事件时间线

| 时间 | 事件 | 影响 |
|------|------|------|
| 2025.04 | WhatsApp MCP 工具投毒 | 整个 WhatsApp 历史被静默窃取 |
| 2025.05 | GitHub MCP Prompt 注入 | 私有仓库内容泄露 |
| 2025.07 | mcp-remote 命令注入 | 影响 437,000+ 安装 |
| 2025.08 | Filesystem MCP 沙箱逃逸 | 宿主文件系统和凭证被访问 |
| 2025.09 | 恶意 Postmark MCP 包 | 供应链攻击，用户邮件被拦截 |
| 2025.10 | Smithery 路径穿越 | 3000+ 服务受影响 |

另外：**492 个 MCP Server 暴露在互联网上且零认证**；约 **36.7% 存在 SSRF 漏洞**。

### 1.4 生产环境安全数据

- **88%** 的组织报告已确认或疑似 Agent 安全事件
- 仅 **14.4%** 获得完整安全审批
- 仅 **21%** 管理者对 Agent 权限有完整可见性
- Shadow AI 泄露平均比传统事件多花费 **$670,000**

---

## 二、信任与授权模型

### 2.1 传统 IAM 为什么失效

| 问题 | Agent 场景影响 |
|------|---------------|
| 粗粒度权限 | Agent 获得远超任务需要的权限 |
| 缺乏运行时上下文 | 无法检测异常行为 |
| 单体身份假设 | 委托链中责任无法归属 |
| 扩展困难 | 数千短生命周期 Agent 难以管理 |

### 2.2 新兴 Agent 身份框架

**零信任四层架构（arXiv 2505.19301）：**

```
第四层：统一全局会话管理
第三层：动态访问控制（ABAC/PBAC + JIT Access）
第二层：发现与信任建立（DID 解析器 + ANS + 声誉系统）
第一层：身份与凭证管理（DID + VC + 密钥生命周期）
```

核心创新：
1. **Agent Naming Service (ANS)**——"Agent 的 DNS"
2. **即时访问 (JIT)**——15 分钟有效期的可验证凭证
3. **多维授权**——身份 x 资源属性 x 行为类型 x 上下文
4. **可追踪委托链**——整条链通过 DID 可审计

**行业标准进展**：
- AAuth：专为 Agent 场景设计的 OAuth 2.1 扩展（IETF Draft）
- ARIA：Agent 关系型身份授权
- ANS：Agent 命名服务（IETF 提案）
- OpenID Foundation 和 CSA 也在推进方案

### 2.3 全权委托模型的现实案例

OpenCLI（2026-03，3,300+ stars）展示了零信任的反面——**全权委托模型**：Agent 通过浏览器 Daemon 获得用户在 Chrome 中的全部权限（发消息、提交表单、社交操作），无分级授权、无操作确认、Daemon 监听 localhost:19825 且无身份验证。

这进一步证明了零信任架构的必要性：即使是快速增长的开源项目，如果不在架构层面设计权限边界，Agent 就会默认获得过大的权限。"凭证不离开浏览器"的安全承诺不等于真正的安全——关键在于 Agent 能用这些凭证做什么。

---

## 三、可观测性与合规

### 3.1 EU AI Act 合规要求（2026.08.02 生效）

- 日志必须捕获：用户 ID、请求参数、模型细节、输出、置信度、人类监督动作
- 强制性决策日志保留期至少 6 个月
- 处罚：高达 **3500 万欧元或全球年收入 7%**
- 高风险系统必须第三方一致性评估

---

## 四、对 Agent-Native 设计的启示

### 4.1 安全必须是原生的

MCP 生态的教训：先追求功能再补安全行不通。
- **Skill 签名与验证**——密码学签名，类似 code signing
- **能力声明**——Skill 显式声明所需权限，运行时强制执行
- **沙箱执行**——默认沙箱运行

### 4.2 Agent 身份是新的基础设施

- 每个 Agent 应有自己的身份（DID 或类似机制）
- 权限应是 JIT 的、范围受限的、短生命周期的
- 委托链必须完整可追踪

### 4.3 Skill 市场需要信任机制

OpenClaw 事件（1,184 个恶意技能）表明 Skill 生态需要：
- 发布者身份验证
- 权限审查（类似 Android 权限声明）
- 行为监控
- 声誉系统

### 4.4 "软件即耗材"的安全悖论

- **Skill 是耗材，但发布者身份不是**——信任锚定在发布者
- **Agent 会话是短暂的，但 Agent 身份是持久的**——权限跟随身份
- **工具是动态加载的，但能力声明是静态可验证的**——发现时验证，执行时强制

---

## 参考来源

- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
- [AuthZed MCP Breaches Timeline](https://authzed.com/blog/timeline-mcp-breaches)
- [Pillar Security: MCP Security Risks](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp)
- [arXiv: Zero Trust Agent Framework (2505.19301)](https://arxiv.org/abs/2505.19301)
- [IETF AAuth Draft](https://www.ietf.org/archive/id/draft-rosenberg-oauth-aauth-00.html)
- [Gravitee: State of AI Agent Security 2026](https://www.gravitee.io/blog/state-of-ai-agent-security-2026-report)
- [EU AI Act Compliance](https://abv.dev/blog/eu-ai-act-compliance-checklist-2025-2027)
- [Adversa AI 2025 Report](https://adversa.ai/blog/adversa-ai-unveils-explosive-2025-ai-security-incidents-report/)
