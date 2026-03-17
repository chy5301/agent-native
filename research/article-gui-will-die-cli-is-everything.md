# GUI 将死，CLI 才是一切

> 来源：微信公众号文章

---

GUI 赢了上一个十年，但它的时代快到头了。

CLI 正在赢下一个十年，只不过这次，CLI 的用户不再是开发者，而是 Agent。

这个判断听起来有点反直觉，毕竟 CLI 是计算机世界里最古老的交互方式之一了。但回头看这半年的趋势，Claude Code 是 CLI，Codex 是 CLI，OpenClaw 也是 CLI。

Agent 操控计算机的方式，正在从「看屏幕点鼠标」转向「读文档敲命令」。

---

## CLI-Anything

有个项目叫 CLI-Anything，来自香港大学的 HKUDS 团队，已经拿到了 15000 颗星，也算是进一步把这件事挑明了。

该项目说起来，也并不复杂：**给任意软件自动生成一套 CLI 接口，让 AI Agent 能直接用命令行操控 GIMP、Blender、Audacity、LibreOffice……** 几乎你能想到的桌面软件，它都能包一层。

---

## 为什么是 CLI

此前的文章《一切软件，都将为 Agent 重写》里给出了一个观点：

> GUI 本质上是一个翻译层。人类花了 40 年给计算机套上图形界面，是因为人类不擅长记命令。

但 Agent 不需要图形界面。

Agent 天生就是说命令行语言的。你让它点按钮、拖滑块、填表单……那是在强迫一个天生会说 API 的实体，去学一门它根本不需要的语言。

CLI-Anything 做的事情，本质上就是**把那层「翻译层」给剥掉了**，让 Agent 直接和软件的底层对话。

---

## 怎么做到的

CLI-Anything 的实现分七步，全自动：

1. 扫描源码，把 GUI 操作映射到底层 API
2. 设计命令结构和状态模型
3. 用 Python Click 框架生成 CLI
4. 自动写测试
5. 自动写文档
6. 最后打包安装到系统 PATH

整个过程只需要一条命令：

```
/cli-anything:cli-anything ./gimp
```

生成出来的 CLI 有几个特点值得留意：

- 每个命令都支持 `--json` 输出，Agent 拿到的是结构化数据，不用解析一坨文本
- 每个命令都有 `--help`，Agent 可以自己「发现」能力，不需要你提前告诉它有哪些功能
- 还有 undo/redo 和持久化的项目状态

目前已经通过测试的软件有 GIMP、Blender、OBS Studio、Audacity、Kdenlive、Inkscape、Draw.io、LibreOffice、Zoom 等等，1508 个测试全部通过。

要知道，这是能跑在生产环境里的东西。

---

## 我早就这么干了

看到 CLI-Anything 火了，我其实有点感慨。因为我很早就在用 CLI + Skill 的方式来操控各种服务了：

- 用 Cloudflare 的 CLI 管理域名解析、Workers 部署、Pages 配置
- 用 aliyun CLI 管理阿里云的 RAM 用户、OSS 存储、ECS 实例
- 用 gcloud CLI 操作 Google Cloud 的各种资源

这些 CLI 本来是给人类工程师用的。但当你把它们写成 Skill，告诉 Claude Code 每个命令该怎么用、什么场景该调哪个……Agent 就能直接上手了。

比如我有一个 RAM 用户管理的 Skill，Claude Code 读完 SKILL.md 之后，就能帮我创建子账号、配权限、生成 AccessKey，全程命令行操作，我只需要说一句「帮我建个只读 OSS 的子账号」。

**CLI 本来就是 Agent 的母语。** CLI-Anything 的贡献在于，它把这条路从「手动写 Skill」变成了「自动生成 CLI」，覆盖了那些原本没有好用命令行的软件。

---

## 组合拳

去年 11 月的文章《MCP 或将成弃子》里写到过，与其让 Agent 在 context 里塞满工具描述（MCP 的做法），不如让它直接写代码调用。

而 CLI + Skill 的组合，其实是这个思路的自然延伸：

- **Skill 是知识**，是「怎么干」
- **CLI 是接口**，是「用什么干」

二者组合起来，Agent 只需要读一份简短的 SKILL.md（几百 tokens），就知道该调哪些命令、参数怎么传、结果怎么解析。

不需要 MCP 那样把所有工具定义塞进 context，也不需要 GUI 自动化那种脆弱的截图 + 点击。

CLI-Anything 目前已经支持 Claude Code、OpenClaw、OpenCode、Qodercli 等平台，Cursor 和 Windsurf 也在路线图上。

---

## 真正的问题

但 CLI-Anything 本身并不是什么新的智能，它是管道工程。把软件变成 Agent 能调用的命令，这件事技术上并不难，难的是后面那一步。

**当 Agent 发出命令之后，谁来决定这条命令能不能执行？**

这其实才是 Agent 时代最核心的问题了。当 Agent 能直接操控 Blender 渲染、Audacity 处理音频、LibreOffice 生成 PDF 的时候……**权限边界在哪里，谁来兜底呢？**

CLI 解决的是 Agent「能不能干」的问题。但「该不该干」「谁说了算」，这些问题才刚刚浮出水面。

---

## 接下来会怎样

我的判断是，**CLI 会成为 Agent 时代的标准交互协议**。

不是因为 CLI 本身有多先进。恰恰相反，它应该算是计算机最古老的交互方式之一了。但它天然适合 Agent：文本输入，结构化输出，可组合，可发现，确定性强。

人类花了 40 年给计算机加上 GUI 这层翻译层。现在 Agent 来了，它不需要这层翻译。

就像当年 USB 统一了硬件接口一样，CLI 正在成为 Agent 世界的「通用插口」。

整个软件行业，正在把那件给人类穿的外衣，一件一件脱掉。

而 Agent，终于可以穿自己的衣服了。

---

## 相关链接

- [CLI-Anything GitHub](https://github.com/HKUDS/CLI-Anything)
- [CLI-Anything 官网](https://clianything.org/)
