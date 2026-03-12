# Windows WSL2 Ubuntu 安装 OpenClaw 超级详细指南

> 适用于 Windows 10/11 + WSL2 (Ubuntu) | 基于官方文档优化 | 最后更新：2026-03-12

---

## 📋 目录

1. [课前必做：环境前置检查](#一课前必做环境前置检查)
2. [WSL2 一键安装](#二wsl2-一键安装)
3. [Ubuntu 初始化设置](#三ubuntu-初始化设置)
4. [WSL 网络配置（重要！）](#四wsl-网络配置)
5. [OpenClaw 环境配置](#五openclaw-环境配置)
6. [安装 OpenClaw](#六安装-openclaw)
7. [飞书连接配置](#七飞书连接配置)
8. [远程访问配置](#八远程访问配置)
9. [常见问题解决](#九常见问题解决)

---

## 一、课前必做：环境前置检查 ⚠️

**必须先完成这一步，规避 90% 的新手报错！**

### 1.1 检查 Windows 系统版本

1. 按 `Win + R`，输入 `winver`，回车
2. 检查版本要求：
   - **Windows 10**：版本 2004+，内部版本号 ≥ 19041
   - **Windows 11**：任意版本均可

若不达标：打开 **设置 → Windows 更新 → 检查更新**

### 1.2 确认 CPU 虚拟化已开启

1. 按 `Ctrl + Shift + Esc` 打开任务管理器
2. 切换到 **性能** 选项卡 → 点击 **CPU**
3. 右下角查看 **虚拟化：已启用**

若显示 **未启用**：
- 重启电脑
- 进入 BIOS 开启 CPU 虚拟化（VT-x/AMD-V）
- 或搜索「你的电脑型号 + 开启虚拟化」

### 1.3 准备管理员权限终端

**后续所有操作必须以管理员身份运行！**

1. 按 `Win + X`
2. 选择 **Windows 终端（管理员）**（Windows 10 选 **PowerShell**）
3. 弹出用户账户控制，点击 **是** ✅

---

## 二、WSL2 一键安装

### 2.1 执行一键安装命令

在管理员终端中执行：

```powershell
wsl --install
```

**命令核心作用（会自动完成）：**
- ✅ 启用「适用于Linux的Windows子系统」核心组件
- ✅ 启用「虚拟机平台」虚拟化组件
- ✅ 下载并安装 WSL2 最新 Linux 内核
- ✅ 下载并安装默认的 Ubuntu Linux 发行版
- ✅ 自动设置 WSL2 为默认版本

### 2.2 重启电脑 ⚠️

**命令执行完成后，必须重启电脑！** 否则组件不会生效。

### 2.3 Ubuntu 安装及系统初始化

重启后，打开 **Ubuntu** 应用，按提示创建 UNIX 用户账户和密码。

---

## 三、Ubuntu 初始化设置

### 3.1 验证安装是否成功

在管理员终端执行：

```powershell
wsl --list --verbose
```

**成功标准：**
| 状态 | 要求 |
|------|------|
| STATE | Running |
| VERSION | 2（必须为 2！）|
| 发行版 | Ubuntu |

### 3.2 系统更新与软件源配置

```bash
# 换清华源（大幅提升下载速度）
sudo sed -i 's|http://archive.ubuntu.com/|https://mirrors.tuna.tsinghua.edu.cn/|g' /etc/apt/sources.list

# 更新软件包
sudo apt update && sudo apt upgrade -y
```

---

## 四、WSL 网络配置 ⚠️ 重要！

### 4.1 创建 .wslconfig 文件

1. 打开文件资源管理器，导航到：
   ```
   C:\Users\你的用户名\
   ```
2. 创建名为 `.wslconfig` 的文件（注意前面的点）
3. 添加以下内容：

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
autoProxy=true
```

4. 保存文件

### 4.2 应用配置

在管理员终端执行：

```powershell
wsl --shutdown
```

**等待约 8 秒**，然后重新启动 Ubuntu。

### 4.3 验证网络模式

```bash
# 在 WSL 中执行
ip addr show eth0
```

**成功标准**：IP 应该以 `127.0.0.1` 或 `::1` 开头，而不是 `172.x.x.x`

### 4.4 配置 Windows 防火墙

以管理员身份打开 PowerShell，执行：

```powershell
# 允许 18789 端口通过防火墙
netsh advfirewall firewall add rule name="OpenClaw" dir=in action=allow protocol=tcp localport=18789

# 或者允许 WSL 入站
netsh advfirewall firewall add rule name="WSL" dir=in action=allow program="%SystemRoot%\System32\wsl.exe" enable=yes
```

---

## 五、OpenClaw 环境配置

### 5.1 基础环境配置（免密设置）

为了避免后续安装中频繁输入密码：

```bash
# 编辑 sudoers 文件
sudo visudo

# 在文件末尾添加（替换为你的用户名）
你的用户名 ALL=(ALL) NOPASSWD: ALL
```

保存：`Ctrl + O` → `Enter` → `Ctrl + X`

### 5.2 安装基础工具

```bash
sudo apt install -y curl git build-essential
```

---

## 六、安装 OpenClaw

### 6.1 安装 nvm 和 Node.js

```bash
# 安装 nvm（Node Version Manager）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 重新加载环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 安装 LTS 版 Node.js（推荐 22.x）
nvm install 22

# 设置默认版本
nvm alias default 22
nvm use 22

# 验证
node -v
npm -v
```

### 6.2 安装 OpenClaw

```bash
# 换国内 npm 源
npm config set registry https://registry.npmmirror.com

# 安装 OpenClaw
npm install -g openclaw

# 验证安装
openclaw --version
```

### 6.3 启用 systemd（重要！）

**如果启动 gateway 报错，必须启用 systemd：**

```bash
# 在 WSL 中执行
sudo vi /etc/wsl.conf
```

添加以下内容：

```ini
[boot]
systemd=true
```

保存退出后，在 **Windows PowerShell** 中执行：

```powershell
wsl --shutdown
```

然后重新启动 WSL，再执行：

```bash
openclaw gateway run
```

### 6.4 运行初始化向导

```bash
# 交互式配置
openclaw configure
```

或手动配置：

```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

**配置文件示例（飞书 + MiniMax 模型）：**

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

### 6.5 启动 Gateway

```bash
# 前台运行（调试用）
openclaw gateway run

# 或后台运行
openclaw gateway run &
```

看到以下输出就是成功：

```
Gateway started on http://127.0.0.1:18789
```

### 6.6 验证安装

```bash
# 检查进程
ps aux | grep openclaw

# 检查端口
netstat -tlnp | grep 18789

# 测试本地访问
curl http://127.0.0.1:18789
```

---

## 七、飞书连接配置

### 7.1 创建飞书应用

1. 打开 [飞书开放平台](https://open.feishu.cn/)
2. 创建企业自建应用
3. 获取 **App ID** 和 **App Secret**
4. 添加权限：
   - `im:message:send_as_bot`
   - `im:message:receive_as_bot`
   - `im:chat:message`

### 7.2 配置 webhook

在应用配置页面创建 webhook 事件订阅：
- 请求 URL：`你的TailscaleIP:18789/callback/feishu`
- 事件：`message` → `im.message.message.created`

### 7.3 添加 Bot 到群聊

将应用添加到群聊

### 7.4 在 WSL 中设置飞书

```bash
openclaw config edit channels.feishu.appId "你的AppId"
openclaw config edit channels.feishu.appSecret "你的AppSecret"

# 重启 Gateway
pkill -f openclaw
openclaw gateway run &
```

---

## 八、远程访问配置

### 方案 A：Tailscale 异地组网（推荐）

1. **在 Windows 端安装 Tailscale**：
   - 下载：https://tailscale.com/download
   - 登录你的 Tailscale 账号

2. **获取 Tailscale IP**：
   - 打开 Tailscale 客户端
   - 顶部的 IP 就是，格式 `100.x.x.x`

3. **从其他设备访问**：
   - 浏览器访问：`http://100.x.x.x:18789`

### 方案 B：在 WSL 里也装 Tailscale

```bash
# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 启动
sudo tailscale up

# 获取 WSL 里的 Tailscale IP
tailscale ip -4
```

---

## 九、常见问题解决

### ❌ 问题 1：执行 wsl --install 报错，需要更新内核

**解决方案**：手动下载安装微软官方 WSL2 内核更新包
- 地址：https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package

### ❌ 问题 2：wsl --list --verbose 显示 VERSION 是 1

**解决方案**：
```powershell
wsl --set-default-version 2
wsl --set-version 你的发行版名称 2
```

### ❌ 问题 3：忘记 Ubuntu 用户密码

**解决方案**：
```bash
wsl -d 发行版名称 -u root
passwd 你的用户名
exit
```

### ❌ 问题 4：镜像模式不生效，IP 还是 172 开头

**解决方案**：
1. 确认 `.wslconfig` 文件在正确的用户目录下
2. 执行 `wsl --shutdown`，等待 10 秒后再启动
3. 执行 `wsl --update` 更新 WSL

### ❌ 问题 5：启动 gateway 报错需要 systemd

**解决方案**：
```bash
# 创建 wsl.conf
sudo vi /etc/wsl.conf

# 添加
[boot]
systemd=true

# 退出 WSL，在 Windows 终端执行
wsl --shutdown

# 重新打开 WSL，启动 OpenClaw
openclaw gateway run
```

### ❌ 问题 6：npm 安装到错误路径

**解决方案**：
```bash
# 重新配置 npm
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 十、常用命令一览

| 命令 | 说明 |
|------|------|
| `openclaw gateway run` | 启动 Gateway |
| `openclaw status` | 查看状态 |
| `openclaw config edit` | 修改配置 |
| `openclaw models list` | 查看可用模型 |
| `openclaw skills list` | 查看已安装技能 |
| `openclaw logs` | 查看日志 |
| `openclaw update` | 更新 OpenClaw |
| `wsl --list --verbose` | 查看 WSL 状态 |
| `wsl --shutdown` | 关闭 WSL |

---

## 十一、推荐模型（国内可白嫖）

| 模型 | 注册地址 | 特点 |
|------|---------|------|
| **GLM** | https://open.bigmodel.cn/activity/trial-card/FRTMYUEKJL | 新用户免费 |
| **阿里百炼** | https://bailian.console.aliyun.com/ | 首月免费 CODE |
| **硅基流动** | https://cloud.siliconflow.cn/i/zrH6CauM | 新手券 |

---

## 十二、相关资源

- 官方文档：https://docs.openclaw.ai
- GitHub：https://github.com/openclaw/openclaw
- 问题反馈：https://github.com/openclaw/openclaw/issues

---

> 📝 教程编写：小雅 | 📅 更新：2026-03-12
> 📚 参考：朋友提供的详细部署文档 + OpenClaw 官方文档
