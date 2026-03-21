# OpenCLI 案例研究：浏览器路线的万物 CLI 化

> 调研日期：2026-03-21
> 信息来源：GitHub jackwener/opencli 仓库源码 + 微信公众号文章
> 性质：开源项目（Apache-2.0），2026-03-14 创建，3,282 stars
> 核心定位：通过 Chrome 扩展 + 本地 Daemon，将网站和 Electron 应用转化为 CLI 接口

---

## 一、项目概况

| 指标 | 数据 |
|------|------|
| GitHub | https://github.com/jackwener/opencli |
| Stars | 3,282（一周内） |
| 主语言 | TypeScript |
| 依赖 | 仅 5 个 runtime（chalk, cli-table3, commander, js-yaml, ws） |
| 命令数 | 150+ |
| 覆盖站点/应用 | 36 |
| 贡献者 | 28 人，但 jackwener 贡献 267/300+ commits |
| npm | @jackwener/opencli |

**核心定位**：给 Agent 用的"万能遥控器"——将网站和 Electron 应用 CLI 化，让 AI Agent 通过命令行操控一切。与 CLI-Anything（源码路线）互补，覆盖无源码的软件。

---

## 二、技术架构

### 2.1 三层结构

```
┌──────────────────────────────────────────────────┐
│ CLI 层：Commander.js (src/cli.ts)                │
├──────────────────────────────────────────────────┤
│ 运行时层：browserSession() 自动选择通信路径       │
│          (src/runtime.ts)                        │
├──────────────┬───────────────────────────────────┤
│ BrowserBridge│ CDPBridge                         │
│ (扩展路径)    │ (直连路径)                        │
│ CLI→HTTP→    │ CLI→WebSocket→                    │
│ Daemon→WS→   │ CDP endpoint→                     │
│ Extension→   │ 页面                              │
│ chrome.      │                                   │
│ debugger→页面│                                   │
└──────────────┴───────────────────────────────────┘
```

### 2.2 双引擎设计

这是仓库调研中发现的关键设计决策（文章未提及）：

**YAML 声明式引擎**（`src/pipeline/`）：
```yaml
# 示例：声明式抓取知乎热榜
steps:
  - navigate: https://www.zhihu.com/hot
  - evaluate: |
      // 提取热榜数据
  - select: ".HotItem"
  - map: { title: ".HotItem-title", heat: ".HotItem-metrics" }
```
步骤类型：navigate, fetch, evaluate, select, map, filter, sort, limit, intercept, tap

**TypeScript 编程引擎**：
```typescript
// 示例：编程式操作 Twitter
export default {
  func: async (page: IPage, args: Args) => {
    await page.goto('https://twitter.com/bookmarks');
    // 复杂交互逻辑...
  }
}
```

**IPage 接口**（核心抽象）：
- 导航：`goto()`, `wait()`
- DOM：`evaluate()`, `click()`, `typeText()`, `pressKey()`
- 滚动：`scroll()`, `autoScroll()`
- 网络：`networkRequests()`, `installInterceptor()`, `getInterceptedRequests()`
- 截图：`screenshot()`（支持全页）
- Tab：`tabs()`, `newTab()`, `closeTab()`, `selectTab()`
- 辅助：`snapshot()`（无障碍树快照）, `getCookies()`

**设计启示**：YAML 适合 Agent 自动生成（synthesize 命令产出的就是 YAML），TypeScript 适合人工编写复杂逻辑。两者互补。

### 2.3 适配器动态加载

文件放入 `src/clis/<site>/` 目录后自动注册，构建时通过 `build-manifest.ts` 生成清单。无需手动 import。

### 2.4 Automation Window Isolation

Chrome Extension 的 `background.ts` 实现了隔离机制——所有 OpenCLI 操作在专用的 Chrome 窗口中执行，不干扰用户正常浏览。窗口 30 秒空闲后自动关闭。

---

## 三、AI 自发现管线（核心创新）

### 3.1 完整链路

```
explore → synthesize → cascade → generate
  ↓           ↓           ↓          ↓
发现API    生成适配器   检测认证   一键全流程
```

### 3.2 explore（`src/explore.ts`，约 300 行）

**实现细节**：
1. BrowserBridge 导航到目标 URL
2. `autoScroll` 触发懒加载
3. 可选 fuzzing：`--auto --click "字幕,CC"`，点击按钮触发隐藏 API
4. 捕获网络流量，分析 JSON 端点
5. iframe 重新 fetch 获取缺失的响应 body
6. 框架检测（Vue/React）+ Pinia/Vuex Store 发现
7. **端点评分系统**：综合 JSON 内容质量、itemCount、字段检测、搜索/分页参数打分
8. **能力推断**：从 URL 模式推断 hot/search/feed/comments 等能力名
9. **认证策略推断**：检测 Authorization/CSRF/签名 header

**产出物**：`.opencli/explore/<site>/`
- `manifest.json` — 站点元数据
- `endpoints.json` — 发现的 API 端点
- `capabilities.json` — 推断的能力
- `auth.json` — 认证策略
- `stores.json` — 前端 Store（Pinia/Vuex）

### 3.3 synthesize（`src/synthesize.ts`，约 200 行）

从 explore 产出物生成 YAML 适配器：
- 为 top N 能力各选择最佳端点
- 三种生成模式：
  - **public**：直接 fetch（无需浏览器）
  - **browser**：navigate + evaluate
  - **store-action**：Pinia/Vuex store 触发
- 自动模板化 URL 参数

### 3.4 cascade（`src/cascade.ts`，约 150 行）

五级认证策略递进，每级构建不同的 fetch probe：

| 级别 | 策略 | 实现 |
|------|------|------|
| 1 | PUBLIC | 无 credentials |
| 2 | COOKIE | `credentials: 'include'` |
| 3 | HEADER | 提取 CSRF token（ct0/csrf_token） |
| 4 | INTERCEPT | XHR/Fetch 猴子补丁 |
| 5 | UI | 完整 UI 自动化（标记为需站点特定实现） |

返回最简可用策略 + 置信度分数。

### 3.5 generate（`src/generate.ts`，约 130 行）

一键全流程：explore → synthesize → 选择最佳候选 → **注册（仍是 TODO stub）**。

支持中英文别名归一化（"热门" → hot）。

**注意**：注册部分尚未实现，说明项目仍处于早期阶段。

---

## 四、Electron/CDP 支持

### 4.1 实现细节

**Extension CDP**（`extension/src/cdp.ts`）：
- `chrome.debugger` API，protocol version 1.3
- `ensureAttached()` 防止重复 attach
- 支持 `Runtime.evaluate` + `Page.captureScreenshot`

**直连 CDP Client**（`src/browser/cdp.ts`）：
- 通过 `OPENCLI_CDP_ENDPOINT` 环境变量连接
- 30 秒超时保护
- `Page.loadEventFired` 等待（替代硬编码 sleep）
- **Target 选择算法**：根据 type/url/title 综合打分，优先 app/webview 类型

### 4.2 React 框架绕过

处理富文本编辑器时的技术细节：
- **不能**直接设置 `.value`（框架内部状态不会更新）
- **使用** `document.execCommand('insertText')` 模拟真实文本输入
- 这样 React/Vue 等框架的 onChange 事件会正确触发

### 4.3 已支持的 Electron 应用

| 应用 | 调试端口 |
|------|---------|
| Cursor | 9222 |
| ChatGPT | 9224 |
| Notion | 9230 |
| Antigravity | - |
| Codex | - |
| ChatWise | - |
| Discord | - |
| Grok | - |
| 飞书 | - |
| 微信 | - |
| 网易云音乐 | - |

---

## 五、安全模型分析

### 5.1 设计理念

**"Account-safe"**：凭证永远不离开浏览器。不存储密码或 token。

### 5.2 实际安全缺陷

| 问题 | 风险等级 | 说明 |
|------|---------|------|
| Daemon 无身份验证 | 高 | 本机任何进程都可发送命令到 localhost:19825 |
| 无沙箱 | 高 | evaluate 中的 JS 可做任何事 |
| 无权限确认 | 中 | 运行命令即授权，无细粒度控制 |
| XHR 全局拦截 | 中 | 猴子补丁可捕获所有网络请求 |
| CORS 设为 * | 低 | 仅本地服务，但不够严谨 |
| 全权委托模型 | 高 | Agent 拥有用户在浏览器中的全部权限 |

### 5.3 与权限模型调研的关联

OpenCLI 完美展示了**无约束 Agent 权限**的风险场景：
- Agent 能发弹幕、发消息、删数据——都用用户的真实账号
- 网络操作不可撤回（不像本地文件可以 undo）
- 作用域无隔离（Agent 能访问用户登录的所有网站）

这进一步验证了我们在安全模型调研（G-06）中的结论：**Agent 工具需要分级权限控制**，不能简单地全权委托。

---

## 六、与 CLI-Anything 的对比

### 6.1 无直接技术关联

从代码和贡献者来看，两个项目完全独立：
- jackwener 不是 CLI-Anything 的贡献者
- HKUDS 也不是 OpenCLI 的贡献者
- CLI-Anything 用 Python，OpenCLI 用 TypeScript

### 6.2 互补覆盖

```
有源码的桌面软件 ──→ CLI-Anything（Python Click, 源码路线）
无源码的网站     ──→ OpenCLI（YAML/TS, 浏览器路线）
Electron 应用    ──→ OpenCLI（CDP 路线）
外部 CLI 工具    ──→ OpenCLI CLI Hub（passthrough）
```

### 6.3 关键差异

| 维度 | CLI-Anything | OpenCLI |
|------|-------------|---------|
| 深度 vs 广度 | 深度集成，调用真实软件后端 | 广度覆盖，通过浏览器操作 |
| 稳定性 | 依赖 API 映射（较稳定） | 依赖 DOM 结构（可能脆弱） |
| 自发现 | 需要人类提供源码 | explore/synthesize 自动发现 |
| 输出质量 | 直接调用软件引擎，输出精确 | 从网页提取，可能有解析误差 |
| 认证复杂度 | 不涉及 | 五级认证策略递进 |

---

## 七、对本项目的直接输入

### 7.1 核心命题验证

| 命题 | OpenCLI 提供的证据 |
|------|-------------------|
| GUI 是翻译层 | 将网站 GUI 剥离，Agent 直接操作底层 API |
| 产品形态从 App 变成 Skill | 每个适配器本质上就是一个 Skill |
| 软件从资产变成耗材 | explore/synthesize 按需生成适配器，用完即弃 |
| 中间层消亡 | 网站 GUI 这个人机中间层正在被移除 |

### 7.2 新的设计模式

- **双引擎模式**：YAML 声明式（低门槛，Agent 可自动生成）+ 编程式（高灵活度）
- **自发现管线**：explore → synthesize → cascade → generate
- **Automation Window Isolation**：在专用窗口中执行操作，不干扰用户
- **认证策略递进**：从简到繁逐级尝试，找到最简可用方案

### 7.3 安全模型警示

- 全权委托是最大风险——Agent 不应该默认拥有用户的全部权限
- 网络操作的不可撤回性需要在权限模型中特别处理
- Daemon 无认证是严重的本地安全漏洞——任何本机进程都能冒充用户

### 7.4 成熟度判断

- **概念验证**：已完成，核心功能可用
- **生产就绪**：远未到达（安全模型粗糙、部分功能 TODO、DOM 依赖脆弱）
- **方向判断**：值得关注，但需要持续跟踪其安全和稳定性方面的演进

---

## 参考来源

- [OpenCLI GitHub](https://github.com/jackwener/opencli)
- [OpenCLI 官网](https://opencli.info/)
- [CLI-Anything GitHub](https://github.com/HKUDS/CLI-Anything)（对比参考）
- 微信公众号文章《OpenCLI：万物皆可 CLI》
