# OpenCLI 浏览器集成指南

本文档详细说明 OpenCLI 与 `browser` 工具的分工协作，以及降级策略。

---

## 分工说明

| 工具 | 职责 | 优势 |
|------|------|------|
| **OpenCLI** (`opencli doctor --live`) | 环境检测、Chrome 启动、系统配置 | 自动检测、一键启动、处理符号链接 |
| **browser 工具** | 页面操作、元素交互、截图、执行 JS | 精细控制、与 Chrome DevTools Protocol 直接通信 |

### 工作流程

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   OpenCLI 启动   │ --> │  Chrome 就绪    │ --> │  browser 工具   │
│  doctor --live   │     │  port 9222      │     │  执行操作       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

1. **OpenCLI 阶段**：确保 Chrome 在正确的模式下运行
2. **browser 阶段**：执行实际的浏览器自动化任务

---

## 使用示例

### 基础模式

```javascript
// 1. 自动检测并启动 Chrome
exec("opencli doctor --live")

// 2. 等待 Chrome 就绪
Start-Sleep -Seconds 3

// 3. 使用 browser 工具操作
browser(action: "status", profile: "user")
browser(action: "open", profile: "user", url: "https://kimi.com")
```

### 完整搜索流程

```javascript
// 智能启动
exec("opencli doctor --live")
Start-Sleep -Seconds 3

// 检查状态
$status = browser(action: "status", profile: "user")

// 打开 Kimi
browser(action: "open", profile: "user", url: "https://kimi.com")
Start-Sleep -Seconds 2

// 获取页面结构
$snapshot = browser(action: "snapshot", profile: "user")

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

---

## 降级策略

当 OpenCLI 不可用时，自动回退到传统启动方式：

### 检测 OpenCLI 可用性

```javascript
// 尝试执行 OpenCLI
try {
    exec("opencli doctor --live")
    Start-Sleep -Seconds 3
} catch {
    // OpenCLI 失败，使用传统方式
    Write-Host "OpenCLI 不可用，切换到传统启动方式..."
    
    // 传统 PowerShell 启动脚本
    taskkill /F /IM chrome.exe 2>$null
    Start-Sleep -Seconds 3
    
    $debugDir = "C:\Users\Administrator\ChromeDebug"
    New-Item -ItemType Directory -Force -Path $debugDir | Out-Null
    Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" `
        -ArgumentList "--remote-debugging-port=9222", "--user-data-dir=$debugDir"
    
    Start-Sleep -Seconds 4
    
    // 创建符号链接
    $systemDir = "C:\Windows\system32\config\systemprofile\AppData\Local\Google\Chrome\User Data"
    cmd /c "rmdir /s /q `"$systemDir`"" 2>$null
    cmd /c "mklink /J `"$systemDir`" `"$debugDir`""
    
    // 验证端口
    Invoke-RestMethod -Uri "http://127.0.0.1:9222/json" -TimeoutSec 5
}

// 无论哪种方式，最后都使用 browser 工具
browser(action: "status", profile: "user")
```

### 降级触发条件

| 条件 | 说明 |
|------|------|
| `opencli: command not found` | OpenCLI 未安装 |
| 命令超时 | OpenCLI 无响应 |
| 返回非零退出码 | OpenCLI 执行失败 |
| Chrome 启动失败 | OpenCLI 无法启动 Chrome |

---

## 故障排查对照表

### OpenCLI 相关问题

| 症状 | 可能原因 | 解决方案 |
|------|----------|----------|
| `opencli: command not found` | OpenCLI 未安装或未添加到 PATH | 安装 OpenCLI 或检查环境变量 |
| `doctor --live` 卡住 | Chrome 被其他程序占用 | 手动关闭所有 Chrome 实例后重试 |
| 符号链接创建失败 | 权限不足 | 以管理员身份运行 |
| Chrome 启动但无法连接 | 端口被占用 | 检查 9222 端口占用情况 |

### browser 工具相关问题

| 症状 | 可能原因 | 解决方案 |
|------|----------|----------|
| `Could not connect to Chrome` | Chrome 未在调试模式运行 | 重新执行 OpenCLI 或手动启动 |
| `Navigation timeout` | 网络问题或网站响应慢 | 增加 timeoutMs 参数 |
| `Element not found` | ref 已过期或页面未加载 | 重新获取 snapshot |
| 截图空白 | Chrome 窗口被最小化 | 确保 Chrome 窗口可见 |

### 系统集成问题

| 症状 | 可能原因 | 解决方案 |
|------|----------|----------|
| Gateway 超时 | OpenClaw Gateway 未运行 | 重启 Gateway 服务 |
| 权限拒绝 | SYSTEM 用户无法访问 Chrome | 确保符号链接正确创建 |
| 配置错误 | openclaw.json 配置不当 | 检查 browser profiles 配置 |

---

## 最佳实践

1. **始终先尝试 OpenCLI** - 它是更简洁、可靠的启动方式
2. **处理异常情况** - 代码中应包含降级逻辑
3. **验证连接状态** - 使用 `browser(action: "status")` 确认就绪
4. **合理等待** - 给 Chrome 启动和页面加载预留足够时间
5. **保持幂等** - 多次执行不应产生副作用

---

## 参考链接

- 主文档：`SKILL.md`
- OpenCLI 文档：https://opencli.dev
- Chrome DevTools Protocol：https://chromedevtools.github.io/devtools-protocol/
