# Chrome Browser Automation

基于 OpenClaw 3.13+ Chrome Attach 模式的浏览器自动化 Skill。

## 功能特性

- ✅ **AI 优先搜索** - 支持 Kimi、Gemini Web 智能搜索
- ✅ **浏览器控制** - 打开网页、点击、输入、截图
- ✅ **智能启动** - OpenCLI 自动检测并启动 Chrome，无需手动配置
- ✅ **无需重复登录** - 复用 Chrome 已有登录态
- ✅ **自然语言操作** - 像人一样描述需求

## 安装

### 前提条件

1. **安装 Chrome**
2. **安装 OpenCLI**（用于智能启动）
   ```bash
   npm install -g @jackwener/opencli
   ```
3. **配置 OpenClaw**

### 安装步骤

```bash
# 克隆仓库
git clone https://github.com/zhangzeyu99-web/chrome-browser-automation.git

# 复制到 OpenClaw skills 目录
cp -r chrome-browser-automation ~/.openclaw/workspace/skills/

# 重启 OpenClaw Gateway
openclaw gateway restart
```

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

## 快速开始（智能启动模式）

### 方式 1：OpenCLI 自动启动（推荐）

```javascript
// 1. 自动检测并启动 Chrome
exec("opencli doctor --live")

// 2. 等待就绪
Start-Sleep -Seconds 3

// 3. 直接使用 browser 工具
browser(action: "open", profile: "user", url: "https://kimi.com")
```

**优势**：
- ✅ 自动检测 Chrome 安装位置
- ✅ 自动启动 Remote Debugging 模式
- ✅ 自动处理 SYSTEM 用户符号链接
- ✅ 无需手动 PowerShell 脚本

### 方式 2：传统手动启动

如 OpenCLI 不可用，使用传统方式：

```powershell
# 关闭所有 Chrome
taskkill /F /IM chrome.exe 2>$null
Start-Sleep -Seconds 3

# 用独立数据目录启动 Chrome（调试模式）
$debugDir = "C:\Users\Administrator\ChromeDebug"
New-Item -ItemType Directory -Force -Path $debugDir | Out-Null
Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" `
    -ArgumentList "--remote-debugging-port=9222", "--user-data-dir=$debugDir"

# 等待启动
Start-Sleep -Seconds 4

# 更新 SYSTEM 用户目录的符号链接
$systemDir = "C:\Windows\system32\config\systemprofile\AppData\Local\Google\Chrome\User Data"
cmd /c "rmdir /s /q `"$systemDir`"" 2>$null
cmd /c "mklink /J `"$systemDir`" `"$debugDir`""

# 验证连接
browser(action: "status", profile: "user")
```

## 使用示例

### 搜索今天的新闻

```javascript
// 智能启动
exec("opencli doctor --live")
Start-Sleep -Seconds 3

// 打开 Kimi
browser(action: "open", profile: "user", url: "https://kimi.com")
Start-Sleep -Seconds 2

// 获取页面结构
browser(action: "snapshot", profile: "user")

// 点击搜索框（根据实际 ref 调整）
browser(action: "act", profile: "user", request: {
    kind: "click",
    ref: "5_41"
})

// 输入搜索内容
browser(action: "act", profile: "user", request: {
    kind: "type",
    ref: "5_41",
    text: "今天的重要新闻"
})

// 提交搜索
browser(action: "act", profile: "user", request: {
    kind: "press",
    key: "Enter"
})

// 等待结果
Start-Sleep -Seconds 10

// 截图保存结果
browser(action: "screenshot", profile: "user", fullPage: true)
```

### 截图当前页面

```javascript
exec("opencli doctor --live")
Start-Sleep -Seconds 3
browser(action: "screenshot", profile: "user", fullPage: true)
```

### 打开指定网站

```javascript
exec("opencli doctor --live")
Start-Sleep -Seconds 3
browser(action: "open", profile: "user", url: "https://github.com")
```

### 表单填写

```javascript
exec("opencli doctor --live")
Start-Sleep -Seconds 3

// 获取表单元素 ref
browser(action: "snapshot", profile: "user")

// 填写多个字段
browser(action: "act", profile: "user", request: {
    kind: "fill",
    ref: "1_10",
    text: "用户名"
})
browser(action: "act", profile: "user", request: {
    kind: "fill",
    ref: "1_11",
    text: "密码"
})
browser(action: "act", profile: "user", request: {
    kind: "click",
    ref: "1_12"  // 提交按钮
})
```

## 目录结构

```
chrome-browser-automation/
├── SKILL.md                    # 技能定义（主要文档）
├── references/
│   └── opencli-integration.md  # OpenCLI 集成指南
├── evals/                      # 测试用例
│   └── evals.json
├── xskill/                     # 经验模式
│   └── patterns/
├── auto-skill/                 # 自动化配置
│   └── regression-tests.json
├── CHANGELOG.md                # 变更日志
└── README.md                   # 本文件
```

## 完整能力清单

### 基础操作

| 能力 | 工具调用 |
|------|----------|
| 检查状态 | `browser(action: "status", profile: "user")` |
| 列出标签页 | `browser(action: "tabs", profile: "user")` |
| 打开网页 | `browser(action: "open", profile: "user", url: "...")` |
| 页面快照 | `browser(action: "snapshot", profile: "user")` |
| 截图 | `browser(action: "screenshot", profile: "user", fullPage: true)` |

### 元素交互

| 操作 | 工具调用 |
|------|----------|
| 点击 | `browser(action: "act", request: {kind: "click", ref: "1_2"})` |
| 输入文字 | `browser(action: "act", request: {kind: "type", ref: "1_3", text: "..."})` |
| 填写表单 | `browser(action: "act", request: {kind: "fill", ref: "1_4", text: "..."})` |
| 按键 | `browser(action: "act", request: {kind: "press", key: "Enter"})` |

### 高级操作

| 能力 | 工具调用 |
|------|----------|
| 执行 JS | `browser(action: "act", request: {kind: "evaluate", fn: "() => document.title"})` |
| 滚动页面 | `browser(action: "act", request: {kind: "evaluate", fn: "() => window.scrollTo(0, 500)"})` |
| 调整窗口 | `browser(action: "act", request: {kind: "resize", width: 1920, height: 1080})` |

## 故障排查

### OpenCLI 相关问题

| 症状 | 解决方案 |
|------|----------|
| `opencli: command not found` | 安装 OpenCLI: `npm install -g @jackwener/opencli` |
| `doctor --live` 卡住 | 手动关闭所有 Chrome 实例后重试 |

### browser 工具相关问题

| 症状 | 解决方案 |
|------|----------|
| `Could not connect to Chrome` | 重新执行 OpenCLI 或手动启动 |
| `Navigation timeout` | 增加 timeoutMs 参数 |
| `Element not found` | 重新获取 snapshot，ref 是动态生成的 |

详见 [SKILL.md](SKILL.md) 完整故障排查指南。

## 版本历史

- **v1.3.0** (2026-03-23) - 🚀 **新增 OpenCLI 智能启动模式**，简化启动流程
- **v1.2.0** (2026-03-18) - 改进触发词、故障排查、使用案例
- **v1.1.0** (2026-03-18) - AI 搜索优先，支持 Kimi/Gemini
- **v1.0.0** (2026-03-18) - 初始版本，基础浏览器控制

## 贡献

欢迎提交 Issue 和 PR！

## License

MIT
