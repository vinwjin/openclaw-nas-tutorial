# AI Assistant OPS Wiki

> 你的AI助手运维知识库 | OpenClaw部署与维护指南

---

## 📋 这是什么？

这是一个关于 **AI助手运维** 的知识库，记录了：

- OpenClaw 在各种平台（飞牛NAS、Windows，WSL）上的部署教程
- 常见问题的解决方案
- 自动化运维技巧和脚本
- 远程维护指南（Tailscale + SSH）
- 最佳实践和经验总结

## 🎯 适合谁？

- 部署了 OpenClaw/小安/牛牛 的用户
- 需要远程维护 AI 助手的技术爱好者
- 想学习 AI Agent 运维的朋友
- 希望搭建自己AI助手的朋友

## 📚 内容导航

### 🖥️ 平台部署

| 文档 | 平台 | 说明 |
|------|------|------|
| [飞牛NAS部署](./README-NAS-DEPLOY.md) | 飞牛NAS | fnOS上通过npm部署OpenClaw |
| [Windows部署维护](./README-WINDOWS-MAINTENANCE.md) | Windows 11 | Windows电脑部署+远程维护 |
| [WSL Ubuntu部署](./WSL-Ubuntu-OpenClaw-安装指南.md) | WSL | Windows子系统上运行 |

### 🔧 运维指南

| 文档 | 说明 |
|------|------|
| [远程维护指南](./README-WINDOWS-MAINTENANCE.md) | Tailscale内网穿透 + SSH远程连接 |
| [常用命令](./README-COMMANDS.md) | OpenClaw CLI 常用命令大全 |
| [技能管理](./README-SKILLS-REPORT.md) | 安装和管理Agent技能 |
| [SSH隧道](./README-SSH-TUNNEL.md) | 远程端口转发方案 |

### ❓ 问题解决

遇到问题先看这里：

#### 1. 飞书连接失败
```bash
# 检查状态
openclaw status

# 查看详细日志
openclaw logs --follow

# 重启Gateway
openclaw gateway restart
```

#### 2. 模型调用失败
```bash
# 检查模型配置
openclaw models status

# 检查API Key
openclaw config get models.providers

# 测试API连通性
curl https://api.minimaxi.com -v
```

#### 3. 内存/Token爆满
```bash
# 手动压缩上下文
/compact

# 查看当前Token使用
/openclaw status

# 开启新会话
/new
```

#### 4. SSH无法远程连接 (Windows)
```powershell
# 检查SSH服务状态
Get-Service sshd

# 启动SSH服务
Start-Service sshd

# 检查防火墙
Get-NetFirewallRule -Name *ssh*
```

#### 5. 开机自启失败 (Linux/NAS)
```bash
# 检查定时任务
crontab -l

# 添加开机任务
crontab -e
# 添加: @reboot /path/to/openclaw gateway run
```

#### 6. Tailscale无法连接
```bash
# 检查Tailscale状态
tailscale status

# 重新登录
tailscale logout
tailscale login

# 检查网络
ping 100.x.x.x
```

## 🚀 快速开始

### 方式一：飞牛NAS部署

详细教程见 [飞牛NAS部署](./README-NAS-DEPLOY.md)

```bash
# 1. SSH登录NAS
ssh user@nas-ip

# 2. 安装Node.js 22
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 22

# 3. 安装OpenClaw
npm i -g openclaw

# 4. 配置并启动
openclaw configure
openclaw gateway run
```

### 方式二：Windows部署

详细教程见 [Windows维护指南](./README-WINDOWS-MAINTENANCE.md)

```powershell
# 1. 安装OpenSSH (管理员PowerShell)
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd

# 2. 安装OpenClaw
npm i -g openclaw

# 3. 配置飞书机器人并启动
openclaw gateway run
```

## 🔧 常用命令速查

| 命令 | 说明 |
|------|------|
| `openclaw status` | 查看运行状态 |
| `openclaw health` | 健康检查 |
| `openclaw logs` | 查看日志 |
| `openclaw gateway restart` | 重启Gateway |
| `openclaw security audit` | 安全审计 |
| `openclaw models list` | 列出可用模型 |
| `/compact` | 压缩上下文 |
| `/new` | 开始新会话 |

## 📊 部署案例

| 平台 | 状态 | 说明 |
|------|------|------|
| 飞牛NAS | ✅ 运行中 | 7x24运行 |
| Windows (小安) | ✅ 运行中 | 开机自启 |
| WSL | ✅ 可选 | 开发测试用 |

## 🤝 如何贡献

1. **提交问题**：发现bug或有疑问
2. **提交PR**：完善文档或添加新教程
3. **分享经验**：把你的部署经验写上来

## 📄 License

MIT License - 欢迎自由使用和修改

## 🔗 相关链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [飞牛NAS论坛](https://club.fnnas.com)
- [MiniMax API](https://platform.minimaxi.com)
- [飞书开放平台](https://open.feilank.cn)

---

*有问题？先看上面的 ❓ 问题解决 部分，还解决不了就在GitHub提Issue！*
