# Windows WSL2 Ubuntu 安装 OpenClaw 完整指南

> 适用于 Windows 10/11 + WSL2 (Ubuntu) | 最后更新：2026-03-12

---

## 一、环境要求

| 项目 | 最低配置 | 推荐配置 |
|------|---------|---------|
| Windows 版本 | Windows 10 2004+ | Windows 11 |
| WSL 版本 | WSL2 | WSL2 |
| Ubuntu 版本 | 20.04 LTS | 22.04 LTS / 24.04 LTS |
| 内存 | 4GB | 8GB+ |
| 磁盘空间 | 10GB | 20GB+ |
| 网络 | 能联网 | 能联网（用于 API 调用） |

---

## 二、安装步骤

### 2.1 启用 WSL2（如果未启用）

以管理员身份打开 PowerShell，执行：

```powershell
wsl --install
```

重启电脑后自动安装 Ubuntu。

> 💡 **提示**：如果安装缓慢，可手动下载 [Ubuntu 22.04 LTS](https://aka.ms/wslubuntu2204) 或 [Ubuntu 24.04 LTS](https://aka.ms/wslubuntu2404) 的 .appx 文件，双击安装。

### 2.2 打开 WSL Ubuntu 终端

开始菜单 → 搜索 "Ubuntu" → 打开

### 2.3 换国内源（推荐，能大幅提升下载速度）

```bash
# 备份原 sources.list
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 替换为清华源（Ubuntu 22.04）
sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list

# 更新软件包列表
sudo apt update && sudo apt upgrade -y
```

### 2.4 安装 Node.js（必须）

OpenClaw 需要 Node.js 18+，推荐用 nvm 管理：

```bash
# 安装 nvm（Node Version Manager）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 重新加载环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 安装 LTS 版 Node.js
nvm install --lts

# 验证安装
node -v    # 应显示 v20.x.x 或 v22.x.x
npm -v
```

> ⚠️ **重要**：不要使用 `sudo apt install nodejs`，版本太老！

### 2.5 安装 OpenClaw

```bash
# 换国内 npm 源（重要！）
npm config set registry https://registry.npmmirror.com

# 安装 OpenClaw（全局安装）
npm install -g openclaw

# 验证安装
openclaw --version
```

### 2.6 首次运行与配置

```bash
# 启动交互式配置向导
openclaw configure
```

或手动配置：

```bash
# 创建配置目录
mkdir -p ~/.openclaw

# 编辑配置文件
nano ~/.openclaw/openclaw.json
```

**最小配置示例**：

```json
{
  "model": {
    "provider": "minimax",
    "default": "MiniMax-M2.1"
  },
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "你的AppId",
      "appSecret": "你的AppSecret"
    }
  }
}
```

### 2.7 启动 Gateway

```bash
# 前台运行（调试用）
openclaw gateway run

# 或后台运行
openclaw gateway run &
```

---

## 三、配置飞书渠道（可选）

### 3.1 创建飞书应用

1. 打开 [飞书开放平台](https://open.feishu.cn/)
2. 创建企业自建应用
3. 获取 **App ID** 和 **App Secret**
4. 添加权限：
   - `im:message:send_as_bot`
   - `im:message:receive_as_bot`
   - `im:chat:message`

### 3.2 配置 webhook

在应用配置页面创建 webhook 事件订阅：
- 请求 URL：`你的公网IP或域名/callback/feishu`
- 事件：`message` → `im.message.message.created`

### 3.3 添加 Bot 到群聊

将应用添加到群聊，获得 `app_id` 用于配置。

### 3.4 配置 OpenClaw

```bash
# 设置飞书配置
openclaw config edit channels.feishu.appId "你的AppId"
openclaw config edit channels.feishu.appSecret "你的AppSecret"

# 重启 Gateway
pkill -f openclaw
openclaw gateway run &
```

---

## 四、远程访问配置

### 方案 A：Tailscale 异地组网（推荐）

1. **在 Ubuntu WSL 内安装 Tailscale**：

```bash
# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 启动 Tailscale
sudo tailscale up

# 会得到一个 100.x.x.x 的 Tailscale IP
```

2. **在 Windows 端也安装 Tailscale**

3. **从其他设备访问**：
   - 通过 Tailscale IP + 端口访问
   - 例如：`http://100.x.x.x:18789`

### 方案 B：SSH 隧道（适合开发调试）

```powershell
# 在 Windows PowerShell 中执行
ssh -L 18789:localhost:18789 用户名@WSL的IP
```

---

## 五、常用命令一览

| 命令 | 说明 |
|------|------|
| `openclaw gateway run` | 启动 Gateway |
| `openclaw status` | 查看状态 |
| `openclaw config edit` | 修改配置 |
| `openclaw models list` | 查看可用模型 |
| `openclaw skills list` | 查看已安装技能 |
| `openclaw logs` | 查看日志 |
| `openclaw update` | 更新 OpenClaw |

---

## 六、开机自启配置

### 方法 1：systemd 用户服务（推荐）

```bash
# 创建 systemd 服务文件
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/openclaw.service << 'EOF'
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
ExecStart=/home/用户名/.nvm/versions/node/版本号/bin/openclaw gateway run
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
EOF

# 启用服务
systemctl --user enable openclaw
systemctl --user start openclaw
```

### 方法 2：crontab

```bash
# 编辑 crontab
crontab -e

# 添加启动命令
@reboot /home/用户名/.nvm/versions/node/版本号/bin/openclaw gateway run
```

---

## 七、问题排查

### 问题 1：Node 版本太旧

```bash
# 检查版本
node -v

# 如果版本 < 18，用 nvm 切换
nvm install 22
nvm use 22
nvm alias default 22
```

### 问题 2：端口被占用

```bash
# 查找占用 18789 端口的进程
lsof -i :18789

# 杀掉进程
kill -9 <PID>
```

### 问题 3：飞书连接失败

```bash
# 查看飞书渠道日志
openclaw logs --channel feishu

# 检查配置
openclaw config get channels.feishu
```

### 问题 4：Tailscale 无法连接

```bash
# 检查 Tailscale 状态
tailscale status

# 重新登录
tailscale logout
tailscale login
```

---

## 八、完整安装脚本

> 将以下内容保存为 `install.sh`，在 WSL 中执行：

```bash
#!/bin/bash
set -e

echo "=== 开始安装 OpenClaw ==="

# 1. 换源
echo "[1/6] 更换软件源..."
sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list

# 2. 安装依赖
echo "[2/6] 更新软件包..."
sudo apt update && sudo apt upgrade -y

# 3. 安装 nvm 和 Node.js
echo "[3/6] 安装 Node.js..."
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] || {
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
}
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts
nvm alias default lts/*
node -v

# 4. 安装 OpenClaw
echo "[4/6] 安装 OpenClaw..."
npm config set registry https://registry.npmmirror.com
npm install -g openclaw

# 5. 配置
echo "[5/6] 初始化配置..."
mkdir -p ~/.openclaw

# 6. 启动
echo "[6/6] 启动 Gateway..."
openclaw gateway run

echo "=== 安装完成 ==="
```

执行脚本：
```bash
chmod +x install.sh
./install.sh
```

---

## 九、相关资源

- 官方文档：https://docs.openclaw.ai
- GitHub：https://github.com/openclaw/openclaw
- 问题反馈：https://github.com/openclaw/openclaw/issues

---

## 十、注意事项

1. **API Key**：需要准备模型提供商的 API Key（如 MiniMax、OpenAI 等）
2. **网络**：中国大陆用户建议使用国内模型或配置代理
3. **安全**：不要将包含敏感信息的配置文件推送到公开仓库
4. **备份**：定期备份 `~/.openclaw/` 目录
5. **更新**：定期执行 `openclaw update` 保持最新

---

> 📝 作者：基于 OpenClaw 官方文档和实际安装经验编写
> 📅 更新：2026-03-12
