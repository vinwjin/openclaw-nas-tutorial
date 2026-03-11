# OpenClaw 常用命令大全

> 整理：王靖 | 更新：2026-03-12

---

## 🚀 基础命令

```bash
# 查看版本
openclaw --version

# 查看帮助
openclaw --help

# 配置向导（首次设置）
openclaw configure
```

---

## 📊 状态查看

```bash
# 查看基本状态
openclaw status

# 查看详细状态（所有渠道、会话）
openclaw status --all

# 查看深度状态（健康检查）
openclaw status --deep

# 查看 Gateway 探测
openclaw gateway probe
```

---

## 🎛️ Gateway 管理

```bash
# 启动 Gateway
openclaw gateway run

# 停止 Gateway
openclaw gateway stop

# 重启 Gateway
openclaw gateway restart

# 查看 Gateway 状态
openclaw gateway status

# 安装 Gateway 服务（开机自启）
openclaw gateway install
```

---

## 💬 渠道管理

```bash
# 查看渠道列表
openclaw channels list

# 添加渠道
openclaw channels add

# 查看渠道详细状态
openclaw channels status feishu
```

---

## 🔐 配对管理

```bash
# 查看待批准的配对列表
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu <配对码>

# 拒绝配对
openclaw pairing reject feishu <配对码>
```

---

## 🔧 技能管理

```bash
# 查看技能列表
openclaw skills list

# 查看已加载的技能
openclaw skills loaded

# 安装技能
openclaw skills install <技能名>

# 卸载技能
openclaw skills uninstall <技能名>
```

---

## 📝 日志与调试

```bash
# 实时查看日志
openclaw logs --follow

# 查看最近日志（最后50行）
openclaw logs -n 50

# 查看特定级别的日志
openclaw logs --level error
```

---

## 🖥️ 控制面板

```bash
# 打开控制面板（带 Token，自动打开浏览器）
openclaw dashboard

# 只输出 URL，不打开浏览器
openclaw dashboard --no-open

# 打开 TUI 终端界面
openclaw tui
```

---

## 🔒 安全与健康

```bash
# 健康检查
openclaw doctor

# 安全审计
openclaw security audit

# 深度安全审计
openclaw security audit --deep

# 模型列表
openclaw models list
```

---

## 🔄 更新管理

```bash
# 检查更新
openclaw update status

# 更新到最新版
openclaw update

# 切换更新通道（stable/beta/dev）
openclaw update --# 非交互式更新
openclaw update --yes
```

---

## ⭐ 常用组合命令

```bash
# 完整检查channel stable

状态
openclaw status --all && openclaw gateway probe

# 实时监控日志
openclaw logs --follow

# 调试飞书渠道
openclaw logs --follow | grep feishu

# 查看 Gateway 是否运行
openclaw gateway status
```

---

## 📌 快速诊断流程

```bash
# 1. 检查 Gateway 是否运行
openclaw gateway status

# 2. 检查渠道状态
openclaw channels list

# 3. 查看实时日志
openclaw logs --follow

# 4. 安全审计
openclaw security audit
```

---

## 🆘 常见问题

### Gateway 启动失败：端口被占用

```bash
# 查看占用端口的进程
lsof -i :18789

# 停止占用进程
kill <PID>

# 或者直接停止 Gateway
openclaw gateway stop
```

### 飞书渠道未连接

```bash
# 查看飞书渠道状态
openclaw channels status feishu

# 查看飞书相关日志
openclaw logs --follow | grep feishu

# 重启 Gateway
openclaw gateway restart
```

### 配对问题

```bash
# 查看配对列表
openclaw pairing list feishu

# 批准配对
openclaw pairing approve feishu <配对码>
```

---

## 📋 环境说明

| 项目 | 说明 |
|------|------|
| Gateway 端口 | 18789 |
| 控制面板 | http://127.0.0.1:18789 |
| 配置文件 | ~/.openclaw/openclaw.json |
| 工作目录 | ~/.openclaw/workspace |

---

*持续更新中...*
