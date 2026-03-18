---
anti_pattern_id: insufficient-wait-time
detected_in: chrome-browser-automation v1.0.0
severity: medium
status: fixed
---

# 反面模式：等待时间不足

## 问题描述
页面操作后没有给予足够的加载时间，导致获取到不完整或过期的页面状态。

## 错误示例
```javascript
// ❌ 错误：等待时间不足
browser(action: "open", profile: "user", url: "https://kimi.com")
Start-Sleep -Seconds 1  // 太短了！
browser(action: "snapshot", profile: "user")  // 可能页面还没加载完
```

## 后果
1. **Snapshot 不完整** - 获取不到页面元素
2. **操作失败** - 点击未加载的元素会报错
3. **错误结果** - 搜索结果显示"加载中"

## 正确做法
```javascript
// ✅ 正确：根据场景设置合适等待时间
browser(action: "open", profile: "user", url: "https://kimi.com")
Start-Sleep -Seconds 2  // 普通页面 2s
// 或 AI 搜索专用
Start-Sleep -Seconds 15  // Kimi 需要 15s 处理
```

## 修复记录
- v1.0.0: 使用统一 2s 等待，AI 搜索失败
- v1.1.0: 根据场景设置不同等待时间，AI 搜索专用 15s
