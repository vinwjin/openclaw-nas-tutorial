# 远程帮朋友在 WSL Ubuntu 安装 OpenClaw 完整方案

> 目标：朋友的 Windows 电脑 → 加入 Tailscale 局域网 → 我远程 SSH 进去 → 在 WSL Ubuntu 里装好 OpenClaw

---

## 📋 完整流程一览

| 步骤 | 操作人 | 内容 |
|------|--------|------|
| 1 | 朋友 | 按教程安装 WSL2 + Ubuntu |
| 2 | 朋友 | 安装 Tailscale 并加入你的账号 |
| 3 | 朋友 | 开启 Windows SSH 服务 |
| 4 | 朋友 | 告诉我连接信息 |
| 5 | 我 | SSH 登录 → 安装 OpenClaw |
| 6 | 我 | 验证成功 → 完工 ✅ |

---

## 第一步：朋友安装 WSL2 + Ubuntu

**让你朋友按这个超级详细教程操作：**

📄 **教程文档**：`WSL-Ubuntu-OpenClaw-安装指南.md`

### 核心步骤（简化版）：

1. **环境检查**（Win+R → winver，确认版本 2004+）
2. **管理员终端**执行：`wsl --install`
3. **重启电脑**
4. **设置 Ubuntu 用户名和密码**
5. **换清华源**：
   ```bash
   sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list
   sudo apt update
   ```
6. **配置 .wslconfig**（重要！）

---

## 第二步：加入 Tailscale 局域网

### 2.1 安装 Tailscale

让你朋友：
1. 打开 https://tailscale.com/download
2. 下载 **Windows** 版本
3. 安装并启动
4. **登录你的 Tailscale 账号**（需要你同意或分享邀请链接）

### 2.2 确认连接

告诉我：
- 他电脑在 Tailscale 上的名称
- 他的 Tailscale IP（100.x.x.x 格式）

---

## 第三步：开启 Windows SSH

**让你朋友执行：**

1. 打开 **设置 → 应用 → 可选功能**
2. 点击 **添加功能**
3. 搜索 **OpenSSH Server** → 安装

**或者管理员 PowerShell 执行：**

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

---

## 第四步：获取连接信息

让你朋友告诉我：

| 信息 | 获取方法 |
|------|---------|
| Tailscale IP | 打开 Tailscale 客户端查看（100.x.x.x）|
| Windows 用户名 | 打开 cmd，输入 `whoami` |
| Windows 密码 | 他的登录密码 |

---

## 第五步：我远程安装

### 5.1 SSH 登录朋友电脑

```bash
ssh 用户名@100.x.x.x
```

### 5.2 进入 WSL

```bash
wsl -d Ubuntu
# 或者
wsl
```

### 5.3 执行安装（详细版）

按照 `WSL-Ubuntu-OpenClaw-安装指南.md` 执行：

```bash
# 1. 更新系统
sudo apt update && sudo apt upgrade -y

# 2. 安装基础工具
sudo apt install -y curl git build-essential

# 3. 安装 nvm + Node.js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install 22
nvm use 22
nvm alias default 22

# 4. 换 npm 源
npm config set registry https://registry.npmmirror.com

# 5. 安装 OpenClaw
npm install -g openclaw

# 6. 创建配置目录
mkdir -p ~/.openclaw

# 7. 启动 Gateway
openclaw gateway run
```

### 5.4 启用 systemd（如需）

```bash
# 创建 wsl.conf
sudo vi /etc/wsl.conf
```

添加：
```ini
[boot]
systemd=true
```

保存退出，然后在 Windows PowerShell 执行：
```powershell
wsl --shutdown
```

重新打开 WSL，执行：
```bash
openclaw gateway run
```

---

## 第六步：验证安装

我会验证以下全部通过：

- [ ] `openclaw --version` 显示版本
- [ ] `openclaw status` 显示健康
- [ ] `netstat -tlnp | grep 18789` 端口在监听
- [ ] 浏览器访问 http://localhost:18789 能打开

---

## 常见问题速查

| 问题 | 解决方案 |
|------|---------|
| SSH 连接被拒绝 | 在朋友电脑管理员 PowerShell 执行 `Start-Service sshd` |
| WSL 版本是 1 | `wsl --set-default-version 2` |
| 忘记 Ubuntu 密码 | `wsl -d Ubuntu -u root` → `passwd 用户名` |
| 镜像模式不生效 | 确认 .wslconfig 在正确位置，执行 `wsl --shutdown` 重启 |
| gateway 报 systemd 错误 | 创建 /etc/wsl.conf 启用 systemd |

---

## 给你朋友的操作清单

### 🚀 请按顺序执行：

#### 第 1 步：检查电脑

- [ ] 按 `Win + R` → 输入 `winver` → 确认版本 2004+
- [ ] 按 `Ctrl + Shift + Esc` → 性能 → CPU → 确认虚拟化已启用

#### 第 2 步：安装 WSL2

- [ ] 管理员 PowerShell 执行：`wsl --install`
- [ ] **重启电脑**
- [ ] 打开 Ubuntu，设置用户名和密码

#### 第 3 步：配置 WSL

- [ ] 换清华源：
  ```bash
  sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list
  sudo apt update
  ```
- [ ] 创建 `.wslconfig` 文件（C:\Users\你的用户名\）

#### 第 4 步：安装 Tailscale

- [ ] 下载安装：https://tailscale.com/download
- [ ] 登录我的 Tailscale 账号
- [ ] **告诉我你的 Tailscale IP**

#### 第 5 步：开启 SSH

- [ ] 管理员 PowerShell 执行：
  ```powershell
  Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  Start-Service sshd
  ```
- [ ] **告诉我你的 Windows 用户名和密码**

---

### 📋 把以下信息发给我：

1. 你的 Tailscale IP：_____
2. 你的 Windows 用户名：_____
3. 你的 Windows 密码：_____

---

## 总结

这个方案的核心：

1. **WSL2** → 在 Windows 里跑 Ubuntu
2. **Tailscale** → 让你朋友进入我们的局域网
3. **Windows SSH** → 让我能远程登录
4. **OpenClaw** → 在 WSL 里装好 AI 助手

全部搞定后，你朋友就能通过飞书/Telegram 等渠道跟 AI 对话辣！🎉

---

> 📝 方案编写：小雅 | 📅 2026-03-12
> 📚 参考：朋友提供的详细部署文档
