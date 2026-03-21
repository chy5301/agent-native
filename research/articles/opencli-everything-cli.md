# OpenCLI：万物皆可 CLI

> 来源：微信公众号文章 + GitHub 仓库调研
> 阅读日期：2026-03-21
> 仓库：https://github.com/jackwener/opencli（3,282 stars，2026-03-14 创建）

---

## 核心论点

CLI-Anything 给桌面软件装上了 CLI，OpenCLI 给网站和 Electron 应用也装上了 CLI。两者合在一起，基本覆盖了"一切软件"的 CLI 化。

---

## 两条路线对比

| 维度 | CLI-Anything | OpenCLI |
|------|-------------|---------|
| 思路 | 从源码出发 | 从浏览器出发 |
| 覆盖范围 | 有开源代码的桌面软件 | 网站 + Electron 应用 |
| 技术手段 | 扫描源码，映射 GUI→API，Python Click | Chrome 扩展 + WebSocket + 本地 Daemon |
| 需要源码 | 是 | 否 |
| 认证方式 | 不涉及（本地软件） | 复用浏览器登录态 |
| 团队 | 香港大学 HKUDS（学术团队） | 个人开发者 jackwener |
| 技术关联 | 无 | 无（独立项目，理念相似） |

文章的比喻：CLI-Anything 像逆向工程师（拆开外壳理解内部结构），OpenCLI 像老练用户（不拆机器但知道每个按钮的作用）。

---

## OpenCLI 实际架构（基于仓库调研）

### 三层结构

1. **CLI 层**：基于 Commander.js（TypeScript），`src/cli.ts` + `src/commanderAdapter.ts`
2. **运行时层**：`src/runtime.ts`，提供 `browserSession()` 抽象，自动选择 BrowserBridge 或 CDPBridge
3. **浏览器通信层**：两条路径
   - **BrowserBridge 路径**：CLI → HTTP → Daemon(localhost:19825) → WebSocket → Chrome Extension → `chrome.debugger` API → 页面
   - **CDP 直连路径**：CLI → WebSocket → Chrome/Electron CDP endpoint → 页面

### 工作链路

```
CLI 命令 → 本地 Daemon(localhost:19825) → WebSocket → Chrome 扩展(Browser Bridge) → 网页操作
```

- **Chrome 扩展**：实现了 **Automation Window Isolation**——所有操作在专用窗口中执行，不干扰用户正常浏览，窗口 30 秒空闲后自动关闭
- **本地 Daemon**：空闲 5 分钟自动退出，默认监听 localhost:19825
- **认证**：复用 Chrome 已登录的 session，凭据始终留在浏览器内

### 双引擎设计（文章未提及的关键细节）

- **YAML 声明式引擎**（`src/pipeline/`）：声明式数据管道，步骤包括 navigate/fetch/evaluate/select/map/filter/sort/limit/intercept/tap 等。适合简单场景。
- **TypeScript 编程引擎**：直接编写 `func` 回调，通过 `IPage` 接口操作浏览器。适合复杂逻辑。

**IPage 接口**是核心抽象，提供：goto, evaluate, getCookies, click, typeText, pressKey, wait, scroll, autoScroll, screenshot, networkRequests, installInterceptor, getInterceptedRequests, snapshot（无障碍树快照）, tabs/newTab/closeTab/selectTab。

**适配器动态加载**：`.ts` 或 `.yaml` 文件放入 `src/clis/<site>/` 目录后自动注册，构建时通过 `build-manifest.ts` 生成清单。

### Electron 应用支持（CDP 路线）

两层 CDP 实现：

**层1 — Chrome Extension CDP**（`extension/src/cdp.ts`）：
- 利用 `chrome.debugger` API
- 支持 `Runtime.evaluate` 执行 JS、`Page.captureScreenshot` 截图
- 全页截图通过 `Page.getLayoutMetrics` + `Emulation.setDeviceMetricsOverride` 设置视口

**层2 — 直连 CDP Client**（`src/browser/cdp.ts`）：
- 通过 `OPENCLI_CDP_ENDPOINT` 环境变量指定 WebSocket URL
- 30 秒超时保护
- **CDP Target 选择算法**：根据 type/url/title 综合打分，优先选择 app/webview 类型，特别照顾 Cursor/Codex/Notion 等
- 处理 React 等框架的富文本编辑器时，使用 `document.execCommand('insertText')` 模拟真实文本输入

**已支持应用**：Cursor(9222), ChatGPT(9224), Notion(9230), Discord, 飞书, 微信, 网易云音乐, Antigravity, Codex, ChatWise, Grok

---

## 三个 AI 命令（基于源码分析）

### 1. explore（`src/explore.ts`，约 300 行）

**已完整实现。** 核心流程：
1. 通过 BrowserBridge 导航到目标 URL
2. 自动滚动触发懒加载（`autoScroll`）
3. 可选交互式 fuzzing（`--auto --click "字幕,CC"`）
4. 捕获网络流量，分析 JSON 端点
5. 对缺失 body 的 JSON 端点，通过 iframe 重新 fetch 获取响应
6. 框架检测（Vue/React/Pinia/Vuex）+ Store 发现
7. **端点评分系统**：基于 JSON 内容、itemCount、字段检测、搜索/分页参数等综合打分
8. **能力推断**：从 URL 模式自动推断 hot/search/feed/comments 等能力名
9. **认证策略推断**：检测 Authorization/CSRF/签名 header
10. 产出物写入 `.opencli/explore/<site>/`（manifest.json, endpoints.json, capabilities.json, auth.json, stores.json）

### 2. synthesize（`src/synthesize.ts`，约 200 行）

**已完整实现。** 从 explore 产出物生成 YAML 适配器：
- 为 top N 能力各选择最佳端点
- 支持三种模式：public（直接 fetch）、browser（navigate + evaluate）、store-action（Pinia/Vuex store 触发）
- 自动模板化 URL 参数

### 3. cascade（`src/cascade.ts`，约 150 行）

**已完整实现。** 五级认证策略递进：

| 级别 | 策略 | 方法 |
|------|------|------|
| 1 | PUBLIC | 无认证，Node.js 直接 fetch |
| 2 | COOKIE | 浏览器 fetch + `credentials: 'include'` |
| 3 | HEADER | 提取 CSRF token + 自定义 header |
| 4 | INTERCEPT | XHR/Fetch 猴子补丁拦截 |
| 5 | UI | 完整 UI 自动化 |

### 4. generate（`src/generate.ts`，约 130 行）

一键全流程：explore → synthesize → 选择最佳候选 → 注册（注册部分仍是 TODO stub）。

```bash
opencli generate https://example.com --goal "hot"
```

---

## 实际覆盖（比文章描述更丰富）

150+ 命令，36 个站点/应用（文章说 80+ 命令、30+ 站点，已增长）。

**浏览器适配器（约 30 个）**：
- 中国：bilibili, zhihu, xiaohongshu, xueqiu, weibo, v2ex, boss, ctrip, smzdm, sinafinance, chaoxing, jike, jimeng, weread, linux-do
- 国际：twitter, reddit, youtube, hackernews, bbc, bloomberg, arxiv, wikipedia, yahoo-finance, barchart, reuters, linkedin, stackoverflow, steam, coupang, apple-podcasts, xiaoyuzhou, hf

**桌面应用适配器（10 个）**：cursor, codex, antigravity, chatgpt, chatwise, notion, discord-app, grok, feishu, wechat, neteasemusic

**外部 CLI Hub**：gh, obsidian, docker, kubectl, readwise（纯 passthrough）

所有命令支持多种输出格式：json、yaml、markdown、csv。

---

## 安全模型（实际状况）

### 设计理念："Account-safe"
- 凭证永远不离开浏览器
- 不存储密码或 token
- 操作在专用的 Automation Window 中执行

### 实际安全缺陷
- **Daemon 无身份验证**——本机任何进程都可以发送命令到 localhost:19825
- **无沙箱**——evaluate 中的 JS 代码可以做任何事
- **无权限确认机制**——用户运行命令即视为授权，没有细粒度的权限控制
- **XHR 拦截器**通过猴子补丁全局 `fetch` 和 `XMLHttpRequest`，理论上可以捕获所有网络请求
- **CORS 头设置为 `*`**（虽然是本地服务）

**CI 层面**：有 `security.yml`，每周 npm audit，使用 `audit-ci` 检查已知漏洞。

---

## 文章描述 vs 仓库实际的差异

| 维度 | 文章描述 | 仓库实际 |
|------|---------|---------|
| Star 数 | 2,300 | 3,282（快速增长中） |
| 命令数 | 80+ | 150+ |
| 站点数 | 30+ | 36 |
| 创建时间 | 未提及 | 仅一周（2026-03-14） |
| 双引擎 | 未提及 | YAML 声明式 + TypeScript 编程式 |
| explore 实现 | 概念描述 | 完整实现（300 行，含评分系统） |
| generate 注册 | "一条命令搞定" | 注册部分仍是 TODO stub |
| 安全问题 | 概念讨论 | Daemon 无认证、无沙箱、无权限确认 |
| 贡献者 | 未提及 | 28 人，但 jackwener 占 267/300+ commits |
| 技术栈 | 未提及 | TypeScript, Commander.js, 仅 5 个 runtime 依赖 |

---

## 与调研主线的关联

### 1. 印证核心命题
- **"GUI 是翻译层"**：OpenCLI 把网站的 GUI 也给剥掉了
- **"软件从资产变成耗材"**：explore/synthesize 让 Agent 按需生成适配器，用完即弃
- **"中间层消亡"**：网站 GUI 就是人机中间层，OpenCLI 正在移除它

### 2. 自发现能力的演进阶梯

```
手动编写 CLI → CLI-Anything 半自动生成(需源码) → OpenCLI explore/synthesize 全自动发现(无需源码)
```

这是从"人类为 Agent 准备工具"到"Agent 自己制造工具"的关键跨越。

### 3. 安全问题比文章描述更严峻
文章讨论的是概念层面的权限风险。实际仓库调研发现更具体的安全缺陷：
- Daemon 无认证 = 本机任何恶意进程都可以冒充用户操作浏览器
- 无沙箱 = 适配器可以在页面中执行任意 JS
- 无权限确认 = Agent 发弹幕、删数据都不需要用户确认

### 4. 双引擎设计值得关注
YAML 声明式 + TypeScript 编程式的双引擎架构，是一个实用的设计模式：
- 简单的数据抓取用 YAML 声明即可（低门槛）
- 复杂的交互逻辑用 TypeScript（高灵活度）
- 对 Agent 来说，自动生成 YAML 适配器比生成 TypeScript 代码更可靠

### 5. 项目成熟度判断
- 一周内从 0 到 3000+ star，增长极快
- 代码量大但高度集中于单一贡献者（267/300+ commits）
- 部分功能仍是 TODO（如 generate 的注册部分）
- 安全模型粗糙
- **判断**：处于早期爆发阶段，概念验证已完成，但生产就绪尚远

---

## 相关链接

- [OpenCLI GitHub](https://github.com/jackwener/opencli)
- [CLI-Anything GitHub](https://github.com/HKUDS/CLI-Anything)
- [OpenCLI 官网](https://opencli.info/)
