# SKILL.md 模板与编写指南

> 本文件提供可直接复制使用的 SKILL.md 模板和编写指导。
> 关联原则：P1 可发现性、P4 Gotchas 驱动迭代、P5 渐进式信息披露、P6 约束意图而非步骤（见 design-principles.md）

---

## Skill 类型速查

选择最接近你场景的类型，用对应模板变体。好的 Skill 干净地落在一个类别里，跨多个类别的 Skill 通常需要拆分。

| 类型 | 场景 | 模板重点 | 典型案例 |
|------|------|---------|---------|
| 库/API 参考 | 帮助正确使用某个库、CLI、SDK | Gotchas 和边界情况 | billing-lib 踩坑清单 |
| 产品验证 | 描述如何测试和验证代码 | 测试工作流 + 环境配置 | Playwright 录制验证 |
| 数据查询 | 连接数据源和监控系统 | 数据源配置 + 查询模式 | funnel-query |
| 流程自动化 | 重复性工作流浓缩为一条命令 | 触发条件 + 输出格式 | standup-post |
| 代码脚手架 | 为特定功能生成框架代码 | 模板结构 + 约定 | 组件生成器 |
| 代码质量 | 强制执行代码规范 | 规则清单 + 检查流程 | adversarial-review |
| CI/CD 部署 | 拉代码、推代码、部署代码 | 流程步骤 + 回滚方案 | babysit-pr |
| 事故排查 | 给定症状引导调查流程 | 症状→调查→报告决策树 | 告警诊断手册 |
| 基础设施运维 | 日常维护和运维操作 | 安全护栏 + 确认机制 | 孤儿 Pod 清理 |

---

## 通用模板

以下是可直接复制使用的 SKILL.md 模板。`<!-- 注释 -->` 部分是编写指导，使用时删除。

````markdown
---
name: my-tool
description: |
  <!-- description 是给模型看的，重点是"什么时候该触发"。
       差："A comprehensive tool for monitoring pull request status."
       好："Monitors a PR until it merges. Trigger on 'babysit', 'watch CI', 'make sure this lands'." -->
  简要说明 + 触发词列表。Trigger on '关键词1', '关键词2', '关键词3'.
---

# my-tool

<!-- 一句话说明这个工具做什么、为谁做。 -->

## 前置条件

<!-- 安装命令、环境依赖、首次配置检测 -->

```bash
pip install my-tool
# 或
uv tool install my-tool
```

<!-- 需要用户配置时，检测 config.json 是否存在：
     如果不存在，主动问用户获取配置信息。 -->

## 快速开始

<!-- 2-3 个最常用的工作流示例，展示典型用法。
     用自然语言描述意图，再给出对应命令。 -->

**场景 1：<!-- 场景名称 -->**
```bash
my-tool resource action --json
```

**场景 2：<!-- 场景名称 -->**
```bash
my-tool resource action --output report.html --report
```

## 命令速查

<!-- 按标准五组分类（适用时）：Project / Core / Import-Export / Config / Session -->

| 命令 | 说明 | 常用选项 |
|------|------|---------|
| `my-tool project create` | 创建项目 | `--name`, `--template` |
| `my-tool core process` | 核心处理 | `--input`, `--json` |
| `my-tool export csv` | 导出 CSV | `--fields`, `--output` |

<!-- 全局选项：--json, --quiet, --fields, --dry-run -->

## Agent 使用规则

<!-- 3-5 条约束意图的规则。说清楚"做什么"和"不做什么"，
     把"怎么做"交给 Agent（原则 P6）。

     过度约束（反面）：
     "Step 1: 运行 git log。Step 2: 解析输出。Step 3: ..."

     恰当约束（正面）：
     "获取最近的变更记录，识别影响范围。如果范围超过 3 个模块，先汇报再操作。" -->

1. 所有输出默认使用 `--json`，需要人类查看时加 `--report`
2. 修改操作前先 `--dry-run` 确认影响范围
3. <!-- 领域特定规则 -->

## Gotchas

<!-- 这是 Skill 的灵魂——信息密度最高的部分。
     初始为空是正常的。每次 Agent 踩坑，追加一条。

     演化示例：
     - Day 1：空
     - Week 2：+1 条（"参数 X 在 Y 场景下行为不同"）
     - Month 3：+4 条（积累实战经验） -->

- <!-- 踩坑记录 1：症状 → 原因 → 正确做法 -->

## 详细文档

<!-- 指向 references/ 子目录中的详细文档（如有）。
     简单 Skill 可省略此节。 -->

- [API 参考](references/api.md)
- [配置说明](references/config.md)
````

---

## description 写作指南

description 是模型决定是否触发 Skill 的关键字段。写给模型看，不是给人看。

### 好/坏对比

| 质量 | description | 问题 |
|------|------------|------|
| **差** | "A comprehensive tool for managing database migrations and schema changes across environments." | 描述功能而非触发条件，模型不知道何时该用 |
| **好** | "Run and manage DB migrations. Trigger on 'migrate', 'schema change', 'add column', 'rollback migration'." | 明确触发词，模型看到用户输入就知道该触发 |
| **差** | "Helps with code review processes." | 太模糊，什么时候触发？ |
| **好** | "Adversarial code review — spawns a sub-agent to find bugs. Trigger on 'review', 'find bugs', 'check quality', 'nitpick'." | 说明做法 + 列出触发词 |

### 写作要点

1. **第一句话说"做什么"**，后面列"什么时候触发"
2. **列出触发词**——用户可能说的关键词和短语
3. **不说废话**——Agent 已知编程常识，只写能把 Agent 推出默认思维模式的信息
4. **长度控制**——1-3 句话，不超过 100 词

---

## 文件夹结构指南

把文件系统当作 context engineering 工具。Agent 按需读取，不一次性加载全部上下文。

### 简单 Skill（单文件）

```
my-tool/
└── SKILL.md          # 全部内容在一个文件中
```

适用于：简单 CLI 工具、单一功能的 Skill。SKILL.md 控制在 200 行以内。

### 中等 Skill（入口 + 参考文档）

```
my-tool/
├── SKILL.md          # ~30-50 行"中枢"，根据场景指向子文件
├── references/
│   ├── api.md        # 详细函数签名和参数说明
│   ├── gotchas.md    # 独立的踩坑记录（内容多时从 SKILL.md 分出）
│   └── config.md     # 配置选项说明
└── examples/
    └── workflow.sh   # 可运行的示例脚本
```

适用于：数据查询、流程自动化、API 包装类 Skill。SKILL.md 作为入口做分发，Agent 按需读取子文件。

### 复杂 Skill（完整结构）

```
my-tool/
├── SKILL.md          # 入口 + 高层概述
├── references/
│   ├── api.md
│   ├── architecture.md
│   ├── security.md
│   └── gotchas.md
├── examples/
│   ├── basic.py
│   └── advanced.py
├── scripts/          # 预定义脚本库（Agent 组合使用）
│   ├── setup.sh
│   └── validate.sh
└── config.json       # 用户配置（首次运行时创建）
```

适用于：复杂平台包装、企业级工具。scripts/ 提供预定义函数库——给代码而非指令，让 Agent 直接组合使用而不必重新理解数据结构。

---

## 多平台分发

同一个 CLI 核心实现可通过不同包装分发到多个 Agent 平台：

| 平台 | 包装形式 | Skill 入口 |
|------|---------|-----------|
| Claude Code | Plugin（.claude/plugins/） | SKILL.md |
| OpenClaw | Skill（ClawHub） | SKILL.md |
| OpenCode | Commands | `--help --json`（无 SKILL.md 支持） |

**核心原则**：一次构建 CLI，多平台包装分发。CLI 是能力的唯一实现，SKILL.md / Plugin 配置只是不同平台的"安装说明"。

---

## 写作检查清单

编写完 SKILL.md 后逐项确认：

- [ ] **description** 包含触发词，写给模型看而非人看
- [ ] **前置条件** 包含安装命令，首次配置有检测逻辑
- [ ] **快速开始** 至少 2 个场景示例，覆盖最常用工作流
- [ ] **命令速查** 按标准分组，列出常用选项
- [ ] **Agent 规则** 约束意图而非步骤，3-5 条
- [ ] **Gotchas** 节存在（即使初始为空），标注"持续追加"
- [ ] **输出格式** 所有命令支持 `--json`
- [ ] **文件长度** 简单 Skill <200 行，复杂 Skill 的 SKILL.md 入口 <50 行
- [ ] **类型单一** Skill 干净地落在一个类型类别中
- [ ] **自包含** 不依赖外部链接或隐含知识即可理解和使用
