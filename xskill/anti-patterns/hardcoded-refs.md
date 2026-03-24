---
anti_pattern_id: hardcoded-refs
detected_in: chrome-browser-automation v1.0.0
severity: high
status: fixed
---

# 反面模式：硬编码元素 Ref

## 问题描述
在代码中直接硬编码元素的 ref ID（如 `ref=1_32`），而不是通过 snapshot 动态获取。

## 错误示例
```javascript
// ❌ 错误：硬编码 ref
browser(action: "act", profile: "user", request: {kind: "click", ref: "1_32"})

// 假设所有页面的搜索框都是 1_32 - 这是错误的！
```

## 为什么这是问题
1. **Ref 是动态生成的** - 每次页面加载都不同
2. **不同页面结构不同** - Bing 搜索框是 `4_23`，Kimi 是 `5_41`
3. **页面更新会改变 ref** - 网站改版后 ref 会变化

## 正确做法
```javascript
// ✅ 正确：先获取 snapshot
$snapshot = browser(action: "snapshot", profile: "user")
// 从 snapshot 中找到正确的 ref
// 例如：textbox [ref=5_41]
browser(action: "act", profile: "user", request: {kind: "click", ref: "5_41"})
```

## 修复记录
- v1.0.0: 早期示例中使用了硬编码 ref
- v1.1.0: 修正所有示例，强调动态获取 ref
