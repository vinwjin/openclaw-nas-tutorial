# OpenClaw 远程访问方案：Tailscale + SSH 隧道

## 方案概述

通过 **Tailscale 异地组网** + **SSH 隧道**，在外网安全访问家中的 OpenClaw 控制台。

## 适用场景

- ✅ 家中部署了 OpenClaw (如飞牛 NAS)
- ✅ 需要在外网安全访问控制台
- ✅ 不想暴露公网端口
- ✅ 官方推荐的方案

## 核心原理

### Tailscale 组网

Tailscale 可以安装在局域网**任意设备**上（NAS、Ubuntu、路由器、其他电脑等），只需要一台设备安装并设置**子网路由**，就能让整个 Tailscale 网络像虚拟局域网一样工作。

### SSH 隧道

SSH 隧道是**点对点**的，不是中继。直接连接到目标设备。

## 准备工作

### 1. 确定设备信息

| 设备 | IP | 用户名 | 说明 |
|------|-----|--------|------|
| NAS | 你的NAS IP | 你的用户名 | 运行 OpenClaw |
| 安装 Tailscale 的设备 | 你的设备IP | 你的用户名 | 可以是 NAS/电脑/路由器 |
| 电脑 | 外网 | - | 外出时使用 |

### 2. 需要安装的软件

| 软件 | 说明 |
|------|------|
| Tailscale | 安装在局域网任意设备上，用于组网 |
| SSH 客户端 | Windows: Git Bash / PowerShell / Xshell |

---

## 配置步骤

### 第一步：在局域网设备上安装 Tailscale

Tailscale 可以安装在任意局域网设备上，推荐安装在 NAS 或稳定的长时间运行的设备上。

#### 方法1：通过 NAS 应用中心

1. 登录 NAS 后台
2. 打开 **应用中心**
3. 搜索 **Tailscale** 并安装

#### 方法2：手动安装（需要 root 权限）

```bash
# SSH 到设备
ssh 你的用户名@设备IP

# 安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
```

---

### 第二步：在电脑上安装 Tailscale

1. 打开 https://tailscale.com
2. 下载并安装对应版本 (Windows/Mac/Linux)
3. 登录（使用与局域网设备相同的账号）

---

### 第三步：配置 Tailscale

#### 1. 启用子网路由

在**安装了 Tailscale 的局域网设备**上执行：

```bash
sudo tailscale up --advertise-routes=你的局域网段

# 例如：
sudo tailscale up --advertise-routes=192.168.1.0/24
# 或
sudo tailscale up --advertise-routes=10.0.0.0/24
```

#### 2. 启用出口节点（可选）

在同一设备上执行：

```bash
sudo tailscale up --advertise-exit-node
```

#### 3. 在 Tailscale 后台批准

1. 打开 https://login.tailscale.com/admin
2. 登录你的账号
3. 找到安装 Tailscale 的设备
4. 启用 **Subnet routes** 和 **Exit node**

---

### 第四步：配置 OpenClaw 为本地模式

#### 1. SSH 到 NAS

```bash
ssh 你的用户名@你的NASIP
```

#### 2. 修改 OpenClaw 配置

```bash
# 编辑配置文件
nano ~/.openclaw/openclaw.json
```

找到 gateway 部分，添加或修改：

```json
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "你的token"
    }
  }
}
```

#### 3. 重启 OpenClaw

```bash
# 停止现有进程
pkill -f openclaw

# 重新启动
openclaw gateway run --port 18789
```

---

### 第五步：建立 SSH 隧道

在**你的电脑**上打开终端，执行：

```bash
ssh -L 18789:localhost:18789 你的NAS用户名@你的NASIP

# 完整示例：
ssh -L 18789:localhost:18789 vinwjin@10.0.0.222
```

> ⚠️ **重要**：确保电脑和 NAS 都已连接 Tailscale

---

### 第六步：访问控制台

1. 浏览器打开：`http://localhost:18789`
2. 输入 Token 登录

#### 获取 Token

在 NAS 上执行：

```bash
cat ~/.openclaw/openclaw.json | jq -r '.gateway.auth.token'
```

---

## 进阶配置

### 使用 SSH 工具保存会话（推荐）

#### Xshell / SecureCRT

1. 新建 SSH 会话
2. 连接地址：你的 NAS IP
3. 设置隧道：
   - 本地端口：18789
   - 远程端口：127.0.0.1:18789
4. 保存会话，下次一键连接

#### Windows PowerShell 脚本

保存为 `connect.ps1`：

```powershell
$NAS_IP = "你的NASIP"
$NAS_USER = "你的用户名"

ssh -L 18789:localhost:18789 $NAS_USER@$NAS_IP
```

---

## 原理解释

### 为什么用 loopback 模式？

OpenClaw 默认监听 `0.0.0.0`（所有网卡），局域网任何设备都能访问。

改为 `loopback` (127.0.0.1) 后：
- ❌ 局域网其他设备无法直接访问
- ✅ 更安全
- ✅ 配合 SSH 隧道使用

### SSH 隧道是点对点的

```
[电脑localhost:18789] ←SSH隧道→ [NAS 127.0.0.1:18789]
```

SSH 隧道直接连接到目标设备，不是中继转发。

### Tailscale 组网优势

- 不需要公网 IP
- 不需要配置路由器端口转发
- 全程加密
- 速度稳定
- 安装在任意局域网设备上即可

---

## 常见问题

### Q1: SSH 隧道是什么？

**答**：SSH 隧道是一种点对点的加密通道，把远程服务器的端口直接映射到本地电脑。

### Q2: 为什么不用公网访问？

**答**：公网访问需要暴露端口，有安全风险。Tailscale + SSH 隧道方案：
- 全程加密
- 不暴露公网端口
- 官方推荐

### Q3: Tailscale 装在哪台设备上？

**答**：装在局域网任意设备上都可以（NAS、电脑、路由器等），只需要一台设备设置子网路由。

### Q4: 每次都要手动建立隧道吗？

**答**：可以用 SSH 工具 (Xshell, SecureCRT, Termius 等) 保存会话，下次一键连接。

### Q5: Token 是什么？

**答**：Token 是 OpenClaw 的认证令牌，用于登录控制台。

---

## 快速命令汇总

| 操作 | 命令 |
|------|------|
| 建立隧道 | `ssh -L 18789:localhost:18789 用户名@NAS_IP` |
| 访问控制台 | http://localhost:18789 |
| 获取 Token | `cat ~/.openclaw/openclaw.json \| jq -r '.gateway.auth.token'` |
| 重启 OpenClaw | `pkill -f openclaw && openclaw gateway run --port 18789` |

---

## 总结

1. **局域网任意设备安装 Tailscale** → 设置子网路由
2. **NAS 配置 OpenClaw** → 绑定 127.0.0.1
3. **电脑连接 Tailscale** → 加入网络
4. **电脑建立 SSH 隧道** → `ssh -L 18789:localhost:18789 用户名@NAS_IP`
5. **浏览器访问** → http://localhost:18789

---

*文档版本: 1.1*  
*更新日期: 2026-03-11*
