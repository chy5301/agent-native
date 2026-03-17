# Agent-Native 生态图谱

> 追踪正在实践 Agent-Native 理念的项目、协议和工具。

## Agent 运行时

| 项目 | 类型 | 交互方式 | 备注 |
|------|------|---------|------|
| Claude Code | CLI Agent | CLI | Anthropic 官方，Skill 生态 |
| Codex | CLI Agent | CLI | OpenAI 官方 |
| OpenClaw | CLI Agent | CLI | 开源，让大众认知了 Agent |
| Aider | CLI Agent | CLI | 专注代码编辑 |
| OpenCode | CLI Agent | CLI | Claude Code 的开源替代 |

## 能力封装协议

| 协议/规范 | 思路 | 优劣 |
|-----------|------|------|
| MCP (Model Context Protocol) | 工具描述塞进 context | 灵活，但 token 开销大 |
| CLI + Skill | Agent 读 Skill 文档，调 CLI 命令 | 轻量，可组合，确定性强 |
| A2A (Agent-to-Agent) | Agent 间直接通信 | Google 提出，生态待建设 |
| OpenAPI / REST | 传统 API 描述 | 成熟但为人类开发者设计 |

## GUI → CLI 桥接

| 项目 | 做什么 |
|------|--------|
| CLI-Anything (HKUDS) | 为任意桌面软件自动生成 CLI 接口 |
| 各云厂商 CLI | aliyun / gcloud / aws cli — 天然的 Agent 接口 |

## 待调研

- [ ] Claude Code Skill 生态的设计模式梳理
- [ ] CLI-Anything 的实现原理与局限性
- [ ] MCP vs CLI+Skill 的实际对比测试
- [ ] Agent 权限模型的现有方案调研
- [ ] 面向 Agent 的"用户体验"评估框架
