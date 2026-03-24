# Chrome Browser Automation

基于 OpenClaw Chrome Attach 模式的浏览器自动化 Skill。

## 功能特性

- **截图核心能力** - `browser(action: "screenshot")` 是系统中网页截图的唯一入口
- **AI 优先搜索** - 支持 Kimi、Gemini Web 智能搜索
- **浏览器控制** - 打开网页、点击、输入、截图、JS 执行
- **智能启动** - OpenCLI 自动检测并启动 Chrome，无需手动配置
- **无需重复登录** - 复用 Chrome 已有登录态
- **多层降级** - Gateway → agent-browser CLI → Playwright MCP

## v2.0.0 新特性

- **截图独占声明**：明确截图为本技能核心能力
- **降级方案**：集成 agent-browser CLI 和 Playwright MCP 作为 Gateway 不可用时的备选
- **传统搜索引擎**：新增 Bing/Google 工作流
- **JS 执行示例**：滚动、获取链接、获取标题等常用场景
- **限制列表**：明确 5 条结构化限制
- **精简 63%**：从 488 行精简至 181 行，去除冗余代码示例

## 安装

### 一键安装（推荐）

```bash
# 克隆到 OpenClaw skills 目录
cd ~/.openclaw/workspace/skills/
git clone https://github.com/zhangzeyu99-web/chrome-browser-automation.git

# 重启 Gateway
openclaw gateway restart
```

### 前置条件

1. **Chrome 已安装**
2. **OpenCLI 已安装**（用于智能启动）：`npm install -g @jackwener/opencli`
3. **OpenClaw Gateway 运行中**

### 配置 OpenClaw

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "browser": {
    "enabled": true,
    "defaultProfile": "user",
    "profiles": {
      "user": {
        "driver": "existing-session",
        "attachOnly": true
      }
    }
  }
}
```

### 启动 Chrome

```bash
# 方式 1：OpenCLI 自动启动（推荐）
opencli doctor --live

# 方式 2：手动启动
# Windows:
# & "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
# macOS:
# /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
# Linux:
# google-chrome --remote-debugging-port=9222
```

### 验证

```javascript
browser(action: "status", profile: "user")
```

## 使用示例

### 截图当前页面

```javascript
browser(action: "screenshot", profile: "user", fullPage: true)
```

### 用 Kimi 搜索

```javascript
browser(action: "open", profile: "user", url: "https://kimi.com")
// → snapshot → click 输入框 → type 问题 → press Enter → sleep 15s → screenshot
```

### 表单填写

```javascript
browser(action: "snapshot", profile: "user")
browser(action: "act", profile: "user", request: {kind: "fill", ref: "1_10", text: "用户名"})
browser(action: "act", profile: "user", request: {kind: "fill", ref: "1_11", text: "密码"})
browser(action: "act", profile: "user", request: {kind: "click", ref: "1_12"})
```

## 与 opencli-natural-commands 的协调

本技能与 [opencli-natural-commands](https://github.com/zhangzeyu99-web/opencli-natural-commands) 共享同一个 Chrome CDP 端口 (9222)，职责分工：

| 任务 | 使用技能 |
|------|---------|
| 截图/AI搜索(Kimi/Gemini)/打开网页/表单 | **chrome-browser-automation** (本技能) |
| B站/YouTube/Cursor/爬虫 | **opencli-natural-commands** |

两者不得同时操作 Chrome，串行执行即可。

## 目录结构

```
chrome-browser-automation/
├── SKILL.md                    # 技能定义（主文档）
├── references/
│   └── opencli-integration.md  # OpenCLI 集成指南
├── evals/                      # 测试用例
├── xskill/                     # 经验模式
├── auto-skill/                 # 自动化配置
├── CHANGELOG.md
└── README.md
```

## 故障排查

| 症状 | 解决 |
|------|------|
| Gateway 超时 | 重启 Gateway 服务 |
| Chrome 连不上 | 确认以 `--remote-debugging-port=9222` 启动 |
| ref 无效 | 重新 snapshot |
| 截图空白 | 确保 Chrome 窗口可见 |

详见 [SKILL.md](SKILL.md) 完整故障排查指南。

## License

MIT
