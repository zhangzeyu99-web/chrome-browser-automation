---
pattern_id: browser-automation/wait-for-ai-search
skill_source: chrome-browser-automation
version: v1.1.0
date: 2026-03-18
category: browser-automation
---

# 智能等待模式

## 适用场景
AI 搜索类页面需要较长时间处理请求。

## 解决方案

### 方案 1：固定等待（AI 搜索专用）
```powershell
Start-Sleep -Seconds 15  # Kimi/Gemini 需要的时间
```

### 方案 2：渐进式等待
```powershell
# 短等待 + 检查 + 长等待
Start-Sleep -Seconds 5
$snapshot = browser(action: "snapshot", profile: "user")
if ($snapshot -notmatch "结果") {
    Start-Sleep -Seconds 10
}
```

### 方案 3：最大超时保护
```powershell
$maxWait = 30
$elapsed = 0
while ($elapsed -lt $maxWait) {
    $snapshot = browser(action: "snapshot", profile: "user")
    if ($snapshot -match "完成|结果|答案") {
        break
    }
    Start-Sleep -Seconds 1
    $elapsed++
}
```

## 推荐配置

| 场景 | 等待时间 | 说明 |
|------|----------|------|
| 普通网页 | 2-3s | 一般页面加载 |
| 复杂应用 | 5s | SPA 应用 |
| AI 搜索 | 15s | Kimi/Gemini |
| 文件上传 | 10-20s | 取决于文件大小 |

## 注意事项

- 过短等待会导致获取到不完整内容
- 过长等待影响用户体验
- 建议截图验证结果是否完整
