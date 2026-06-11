# OpenClaw 安装部署指南

> 本文档总结了 OpenClaw Gateway 和 Node 的安装配置经验，供后续部署参考。

---

## 1. 架构概述

```
┌─────────────┐                  ┌─────────────┐                  ┌─────────────┐
│   手机/PC    │  ─── 消息 ───►  │   Gateway   │  ─── 任务 ───►  │    Node     │
│  (控制端)    │  ◄─── 结果 ───  │  (主控服务器) │  ◄─── 返回 ───  │  (执行节点)  │
└─────────────┘                  └─────────────┘                  └─────────────┘
```

| 组件 | 部署位置 | 作用 |
|------|----------|------|
| **Gateway** | 云服务器（公网 IP） | 消息路由、任务调度、会话管理 |
| **Node** | 本地电脑/服务器 | 执行命令、提供算力和能力 |
| **Channel** | Gateway 上配置 | 钉钉/飞书/Telegram 等消息入口 |

---

## 2. Gateway 安装（云服务器）

### 2.1 环境要求

| 依赖 | 版本要求 |
|------|----------|
| Node.js | ≥ 22.12.0 |
| pnpm | 最新版 |
| 系统 | Linux (推荐) / Windows / macOS |

### 2.2 安装步骤（Linux）

```bash
# 1. 安装 Node.js 22+
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo yum install -y nodejs

# 2. 安装 pnpm
npm install -g pnpm

# 3. 安装 OpenClaw
npm install -g openclaw

# 4. 验证安装
openclaw --version

# 5. 运行初始化向导
openclaw onboard
```

### 2.3 配置 Gateway

```bash
# 交互式配置
openclaw configure

# 或直接编辑配置文件
vim ~/.openclaw/openclaw.json
```

### 2.4 启动 Gateway

```bash
# 前台运行（调试用）
openclaw gateway run --port 18789 --verbose

# 安装为系统服务（生产环境）
openclaw gateway install --port 18789

# 管理服务
openclaw gateway status
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
```

### 2.5 Gateway 配置要点

**配置文件位置**: `~/.openclaw/openclaw.json`

**关键配置项**:

```json
{
  "gateway": {
    "port": 18789,
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "<你的Gateway Token>"
    },
    "trustedProxies": ["127.0.0.1", "::1"]
  },
  "channels": {
    "dingtalk": {
      "clientId": "<钉钉ClientId>",
      "clientSecret": "<钉钉ClientSecret>",
      "corpId": "<企业ID>"
    },
    "feishu": {
      "appId": "<飞书AppId>",
      "appSecret": "<飞书AppSecret>",
      "enabled": true
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "zai/glm-5"
      }
    }
  }
}
```

**bind 选项说明**:

| 值 | 说明 | 适用场景 |
|-----|------|----------|
| `loopback` | 仅本机访问 | 需要通过 SSH 隧道或 Nginx 代理 |
| `lan` | 监听所有接口 | 允许局域网/公网直接访问 |

### 2.6 Nginx 反向代理（可选）

如果需要 HTTPS 访问，配置 Nginx：

```nginx
server {
    listen 3001 ssl;
    server_name your-domain.com;

    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

---

## 3. Node 安装（执行节点）

### 3.1 环境要求

| 依赖 | 版本要求 |
|------|----------|
| Node.js | ≥ 22.12.0 |
| 系统 | Windows / Linux / macOS |

### 3.2 安装步骤

```bash
# 安装 Node.js 22+ (Windows: 下载安装包)
# https://nodejs.org/

# 安装 OpenClaw
npm install -g openclaw

# 验证安装
openclaw --version
```

### 3.3 Node 配置文件

**配置文件位置**: `~/.openclaw/node.json` (Windows: `C:\Users\<用户名>\.openclaw\node.json`)

**配置模板**:

```json
{
  "version": 1,
  "nodeId": "<从Gateway获取>",
  "token": "<从Gateway获取>",
  "displayName": "<节点名称>",
  "gateway": {
    "host": "<Gateway IP地址>",
    "port": 18789,
    "tls": false
  }
}
```

**配置字段说明**:

| 字段 | 必填 | 说明 | 获取方式 |
|------|------|------|----------|
| `nodeId` | ✅ | 设备唯一标识 | 首次配对后由 Gateway 生成 |
| `token` | ✅ | 设备认证 Token | 首次配对或轮换时生成 |
| `displayName` | ✅ | 节点显示名称 | 自定义，如 `ZEN_LIZ` |
| `gateway.host` | ✅ | Gateway IP | Gateway 服务器的公网 IP |
| `gateway.port` | ✅ | Gateway 端口 | 默认 18789 (HTTP) 或 3001 (HTTPS) |
| `gateway.tls` | ✅ | 是否使用 TLS | 18789 用 false，3001 用 true |

### 3.4 首次配对流程

**步骤 1**: 在 Node 上发起连接

```bash
# Windows PowerShell
$env:OPENCLAW_GATEWAY_TOKEN = "<Gateway Token>"
openclaw node run --host <Gateway IP> --port 18789 --display-name "NODE_NAME"
```

**步骤 2**: 在 Gateway 上批准配对

```bash
# 查看待配对设备
openclaw devices list

# 批准配对
openclaw devices approve <requestId>
```

**步骤 3**: Node 会自动获取 nodeId 和 token，保存在 `node.json` 中

### 3.5 启动 Node

```powershell
# Windows PowerShell
$env:OPENCLAW_GATEWAY_TOKEN = "<Gateway Token>"
openclaw node run

# Linux/macOS
export OPENCLAW_GATEWAY_TOKEN="<Gateway Token>"
openclaw node run
```

### 3.6 验证连接

在 Gateway 上执行：

```bash
openclaw nodes status
# 应该看到节点显示 paired · connected
```

---

## 4. Token 管理

### 4.1 Token 类型

| Token 类型 | 存储位置 | 用途 | 命令 |
|------------|----------|------|------|
| **Gateway Token** | Gateway 配置 | Gateway 访问认证 | 环境变量 `OPENCLAW_GATEWAY_TOKEN` |
| **设备 Token** | node.json | Node 身份认证 | `openclaw devices rotate` |

### 4.2 查看 Gateway Token

```bash
# 在 Gateway 服务器上
openclaw config get gateway.auth
```

### 4.3 轮换设备 Token

如果遇到 `device token mismatch` 错误：

```bash
# 在 Gateway 上执行
# 1. 获取完整的 nodeId
openclaw nodes status

# 2. 轮换 Token
openclaw devices rotate --device <完整nodeId> --role node --json

# 3. 将返回的新 token 更新到 Node 的 node.json 中
```

---

## 5. 常见问题排查

### 5.1 错误对照表

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| `Unexpected server response: 400` | 端口或 TLS 配置错误 | 检查 node.json 的 port 和 tls |
| `device token mismatch` | Token 不匹配 | 执行 `openclaw devices rotate` |
| `pairing required` | 设备未配对 | 执行 `openclaw devices approve` |
| `self-signed certificate` | 自签名证书问题 | 添加 `--tls-fingerprint` 参数 |
| `Gateway connection refused` | Gateway 未运行或端口不通 | 检查 Gateway 状态和防火墙 |

### 5.2 完整排查流程

```bash
# 1. 检查 Gateway 状态
openclaw gateway status

# 2. 检查网络连通性
curl http://<Gateway IP>:18789/health

# 3. 检查 Node 配置
cat ~/.openclaw/node.json

# 4. 检查节点状态
openclaw nodes status

# 5. 检查设备列表
openclaw devices list

# 6. 查看 Gateway 日志
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

### 5.3 nodeId 不匹配问题

**症状**: `device token mismatch`

**原因**: 本地 node.json 的 nodeId 与 Gateway 上记录的不一致

**解决**:
1. 在 Gateway 上执行 `openclaw nodes status` 获取正确的 nodeId
2. 更新本地 node.json 中的 nodeId
3. 如果仍失败，执行 Token 轮换

---

## 6. 生产环境部署建议

### 6.1 Gateway 服务器

- 使用 systemd 管理 Gateway 服务
- 配置 Nginx 反向代理 + HTTPS
- 设置防火墙白名单
- 定期备份配置文件

### 6.2 Node 节点

- 使用 `openclaw node install` 安装为系统服务
- 配置自动重连机制
- 保持 Node.js 版本更新

### 6.3 安全建议

- 定期轮换 Token
- 限制 Gateway 访问 IP
- 使用 HTTPS 而非 HTTP
- 不要在代码仓库中提交 Token

---

## 7. 快速参考卡片

### Gateway 常用命令

```bash
openclaw gateway status      # 查看状态
openclaw gateway start       # 启动
openclaw gateway stop        # 停止
openclaw gateway restart     # 重启
openclaw gateway install     # 安装为服务
openclaw nodes status        # 查看连接的 Node
openclaw devices list        # 查看设备列表
openclaw devices approve ID  # 批准配对
openclaw devices rotate --device ID --role node  # 轮换 Token
```

### Node 常用命令

```bash
# Windows
$env:OPENCLAW_GATEWAY_TOKEN = "<token>"
openclaw node run

# Linux/macOS
export OPENCLAW_GATEWAY_TOKEN="<token>"
openclaw node run

openclaw node install        # 安装为服务
openclaw node restart        # 重启服务
```

---

## 8. 当前部署信息

| 项目 | 值 |
|------|-----|
| **Gateway 服务器** | 114.55.248.111 |
| **Gateway 端口** | 18789 (HTTP) / 3001 (HTTPS) |
| **Gateway Token** | `d6d7858143a8faad916306a8d3393ff0e6509f3ed29b1738` |
| **已配对 Node** | ZEN_LIZ (Windows 笔记本) |
| **ZEN_LIZ Node ID** | `5a620187398452c733e6bcf68589e3671bc1230a76fc8009993e9ba8cdf84238` |
| **ZEN_LIZ Token** | `5hhf9KayJrcV1NYI0N3FY8tZglHXFjbPGQoJiyD9xjA` |

---

*文档版本: 1.0*
*创建时间: 2026-02-20*
*基于实际部署经验总结*
