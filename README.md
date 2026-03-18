# Chrome Browser Automation

基于 OpenClaw 3.13+ Chrome Attach 模式的浏览器自动化 Skill。

## 功能特性

- ✅ **AI 优先搜索** - 支持 Kimi、Gemini Web 智能搜索
- ✅ **浏览器控制** - 打开网页、点击、输入、截图
- ✅ **无需重复登录** - 复用 Chrome 已有登录态
- ✅ **自然语言操作** - 像人一样描述需求

## 安装

### 前提条件

1. **安装 Chrome** 并开启 Remote Debugging
2. **配置 OpenClaw**

### 安装步骤

```bash
# 克隆仓库
git clone https://github.com/zhangzeyu99-web/chrome-browser-automation.git

# 复制到 OpenClaw skills 目录
cp -r chrome-browser-automation ~/.openclaw/workspace/skills/

# 重启 OpenClaw Gateway
openclaw gateway restart
```

### 配置 Chrome

1. 打开 Chrome，访问 `chrome://inspect/#remote-debugging`
2. 勾选 "Allow remote debugging for this browser instance"
3. 确认看到 `Server running at: 127.0.0.1:xxxx`

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

## 使用示例

### 搜索今天的新闻

```
用户：用 Kimi 搜一下今天的新闻
AI：打开 Kimi → 输入搜索 → 返回结果截图
```

### 截图当前页面

```
用户：截图
AI：截取当前浏览器页面
```

### 打开指定网站

```
用户：打开 GitHub
AI：打开 https://github.com
```

## 目录结构

```
chrome-browser-automation/
├── SKILL.md              # 技能定义
├── evals/                # 测试用例
│   └── evals.json
├── xskill/               # 经验模式
│   └── patterns/
├── auto-skill/           # 自动化配置
│   └── regression-tests.json
├── CHANGELOG.md          # 变更日志
└── README.md             # 本文件
```

## 测试

```bash
# 运行测试
openclaw skill test chrome-browser-automation
```

## 版本历史

- **v1.2.0** (2026-03-18) - 改进触发词、故障排查、使用案例
- **v1.1.0** (2026-03-18) - AI 搜索优先，支持 Kimi/Gemini
- **v1.0.0** (2026-03-18) - 初始版本，基础浏览器控制

## 贡献

欢迎提交 Issue 和 PR！

## License

MIT
