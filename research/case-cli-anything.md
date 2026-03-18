# CLI-Anything 案例研究：GUI 软件的 Agent-Native 改造流水线

> 调研日期：2026-03-18
> 信息来源：GitHub HKUDS/CLI-Anything、HARNESS.md（763 行 SOP）、各软件 SKILL.md
> 性质：香港大学 HKUDS 团队的开源项目，目标是"Making ALL Software Agent-Native"

---

## 一、项目概况

**CLI-Anything** 是香港大学数据科学实验室（HKUDS）的开源项目，通过自动化流水线将已有桌面 GUI 软件转换为 Agent 可用的 CLI 接口。

- **GitHub**：https://github.com/HKUDS/CLI-Anything
- **规模**：18k+ stars
- **已改造软件**：11 个——Blender、GIMP、Inkscape、LibreOffice、Shotcut、Kdenlive、Audacity、OBS、Draw.io、ComfyUI、AnyGen
- **核心产物**：HARNESS.md（763 行的 7 阶段 SOP）+ 每个软件的 CLI 实现 + 自动生成的 SKILL.md
- **多平台适配**：Claude Code plugin、OpenClaw skill、OpenCode commands、Codex skill、Qoder plugin

**核心定位**：不是从零构建 Agent-Native 工具，而是将**已有 GUI 软件**批量改造为 Agent 可操作的 CLI 接口。HARNESS.md 本质上是一份"领域特化的设计指南 Skill"——Agent 阅读它就能完成 GUI→CLI 的改造任务。

---

## 二、技术架构

### 2.1 七阶段 SOP（HARNESS.md）

HARNESS.md 是整个项目的方法论核心，定义了从分析到发布的完整流水线：

| 阶段 | 名称 | 核心任务 |
|------|------|---------|
| Phase 1 | Codebase Analysis | 识别后端引擎、映射 GUI→API、识别数据模型、查找已有 CLI、编目命令/撤销系统 |
| Phase 2 | CLI Architecture Design | 选择交互模型、定义命令分组、设计状态模型、规划输出格式 |
| Phase 3 | Implementation | 从数据层开始实现，逐步添加探查/变更/渲染/会话命令 |
| Phase 4 | Test Planning | 先写 TEST.md 测试计划，再写测试代码 |
| Phase 5 | Test Implementation | 4 层测试：单元、E2E native、E2E true backend、CLI subprocess |
| Phase 6 | Test Documentation | 追加测试结果到 TEST.md |
| Phase 6.5 | SKILL.md Generation | 从 CLI 元数据自动生成 SKILL.md |
| Phase 7 | PyPI Publishing | PEP 420 namespace packages 发布 |

### 2.2 CLI 架构设计决策

Phase 2 中沉淀了一系列关键设计决策：

**交互模型**：推荐 Stateful REPL + Subcommand CLI 双模式。无参数时自动进入 REPL，有参数时作为标准 CLI 使用。技术实现为 Click 框架 + `invoke_without_command=True`。

**命令分组**：标准五组——Project（项目管理）、Core（核心领域操作）、Import-Export（格式转换）、Config（配置管理）、Session（会话/撤销）。

**输出格式**：人类可读 + JSON 双模，通过 `--json` flag 切换。这是 Agent-Native 的关键：Agent 消费 JSON，人类看格式化文本。

**状态模型**：会话文件持久化，支持 undo/redo。文件锁定使用 `fcntl.flock`。

### 2.3 标准目录结构

每个改造的软件遵循统一布局：

```
<software>/
└── agent-harness/
    ├── <SOFTWARE>.md          ← 软件特定分析（Phase 1 产物）
    ├── setup.py
    └── cli_anything/
        └── <software>/
            ├── <software>_cli.py  ← Click CLI 入口
            ├── core/              ← 领域模块
            ├── utils/             ← 后端包装 + REPL 皮肤
            │   └── <software>_backend.py  ← 包装真实软件
            ├── skills/SKILL.md    ← 自动生成的 Skill
            └── tests/             ← 测试 + TEST.md
```

关键设计：
- **PEP 420 namespace packages**：`cli_anything/` 目录无 `__init__.py`，允许多个软件的包共存
- **registry.json**：统一索引所有已改造软件
- **ReplSkin**：统一交互皮肤，确保所有软件的 REPL 体验一致
- **后端包装模式**：`utils/<software>_backend.py` 包装真实软件的 CLI（如 `libreoffice --headless`、`blender --background --python`）

### 2.4 四层测试策略

Phase 5 定义的测试层次值得注意：

| 层次 | 说明 | 是否需要真实软件 |
|------|------|----------------|
| 单元测试 | 核心逻辑、数据模型 | 否 |
| E2E native | 端到端，使用 mock | 否 |
| E2E true backend | 端到端，调用真实软件 | **是** |
| CLI subprocess | 通过子进程调用完整 CLI | **是** |

"E2E true backend"是核心——必须验证 CLI 确实正确调用了真实软件，而非仅测试 Python 包装层。

---

## 三、实战教训（Critical Lessons Learned）

HARNESS.md 中沉淀了一系列 Gotchas，这些是 11 个软件改造过程中的血泪经验，极具价值。

### 3.1 "Use the Real Software — Don't Reimplement It"

**第一规则**。CLI 必须调用真实软件进行渲染/导出，而非用 Python 重新实现。

反模式示例：用 Pillow 替代 GIMP 来处理图像。看似能工作，但会丢失 GIMP 的滤镜效果、色彩管理、格式兼容性。

正确做法：CLI 始终是真实软件的**包装层**，不是**替代品**。

### 3.2 "The Rendering Gap"

**第二大陷阱**。GUI 应用在渲染时会应用效果（滤镜、转场、色彩校正等），但 CLI 操作项目文件时必须显式处理渲染——朴素方法会静默丢失效果。

解决方案按优先级：
1. 优先使用原生渲染器（如 `blender --render`）
2. 构建滤镜翻译层（将项目文件中的效果描述翻译为渲染命令）
3. 最后手段：生成渲染脚本让真实软件执行

### 3.3 Filter Translation Pitfalls

格式间翻译效果时的具体陷阱：
- **重复滤镜合并**：同类型滤镜多次出现时需要合并而非叠加
- **排序约束**：ffmpeg concat 要求交错流顺序（video0, audio0, video1, audio1...），乱序会失败
- **参数空间差异**：同一效果在不同格式中的参数范围不同。例如 MLT `brightness 1.15` 对应 ffmpeg `eq=brightness=0.06`，不是简单的数值映射

### 3.4 Timecode Precision

非整数帧率（如 29.97fps）导致累积舍入误差：
- 用 `round()` 不用 `int()`
- 用整数运算显示时间码
- 接受 ±1 帧容差

### 3.5 "No Graceful Degradation"

真实软件是硬依赖。缺少时必须**明确报错**而非静默降级。

如果 Blender 未安装，CLI 不应该尝试用简化方法完成任务——应该直接告诉用户"Blender not found, please install it"。静默降级会产生错误结果，比报错更糟糕。

### 3.6 会话文件锁定

一个具体的并发 bug：`open("w")` 会在锁获取前截断文件。

```python
# 错误：open("w") 在获取锁之前就已经截断文件内容
with open(session_file, "w") as f:
    fcntl.flock(f, fcntl.LOCK_EX)  # 太晚了，文件已被截断
    f.write(data)

# 正确：先 open("r+")，获取锁后再 truncate
with open(session_file, "r+") as f:
    fcntl.flock(f, fcntl.LOCK_EX)
    f.truncate()
    f.write(data)
```

---

## 四、SKILL.md 生态

### 4.1 自动生成流程

`skill_generator.py` 从结构化的 CLI 元数据自动生成 SKILL.md：

- **输入源**：setup.py（包元数据）、Click decorators（命令签名）、README（描述文本）
- **输出**：标准化的 SKILL.md 文件

### 4.2 SKILL.md 标准结构

```yaml
---
name: cli-anything-blender
description: Agent-native CLI for Blender 3D
---
```

之后依次包含：
1. **安装说明**（pip install）
2. **命令表**（分组列出所有命令及参数）
3. **使用示例**（覆盖典型场景）
4. **Agent 使用指南**（5 条规则，指导 Agent 正确使用）

### 4.3 以 Blender 为例

Blender 的 SKILL.md 包含 9 个命令组：Scene、Object、Material、Modifier、Camera、Light、Animation、Render、Session。

双输出模式说明：
- 默认输出人类可读格式
- `--json` 输出机器可读 JSON

Agent 特定指南（5 条规则）指导 Agent：
- 优先使用 `--json` 获取结构化数据
- 用 `info` 命令探查状态后再操作
- 渲染必须通过 CLI 的 render 命令（调用真实 Blender），不要尝试其他方式
- 利用 session undo/redo 进行安全探索
- 错误时检查 Blender 是否已安装

### 4.4 多平台分发

同一 CLI 实现通过不同包装分发到多个 Agent 平台：

| 平台 | 格式 |
|------|------|
| Claude Code | plugin |
| OpenClaw | skill |
| OpenCode | commands |
| Codex | skill |
| Qoder | plugin |

核心能力（CLI 实现）只有一份，SKILL.md 的生成器根据目标平台调整格式。

---

## 五、与本项目的关系

### 5.1 形态对比

| 维度 | CLI-Anything | 我们的设计指南 Skill |
|------|-------------|-------------------|
| 核心产物 | HARNESS.md（SOP）+ 代码实现 | SKILL.md + references/ 知识库 |
| 受众 | Agent（执行 GUI→CLI 改造） | Agent（参考设计原则构建新工具） |
| 关注点 | 改造已有 GUI 软件 | 从零设计 Agent-Native 工具 |
| 覆盖范围 | CLI 架构 + 状态管理 + 测试 + 打包 | 设计原则 + 协议选择 + 双模架构 + 安全 |
| 前提假设 | 存在一个已有的 GUI 软件（有后端引擎、原生格式、已有 CLI） | 无前提，适用于全新工具 |

### 5.2 互补关系

两个项目覆盖了 Agent-Native 转型的两个方向：

- **CLI-Anything**：解决"已有 GUI 软件如何变成 Agent 可用的"——存量改造
- **我们的设计指南**：解决"从零设计工具时如何天然 Agent-Native"——增量设计

CLI-Anything 默认存在一个已有的 GUI 软件需要改造（有后端引擎、原生格式、已有 CLI），我们的指南覆盖更广（协议选择、双模架构、安全模型、经济学视角），且不依赖已有软件的存在。

### 5.3 可借鉴点

**1. HARNESS.md 是"设计指南 Skill"的范本。** 763 行的方法论文档直接指导 Agent 完成复杂的 GUI→CLI 改造任务。它的结构——分阶段 SOP + 标准布局 + Gotchas 沉淀——与我们计划中的 SKILL.md 形态高度一致。

**2. Gotchas 沉淀印证"Gotchas 是灵魂"原则。** Anthropic 在 Skill 打造经验中强调"Gotchas 是 Skill 最有价值的部分"，CLI-Anything 的实战教训完美印证了这一点。6 条 Gotchas 每一条都来自真实踩坑，无法通过理论推导获得。

**3. 多平台分发策略。** 同一核心内容适配 Claude Code plugin、OpenClaw skill、OpenCode commands 等多个 Agent 平台，展示了"一次构建，多平台分发"的可行路径。

**4. 自动化生成。** `skill_generator.py` 展示了从结构化代码（Click decorators、setup.py）自动生成 SKILL.md 的可能性。如果我们的设计指南产出的工具遵循一致的结构约定，也可以实现类似的自动化。

**5. "真实软件是硬依赖"原则的泛化。** 这个原则可以抽象为更通用的设计原则："Agent 工具应包装而非替代"——不要用简化实现替代专业工具，不要静默降级，明确声明依赖。

### 5.4 对我们任务计划的直接输入

| 任务 | CLI-Anything 提供的输入 |
|------|----------------------|
| G-05（CLI 接口设计） | 7-Phase SOP 的 CLI 架构设计模式：双模交互（REPL + Subcommand）、五组命令分类、状态模型、`--json` 输出格式 |
| G-08（接口规范参考） | `--json` flag 标准、命令表结构、Agent 使用指南格式（5 条规则） |
| G-09（架构模式） | 后端包装模式、PEP 420 namespace packages、SKILL.md 自动生成流程 |
| G-12（SKILL.md 入口） | HARNESS.md 的结构作为"长篇设计指南 Skill"的参考范本 |

---

## 参考来源

- [HKUDS/CLI-Anything GitHub 仓库](https://github.com/HKUDS/CLI-Anything)
- HARNESS.md（763 行，项目核心 SOP）
- 各软件的 SKILL.md（以 Blender 为代表）
- skill_generator.py（SKILL.md 自动生成器）
