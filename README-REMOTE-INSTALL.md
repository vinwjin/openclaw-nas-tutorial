# 远程帮朋友在 WSL Ubuntu 安装 OpenClaw 完整方案

> 目标：朋友的 Windows 电脑 → 加入 Tailscale 局域网 → 我远程 SSH 进去 → 在 WSL Ubuntu 里装好 OpenClaw

---

## 一句话概括

让你朋友在电脑下安装 Tailscale → 开启 Windows SSH → 告诉我电脑的 Tailscale IP → 我远程登进去装

---

## 详细步骤

### 第一步：让你朋友加入 Tailscale 局域网

#### 1.1 下载安装 Tailscale

让你朋友在他电脑上：

1. 打开浏览器，访问：https://tailscale.com/download
2. 下载 Windows 版本
3. 安装并登录（用你的 Tailscale 账号）

#### 1.2 验证连接

安装成功后，告诉我：
- 他电脑在 Tailscale 上的名称（如 `friend-laptop`）
- 他分配到的 Tailscale IP（通常是 100.x.x.x 格式）

---

### 第二步：开启 Windows SSH 服务

**让你朋友在他电脑上执行：**

#### 2.1 安装 OpenSSH Server

1. 打开 **设置** → **应用** → **可选功能**
2. 点击 **添加功能**
3. 搜索 **OpenSSH Server** → 点击安装

或者以管理员身份打开 PowerShell 执行：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

#### 2.2 启动 SSH 服务

以管理员身份打开 PowerShell，执行：

```powershell
# 启动服务
Start-Service sshd

# 设置开机自启
Set-Service -Name sshd -StartupType Automatic

# 检查服务状态
Get-Service sshd
```

#### 2.3 允许防火墙

```powershell
# 允许 SSH 通过防火墙
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

---

### 第三步：获取连接信息

让你朋友告诉我：

1. **他的 Windows 用户名**（查看方法：打开cmd，输入 `whoami`）
2. **他的 Windows 登录密码**（需要密码登录 SSH）
3. **他的 Tailscale IP**（打开 Tailscale 客户端可见，格式 100.x.x.x）

---

### 第四步：我远程连接安装

拿到信息后，我会执行：

```bash
# 通过 Tailscale IP  SSH 登录他的 Windows
ssh 用户名@100.x.x.x

# 进入 WSL
wsl -d Ubuntu

# 然后在 WSL 里执行安装...
```

---

## 完整安装脚本（提前给你朋友准备好）

让你朋友先在 WSL 里创建这个脚本，我远程进去直接跑：

```bash
#!/bin/bash
# 保存为 ~/install-openclaw.sh

set -e

echo "=== OpenClaw 安装脚本 ==="

# 1. 换国内源
echo "[1/7] 更换软件源..."
sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list
sudo apt update

# 2. 安装基础依赖
echo "[2/7] 安装依赖..."
sudo apt install -y curl git build-essential

# 3. 安装 nvm + Node.js
echo "[3/7] 安装 Node.js..."
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] || {
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
}
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install 22
nvm use 22
nvm alias default 22
node -v

# 4. 换 npm 源并安装 OpenClaw
echo "[4/7] 安装 OpenClaw..."
npm config set registry https://registry.npmmirror.com
npm install -g openclaw

# 5. 创建配置目录
echo "[5/7] 创建配置..."
mkdir -p ~/.openclaw

# 6. 启动 Gateway
echo "[6/7] 启动 Gateway..."
openclaw gateway run

echo "=== 安装完成 ==="
```

---

## 远程操作流程（我来做）

### 步骤 1：SSH 登录朋友电脑

```bash
ssh friend@100.x.x.x
```

### 步骤 2：进入 WSL

```bash
wsl -d Ubuntu
# 或者
wsl # 进入默认发行版
```

### 步骤 3：执行安装

```bash
# 方式一：运行准备好的脚本
bash ~/install-openclaw.sh

# 方式二：一步步执行
# （见下文）
```

### 步骤 4：验证安装

```bash
# 检查 OpenClaw 是否运行
openclaw status

# 检查端口
netstat -tlnp | grep 18789
```

---

## 详细安装命令（一步步版）

### 1. 进入 WSL 后，先更新

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. 安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

然后重新打开终端，或者执行：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### 3. 安装 Node.js

```bash
nvm install 22
nvm use 22
nvm alias default 22
node -v  # 应该显示 v22.x.x
```

### 4. 换 npm 源

```bash
npm config set registry https://registry.npmmirror.com
```

### 5. 安装 OpenClaw

```bash
npm install -g openclaw
openclaw --version
```

### 6. 首次配置

```bash
# 方式一：交互式配置
openclaw configure

# 方式二：手动配置
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

**配置文件示例**：

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

### 7. 启动 Gateway

```bash
openclaw gateway run
```

看到类似输出就是成功：

```
Gateway started on http://127.0.0.1:18789
```

### 8. 验证

打开另一个终端：

```bash
# 检查进程
ps aux | grep openclaw

# 检查端口
netstat -tlnp | grep 18789

# 测试本地访问
curl http://127.0.0.1:18789
```

---

## 可能的问题及解决

### 问题 1：SSH 连接被拒绝

**原因**：Windows SSH 服务没开

**解决**：
```powershell
# 在朋友电脑的管理员 PowerShell 里执行
Start-Service sshd
```

### 问题 2：SSH 登录需要密码

**解决**：朋友需要设置 Windows 登录密码，或者我尝试无密码登录

### 问题 3：WSL 不是默认发行版

```bash
# 查看已安装的 WSL
wsl -l -v

# 设置默认
wsl -s Ubuntu
```

### 问题 4：Node 版本问题

```bash
# 检查版本
node -v

# 如果版本不对，切换版本
nvm list
nvm use 22
nvm alias default 22
```

### 问题 5：npm 安装慢

```bash
# 确认换源成功
npm config get registry
# 应该显示 https://registry.npmmirror.com
```

---

## 验证清单

安装完成后，我会验证：

- [ ] `openclaw --version` 能显示版本
- [ ] `openclaw status` 显示健康
- [ ] `netstat -tlnp | grep 18789` 端口在监听
- [ ] 浏览器访问 http://localhost:18789 能打开控制台

---

## 完整流程时间线

| 步骤 | 操作人 | 内容 |
|------|--------|------|
| 1 | 朋友 | 下载安装 Tailscale，告诉我 Tailscale IP |
| 2 | 朋友 | 开启 Windows SSH 服务 |
| 3 | 朋友 | 告诉我 Windows 用户名 + 密码 |
| 4 | 我 | SSH 登录朋友电脑 |
| 5 | 我 | 进入 WSL Ubuntu |
| 6 | 我 | 执行安装脚本 |
| 7 | 我 | 验证安装成功 |
| 8 | 我 | 回复"安装完成" |

---

## 给你朋友的操作清单

让你朋友复制下面的内容执行：

---

### 🚀 快速开始 - 请执行以下步骤

#### 第 1 步：安装 Tailscale

1. 打开浏览器，访问：https://tailscale.com/download
2. 点击 **Download for Windows**
3. 安装并启动
4. 登录（用我的 Tailscale 账号，如果你没有问我）
5. **告诉我你电脑的 Tailscale IP 地址**（打开 Tailscale 客户端可以看到，格式是 100.x.x.x）

#### 第 2 步：开启 SSH 服务

1. 在开始菜单搜索 **PowerShell**
2. 右键 → **以管理员身份运行**
3. 复制粘贴下面的命令并回车：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

4. **告诉我你的 Windows 用户名和密码**

#### 第 3 步：确认 WSL 已安装

1. 打开 Microsoft Store
2. 搜索 **Ubuntu**
3. 点击 **安装**（建议选 Ubuntu 22.04 LTS）
4. 安装完成后打开，设置用户名和密码

---

### 📋 把以下信息发给我：

1. 你的 Tailscale IP：_____
2. 你的 Windows 用户名：_____
3. 你的 Windows 密码：_____

---

## 总结

这个方案的核心是：

1. **Tailscale** → 让你朋友电脑进入我们的局域网
2. **Windows SSH** → 让我能远程登录他的 Windows
3. **WSL** → 在 Windows 的 Linux 子系统里装 OpenClaw

全部搞定后，你朋友就能通过飞书/Telegram 等渠道跟 AI 对话辣！🎉

---

> 📝 方案编写：小雅 | 📅 2026-03-12
