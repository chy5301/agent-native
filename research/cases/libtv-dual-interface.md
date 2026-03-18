# LibTV 案例研究：画布编排型双接口产品

> 调研日期：2026-03-18
> 信息来源：微信公众号文章、GitHub libtv-labs/libtv-skills、liblib.tv 产品页面

---

## 一、产品概况

**LibTV** 是 LiblibAI（哩布哩布AI）推出的 AI 视频创作平台，定位为"专业 AI 视频创作工具"。

- **母公司**：LiblibAI，国内头部 AI 创作平台，2500 万+用户
- **关联产品**：Lovart AI（AI 设计 Agent，前字节跳动高管创办）
- **积分体系**：与 LiblibAI 主站打通，已有会员和积分可无缝迁移
- **产品地址**：https://www.liblib.tv/
- **Skill 仓库**：https://github.com/libtv-labs/libtv-skills（58 stars, MIT）

**核心定位**：同时为人类专业创作者和 AI Agent 设计的 AI 视频产品——首批明确采用双接口设计的生产级产品之一。

**⚠️ 适用性说明**：LibTV 的模式有强烈的**画布/蓝图编排工作流**特征（节点连线、多步创作管线），天然适合视频/图片等创意生成场景。它并非通用的 Agent-Native 设计范式——对于非画布型工具（如数据处理、系统管理、文本分析等），其双接口架构中"无限画布"这一人类入口的设计不具有普适性。本案例的参考价值主要在于：(1) "Skill 即薄中继"的后端架构模式、(2) IM 异步会话协议、(3) Skill 生态中的 IP 保护策略。

---

## 二、双接口架构

LibTV 的最大特点是**同一个产品通过两个完全不同的入口，服务两类完全不同的用户**。

### 2.1 人类入口：无限画布

面向专业创作者的节点式无限画布（Web GUI），功能极其全面：

- **全链路覆盖**：剧本 → 图片 → 视频 → 音频，一站式完成
- **图片处理**：高清、扩图、重绘、擦除、抠图、多角度生成、灯光调节
- **视频生成**：支持几乎所有主流视频模型（Kling 3.0/O3、Wan 2.6 等）
- **专业控制**：摄像机控制（真实镜头、光圈、焦距）、网格切分、剧情推演
- **脚本系统**：剧本生成分镜、角色设定驱动创作
- **音频节点**：音频驱动数字人、音乐生成

**目标用户**：希望精细控制每个创作环节的专业 AI 短片/广告创作者。

### 2.2 Agent 入口：OpenClaw Skill

面向 Agent 的极简接口：

```bash
# 安装
npx skills add libtv-labs/libtv-skills --skill libtv-skill

# 使用（通过 Agent 自然语言）
"帮我生成一下图片：黑白、模糊的歌剧芭蕾舞者，使用 Canon K-35 拍摄"
```

**目标用户**：Claude Code、Codex、OpenClaw 等 Agent 平台的用户——无需理解画布，一句话完成创作。

### 2.3 共享后端

关键设计：**两个入口共享同一个 project/session 状态**。

- Agent 通过 Skill 创建的内容会自动出现在画布项目中，所有节点串联完成
- 人类可以在画布上继续精调 Agent 生成的内容
- 反之亦然：画布上的项目可被 Agent 继续操作

这意味着：**Agent 出初稿（70 分），人类在画布上优化到 100 分**——可能是未来最普遍的创作方式。

---

## 三、"Skill 即薄中继"模式

LibTV 的 Skill 设计体现了一种此前未被明确命名的架构模式。

### 3.1 核心原则

SKILL.md 中明确声明：**"用户侧 Agent 只转发，不创作"**。

用户侧 Skill 只做三件事：
1. **上传**：将本地文件传到 OSS
2. **转发**：将用户的原始请求原封不动传给后端
3. **取回**：轮询结果、下载文件、展示给用户

### 3.2 明确禁止

SKILL.md 明确禁止用户侧 Agent：
- ❌ 扩写、润色或翻译 prompt
- ❌ 将任务拆解为步骤
- ❌ 自行添加提示工程（如"ultra-realistic, cinematic lighting, 8K"）
- ❌ 自行编排多步工作流

### 3.3 后端 Agent 完成一切

真正的创作能力全部运行在 LibTV 的后端 Agent 上：
- 理解需求、拆解分镜
- 选择合适的模型（Seedance 2.0, Kling 3.0, Seedream 5.0 等）
- 编写专业 prompt
- 编排多步工作流
- 返回最终结果

### 3.4 模式意义

**"接口而非大脑"**——对外发布的是调用接口，核心能力留在后端：

- **IP 保护**：核心 prompt 工程、模型调用策略、分镜生成逻辑不对外暴露
- **持续迭代**：后端 Agent 可以随时升级，用户侧 Skill 无需变动
- **生态开放**：Skill 遵循 OpenClaw 标准，任何兼容 Agent 都能调用
- **质量保证**：专业化的后端 Agent 远比通用 Agent 更擅长创作任务

这解决了 Skill 生态的一个核心矛盾：**开放性 vs 商业壁垒**。你可以开放接口而不开放大脑。

---

## 四、IM 异步会话协议

LibTV 没有采用传统 REST API，而是设计了一套**即时通信（IM）风格的异步会话协议**。

### 4.1 协议流程

```
1. POST /openapi/session        → 创建会话 / 发送消息
2. GET  /openapi/session/:id    → 轮询响应（支持 afterSeq 增量拉取）
3. POST /openapi/session/change-project → 切换项目
4. POST /openapi/file/upload    → 上传文件（最大 200MB）
```

认证方式：Bearer token（`LIBTV_ACCESS_KEY`）

### 4.2 与 REST API 的区别

| 维度 | 传统 REST API | LibTV IM 会话 |
|------|--------------|--------------|
| 交互模式 | 请求-响应 | 异步消息传递 |
| 上下文 | 无状态 | 会话持久化 |
| 复杂任务 | 需客户端编排多个 API 调用 | 一条消息，后端 Agent 自行编排 |
| 轮询 | 不需要 | 每 8 秒轮询，3 分钟超时 |
| 多轮交互 | 需客户端管理状态 | 后端 Agent 保持对话上下文 |

### 4.3 为什么 IM 模式更适合创意型工作流

- 创意任务天然是**对话式**的（"帮我生成→不够好→换个风格→加个元素"）
- 后端 Agent 需要**多步推理**才能完成任务（分镜→选模型→生成→后处理）
- 生成过程**耗时较长**（视频生成可能需要数分钟），异步模式更自然
- 会话上下文使得**迭代优化**成为可能，无需每次重述需求

---

## 五、Skill 仓库结构

### 5.1 文件结构

```
libtv-skills/
├── README.md
├── LICENSE (MIT)
└── skills/
    └── libtv-skill/
        ├── SKILL.md              # OpenClaw skill 规范文件
        └── scripts/
            ├── _common.py        # 认证、HTTP 工具、API 封装
            ├── create_session.py # 创建会话 / 发送消息
            ├── query_session.py  # 轮询会话消息
            ├── change_project.py # 切换绑定项目
            ├── upload_file.py    # 上传图片/视频到 OSS
            └── download_results.py # 批量下载结果到本地
```

### 5.2 技术特点

- **纯 Python 标准库**，零外部依赖
- 遵循 **OpenClaw skill 规范**
- 6 个脚本对应 4 个 API 端点 + 2 个辅助操作
- 用户侧暴露的功能极少——几乎只有触发和通信

### 5.3 生态分发

- **ClawStack/ClawHub**：OpenClaw 技能注册表
- **Termo.ai**：Agent 技能市场
- **LobeHub**：第三方集成

---

## 六、对 Agent-Native 设计范式的启示

### 6.1 可迁移的模式 vs 场景绑定的设计

LibTV 的设计中，有些元素是**可迁移到其他类型工具**的，有些则是**画布编排场景特有**的：

**✅ 可迁移（通用参考价值）**：
- **"Skill 即薄中继"架构**：用户侧 Skill 只做触发和通信，业务逻辑集中在后端。这对任何需要保护核心能力的 SaaS 转 Skill 产品都适用
- **IM 异步会话协议**：对耗时较长、需要多轮交互的任务普遍适用（不限于创意生成）
- **"接口而非大脑"的 IP 保护策略**：开源 Skill 建生态覆盖，核心能力留后端。适用于任何有商业化需求的 Skill
- **Skill 即产品分发**：GitHub 仓库就是面向 Agent 的产品，`npx skills add` 就是安装

**⚠️ 场景绑定（不可直接迁移）**：
- **无限画布作为人类入口**：节点连线的画布式编排是视频/设计工作流的特有模式。数据处理工具不需要画布，系统管理工具不需要节点连线。不同类型的工具需要找到自己的"人类入口"形态
- **"Agent 出初稿，人类在画布上精调"**：这个协作模式依赖画布的可视化编辑能力，不是所有场景都有对应的"精调界面"
- **后端 Agent 做全部创作决策**：对于确定性要求高的工具（如金融计算、系统运维），用户可能需要更多控制权而非完全委托给后端 Agent

### 6.2 双入口的一般性启示

LibTV 证明了"两个入口，一个内核"在产品层面是可行的。但**人类入口不一定是画布**：

| 工具类型 | Agent 入口 | 人类入口（各不相同） |
|---------|-----------|-------------------|
| 创意生成（LibTV） | Skill | 无限画布 |
| 数据分析工具 | CLI `--json` | Web Dashboard / 报表 |
| 系统管理工具 | CLI | TUI / Web 控制台 |
| 文档处理工具 | CLI | 编辑器插件 / Web UI |

通用原则是：**Agent 入口趋同（CLI/Skill/MCP），人类入口因场景而异**。

### 6.3 新的商业模式可能

"接口而非大脑"模式意味着：
- 开源 Skill 建立生态覆盖
- 后端 Agent 的核心能力是真正的商业壁垒
- 按调用量/生成量计费，而非按席位

这一点不依赖画布模式，对所有想在 Skill 生态中商业化的产品都有参考价值。

### 6.4 与其他双模方案的对比

| 方案 | 人类界面 | Agent 界面 | 后端共享 | 适用场景 |
|------|---------|-----------|---------|---------|
| LibTV | 无限画布 | OpenClaw Skill | ✅ 完全共享 | 画布编排型创意工作流 |
| A2UI + AG-UI | Agent 动态生成 | 协议层 | ✅ 协议级 | 需要 Agent 运行时生成 UI 的通用场景 |
| CLI `--json` + `--report` | HTML 报告 / Dashboard | CLI | ✅ 共享核心逻辑 | 大多数工具型软件（推荐 MVP 方案） |
| Streamlit/Gradio | 框架生成 Web UI | Python API | ✅ Python 层 | 数据科学 / ML 演示 |
| CLI + Web Dashboard | 独立 Web UI | CLI | ❌ 仅共享 API | 传统运维/监控工具 |

---

## 参考来源

- [LibTV 官方网站](https://www.liblib.tv/)
- [libtv-labs/libtv-skills GitHub 仓库](https://github.com/libtv-labs/libtv-skills)
- [微信公众号文章：第一个同时为人类和Agent设计的AI视频产品](https://mp.weixin.qq.com/s/A8Unxhp-OU79VsTFa7mJPA)
- [LiblibAI 主站](https://www.liblib.art/)
- [Lovart AI](https://www.lovart.ai/)
