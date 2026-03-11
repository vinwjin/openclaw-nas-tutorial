# OpenClaw Skills 完整报告

> 整理日期：2026-03-12
> 来源：https://open-claw.me/skills

---

## 一、什么是 OpenClaw Skills？

OpenClaw Skills（技能）是扩展 OpenClaw 功能的插件，让 AI 助手能够执行各种任务。

### 核心功能
- 集成第三方服务（GitHub、Discord、Slack 等）
- 自动化工作流
- 扩展 AI 能力边界

---

## 二、Skills 分类目录

### 1. 开发工具类 (Development)

| 技能 | 说明 | 热度 |
|------|------|------|
| **github** | 通过 gh CLI 操作 GitHub (issues, PRs, repos) | ⭐⭐⭐⭐⭐ |
| **conventional-commits** | 规范提交生成 | ⭐⭐⭐⭐ |
| **pr-reviewer** | PR 审查 | ⭐⭐⭐⭐ |
| **nextjs-expert** | Next.js 专家 | ⭐⭐⭐⭐ |
| **shadcn-ui** | shadcn/ui 组件助手 | ⭐⭐⭐⭐ |
| **docker-essentials** | Docker 必备工具 | ⭐⭐⭐⭐ |
| **kubernetes** | K8s 集群管理 | ⭐⭐⭐⭐ |
| **sentry-cli** | Sentry 错误追踪 | ⭐⭐⭐ |

### 2. 云服务类 (Cloud)

| 技能 | 说明 |
|------|------|
| **aws-infra** | AWS 基础设施管理 |
| **coolify** | Coolify 部署平台 |
| **vercel** | Vercel 部署平台 |

### 3. 通讯平台类 (Messaging)

| 技能 | 说明 | 推荐度 |
|------|------|--------|
| **discord** | Discord 服务器管理 | ⭐⭐⭐⭐⭐ |
| **slack** | Slack 工作区自动化 | ⭐⭐⭐⭐ |
| **home-assistant** | Home Assistant 智能家居 | ⭐⭐⭐ |
| **telegram** | Telegram 机器人 | ⭐⭐⭐⭐ |
| **feishu** | 飞书 (已有) | ✅ |

### 4. 安全工具类 (Security)

| 技能 | 说明 |
|------|------|
| **1password** | 1Password 密码管理 |
| **bitwarden** | Bitwarden 开源密码管理 |
| **ggshield-scanner** | ggshield 安全扫描 |

### 5. AI/搜索类

| 技能 | 说明 |
|------|------|
| **fal-ai** | Fal AI 图像/视频生成 |
| **tavily-web-search** | Tavily AI 搜索 |
| **weather** | 天气查询 (已安装) |

### 6. 系统管理类

| 技能 | 说明 |
|------|------|
| **tailscale** | Tailscale 网络管理 |
| **system-resource-monitor** | 系统资源监控 (已安装) |
| **automation-workflows** | 自动化工作流 (已安装) |

---

## 三、安装指南

### 基本命令

```bash
# 安装技能
npx clawhub@latest install <技能名>

# 示例
npx clawhub@latest install github
npx clawhub@latest install discord

# 查看已安装技能
clawhub list

# 卸载技能
clawhub uninstall <技能名>
```

### 安装前检查

```bash
# 查看技能详情
clawhub inspect <技能名>

# 查看技能文件
clawhub inspect <技能名> --files
```

---

## 四、常见问题 FAQ

### Q1: 安装技能需要什么前提条件？

| 技能 | 前提条件 |
|------|----------|
| github | 安装 gh CLI，配置 GitHub Token |
| discord | Discord Bot Token |
| slack | Slack Bot Token |
| telegram | Telegram Bot Token |
| aws-infra | AWS CLI + AWS 凭证 |
| docker-essentials | Docker + Docker Compose |
| kubernetes | kubectl + kubeconfig |
| 1password | 1Password CLI + 订阅 |
| bitwarden | Bitwarden CLI |

### Q2: 技能安全吗？

**建议安装前审查：**

```bash
# 查看技能源代码
clawhub inspect <技能名> --files

# 查看权限
clawhub inspect <技能名>
```

**安全建议：**
- 只安装来自可信作者的技能
- 检查技能权限范围
- 定期更新技能

### Q3: 技能安装失败怎么办？

| 错误 | 解决方案 |
|------|----------|
| 网络超时 | 配置国内镜像：`npm config set registry https://registry.npmmirror.com` |
| 权限不足 | 检查 Node.js 权限 |
| 依赖缺失 | 根据技能文档安装对应 CLI 工具 |

### Q4: 如何更新技能？

```bash
# 更新到最新版
clawhub update <技能名>

# 查看可更新技能
clawhub outdated
```

### Q5: 技能冲突怎么办？

```bash
# 卸载冲突技能
clawhub uninstall <冲突技能>

# 重新安装
clawhub install <技能名>
```

### Q6: 技能放在哪里？

| 类型 | 位置 |
|------|------|
| 全局技能 | `~/.npm-global/lib/node_modules/openclaw/extensions/` |
| 本地技能 | `~/.openclaw/extensions/` |

### Q7: 如何开发自己的技能？

参考官方文档：
- 技能开发指南：https://docs.openclaw.ai/skills

---

## 五、推荐安装组合

### 基础组合（推荐）
```bash
npx clawhub@latest install github
npx clawhub@latest install discord
npx clawhub@latest install weather
```

### 开发者组合
```bash
npx clawhub@latest install docker-essentials
npx clawhub@latest install kubernetes
npx clawhub@latest install nextjs-expert
```

### 运维组合
```bash
npx clawhub@latest install aws-infra
npx clawhub@latest install tailscale
npx clawhub@latest install system-resource-monitor
```

---

## 六、技能权限说明

安装技能时，技能会获得以下权限：

| 权限类型 | 说明 |
|----------|------|
| 文件读写 | 读取/写入文件 |
| 命令执行 | 执行 shell 命令 |
| 网络请求 | 访问外部 API |
| 凭证访问 | 读取配置中的 API Keys |

**安全提醒：**
- 陌生技能先审查再安装
- 敏感凭证不要暴露给不信任的技能

---

## 七、官方资源

| 资源 | 链接 |
|------|------|
| 技能市场 | https://open-claw.me/skills |
| 官方文档 | https://docs.openclaw.ai |
| GitHub | https://github.com/openclaw/openclaw |
| Discord 社区 | https://discord.com/invite/clawd |

---

*持续更新中...*
