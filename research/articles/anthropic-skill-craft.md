# Anthropic 内部 Skill 打造经验：Thariq 长文提炼

> 来源：Anthropic 工程师 Thariq，2026-03-18
> 原文：https://x.com/trq212/article/2033949937936085378
> 性质：Anthropic 内部数百个 Skill 的实践总结，一周半打磨

---

## 一、核心认知：Skill 不只是 markdown

Skill 是一个**文件夹**，可以包含脚本、资源文件、数据、配置等。Agent 可以自己去发现、探索和操作这些文件。在 Claude Code 中，Skill 还支持注册动态 hooks。

Anthropic 内部发现，最值得研究的 Skill 恰恰是那些把文件夹结构和配置选项用到极致的。

---

## 二、九大类型分类

Anthropic 内部梳理所有 Skill 后发现它们自然聚成九个类别。**好用的 Skill 干净地落在某一个类别里，让人困惑的横跨好几个。**

| 类型 | 功能 | 典型案例 |
|------|------|---------|
| **库和 API 参考** | 帮助正确使用某个库/CLI/SDK | billing-lib：边界情况和踩坑清单 |
| **产品验证** | 描述如何测试和验证代码 | 搭配 Playwright/tmux，录制输出视频 |
| **数据查询与分析** | 连接数据源和监控系统 | funnel-query：注册→激活→付费的 join 模式 |
| **业务流程自动化** | 重复性工作流浓缩为一条命令 | standup-post：聚合票务/GitHub/Slack 生成站会汇报 |
| **代码脚手架** | 为特定功能生成框架代码 | 涉及自然语言需求描述时格外有用 |
| **代码质量与审查** | 在组织内强制执行代码规范 | adversarial-review：启动子 Agent 挑刺，反复迭代 |
| **CI/CD 与部署** | 拉代码、推代码、部署代码 | babysit-pr：监控 PR，CI 挂了重试，有冲突就解决 |
| **事故排查手册** | 给定症状引导走完调查流程 | 输入 Slack 告警 → 多工具调查 → 结构化报告 |
| **基础设施运维** | 日常维护和运维操作 | 找孤儿 Pod → Slack 通知 → 冷静期 → 确认 → 清理 |

**对设计指南的启示**：这个分类框架可以作为 Skill 设计的参考模型——帮助判断一个 Skill 该做什么、不该做什么。

---

## 三、九条写作技巧

### 3.1 别说废话

Claude 对代码库和编程已有大量默认认知。知识类 Skill 应把精力放在**能把 Claude 推出默认思维模式的信息**上。

例：frontend-design Skill 专门纠正 Claude 在设计上的"老毛病"（如默认用 Inter 字体和紫色渐变）。

### 3.2 Gotchas 是灵魂

**信息密度最高的部分是踩坑记录。** 应从 Claude 使用 Skill 时反复踩到的坑中积累，持续更新。

演化过程示例（billing-lib）：
- **Day 1**：简单说明文档
- **Week 2**：+1 条 Gotcha（"Proration rounds DOWN, not to nearest cent"）
- **Month 3**：+4 条 Gotcha（test-mode 跳过 webhook、幂等键 24 小时过期等）

> 每次 Claude 踩一次坑，就加一行。这才是 Skill 真正「活」起来的方式。

### 3.3 用文件夹做信息分层

把文件系统当作 context engineering 和渐进式信息披露的工具。

示例（queue-debugging Skill）：
```
queue-debugging/
├── SKILL.md          # ~30 行"中枢"，根据症状指向子文件
├── stuck-jobs.md
├── dead-letters.md
├── retry-storms.md
└── references/
    └── api.md        # 详细函数签名
```

Claude 按需去读，不会一次性加载全部上下文。

### 3.4 别把 Claude 管太死

Skill 是高度复用的，写指令要留足灵活度。

**过度约束**：把 Step 1 到 Step 6 都写死（git log → cherry-pick → git status → 逐个解决冲突……）

**恰当约束**："Cherry-pick the commit onto a clean branch. Resolve conflicts preserving intent. If it can't land cleanly, explain why."

> 把意图说清楚，把方法交给 Claude。

### 3.5 首次配置

需要用户上下文的 Skill，用 `config.json` 存储配置。如果配置不存在，Agent 主动问用户。

```
# SKILL.md 中的检测逻辑
if [ ! -f config.json ]; then
  echo "NOT_CONFIGURED"
  # 引导用户设置
fi
```

### 3.6 description 是给模型看的

description 的重点是**什么时候该触发**，而不是描述它做什么。

- 差："A comprehensive tool for monitoring pull request status across the development lifecycle."
- 好："Monitors a PR until it merges. Trigger on 'babysit', 'watch CI', 'make sure this lands'."

### 3.7 让 Skill 有记忆

通过存储数据实现跨会话记忆：
- 简单：追加写入的文本日志
- 复杂：SQLite 数据库
- 持久化路径：`${CLAUDE_PLUGIN_DATA}`（不会被 Skill 升级删除）

示例：standup-post 每次发完站会汇报后追加到 `standups.log`，下次 Claude 读历史记录自动对比变化。

### 3.8 给代码，别给指令

> 你能给 Claude 的最强大的工具，就是代码。

示例：数据分析 Skill 里的 `lib/signups.py`，定义 `fetch(day)`、`by_referrer(df)`、`by_landing_page(df)` 等函数，docstring 里嵌入 Gotchas。

当用户问"周二发生了什么？"，Claude 自动 import 现成函数组合出分析脚本——不需要重新理解数据表结构。

> 这就像给了一个新来的分析师一个「老员工的工具箱」。

### 3.9 按需 Hooks

Skill 可注册只在被调用时才激活的 hooks，持续到会话结束。

- `/careful`：碰生产环境时触发，拦截 `rm -rf`、`DROP TABLE`、`force-push`、`kubectl delete`
- `/freeze`：锁定特定目录之外的所有文件修改，防止调试时误改不相关代码

---

## 四、分发与度量

### 分发路径

1. **小团队**：直接提交到 `.claude/skills/`
2. **规模化**：内部插件市场，让每个人自己选装
3. **渐进式上市**：sandbox 文件夹 → Slack 推广 → 足够多用户后正式进市场

**警告**：创建糟糕或重复的 Skill 太容易了，正式发布前质量把关不能省。

### 度量

用 PreToolUse hook 记录 Skill 使用情况——哪些用的人多，哪些触发频率低于预期。

---

## 五、与 `/skill-creator` 工具的对比

| 维度 | Thariq 文章 | `/skill-creator` |
|------|------------|-----------------|
| 视角 | 产品/架构——"做什么样的 Skill" | 工具/流程——"怎么做出好 Skill" |
| 核心贡献 | 九大类型 + 九条原则 | draft→test→eval→iterate 工作流 |
| 关注焦点 | Skill 的内容设计 | Skill 的质量保障 |
| 适用阶段 | 0→1（想清楚要做什么） | 1→N（做出来并验证好用） |

**Thariq 独有**：九大类型分类、Gotchas 演化论、首次配置模式、Skill 记忆、按需 Hooks、分发策略

**`/skill-creator` 独有**：量化评估体系（assertions + benchmark）、A/B 基线对比、description 自动优化、子 Agent 并行测试

**结论**：两者互补。Thariq 回答"做什么"和"怎么想"，`/skill-creator` 回答"怎么做"和"怎么验证"。

---

## 六、对 Agent-Native 设计范式的启示

### 6.1 Skill 是渐进式增长的知识资产

Thariq 最重要的洞察：**好的 Skill 需要长期积累才有价值。** Day 1 的三行文字和 Month 3 的完整踩坑手册之间，差的是持续的实践反馈。

这修正了"软件从资产变耗材"的命题——Skill 比传统 App 轻量，但远非"用完即弃"。它更像是**团队知识的结晶体**，通过 Gotchas 不断生长。

### 6.2 Skill 的本质是 Context Engineering

文件夹结构、渐进式信息披露、按需加载——Skill 设计的核心问题不是"写什么指令"，而是**如何高效管理 Agent 的上下文窗口**。

### 6.3 adversarial-review 模式值得推广

启动子 Agent 挑刺 → 修复 → 迭代直到无实质问题——这个模式可以泛化为 Agent-Native 质量保障的标准方法。

### 6.4 安全护栏是 Skill 级别的

`/careful` 和 `/freeze` 示范了一种轻量但有效的安全模式：不是全局永远开启，而是在需要的时候按需激活。这与独立调研中安全方向的发现（沙箱执行、能力声明）形成呼应。

### 6.5 Skill 生态需要质量把控

Anthropic 内部都面临"糟糕/重复 Skill 太容易创建"的问题。这与 MCP 生态的"95% 是垃圾"、OpenClaw 的 1,184 个恶意技能一脉相承——分发门槛低是双刃剑。
