# AI Assistant OPS Wiki

> 你的AI助手运维知识库

---

## 📋 这是什么？

这是一个关于 **AI助手运维** 的知识库，主要包含：

- OpenClaw 部署教程
- 常见问题解决方案
- 自动化运维技巧
- 远程维护指南

## 🎯 适合谁？

- 部署了 OpenClaw/小安/牛牛 的用户
- 需要远程维护 AI 助手的技术爱好者
- 想了解 AI Agent 运维的朋友

## 📚 内容目录

### 部署教程

| 文档 | 说明 |
|------|------|
| [飞牛NAS部署OpenClaw](./README-NAS-DEPLOY.md) | 在飞牛NAS上部署OpenClaw |
| [Windows部署指南](./README-WINDOWS-MAINTENANCE.md) | Windows部署 + 远程维护 |
| [WSL Ubuntu部署](./WSL-Ubuntu-OpenClaw-安装指南.md) | WSL环境部署 |

### 运维指南

| 文档 | 说明 |
|------|------|
| [远程维护指南](./README-WINDOWS-MAINTENANCE.md) | Tailscale + SSH 远程维护 |
| [常用命令](./README-COMMANDS.md) | OpenClaw 常用命令 |
| [技能安装](./README-SKILLS-REPORT.md) | 技能安装与管理 |

### 问题解决

## 🔧 常见问题快速解决

### Q1: 飞书连接失败

```bash
# 检查状态
openclaw status

# 重启Gateway
openclaw gateway restart

# 检查日志
openclaw logs
```

### Q2: 模型调用失败

```bash
# 检查模型配置
openclaw models status

# 检查API Key
openclaw config get models
```

### Q3: 内存/性能问题

```bash
# 压缩上下文
/com pact

# 检查Token使用
/openclaw status
```

### Q4: 无法远程访问

1. 检查 Tailscale 状态：`tailscale status`
2. 检查 SSH 服务：`Get-Service sshd`
3. 检查防火墙：`Get-NetFirewallRule -Name *ssh*`

### Q5: 开机自启失败

```bash
# 检查定时任务
crontab -l

# 或 Windows
schtasks /query /tn "OpenClaw Gateway"
```

## 🚀 快速部署

### Ubuntu/NAS

```bash
# 安装
npm i -g openclaw

# 配置
openclaw configure

# 启动
openclaw gateway run
```

### Windows

参考 [Windows维护指南](./README-WINDOWS-MAINTENANCE.md)

## 🤝 贡献

欢迎提交 Issue 和 PR！

## 📄 License

MIT

---

*持续更新中...*
