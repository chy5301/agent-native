# Agent-Native

> 软件正在脱掉穿给人类的外衣。下一代软件，为 Agent 而生。

## 这是什么

这是一个研究与实践工作空间，探索 **Agent-Native** 软件设计范式——当软件的主要用户从人类变成 Agent 时，设计、构建和分发软件的方式将发生根本性的转变。

## 核心命题

### 范式转移

- **GUI → CLI**：图形界面是给人类的翻译层，Agent 不需要这层翻译。CLI 是 Agent 的母语。
- **App → Skill**：产品形态从独立应用变成可被 Agent 调用的能力原子。
- **资产 → 耗材**：软件从需要长期维护的重资产，变成按需生成、用完即弃的轻量耗材。
- **面向人 → 面向 Agent**：用户体验的优化对象从人类操作路径变成 Agent 决策路径。

### 关键问题

1. **可调用性**：软件如何被 Agent 理解和接入？
2. **可组合性**：能力如何像乐高一样随取随用？
3. **可靠性**：Agent 反复调用时，如何保证稳定正确？
4. **信任度**：Agent 为什么优先调用你而不是别人？
5. **权限边界**：Agent 能干 ≠ Agent 该干，谁来兜底？

## 目录结构

```
agent-native/
├── docs/            # 方法论、设计原则、架构思考
├── research/        # 调研笔记、案例分析、行业观察
└── examples/        # 实践案例、原型验证
```

## 研究范围

- **CLI 接口设计**：面向 Agent 的命令行接口设计原则（参考 CLI-Anything 等项目）
- **Skill 封装规范**：如何将业务能力封装成 Agent 可调用的 Skill
- **Agent 调用协议**：MCP、A2A、CLI+Skill 等不同方案的对比与取舍
- **中间层消亡**：GUI、管理层等传统中间层在 Agent 时代的演变
- **信任与权限模型**：Agent 自主操作时的安全边界设计

## 灵感来源

- Marc Andreessen,《Why Software Is Eating the World》(2011)
- "Software was eaten by AI." — 2026 年的时代注脚
- CLI-Anything (HKUDS) — 为任意软件自动生成 CLI 接口
- Claude Code / OpenClaw / Codex — Agent-Native 工具的早期实践者
