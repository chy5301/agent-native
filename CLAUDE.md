# Agent-Native 工作空间

## 这是什么

这是一个调研与思考工作空间，探索 **Agent-Native** 软件设计范式——当软件的主要用户从人类变成 Agent，软件的设计、构建和分发方式会发生什么根本性变化。

不是一个代码项目，是一个思想实验场。

## 最终产物

设计指南 Skill —— 将调研成果转化为可被 Agent 使用的设计指南。

**Plugin 打包**：如需将 Skill 打包为 Claude Code Plugin 进行分发，参见 `PLUGIN_BUILD_GUIDE.md`。

**已发布的 Plugin 副本**：[chy5301/cc-plugins](https://github.com/chy5301/cc-plugins) 仓库的 `agent-native-design-guide/` 目录。本项目是调研源头和开发工作区，Plugin 副本是面向分发的打包产物。更新 Skill 内容后需同步到 Plugin 副本。

## 核心命题

1. **GUI 是给人类的翻译层，Agent 不需要它。** CLI+Skill 是 Agent 时代的原生交互方式。
2. **产品形态从 App 变成 Skill。** 没有 UI，但它是产品；没有下载，但它有用户——只不过用户是 Agent。
3. **软件从资产变成耗材。** 生产成本趋近于零，按需生成，用完即弃。
4. **中间层消亡。** 软件是人机之间的中间层，管理层是意图与执行之间的中间层，AI 正在同时冲击所有中间层。

## 目录结构

- `docs/` — 方法论、设计原则
  - `docs/design-principles.md` — 设计原则文档
  - `docs/workflow/` — 结构化工作流管理
    - `workflow.json` — 工作流配置
    - `TASK_PLAN.md` — 任务规划与分解
    - `TASK_ANALYSIS.md` — 任务分析
    - `DEPENDENCY_MAP.md` — 依赖关系图
    - `TASK_STATUS.md` — 任务执行状态追踪
- `research/` — 调研笔记、案例分析、生态图谱
  - `research/articles/` — 外部文章的阅读笔记与提炼
  - `research/cases/` — 案例研究
  - `research/references/` — 参考资料整理
  - `research/independent/` — 独立视角与批判分析
  - `research/synthesis/` — 综合分析与设计产出
- `skill/` — 最终设计指南产物
  - `skill/agent-native-design-guide/` — Agent-Native 设计指南 Skill
    - `SKILL.md` — Skill 入口（决策框架 + 原则速查 + 导航索引）
    - `references/` — 参考文档（设计原则、架构模式）
    - `examples/` — 可运行代码示例（JSON 信封、Agent 友好 --help）
- `PLUGIN_BUILD_GUIDE.md` — Plugin 打包构建指南

## 工作方式

以调研和写作为主，不急于写代码。需要验证想法时再做原型。

## 工作流管理

本项目使用结构化工作流（structured-workflow）管理调研任务。
任务规划和状态详见 `docs/workflow/TASK_STATUS.md`。

常用命令：
- `/structured-workflow:task-exec` — 执行单个任务
- `/structured-workflow:phase-review` — 阶段完成检查
- `/structured-workflow:plan-adjust` — 调整计划
