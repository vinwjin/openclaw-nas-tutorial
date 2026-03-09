# 飞牛 NAS 部署 OpenClaw 完整教程

> 作者: 小雅 (Xiǎoyǎ)  
> 日期: 2026-03-10  
> 环境: 飞牛 NAS (fnOS) + OpenClaw + 飞书

---

## 📋 概述

本文档详细介绍如何在飞牛 NAS (fnOS) 上通过 npm 方式部署 OpenClaw，实现通过飞书/Telegram 等渠道与 AI 对话。

**为什么选择 npm 而非 Docker？**
- 更轻量，无需 Docker 环境
- 直接使用主机网络，Telegram API 通信更稳定
- 调试友好，直接查看日志
- 权限可控，配合 allowlist 更安全

---

## 🏠 环境信息

| 项目 | 值 |
|------|-----|
| 飞牛 NAS IP | 你的NAS-IP (示例) |
| SSH 用户 | 你的用户名 |
| OpenClaw 版本 | 2026.3.8 |
| Node.js 版本 | v22.22.1 |
| 部署方式 | npm 系统级安装 |
| 通信渠道 | 飞书 (Feishu) |
| AI 模型 | MiniMax 2.5 |

---

## 📝 安装步骤

### 一、环境检查

SSH 登录飞牛 NAS：
```bash
ssh 你的用户名@你的NAS-IP
```

检查系统信息：
```bash
cat /etc/os-release
uname -m
node -v
npm -v
```

### 二、安装 Node.js 22

飞牛 NAS 通常预装了 Node.js，但版本可能较低。OpenClaw 需要 Node.js 22+。

#### 2.1 如果没有 nvm，先安装 nvm：
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
```

#### 2.2 安装 Node.js 22：
```bash
nvm install 22
nvm use 22
node -v  # 确认版本
```

### 三、安装 OpenClaw

```bash
npm i -g openclaw
```

**三个黄金确认：**
```bash
which openclaw
# 期望: ~/.nvm/versions/node/v22.x.x/bin/openclaw

openclaw --version
# 期望显示版本号

whoami
# 期望: 你的用户名 (普通用户，非 root)
```

### 四、配置 OpenClaw

#### 4.1 创建配置文件

创建配置目录：
```bash
mkdir -p ~/.openclaw
```

编辑配置文件 `~/.openclaw/openclaw.json`：

```json
{
  "meta": {
    "lastTouchedVersion": "2026.3.8",
    "lastTouchedAt": "2026-03-10T00:10:00.000Z"
  },
  "auth": {
    "profiles": {
      "minimax-cn:default": {
        "provider": "minimax-cn",
        "mode": "api_key"
      }
    }
  },
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "你的MiniMax API Key",
        "api": "anthropic-messages",
        "authHeader": true,
        "models": [
          {
            "id": "MiniMax-M2.5",
            "name": "MiniMax-M2.5",
            "reasoning": true,
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.5"
      },
      "workspace": "/home/你的用户名/openclaw_workspace"
    }
  },
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "main": {
          "appId": "你的飞书 App ID",
          "appSecret": "你的飞书 App Secret",
          "mode": "websocket"
        },
        "default": {
          "appId": "你的飞书 App ID",
          "appSecret": "你的飞书 App Secret",
          "mode": "websocket"
        }
      }
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan"
  },
  "plugins": {
    "allow": ["feishu"],
    "entries": {
      "feishu": {
        "enabled": true
      }
    }
  }
}
```

#### 4.2 创建工作目录
```bash
mkdir -p ~/openclaw_workspace
```

### 五、启动 OpenClaw

```bash
source ~/.bashrc
openclaw gateway run --bind lan --port 18789
```

**访问地址：** http://你的NAS-IP:18789

### 六、飞书配对

1. 在飞书给机器人发消息 `/start`
2. 机器人会返回配对码
3. 执行配对：
```bash
openclaw pairing approve feishu <配对码>
```

### 七、配置开机自启

#### 7.1 使用 crontab (推荐)
```bash
crontab -e
# 添加以下行：
@reboot source ~/.bashrc && ~/.nvm/versions/node/v22.22.1/bin/openclaw gateway run --bind lan --port 18789 > ~/openclaw.log 2>&1 &
```

#### 7.2 验证开机自启
```bash
crontab -l | grep openclaw
```

---

## 🔧 安装技能

### 基础技能 (已内置)

```bash
openclaw skills list
```

### 安装额外工具

```bash
# GitHub CLI
npm i -g gh

# MCP 服务器
npm i -g mcporter

# Oracle CLI
npm i -g oracle-cli

# 浏览器自动化
npm i -g agent-browser

# Tavily 搜索
npm i -g tavily
```

### 安装 ClawHub 技能

```bash
# 搜索技能
clawhub search <关键词>

# 安装技能
clawhub install <技能slug>
```

### 手动安装技能

如果 ClawHub 限速，可以手动复制技能：

```bash
# 从本地上传技能
scp -r ~/skills/<技能名> 你的用户名@你的NAS-IP:~/.openclaw/skills/
```

---

## ⚠️ 安全配置建议

### 1. 配置 allowlist

编辑 `~/.openclaw/config.json`：

```json
{
  "fileSystem": {
    "allowlist": [
      "/home/你的用户名/openclaw_workspace"
    ]
  },
  "shell": {
    "enabled": true,
    "allowlist": [
      "ping", "curl", "git", "apt", "npm", "node", "ps", "ls", "cat", "echo"
    ]
  }
}
```

### 2. 创建专用工作目录

```bash
mkdir -p ~/openclaw_workspace
# 所有 Agent 操作的文件放在此目录
```

### 3. 定期安全检查

```bash
openclaw security audit
```

---

## 🔧 常用命令

| 命令 | 说明 |
|------|------|
| `openclaw gateway run` | 启动网关 |
| `openclaw status` | 查看状态 |
| `openclaw skills list` | 列出技能 |
| `openclaw pairing approve <渠道> <码>` | 配对 |
| `openclaw logs` | 查看日志 |
| `openclaw doctor` | 健康检查 |

---

## ❓ 常见问题

### Q1: 端口被占用？
```bash
# 查找占用进程
lsof -i :18789
# 或换端口
openclaw gateway run --port 18790
```

### Q2: 飞书连接失败？
- 检查 App ID 和 App Secret 是否正确
- 确认飞书应用已启用机器人权限
- 检查回调 URL 是否配置

### Q3: npm 安装权限报错？
- 使用 nvm 管理 Node.js
- 不要用 sudo npm install
- 配置 npm prefix 到用户目录

### Q4: 重启后无法自启？
- 检查 crontab 是否生效
- 确认 nvm 环境变量加载
- 查看日志 `~/openclaw.log`

---

## 📊 部署成果

| 项目 | 状态 |
|------|------|
| OpenClaw 版本 | 2026.3.8 |
| Node.js 版本 | v22.22.1 |
| 技能数量 | 24+ |
| 渠道 | 飞书 |
| 模型 | MiniMax 2.5 |
| 访问地址 | http://你的NAS-IP:18789 |
| 开机自启 | ✅ |

---

## 📚 参考链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [飞牛 NAS 论坛](https://club.fnnas.com)
- [MiniMax API](https://platform.minimaxi.com)
- [飞书开放平台](https://open.feishu.cn)

---

*教程制作: 小雅 (Xiǎoyǎ)*  
*2026-03-10*
