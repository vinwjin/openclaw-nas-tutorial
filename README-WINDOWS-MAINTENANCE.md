# Windows 部署 OpenClaw + Tailscale 远程维护指南

> 作者: 牛牛  
> 日期: 2026-03-14  
> 环境: Windows 11 + OpenClaw + Tailscale + 飞书

---

## 📋 概述

本文档介绍如何在 Windows 电脑上部署 OpenClaw，并通过 Tailscale 实现跨网络远程维护。

**适用场景：**
- Windows 电脑部署 OpenClaw（，小安）
- 通过 Tailscale 实现远程 SSH 维护
- 远程查看状态、日志、执行命令

---

## 🏠 环境信息

| 项目 | 值 |
|------|-----|
| 主机 | Windows 11 专业版 |
| Tailscale | 100.x.x.x |
| SSH 端口 | 22 |
| OpenClaw 端口 | 18789 |

---

## 📝 安装步骤

### 一、安装 Tailscale

1. 下载 Tailscale Windows 版：https://tailscale.com/download
2. 安装并登录账号
3. 确保 Tailscale 显示在线状态

### 二、安装 OpenSSH Server

以管理员身份运行 PowerShell：

```powershell
# 安装 OpenSSH 服务端
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动 SSH 服务
Start-Service sshd

# 设置开机自启
Set-Service -Name sshd -StartupType Automatic
```

### 三、安装 OpenClaw

```powershell
# 安装 Node.js (如果没有)
winget install OpenJS.NodeJS.LTS

# 安装 OpenClaw
npm i -g openclaw
```

### 四、配置 OpenClaw

参考 [飞牛NAS部署教程](./README.md) 配置 openclaw.json，重点配置飞书渠道。

### 五、配置公钥登录（推荐）

#### 5.1 生成密钥对（Linux/Mac端）

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

#### 5.2 配置 Windows 公钥

在 Windows PowerShell（管理员）：

```powershell
# 创建 .ssh 目录
New-Item -ItemType Directory -Force -Path C:\Users\你的用户名\.ssh

# 写入公钥（替换为Linux端的公钥）
Set-Content -Path C:\Users\你的用户名\.ssh\authorized_keys -Value "ssh-ed25519 AAAA... your@email"
```

#### 5.3 配置 SSH 允许公钥登录

```powershell
notepad C:\ProgramData\ssh\sshd_config
```

确保以下行没有注释：
```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

保存后重启 SSH：
```powershell
Restart-Service sshd
```

---

## 🔧 远程维护

### 一、获取 Tailscale IP

```bash
tailscale status
```

找到 Windows 主机的 IP（100.x.x.x）

### 二、SSH 远程连接

```bash
# 密码登录
ssh 用户名@100.x.x.x

# 公钥登录（推荐）
ssh 用户名@100.x.x.x
```

### 三、常用远程命令

| 命令 | 说明 |
|------|------|
| `openclaw status` | 查看状态 |
| `openclaw health` | 健康检查 |
| `openclaw logs` | 查看日志 |
| `openclaw gateway stop` | 停止 |
| `openclaw gateway start` | 启动 |

---

## ⚡ 快速连接脚本

### Linux/Ubuntu 端

创建快速连接脚本 `ssh-windows.ps1`：

```powershell
# Windows SSH 连接
ssh 用户名@100.x.x.x
```

或在 Linux 端直接用 sshpass：

```bash
sshpass -p '密码' ssh -o StrictHostKeyChecking=no 用户名@100.x.x.x
```

---

## 🚀 一键启动/停止脚本

### 启动脚本 (启动小安.bat)

```bat
@echo off
start /b openclaw gateway run
```

### 停止脚本 (停止小安.bat)

```bat
@echo off
openclaw gateway stop
```

放置在桌面，双击即可运行。

---

## 🔐 安全配置

### 1. 配置防火墙

```powershell
# 开放 SSH 端口（如需要从外网访问）
New-NetFirewallRule -Name "SSH" -DisplayName "SSH" -Protocol TCP -LocalPort 22 -Enabled True -Action Allow
```

### 2. 配置 allowlist

在 openclaw.json 中配置飞书白名单：

```json
{
  "channels": {
    "feishu": {
      "allowFrom": ["你的飞书用户ID"],
      "groupAllowFrom": ["你的飞书群ID"]
    }
  }
}
```

---

## ❓ 常见问题

### Q1: SSH 连接超时？

检查 Windows SSH 服务是否运行：
```powershell
Get-Service sshd
Start-Service sshd
```

### Q2: 公钥登录失败？

检查 authorized_keys 文件路径：
- 系统级：`C:\ProgramData\ssh\administrators_authorized_keys`
- 用户级：`C:\Users\用户名\.ssh\authorized_keys`

### Q3: Tailscale 显示离线？

重新登录 Tailscale：
```powershell
tailscale logout
tailscale login
```

### Q4: OpenClaw 无法启动？

检查日志：
```powershell
openclaw logs
```

---

## 📊 维护成果

| 项目 | 状态 |
|------|------|
| OpenClaw | ✅ 运行中 |
| Tailscale | ✅ 已连接 |
| SSH | ✅ 可远程访问 |
| 飞书 | ✅ 正常响应 |
| 开机自启 | ✅ 已配置 |

---

## 🔗 相关链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [Tailscale 下载](https://tailscale.com/download)
- [飞书开放平台](https://open.feishu.cn)

---

*教程制作: 牛牛*  
*2026-03-14*
