---
name: chrome-browser-automation
version: 2.0.0
description: |
  Chrome 浏览器自动化 — 截图、AI搜索、网页操作的唯一入口。
  
  **核心独占能力**: 网页截图（`browser(action: "screenshot")`）只能通过本技能完成。
  
  **触发场景**:
  - 截图/截个图/网页截图/保存页面
  - 搜索/查一下/用Kimi搜/问问Gemini
  - 打开网站/去xx看看/访问xx/任意URL
  - 点击/输入/填写/表单操作
  - 看看Chrome开了什么/浏览器状态/标签页
  - 网页监控/页面变化检测
  
  **不处理**: B站/bilibili/Cursor控制/爬虫 → 使用 opencli-natural-commands
triggers: [截图, 搜索, 查一下, 打开, 访问, 浏览器, Chrome, Kimi, Gemini, 网页, 标签页, 点击, 输入, 填写, 表单, http, https]
keywords: [chrome automation, browser control, screenshot, ai search, kimi, gemini]
---

# Chrome 浏览器自动化

基于 OpenClaw Chrome Attach 模式。**截图是本技能的核心独占能力。**

## 前置要求

1. Chrome 已安装并以 `--remote-debugging-port=9222` 启动
2. OpenClaw Gateway 运行中

## 配置检查

确认 `~/.openclaw/openclaw.json` 中浏览器配置正确：

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

## 启动 Chrome

```javascript
exec("opencli doctor --live")   // 自动检测+启动 Chrome 调试模式
Start-Sleep -Seconds 3
browser(action: "status", profile: "user")
```

如果 opencli 不可用，手动启动（含 SYSTEM 用户符号链接）：
```powershell
taskkill /F /IM chrome.exe 2>$null
Start-Sleep -Seconds 3
$debugDir = "$env:USERPROFILE\ChromeDebug"
New-Item -ItemType Directory -Force -Path $debugDir | Out-Null
Start-Process chrome.exe -ArgumentList "--remote-debugging-port=9222", "--user-data-dir=$debugDir"
Start-Sleep -Seconds 4
# SYSTEM 用户符号链接（Windows 服务场景需要）
$systemDir = "C:\Windows\system32\config\systemprofile\AppData\Local\Google\Chrome\User Data"
cmd /c "rmdir /s /q `"$systemDir`"" 2>$null
cmd /c "mklink /J `"$systemDir`" `"$debugDir`""
Invoke-RestMethod -Uri "http://127.0.0.1:9222/json" -TimeoutSec 5
```

## 能力速查

### 基础操作

| 能力 | 调用 |
|------|------|
| 检查状态 | `browser(action: "status", profile: "user")` |
| 列出标签页 | `browser(action: "tabs", profile: "user")` |
| 打开网页 | `browser(action: "open", profile: "user", url: "...")` |
| 页面快照 | `browser(action: "snapshot", profile: "user")` |
| **截图** | `browser(action: "screenshot", profile: "user", fullPage: true)` |
| 激活标签页 | `browser(action: "focus", profile: "user", targetId: "...")` |
| 关闭标签页 | `browser(action: "close", profile: "user", targetId: "...")` |

### 元素交互

| 操作 | 调用 |
|------|------|
| 点击 | `browser(action: "act", profile: "user", request: {kind: "click", ref: "REF"})` |
| 输入 | `browser(action: "act", profile: "user", request: {kind: "type", ref: "REF", text: "..."})` |
| 填写 | `browser(action: "act", profile: "user", request: {kind: "fill", ref: "REF", text: "..."})` |
| 按键 | `browser(action: "act", profile: "user", request: {kind: "press", key: "Enter"})` |
| 悬停 | `browser(action: "act", profile: "user", request: {kind: "hover", ref: "REF"})` |
| 选择 | `browser(action: "act", profile: "user", request: {kind: "select", ref: "REF", values: ["opt"]})` |
| 执行JS | `browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => ..."})` |

## 典型工作流

### AI 搜索（Kimi/Gemini）

```
browser(open) → kimi.com → snapshot → click 输入框 → type 问题 → press Enter → sleep 15s → screenshot
```

### 打开URL + 截图

```
browser(open) → URL → sleep 2s → screenshot(fullPage: true)
```

### 表单填写

```
browser(open) → URL → snapshot → fill 字段1 → fill 字段2 → click 提交
```

### 传统搜索引擎（Bing/Google）

```
browser(open) → 搜索引擎URL → snapshot → click 搜索框 → type 关键词 → press Enter → sleep 3s → screenshot
```

### JavaScript 执行示例

```javascript
// 获取页面标题
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => document.title"})

// 滚动到页面底部
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => window.scrollTo(0, document.body.scrollHeight)"})

// 获取页面所有链接
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => Array.from(document.querySelectorAll('a')).map(a => a.href)"})
```

### 页面监控

```
snapshot(初始) → sleep N → snapshot(当前) → 比较差异 → screenshot 如有变化
```

## 降级方案

当 Gateway 不可用时，可用以下独立工具完成部分操作：

### agent-browser CLI（需 `npm install -g agent-browser`）

```bash
agent-browser open <url>          # 导航
agent-browser snapshot -i         # 获取可交互元素（ref 为 @e1 格式）
agent-browser click @e1           # 点击
agent-browser fill @e2 "text"     # 填写
agent-browser screenshot path.png # 截图
agent-browser close               # 关闭
```

### Playwright MCP（需 `npx @playwright/mcp`）

| MCP 工具 | 说明 |
|----------|------|
| `browser_navigate` | 导航到 URL |
| `browser_click` | 点击元素 |
| `browser_type` | 输入文字 |
| `browser_snapshot` | 获取页面结构 |
| `browser_evaluate` | 执行 JavaScript |
| `browser_get_text` | 获取文本内容 |

## 故障速查

| 症状 | 解决 |
|------|------|
| Gateway 超时 | 重启 Gateway 服务 |
| Chrome 连不上 | 确认 Chrome 以 `--remote-debugging-port=9222` 启动；确认符号链接已创建 |
| ref 无效 | 重新 snapshot，ref 每次页面变化后会更新 |
| 截图空白 | 确保 Chrome 窗口可见且页面已加载，增加等待时间后再截图 |
| 登录失效 | 让用户在 Chrome 中手动重新登录 Kimi/Gemini，勾选保持登录 |
| 端口被占 | `netstat -ano | findstr 9222`，kill 占用进程 |
| 操作过频被限 | 增加操作间隔 `Start-Sleep`，避免短时间大量请求同一网站 |
| 页面加载超时 | `browser(action: "open", url: "...", timeoutMs: 30000)` 增加超时 |

## 最佳实践

1. 每次操作前先 `browser(status)` 确认连接
2. snapshot 获取 ref 后再交互，ref 是动态的
3. 操作后 `Start-Sleep -Seconds 2` 等页面加载
4. 重要操作后截图确认结果

## 限制

- 只能操作已开启 Remote Debugging 的 Chrome 实例
- 无法操作无痕模式窗口
- 某些网站有反自动化检测，可能需要增加等待和随机延迟
- 需要用户保持 Chrome 运行
- Kimi/Gemini Web 需要用户已登录，首次使用需手动登录，后续复用 Cookie

## 参考

- [OpenCLI 集成指南](references/opencli-integration.md)
