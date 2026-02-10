# RedNote Browser Automation MCP

[English](#english) | [中文](#中文)

---

## ⚠️ 免责声明 / Disclaimer

**中文：**
本项目仅供学习研究使用。使用本工具访问小红书平台可能违反小红书的用户协议。使用者需自行承担使用本工具产生的所有风险，包括但不限于账号被封禁、IP 被限制等。本项目开发者不对使用本工具造成的任何后果负责。请遵守相关法律法规和平台规则。

**English:**
This project is for educational and research purposes only. Using this tool to access the Xiaohongshu platform may violate the platform's terms of service. Users assume all risks associated with using this tool, including but not limited to account suspension and IP blocking. The developers are not responsible for any consequences resulting from the use of this tool. Please comply with applicable laws and platform regulations.

---

<a name="english"></a>

## English

> 🤖 **AI-powered browser automation for Xiaohongshu (小红书/RedNote)**
>
> **Built on [iFurySt/RedNote-MCP](https://github.com/iFurySt/RedNote-MCP.git)** with critical bug fixes and architectural improvements:
> - ✅ Fixed xsec_token navigation failures
> - ✅ Fixed hidden element click issues
> - ✅ Fixed login browser crash
> - ✅ Fixed search result timing issues
> - ✅ Replaced hardcoded CSS selectors with generic snapshot extraction
> - ✅ Added anti-detection measures

### 📹 Demo Video

[![Watch Demo](https://img.shields.io/badge/Watch-Demo-red)](https://github.com/user-attachments/assets/2c2f0ae3-89e9-4b61-8406-9e6c40b49977)

---

### ✨ Features

- **🔧 Generic Tools** — 9 universal browser operations (browse, search, click, scroll, snapshot, etc.)
- **📸 Snapshot-Driven** — Returns structured page data instead of hardcoded CSS extractions
- **🛡️ Anti-Detection** — Built-in stealth measures (webdriver masking, realistic delays)
- **🔐 Auto-Login** — Automatic login on first use with cookie persistence
- **🧠 AI-Friendly** — Designed for LLM to freely explore and extract information
- **🚫 Selector-Free** — No CSS selectors = resilient to page structure changes

### 🎯 Use Cases

- Search notes and users on Xiaohongshu
- Extract note content, comments, likes, and collections
- Browse user profiles and public collections
- Automate content discovery workflows
- Build AI agents that understand Xiaohongshu data

### 🏗️ Architecture

This MCP server uses its own Playwright browser instance (not dependent on `@playwright/mcp`):

```
┌────────────────────────────────┐
│   Your MCP Client              │
│   (Cursor, Claude Desktop,     │
│    Custom Agent, etc.)         │
└───────────────┬────────────────┘
                │ STDIO (JSON-RPC)
                ▼
┌────────────────────────────────┐
│   RedNote MCP Server           │
│  ┌──────────────────────────┐  │
│  │  9 Generic Tools         │  │
│  │  - browse(url)           │  │
│  │  - search(keyword)       │  │
│  │  - click(target)         │  │
│  │  - scroll(), snapshot()  │  │
│  │  - type_text(), etc.     │  │
│  └──────────┬───────────────┘  │
│             │                   │
│  ┌──────────▼───────────────┐  │
│  │  Browser Singleton       │  │
│  │  + Mutex Lock            │  │
│  │  + Anti-Detection        │  │
│  └──────────┬───────────────┘  │
│             │                   │
│             ▼                   │
│       Playwright (Chromium)     │
└────────────────────────────────┘
```

**Key Design:**
- **Singleton Browser** — Shared across all tool calls to avoid cold start
- **Mutex Lock** — Serializes concurrent calls for thread safety
- **Generic Snapshots** — No hardcoded `.note-container` or `.user-profile` selectors
- **Click > Navigate** — Avoids Xiaohongshu's `xsec_token` anti-scraping mechanism

### 📦 Installation

#### Option 1: npm Global Install

```bash
npm install -g rednote-mcp
```

#### Option 2: Local Development

```bash
git clone https://github.com/ifuryst/rednote-mcp.git
cd rednote-mcp/mcp/rednote
npm install
npm run build
npm install -g .
```

### ⚙️ Configuration

#### For Cursor / Claude Desktop

Add to your MCP settings file:

**Windows:** `%APPDATA%\Cursor\User\globalStorage\saoudrizwan.claude-dev\settings\cline_mcp_settings.json`
**macOS:** `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`

```json
{
  "mcpServers": {
    "rednote": {
      "command": "rednote-mcp",
      "args": ["--stdio"]
    }
  }
}
```

Or using npx:

```json
{
  "mcpServers": {
    "rednote": {
      "command": "npx",
      "args": ["rednote-mcp", "--stdio"]
    }
  }
}
```

For local development from source:

```json
{
  "mcpServers": {
    "rednote": {
      "command": "node",
      "args": ["/path/to/rednote-mcp/mcp/rednote/dist/index.js", "--stdio"],
      "cwd": "/path/to/rednote-mcp/mcp/rednote"
    }
  }
}
```

#### For Custom Agent Systems

If your agent has MCP auto-discovery:

1. Copy `mcp/rednote/` to your agent's `mcp/` directory
2. Copy `skills/xiaohongshu/SKILL.md` to your agent's `skills/` directory
3. Agent will automatically detect and load the MCP server

### 🚀 Usage

#### Automatic Login

When you first call any tool, the MCP will automatically:
1. Open a browser window
2. Navigate to Xiaohongshu login page
3. Wait for you to scan the QR code
4. Save cookies to `~/.mcp/rednote/cookies.json`
5. Continue with your request

**Cookie expiration:** Call the `login()` tool to re-authenticate.

#### Available Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `browse(url)` | Navigate to any URL and return page snapshot | `url: string` |
| `search(keyword, type?)` | Search for notes (default) or users | `keyword: string`<br>`type?: "note" \| "user"` |
| `click(target)` | Click element by text/URL/noteId/coordinates | `target: string` |
| `scroll(direction?, amount?)` | Scroll page to load more content | `direction?: "down" \| "up"`<br>`amount?: number` |
| `snapshot()` | Get current page snapshot | — |
| `go_back()` | Browser back button | — |
| `type_text(text, press_enter?)` | Type text into focused input | `text: string`<br>`press_enter?: boolean` |
| `press_key(key)` | Press keyboard key | `key: string` |
| `login()` | Re-authenticate | — |

#### Example Workflows

**Search and read notes:**

```typescript
// Search for coffee-related notes
const searchResult = await mcp.callTool('search', {
  keyword: '咖啡'
})

// Click on a note title from the snapshot
const noteDetail = await mcp.callTool('click', {
  target: '建议收藏‼️30秒看懂咖啡店常见8种咖啡配方'
})

// Scroll to load more comments
const moreComments = await mcp.callTool('scroll', {
  direction: 'down'
})

// Go back
await mcp.callTool('go_back', {})
```

**Browse user profile:**

```typescript
// Search for a user
const userSearch = await mcp.callTool('search', {
  keyword: '用户名',
  type: 'user'
})

// Click on the user's name
const profile = await mcp.callTool('click', {
  target: '用户名'
})
```

#### Snapshot Structure

Every tool returns a structured snapshot:

```
URL: https://www.xiaohongshu.com/...
Title: 页面标题

--- Links (50) ---
[1] 笔记标题  →  https://www.xiaohongshu.com/explore/noteId?xsec_token=...
[2] 用户名  →  https://www.xiaohongshu.com/user/profile/userId
...

--- Buttons ---
<button> 综合
<button> 筛选

--- Inputs ---
<input type="text" placeholder="搜索小红书" value="咖啡">

--- Page Content ---
笔记标题
作者名
2025-01-29
笔记正文...
```

### 🔐 Anti-Scraping Best Practices

Xiaohongshu uses `xsec_token` to prevent direct URL access. The token is injected by JavaScript click events.

#### ❌ Wrong

```typescript
browse({ url: 'https://www.xiaohongshu.com/explore/noteId' })
```

#### ✅ Correct

```typescript
// 1. Search first
search({ keyword: '关键词' })

// 2. Click from search results
click({ target: '笔记标题' })
```

### 📋 FAQ

**Q: Why not use Playwright MCP?**
A: We tested `@playwright/mcp` but it has critical issues with XHS: unreliable cookies, search API failures, and IP blocking.

**Q: Why no CSS selectors?**
A: XHS frequently changes class names. Generic DOM traversal survives page structure changes.

**Q: Does this support headless mode?**
A: No, headful mode is required to avoid detection.

### 📄 License

MIT

### 🤝 Contributing

Contributions welcome! Please open an issue or PR.

---

<a name="中文"></a>

## 中文

> 🤖 **小红书 AI 浏览器自动化工具**
>
> **基于 [iFurySt/RedNote-MCP](https://github.com/iFurySt/RedNote-MCP.git) 修复关键 bug 并重构：**
> - ✅ 修复 xsec_token 导航失败问题
> - ✅ 修复隐藏元素点击失败
> - ✅ 修复登录浏览器崩溃
> - ✅ 修复搜索结果加载时机问题
> - ✅ 用通用快照提取替换硬编码 CSS 选择器
> - ✅ 增强反检测措施

### 📹 演示视频

[![观看演示](https://img.shields.io/badge/观看-演示-red)](https://github.com/user-attachments/assets/2c2f0ae3-89e9-4b61-8406-9e6c40b49977)

---

### ✨ 特性

- **🔧 通用工具** — 9 个通用浏览器操作（浏览、搜索、点击、滚动、快照等）
- **📸 快照驱动** — 返回结构化页面数据，而非硬编码提取
- **🛡️ 反检测** — 内置隐身措施（webdriver 属性隐藏、真实延迟）
- **🔐 自动登录** — 首次使用自动登录，cookie 持久化
- **🧠 AI 友好** — 为 LLM 自由探索和提取信息而设计
- **🚫 无选择器** — 不依赖 CSS 选择器，抗改版

### 🎯 使用场景

- 搜索小红书笔记和用户
- 提取笔记内容、评论、点赞、收藏
- 浏览用户主页和公开收藏
- 自动化内容发现工作流
- 构建理解小红书数据的 AI 代理

### 🏗️ 架构

本 MCP 使用自己的 Playwright 浏览器实例（不依赖 `@playwright/mcp`）：

```
┌────────────────────────────────┐
│   你的 MCP 客户端               │
│   (Cursor, Claude Desktop,     │
│    自定义 Agent 等)             │
└───────────────┬────────────────┘
                │ STDIO (JSON-RPC)
                ▼
┌────────────────────────────────┐
│   RedNote MCP 服务器            │
│  ┌──────────────────────────┐  │
│  │  9 个通用工具             │  │
│  │  - browse(url)           │  │
│  │  - search(keyword)       │  │
│  │  - click(target)         │  │
│  │  - scroll(), snapshot()  │  │
│  │  - type_text() 等        │  │
│  └──────────┬───────────────┘  │
│             │                   │
│  ┌──────────▼───────────────┐  │
│  │  浏览器单例               │  │
│  │  + 互斥锁                │  │
│  │  + 反检测                │  │
│  └──────────┬───────────────┘  │
│             │                   │
│             ▼                   │
│       Playwright (Chromium)     │
└────────────────────────────────┘
```

**核心设计：**
- **单例浏览器** — 所有工具调用共享，避免冷启动
- **互斥锁** — 串行化并发调用，保证线程安全
- **通用快照** — 不依赖 `.note-container`、`.user-profile` 等硬编码选择器
- **点击 > 直接导航** — 避免小红书的 `xsec_token` 反爬机制

### 📦 安装

#### 方式 1：npm 全局安装

```bash
npm install -g rednote-mcp
```

#### 方式 2：本地开发

```bash
git clone https://github.com/ifuryst/rednote-mcp.git
cd rednote-mcp/mcp/rednote
npm install
npm run build
npm install -g .
```

### ⚙️ 配置

#### Cursor / Claude Desktop

添加到 MCP 配置文件：

**Windows:** `%APPDATA%\Cursor\User\globalStorage\saoudrizwan.claude-dev\settings\cline_mcp_settings.json`
**macOS:** `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`

```json
{
  "mcpServers": {
    "rednote": {
      "command": "rednote-mcp",
      "args": ["--stdio"]
    }
  }
}
```

或使用 npx：

```json
{
  "mcpServers": {
    "rednote": {
      "command": "npx",
      "args": ["rednote-mcp", "--stdio"]
    }
  }
}
```

本地开发从源码运行：

```json
{
  "mcpServers": {
    "rednote": {
      "command": "node",
      "args": ["/path/to/rednote-mcp/mcp/rednote/dist/index.js", "--stdio"],
      "cwd": "/path/to/rednote-mcp/mcp/rednote"
    }
  }
}
```

#### 自定义 Agent 系统

如果你的 agent 支持 MCP 自动发现：

1. 复制 `mcp/rednote/` 到你 agent 的 `mcp/` 目录
2. 复制 `skills/xiaohongshu/SKILL.md` 到你 agent 的 `skills/` 目录
3. Agent 会自动检测并加载 MCP 服务器

### 🚀 使用

#### 自动登录

首次调用任何工具时，MCP 会自动：
1. 打开浏览器窗口
2. 导航到小红书登录页
3. 等待你扫码登录
4. 保存 cookie 到 `~/.mcp/rednote/cookies.json`
5. 继续执行你的请求

**Cookie 过期：** 调用 `login()` 工具重新登录。

#### 可用工具

| 工具 | 说明 | 参数 |
|------|------|------|
| `browse(url)` | 导航到任意 URL，返回页面快照 | `url: string` |
| `search(keyword, type?)` | 搜索笔记（默认）或用户 | `keyword: string`<br>`type?: "note" \| "user"` |
| `click(target)` | 通过文本/URL/noteId/坐标点击元素 | `target: string` |
| `scroll(direction?, amount?)` | 滚动页面加载更多内容 | `direction?: "down" \| "up"`<br>`amount?: number` |
| `snapshot()` | 获取当前页面快照 | — |
| `go_back()` | 浏览器后退 | — |
| `type_text(text, press_enter?)` | 在输入框输入文本 | `text: string`<br>`press_enter?: boolean` |
| `press_key(key)` | 按键盘键 | `key: string` |
| `login()` | 重新登录 | — |

#### 使用示例

**搜索并阅读笔记：**

```typescript
// 搜索咖啡相关笔记
const searchResult = await mcp.callTool('search', {
  keyword: '咖啡'
})

// 点击快照中的笔记标题
const noteDetail = await mcp.callTool('click', {
  target: '建议收藏‼️30秒看懂咖啡店常见8种咖啡配方'
})

// 滚动加载更多评论
const moreComments = await mcp.callTool('scroll', {
  direction: 'down'
})

// 返回
await mcp.callTool('go_back', {})
```

**浏览用户主页：**

```typescript
// 搜索用户
const userSearch = await mcp.callTool('search', {
  keyword: '用户名',
  type: 'user'
})

// 点击用户名
const profile = await mcp.callTool('click', {
  target: '用户名'
})
```

#### 快照结构

每个工具返回结构化快照：

```
URL: https://www.xiaohongshu.com/...
Title: 页面标题

--- Links (50) ---
[1] 笔记标题  →  https://www.xiaohongshu.com/explore/noteId?xsec_token=...
[2] 用户名  →  https://www.xiaohongshu.com/user/profile/userId
...

--- Buttons ---
<button> 综合
<button> 筛选

--- Inputs ---
<input type="text" placeholder="搜索小红书" value="咖啡">

--- Page Content ---
笔记标题
作者名
2025-01-29
笔记正文...
```

### 🔐 反爬最佳实践

小红书使用 `xsec_token` 防止直接 URL 访问，token 由 JS 点击事件注入。

#### ❌ 错误

```typescript
browse({ url: 'https://www.xiaohongshu.com/explore/noteId' })
```

#### ✅ 正确

```typescript
// 1. 先搜索
search({ keyword: '关键词' })

// 2. 从搜索结果点击
click({ target: '笔记标题' })
```

**其他最佳实践：**
- **用户搜索：** 使用 `type: "user"` 搜索用户，而非关于该用户的笔记
- **真实延迟：** 内置 0.5-3 秒随机延迟
- **错误恢复：** 每次错误后自动运行 `ensurePageHealthy()` 防止状态损坏
- **IP 限流：** 如果看到"安全限制，IP存在风险"，降低请求频率

### 📋 常见问题

**问：为什么不用 Playwright MCP？**
答：我们测试过 `@playwright/mcp`，但它在小红书上有关键问题：cookie 不稳定、搜索 API 失败、IP 更容易被封。

**问：为什么不用 CSS 选择器？**
答：小红书经常改 class 名。通用 DOM 遍历可以应对页面结构变化。

**问：支持无头模式吗？**
答：不支持，需要有头模式避免被检测。

### 🛠️ 开发

```bash
npm install
npm run build
npm run dev  # 开发模式
```

**调试日志：**

日志位于 `~/.mcp/rednote/logs/`：
- `rednote-YYYY-MM-DD.log` — 每日日志
- `stdio.log` — STDIO 通信日志

查看日志：

```bash
# 打开日志目录
node dist/cli.js open-logs

# 打包日志（用于报告 bug）
node dist/cli.js pack-logs
```

### 📄 许可证

MIT

### 🤝 贡献

欢迎贡献！请提交 issue 或 PR。

### 🔗 链接

- **GitHub 仓库：** `https://github.com/ifuryst/rednote-mcp`
- **原始项目：** [iFurySt/RedNote-MCP](https://github.com/iFurySt/RedNote-MCP.git)

---