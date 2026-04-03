# 2026 年，是造 CLI 的最好时机

> 来源：AGI Hunt 公众号（J0hn）
> 日期：2026-04-03
> 链接：https://mp.weixin.qq.com/s/8_4LCvQbOEyX24GuW-gRRA

## 核心论点

CLI 经历三波浪潮，当下是造 CLI 的最佳时机：框架成熟、AI 辅助开发、Agent 驱动需求三者同时就位。

## 三波浪潮

1. **1970-1995 Unix 黄金期**：ls、grep、awk、sed、curl 等经典工具诞生
2. **2015-2024 终端文艺复兴**：Go/Rust 的单二进制分发打破了 20 年的 CLI 创新停滞。Rust 社区重写经典命令（ripgrep、bat、eza、fd、zoxide、dust、sd）
3. **2025+ Agent 时代**：CLI 从开发者工具升级为软件的通用入口

## 五大 CLI 框架对比

| 框架 | 语言 | Stars | 适用场景 |
|------|------|-------|---------|
| **Cobra** | Go | 43.5k | 中大型 CLI（10+ 子命令），飞书 CLI 在用 |
| **Typer** | Python | 18.3k | 快速原型、数据脚本（分发是短板） |
| **Clap** | Rust | 10k+ | 高性能工具，derive macro 风格 |
| **Commander.js** | Node.js | 28k | 零依赖，18ms 启动，Node 团队默认选择 |
| **Picocli** | Java | 5.2k | Java 生态，支持 GraalVM native image |

**选型结论**：Go + Cobra 是"从造出来到分发出去"路径最短的组合。

## 框架选型决策树

- 需要跨平台单二进制？→ Go 或 Rust
- Python 团队？→ Typer（分发用 PyInstaller）
- Node.js 工具链？→ Commander.js（生态一致性 > 语言性能）
- 子命令 > 10 个？→ Cobra 或 Clap
- 追求极致性能？→ Rust + Clap

## AI 辅助造 CLI 的工作流

作者用 Claude Code 辅助完成了一个内部 CLI 工具，流程：
1. 准备三样 context：设计规范（Agent CLI 原则 checklist）、框架知识（Cobra 最佳实践）、业务需求
2. 丢给 Claude Code 生成命令树骨架
3. 人工调整参数命名、补边界情况，2-3 轮迭代定型

关键感受：**瓶颈从"怎么写代码"变成"怎么把需求描述清楚"**。设计规范越清晰，生成质量越高。全程 2 小时。

## Agent 友好六要素

1. **输出必须结构化**：`--json` / `--format json`，进度走 stderr
2. **退出码有语义**：2=参数错误、3=资源不存在、4=权限不足、10=dry-run（Lightning Labs 实践）
3. **支持 --dry-run**：输出 JSON diff 预览
4. **支持 --no-interactive**：Agent 无法回答 `Are you sure?`。反面教训：AWS CLI v2 (2019) 默认 pager 改为 `less`，挂起全球数千 CI 管道
5. **Schema 自省**：`my-cli schema --all` 输出命令树 JSON；飞书 CLI 已实现
6. **错误信息指导修复**：包含错误类型 + 描述 + 修复建议 + 是否可重试

## 引用的关键参考源

- **ScaleKit benchmark**：GitHub MCP 服务器 vs `gh` CLI，75 轮实验。CLI token 消耗便宜 10-32 倍，可靠性 100%（MCP 72%）
- **Lightning Labs PR #14**：系统性实现 10 个 agent-CLI 设计轴，最完整的实战参考
- **InfoQ《Keep the Terminal Relevant》**：从设计模式角度给出 AI Agent 驱动 CLI 的建议
- **Smithery《MCP vs CLI Is the Wrong Fight》**：CLI 赢在 LLM 有训练先验的本地工具，MCP 赢在零训练数据的远程内部 API
- **CircleCI**：内循环用 CLI（速度+token效率），外循环用 MCP（团队规模+结构化鉴权）
- **Gabe Venberg《The Modern CLI Renaissance》**：CLI 工具 1995-2015 的 20 年创新停滞
- **Railway CLI Rust Rewrite**：从 Go 重写为 Rust 的案例，获得更好类型安全但代价是团队投入

## 对本项目的价值

| 发现 | 改进动作 |
|------|---------|
| `--no-interactive` 是 Agent 友好的关键要素，现有设计指南完全未提及 | 补充到 SKILL.md checklist 和 design-principles.md |
| 退出码语义应更细粒度（3=不存在、4=权限、10=dry-run） | 扩展 SKILL.md checklist 和示例代码 |
| CLI vs MCP 的量化对比数据（ScaleKit）可增强现有论据 | 现有架构文档已引用类似数据，无需重复 |
| 框架选型（Cobra/Clap/Typer 等）属于实现层，不属于设计原则范畴 | 记录在此笔记中供参考，不纳入 Skill |
