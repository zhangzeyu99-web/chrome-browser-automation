---
name: chrome-browser-automation
version: 1.2.0
description: |
  使用 OpenClaw 的 Chrome Attach 模式自动化操作浏览器。
  
  **搜索能力（优先使用 Kimi/Gemini Web）**:
  - 用户需要搜索信息时，优先使用 Kimi (kimi.com) 或 Gemini Web (gemini.google.com)
  - 相比传统搜索引擎，Kimi/Gemini 会基于 AI 整理搜索结果，更高效
  - 触发词："搜索xx"、"查一下xx"、"用Kimi搜"、"问问Gemini"
  
  **其他触发场景**:
  - "打开xx网站"、"去xx看看"、"访问xx"
  - "截图"、"截个图"、"网页截图"
  - "看看Chrome开了什么"、"浏览器状态"
  - 用户给出任意 URL 让打开
  - 用户需要操作网页：点击、输入、填写表单
  - 用户需要获取网页内容、监控页面变化
  
  **适用场景**: 任何需要浏览器自动化的任务，包括 AI 搜索、访问网站、填写表单、数据抓取、网页监控等。
triggers:
  - 搜索
  - 查一下
  - 看看
  - 打开
  - 访问
  - 截图
  - 浏览器
  - Chrome
  - Kimi
  - Gemini
  - 网页
  - 标签页
  - 点击
  - 输入
  - 填写
  - 表单
  - 网址
  - http
  - https
  - .com
  - .cn
keywords:
  - chrome automation
  - browser control
  - web scraping
  - ai search
  - kimi
  - gemini
---

# Chrome 浏览器自动化

基于 OpenClaw 3.13+ Chrome Attach 模式的浏览器自动化工具。

## 前置要求

1. **Chrome 已安装** 并开启 Remote Debugging
2. **OpenClaw Gateway 运行中** 且配置正确

## 配置检查

每次使用前检查配置：

```json
// ~/.openclaw/openclaw.json
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

## 完整能力清单

### 基础操作

| 能力 | 工具调用 | 说明 |
|------|----------|------|
| 检查状态 | `browser(action: "status", profile: "user")` | 确认 Chrome 连接正常 |
| 列出标签页 | `browser(action: "tabs", profile: "user")` | 查看所有打开的页面 |
| 打开网页 | `browser(action: "open", profile: "user", url: "...")` | 访问指定 URL |
| 页面快照 | `browser(action: "snapshot", profile: "user")` | 获取页面 DOM 结构 |
| 截图 | `browser(action: "screenshot", profile: "user", fullPage: true)` | 截取页面 |
| 激活标签页 | `browser(action: "focus", profile: "user", targetId: "...")` | 切换到指定标签 |
| 关闭标签页 | `browser(action: "close", profile: "user", targetId: "...")` | 关闭指定标签 |

### 元素交互

| 操作 | 工具调用 | 说明 |
|------|----------|------|
| 点击 | `browser(action: "act", profile: "user", request: {kind: "click", ref: "1_2"})` | 点击元素 |
| 输入文字 | `browser(action: "act", profile: "user", request: {kind: "type", ref: "1_3", text: "..."})` | 在输入框打字 |
| 填写表单 | `browser(action: "act", profile: "user", request: {kind: "fill", ref: "1_4", text: "..."})` | 填写表单字段 |
| 按键 | `browser(action: "act", profile: "user", request: {kind: "press", key: "Enter"})` | 模拟键盘按键 |
| 悬停 | `browser(action: "act", profile: "user", request: {kind: "hover", ref: "1_5"})` | 鼠标悬停 |
| 选择 | `browser(action: "act", profile: "user", request: {kind: "select", ref: "1_6", values: ["option1"]})` | 下拉框选择 |

### 高级操作

| 能力 | 工具调用 | 说明 |
|------|----------|------|
| 执行 JS | `browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => document.title"})` | 运行 JavaScript |
| 滚动页面 | `browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => window.scrollTo(0, 500)"})` | 滚动到指定位置 |
| 调整窗口 | `browser(action: "act", profile: "user", request: {kind: "resize", width: 1920, height: 1080})` | 改变窗口大小 |
| 等待加载 | `Start-Sleep -Seconds 3` | 等待页面/元素加载 |

## 标准工作流程

### 流程 1：使用 Kimi/Gemini 搜索（推荐）

**推荐使用 Kimi (kimi.com) 或 Gemini Web 进行搜索**，它们会基于 AI 整理搜索结果，比传统搜索引擎更高效。

```javascript
// 1. 检查浏览器状态
browser(action: "status", profile: "user")

// 2. 打开 Kimi
browser(action: "open", profile: "user", url: "https://kimi.com")

// 3. 等待页面加载（Kimi 需要登录，确保用户已登录）
Start-Sleep -Seconds 2

// 4. 获取页面 snapshot
browser(action: "snapshot", profile: "user")

// 5. 点击搜索/输入框（ref 根据实际页面确定）
browser(action: "act", profile: "user", request: {kind: "click", ref: "5_41"})

// 6. 输入搜索问题（用自然语言描述需求）
browser(action: "act", profile: "user", request: {kind: "type", ref: "5_41", text: "今天有什么重要新闻"})

// 7. 按回车提交
browser(action: "act", profile: "user", request: {kind: "press", key: "Enter"})

// 8. 等待 AI 搜索和整理结果（Kimi 需要较长时间）
Start-Sleep -Seconds 15

// 9. 截图获取结果
browser(action: "screenshot", profile: "user", fullPage: true)

// 10. 可选：获取页面文本内容
browser(action: "snapshot", profile: "user")
```

**Gemini Web 替代方案**：
```javascript
// 使用 Gemini Web
browser(action: "open", profile: "user", url: "https://gemini.google.com")
// 后续步骤类似，根据页面结构调整 ref
```

### 流程 2：传统搜索引擎（备用）

如需使用百度/Bing/Google 等传统搜索引擎：

```javascript
// 1. 检查浏览器状态
browser(action: "status", profile: "user")

// 2. 打开目标网站
browser(action: "open", profile: "user", url: "https://example.com")

// 3. 等待页面加载
Start-Sleep -Seconds 2

// 4. 获取页面 snapshot（找到搜索框 ref）
browser(action: "snapshot", profile: "user")

// 5. 点击搜索框
browser(action: "act", profile: "user", request: {kind: "click", ref: "1_32"})

// 6. 输入搜索词
browser(action: "act", profile: "user", request: {kind: "type", ref: "1_32", text: "搜索内容"})

// 7. 按回车提交
browser(action: "act", profile: "user", request: {kind: "press", key: "Enter"})

// 8. 等待结果
Start-Sleep -Seconds 3

// 9. 截图或获取内容
browser(action: "screenshot", profile: "user", fullPage: true)
browser(action: "snapshot", profile: "user")
```

### 流程 2：直接打开指定 URL

```javascript
// 快速打开任意链接
browser(action: "open", profile: "user", url: "https://user-provided-url.com")
browser(action: "snapshot", profile: "user")
```

### 流程 3：检查浏览器状态

```javascript
// 列出所有标签页
browser(action: "tabs", profile: "user")

// 查看详细状态
browser(action: "status", profile: "user")
```

### 流程 4：表单填写

```javascript
// 获取表单元素 ref
browser(action: "snapshot", profile: "user")

// 填写多个字段
browser(action: "act", profile: "user", request: {kind: "fill", ref: "1_10", text: "用户名"})
browser(action: "act", profile: "user", request: {kind: "fill", ref: "1_11", text: "密码"})
browser(action: "act", profile: "user", request: {kind: "click", ref: "1_12"}) // 提交按钮
```

### 流程 5：多标签页管理

```javascript
// 列出所有标签页
browser(action: "tabs", profile: "user")

// 激活特定标签页（通过 targetId）
browser(action: "focus", profile: "user", targetId: "ABC123...")

// 关闭不需要的标签页
browser(action: "close", profile: "user", targetId: "DEF456...")
```

### 流程 6：JavaScript 执行

```javascript
// 获取页面标题
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => document.title"})

// 滚动到页面底部
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => window.scrollTo(0, document.body.scrollHeight)"})

// 获取页面所有链接
browser(action: "act", profile: "user", request: {kind: "evaluate", fn: "() => Array.from(document.querySelectorAll('a')).map(a => a.href)"})
```

### 流程 7：页面监控（轮询检查）

```javascript
// 监控特定元素变化
$initial = browser(action: "snapshot", profile: "user")
Start-Sleep -Seconds 30
$current = browser(action: "snapshot", profile: "user")

// 比较差异，检测变化
if ($current -ne $initial) {
    // 发送通知或截图
    browser(action: "screenshot", profile: "user", fullPage: true)
}
```

## 故障排查

### Gateway 超时

**症状**: `timed out. Restart the OpenClaw gateway`

**解决**:
```powershell
schtasks /End /TN "OpenClaw"
Start-Sleep -Seconds 3
schtasks /Run /TN "OpenClaw"
```

**预防**: 定期重启 Gateway，建议设置定时任务每周重启一次。

### Chrome 未检测到

**症状**: `Could not connect to Chrome`

**检查**:
1. Chrome 是否开启 Remote Debugging: `chrome://inspect/#remote-debugging`
2. 端口是否与配置一致（默认 9222）
3. OpenClaw 配置中的 `defaultProfile` 是否正确
4. Chrome 是否以管理员权限运行

**启动 Chrome with Remote Debugging**:
```powershell
# Windows
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# Mac
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222
```

### 页面加载超时

**症状**: `Navigation timeout`

**解决**:
```javascript
browser(action: "open", profile: "user", url: "https://example.com", timeoutMs: 30000)
```

**常见原因**:
- 网络连接问题
- 目标网站响应慢
- 页面有重定向循环

### 元素找不到

**症状**: `Element not found` 或 ref 无效

**解决**: 重新获取 snapshot，ref 是动态生成的

**注意**: 
- 每次页面刷新后 ref 都会变化
- 不同网站的 ref 格式不同
- 某些动态加载的元素需要等待后才能获取

### Chrome Attach 被拒绝

**症状**: `Failed to attach to Chrome: Connection refused`

**原因**: Chrome 未开启远程调试或端口被占用

**解决**:
1. 关闭所有 Chrome 实例
2. 使用命令行重新启动 Chrome（见上文）
3. 检查端口 9222 是否被其他程序占用：`netstat -ano | findstr 9222`

### Kimi/Gemini 登录失效

**症状**: AI 搜索页面显示登录框或"请登录"

**解决**:
1. 让用户手动在 Chrome 中登录 Kimi/Gemini
2. 勾选"保持登录状态"
3. 如果 Cookie 过期，需要重新登录

**提示**: 建议用户使用 Chrome 的密码管理器保存登录信息。

### 截图失败或空白

**症状**: 截图返回黑色或空白图片

**原因**:
- Chrome 窗口被最小化
- 页面尚未完全加载
- 显卡驱动问题

**解决**:
1. 确保 Chrome 窗口可见
2. 增加等待时间后再截图
3. 使用 `fullPage: true` 参数获取完整页面

### 操作频繁被限制

**症状**: 网站返回"操作过于频繁"或验证码

**解决**:
1. 增加操作间隔时间
2. 使用 `Start-Sleep` 在操作间添加延迟
3. 避免短时间内大量请求同一网站

## 最佳实践

1. **先检查状态** - 每次使用前确认浏览器连接正常
2. **使用 snapshot 获取 ref** - 元素引用必须通过 snapshot 获取，且每次页面变化后需要重新获取
3. **适当等待** - 操作后给页面加载时间 `Start-Sleep -Seconds 2`
4. **处理登录** - 如需登录，让用户手动操作或确保 Cookie 已保存
5. **截图确认** - 重要操作后截图确认结果
6. **错误处理** - 操作失败时检查状态并重试

## 限制

- 只能操作已开启 Remote Debugging 的 Chrome 实例
- 无法操作无痕模式窗口
- 某些网站可能有反自动化检测
- 需要用户保持 Chrome 运行
- **Kimi/Gemini Web 需要用户已登录** - 首次使用需手动登录，后续复用 Cookie

## 触发词参考

用户可能用各种方式表达相同需求：

| 意图 | 可能的表达 |
|------|-----------|
| **AI 搜索（优先）** | "用 Kimi 搜今天的新闻"、"问问 Gemini"、"查一下 xxx"、"搜索 xxx" |
| 打开网站 | "打开百度"、"去知乎看看"、"访问 GitHub"、"打开 https://..." |
| 传统搜索 | "在百度搜..."、"Google 一下"（备用方案） |
| 截图 | "截图"、"截个图"、"网页截图"、"保存当前页面" |
| 查看状态 | "看看 Chrome 开了什么"、"浏览器状态"、"当前标签页" |
| 操作网页 | "点击登录按钮"、"填写用户名"、"在搜索框输入..." |

---

*基于 OpenClaw 2026.3.13 Chrome Attach 模式*
