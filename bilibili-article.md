# 飞牛NAS部署OpenClaw：AI助手搭建完整指南

> 作者：vinwjin
> 2026年3月

---

## 前言

你是否想过拥有一 个属于自己的AI助手？它可以帮你查资料、聊聊天、管理文件，甚至帮你完成工作？最近我在飞牛NAS上成功部署了OpenClaw，今天把完整经验分享给大家。

## 什么是OpenClaw？

OpenClaw是一个开源的AI助手框架，可以部署在各种设备上，通过Telegram、飞书等渠道随时和AI对话。它支持多种模型（比如MiniMax、Claude、GLM等），还可以安装各种技能。

## 为什么选择飞牛NAS？

1. **轻量级** - 不需要Docker，直接用npm安装
2. **网络稳定** - 主机网络直接访问，API连接更稳定
3. **权限可控** - 配合allowlist，更安全

## 开始部署

### 准备工作

你需要：
- 一台飞牛NAS
- SSH客户端
- MiniMax API Key（或其他模型）
- 飞书开发者账号

### 第一步：安装Node.js 22

SSH登录NAS后，运行：

```bash
# 安装nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# 安装Node.js 22
nvm install 22
nvm use 22
```

### 第二步：安装OpenClaw

```bash
npm i -g openclaw
```

### 第三步：配置

创建配置文件 `~/.openclaw/openclaw.json`，填入你的配置：

```json
{
  "models": {
    "providers": {
      "minimax-cn": {
        "apiKey": "你的API Key",
        "models": [{ "id": "MiniMax-M2.5" }]
      }
    }
  },
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "main": {
          "appId": "你的飞书App ID",
          "appSecret": "你的飞书App Secret",
          "mode": "websocket"
        }
      }
    }
  },
  "gateway": {
    "port": 18789,
    "bind": "lan"
  }
}
```

### 第四步：启动

```bash
openclaw gateway run --bind lan --port 18789
```

现在访问 `http://你的NASIP:18789` 就能看到控制台。

### 第五步：飞书配对

在飞书给机器人发 `/start`，会收到配对码，然后执行：

```bash
openclaw pairing approve feishu 你的配对码
```

### 第六步：开机自启

```bash
crontab -e
# 添加：
@reboot source ~/.bashrc && ~/.nvm/versions/node/v22.x.x/bin/openclaw gateway run --bind lan --port 18789 > ~/openclaw.log 2>&1 &
```

## 安装技能

部署完成后，可以安装各种技能：

```bash
# 查看技能
openclaw skills list

# 安装常用工具
npm i -g gh mcporter oracle-cli agent-browser tavily
```

## 效果展示

部署完成后，你可以：
- 在飞书和AI对话
- 让AI帮你查资料、总结文章
- 调用各种工具完成任务

## 常见问题

**Q: 端口被占用？**
A: 换端口或关闭占用进程

**Q: 飞书连不上？**
A: 检查App ID和Secret是否正确

**Q: 重启后不能自启？**
A: 检查crontab是否生效

## 总结

整个部署过程大概30分钟，比想象中简单。npm方式比Docker更轻量，调试也更方便。有兴趣的朋友可以试试！

---

*如果觉得有用，欢迎三连支持！*

GitHub仓库：https://github.com/vinwjin/openclaw-nas-tutorial
