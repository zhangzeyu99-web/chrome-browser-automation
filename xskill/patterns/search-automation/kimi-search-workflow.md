---
pattern_id: search-automation/kimi-search-workflow
skill_source: chrome-browser-automation
version: v1.1.0
date: 2026-03-18
category: search-automation
---

# Kimi 搜索工作流模式

## 适用场景
使用 Kimi (kimi.com) 进行 AI 增强搜索，获取整理后的信息摘要。

## 前提条件
- 用户已登录 Kimi 账号
- Chrome 开启 Remote Debugging
- OpenClaw Gateway 运行中

## 标准流程

```javascript
// 1. 检查浏览器状态
browser(action: "status", profile: "user")

// 2. 打开 Kimi
browser(action: "open", profile: "user", url: "https://kimi.com")

// 3. 等待页面加载
Start-Sleep -Seconds 2

// 4. 获取页面 snapshot
browser(action: "snapshot", profile: "user")

// 5. 点击搜索框
browser(action: "act", profile: "user", request: {kind: "click", ref: "5_41"})

// 6. 输入自然语言搜索问题
browser(action: "act", profile: "user", request: {kind: "type", ref: "5_41", text: "搜索问题"})

// 7. 提交搜索
browser(action: "act", profile: "user", request: {kind: "press", key: "Enter"})

// 8. 等待 AI 处理（关键！Kimi 需要较长时间）
Start-Sleep -Seconds 15

// 9. 截图获取结果
browser(action: "screenshot", profile: "user", fullPage: true)

// 10. 可选：获取页面文本
browser(action: "snapshot", profile: "user")
```

## 关键要点

1. **等待时间**：Kimi 需要 15s 左右进行 AI 搜索和整理
2. **自然语言**：使用自然语言描述需求，而非关键词
3. **Ref 动态性**：搜索框 ref 会变化，必须通过 snapshot 获取
4. **登录状态**：确保用户已登录，否则需要引导登录

## 可复用性

此模式也可用于：
- Gemini Web (gemini.google.com)
- 其他 AI 搜索工具

## 经验教训

- v1.0.0 使用传统搜索引擎，效率较低
- v1.1.0 切换到 Kimi，获取的是 AI 整理的摘要而非原始链接
- 等待时间从 3s 增加到 15s，但结果质量显著提升
