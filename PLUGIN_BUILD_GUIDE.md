# Plugin 构建指南

本文档为 Agent 提供将 `skill/agent-native-design-guide/` 打包为 Claude Code Plugin 的完整信息。

## 目标

将现有的 Agent-Native 设计指南 Skill 包装为可通过 Plugin Market 分发的 Claude Code Plugin。

## 源文件清单

所有需要打包的文件位于 `skill/agent-native-design-guide/`：

```
skill/agent-native-design-guide/
├── SKILL.md                          # Skill 入口（已完成，直接使用）
├── references/
│   ├── design-principles.md          # 十原则体系
│   └── architecture-patterns.md      # 三层架构 + 协议选择
└── examples/
    ├── cli-json-output.py            # JSON 信封代码示例
    └── cli-help-design.py            # Agent 友好 --help 代码示例
```

## 目标 Plugin 结构

```
agent-native-design-guide/
├── .claude-plugin/
│   └── plugin.json                   # Plugin 清单（需新建）
├── skills/
│   └── agent-native-design-guide/    # 直接复制 skill/ 下的内容
│       ├── SKILL.md
│       ├── references/
│       │   ├── design-principles.md
│       │   └── architecture-patterns.md
│       └── examples/
│           ├── cli-json-output.py
│           └── cli-help-design.py
└── README.md                         # Plugin 说明（需新建）
```

## plugin.json 内容

```json
{
  "name": "agent-native-design-guide",
  "version": "1.0.0",
  "description": "Agent-Native 软件设计原则与架构模式指南，帮助设计面向 Agent 的工具、CLI 和 Skill",
  "author": {
    "name": "chy5301"
  },
  "repository": "https://github.com/chy5301/agent-native",
  "license": "MIT",
  "keywords": [
    "agent-native",
    "design-guide",
    "cli-design",
    "skill-architecture",
    "agent-first"
  ]
}
```

## Skill 描述（已写入 SKILL.md frontmatter）

```yaml
name: agent-native-design-guide
description: >
  This skill should be used when the user is designing software, tools, CLIs, or
  Skills meant to be operated by AI agents rather than humans. It provides a
  decision framework and design principles for Agent-Native software architecture.
  Trigger on 'design for agents', 'agent-native', 'agent-friendly CLI',
  'make this tool agent-compatible', 'CLI for agents', 'Skill architecture',
  'agent-first design', 'how should agents use this tool',
  'convert to agent-native', 'tool design principles'.
```

## 构建步骤

1. 在目标位置创建 Plugin 目录结构
2. 创建 `.claude-plugin/plugin.json`（内容见上方）
3. 将 `skill/agent-native-design-guide/` 整体复制到 `skills/agent-native-design-guide/`
4. 创建 README.md（从本项目 README.md 的"产出物"部分提取）
5. 验证：确认 `.claude-plugin/plugin.json` 存在且 JSON 合法，`skills/agent-native-design-guide/SKILL.md` 存在

## 注意事项

- Plugin 只包含 `skills/` 组件，不包含 commands、agents、hooks 或 MCP servers
- SKILL.md 已就绪，无需修改，直接使用
- 两个 Python 示例文件是可运行的代码（`uv run python <file>`），保留在 `examples/` 中
- 不要将 `research/` 或 `docs/` 目录打包进 Plugin，那些是调研过程文件
