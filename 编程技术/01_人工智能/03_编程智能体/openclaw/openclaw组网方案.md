# OpenClaw 组网方案设计文档

## 1. 项目概述

### 1.1 核心理念：三层分离架构

```
┌─────────────┐     指令      ┌─────────────┐     调度      ┌─────────────┐
│    手机      │ ───────────► │   Gateway   │ ───────────► │   笔记本     │
│  (控制端)    │ ◄─────────── │  (主控端)    │ ◄─────────── │  (执行端)    │
│             │     结果      │             │    返回      │             │
└─────────────┘              └─────────────┘              └─────────────┘
```

| 层级 | 组件 | 职责 | 特点 |
|------|------|------|------|
| **控制层** | 手机 | 发送指令，接收结果 | 移动便捷，通过聊天 App 操作 |
| **主控层** | 云服务器 Gateway | 消息路由，任务调度，会话管理 | 永远在线，固定 IP |
| **执行层** | 笔记本/Node | 执行任务，提供算力和能力 | 按需启动，能力丰富 |

### 1.2 设计目标

1. **分离关注点**：控制、调度、执行三层解耦
2. **移动优先**：用日常聊天应用控制一切
3. **弹性算力**：Node 按需接入，用完即走
4. **安全可控**：HTTPS + WebSocket + 设备配对认证

---

## 2. OpenClaw 架构原理

### 2.1 核心组件

| 组件 | 说明 | 在本方案中的角色 |
|------|------|------------------|
| **Gateway** | 中央服务器，处理消息路由和会话管理 | 部署在云服务器，永远在线 |
| **Channel** | 消息渠道（钉钉/Telegram/Discord 等） | 连接手机的入口 |
| **Agent** | AI 执行单元，可运行在不同节点 | 负责解释指令并调度执行 |
| **Node** | 能力提供者，通过 WebSocket 连接到 Gateway | 笔记本/手机作为执行节点 |
| **Operator** | 控制平面客户端 | 手机通过 Channel 扮演此角色 |

### 2.2 关键概念：Node 能力

Node 是连接到 Gateway 的**外围设备**，暴露以下能力：

| 能力类型 | 命令前缀 | 说明 |
|----------|----------|------|
| **system** | `system.run`, `system.which` | 执行系统命令（headless node） |
| **canvas** | `canvas.snapshot`, `canvas.eval`, `canvas.present` | WebView 控制、截图、JS 执行 |
| **camera** | `camera.snap`, `camera.clip` | 拍照、录像 |
| **screen** | `screen.record` | 屏幕录制 |
| **location** | `location.get` | 获取位置信息 |
| **sms** | `sms.send` | 发送短信（Android） |

**重要**：
- Nodes 是**外围设备**，不运行 Gateway 服务
- 消息（Telegram/钉钉等）始终发送到 **Gateway**，不是 Node
- Gateway 负责将任务**路由**到对应的 Node 执行

---

## 3. Node 详解

### 3.1 Node 类型

| 类型 | 平台 | 能力 | 适用场景 |
|------|------|------|----------|
| **Headless Node Host** | Windows/Linux/macOS | `system.run`, `system.which` | 服务器命令执行、CI/CD |
| **macOS Companion App** | macOS | `system.run`, `system.notify`, canvas, camera, screen | 开发机、日常办公 |
| **iOS App** | iOS | canvas, camera, screen, location | 移动办公、现场操作 |
| **Android App** | Android | canvas, camera, screen, location, sms | 移动办公、自动化 |

### 3.2 Node 配对流程

**WS nodes 使用 device pairing**（设备配对）：

```
1. Node 发起 WebSocket 连接，携带设备身份
2. Gateway 创建 device pairing request (role: node)
3. 管理员批准配对（CLI 或 Web UI）
4. Node 获得 token，完成配对
```

**CLI 命令**：
```bash
# 查看待配对设备
openclaw devices list

# 批准配对
openclaw devices approve <requestId>

# 拒绝配对
openclaw devices reject <requestId>

# 查看节点状态
openclaw nodes status

# 查看节点详情
openclaw nodes describe --node <idOrNameOrIp>
```

**注意**：
- `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) 是**独立**的 gateway-owned store
- 它**不**控制 WebSocket connect 握手
- WS nodes 必须通过 `openclaw devices approve` 批准

### 3.3 快速启动指南

#### 启动步骤（Windows 笔记本 → 114 Gateway）

**步骤 1：确保 Gateway 运行（114 服务器）**
```bash
# SSH 到 114 检查
openclaw gateway status

# 如果没运行，启动它
openclaw gateway start
```

**步骤 2：本地启动 Node（Windows）**
```powershell
# 设置 Gateway Token 环境变量
$env:OPENCLAW_GATEWAY_TOKEN = "d6d7858143a8faad916306a8d3393ff0e6509f3ed29b1738"

# 启动 Node（前台运行，可看到日志）
openclaw node run
```

**步骤 3：验证连接（在 114 上）**
```bash
openclaw nodes status
# 应该看到 ZEN_LIZ 显示 paired · connected
```

#### Token 过期时的修复步骤

如果遇到 `device token mismatch` 错误：

```bash
# 在 114 服务器上执行，重新颁发 Token
openclaw devices rotate --device <完整设备ID> --role node --json

# 示例（ZEN_LIZ）
openclaw devices rotate --device 5a620187398452c733e6bcf68589e3671bc1230a76fc8009993e9ba8cdf84238 --role node --json
```

然后将返回的新 token 更新到本地 `~/.openclaw/node.json` 文件中。

---

### 3.4 Headless Node Host 配置

#### 启动 Node（前台）
```bash
# 基本启动（使用配置文件）
$env:OPENCLAW_GATEWAY_TOKEN = "<gateway-token>"
openclaw node run

# 指定参数启动
openclaw node run --host 114.55.248.111 --port 18789 --display-name "ZEN_LIZ"
```

#### 启动 Node（后台服务）
```bash
# 安装为系统服务（只需一次）
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"

# 重启服务
openclaw node restart
```

#### 配置文件 (~/.openclaw/node.json)

**重要**：`nodeId` 和 `token` 必须与 Gateway 上的配对信息一致。

```json
{
  "version": 1,
  "nodeId": "5a620187398452c733e6bcf68589e3671bc1230a76fc8009993e9ba8cdf84238",
  "token": "5hhf9KayJrcV1NYI0N3FY8tZglHXFjbPGQoJiyD9xjA",
  "displayName": "ZEN_LIZ",
  "gateway": {
    "host": "114.55.248.111",
    "port": 18789,
    "tls": false
  }
}
```

**配置字段说明**：

| 字段 | 说明 | 获取方式 |
|------|------|----------|
| `nodeId` | 设备唯一标识 | `openclaw nodes status` 查看 |
| `token` | 设备认证 Token | `openclaw devices rotate` 生成 |
| `displayName` | 节点显示名称 | 自定义 |
| `gateway.host` | Gateway IP 地址 | 114.55.248.111 |
| `gateway.port` | Gateway 端口 | 18789 (HTTP) 或 3001 (HTTPS) |
| `gateway.tls` | 是否使用 TLS | 18789 用 false，3001 用 true |

### 3.4 SSH 隧道连接（loopback bind）

如果 Gateway 绑定到 loopback（`gateway.bind=loopback`），远程 Node 无法直接连接。

**解决方案：SSH 隧道**

```bash
# 终端 A：建立隧道（保持运行）
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# 终端 B：通过隧道连接
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

### 3.5 配置默认执行 Node

```bash
# 配置 exec 默认使用 Node
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "ZEN_LIZ"

# 或在会话中临时设置
/exec host=node security=allowlist node=ZEN_LIZ
```

### 3.6 命令审批（Exec Approvals）

审批配置存储在 Node 本地：`~/.openclaw/exec-approvals.json`

```bash
# 从 Gateway 添加允许列表
openclaw approvals allowlist add --node ZEN_LIZ "/usr/bin/git"
openclaw approvals allowlist add --node ZEN_LIZ "/usr/bin/npm"
openclaw approvals allowlist add --node ZEN_LIZ "C:\\Windows\\System32\\cmd.exe"
```

---

## 4. 应用案例

### 4.1 案例一：手机远程执行笔记本上的脚本

**场景**：在钉钉发送消息，让笔记本执行 Python 脚本

**架构**：
```
钉钉消息 → Gateway (114服务器) → Node (笔记本 ZEN_LIZ) → 执行脚本 → 返回结果
```

**操作流程**：
1. 确保 Node 已连接：`openclaw nodes status` 显示 `ZEN_LIZ: paired · connected`
2. 配置默认 Node：
   ```bash
   openclaw config set tools.exec.host node
   openclaw config set tools.exec.node "ZEN_LIZ"
   ```
3. 在钉钉发送：`帮我运行 python --version`
4. Gateway 接收消息，AI 调用 `exec` 工具，路由到 ZEN_LIZ 执行
5. 执行结果返回到钉钉

### 4.2 案例二：远程查看笔记本屏幕

**场景**：在手机上查看笔记本当前屏幕内容

**CLI 方式**：
```bash
# 截图
openclaw nodes canvas snapshot --node ZEN_LIZ --format png

# 屏幕录制（10秒）
openclaw nodes screen record --node ZEN_LIZ --duration 10s --fps 10
```

**通过聊天**：
- 发送：`帮我截个屏`
- AI 调用 `nodes` 工具的 `screen_record` 或 `canvas.snapshot` action
- 返回图片 `MEDIA:<path>`

### 4.3 案例三：远程开发环境管理

**场景**：手机控制笔记本启动/停止开发服务

**操作示例**：
```
钉钉发送：
"帮我启动本地的前端开发服务器"

AI 执行：
1. 调用 exec tool，host=node, node=ZEN_LIZ
2. 执行：cd /path/to/project && npm run dev
3. 返回：服务器已启动在 http://localhost:3000
```

### 4.4 案例四：多节点负载分担

**场景**：两台笔记本分别处理不同任务

**配置**：
```json
{
  "agents": {
    "list": [
      {
        "id": "dev-agent",
        "tools": {
          "exec": {
            "host": "node",
            "node": "ZEN_LIZ"
          }
        }
      },
      {
        "id": "test-agent",
        "tools": {
          "exec": {
            "host": "node",
            "node": "ZEN_LIC_NODE"
          }
        }
      }
    ]
  }
}
```

**使用**：
- 开发任务路由到 ZEN_LIZ
- 测试任务路由到 ZEN_LIC_NODE

### 4.5 案例五：远程摄像头监控

**场景**：通过手机查看笔记本/手机摄像头

**CLI**：
```bash
# 拍照
openclaw nodes camera snap --node <device> --facing front

# 录像（10秒）
openclaw nodes camera clip --node <device> --duration 10s
```

**注意**：Node App 必须在前台运行

### 4.6 案例六：发送系统通知

**场景**：任务完成时在笔记本上弹出通知

**CLI**：
```bash
openclaw nodes notify --node ZEN_LIZ \
  --title "任务完成" \
  --body "构建已完成，耗时 5 分钟"
```

---

## 5. Node 命令速查表

### 5.1 配对管理
```bash
openclaw devices list                    # 查看所有设备
openclaw devices approve <requestId>     # 批准配对
openclaw devices reject <requestId>      # 拒绝配对
openclaw nodes status                    # 查看节点状态
openclaw nodes describe --node <name>    # 查看节点详情
openclaw nodes rename --node <id> --name "NewName"  # 重命名
```

### 5.2 Node Host 操作
```bash
openclaw node run --host <ip> --port <port> --display-name "<name>"  # 前台运行
openclaw node install --host <ip> --port <port>  # 安装为服务
openclaw node restart                          # 重启服务
```

### 5.3 远程调用
```bash
# 执行命令
openclaw nodes run --node ZEN_LIZ -- echo "Hello"

# 发送通知
openclaw nodes notify --node ZEN_LIZ --title "标题" --body "内容"

# 截图
openclaw nodes canvas snapshot --node ZEN_LIZ --format png

# 拍照
openclaw nodes camera snap --node ZEN_LIZ

# 屏幕录制
openclaw nodes screen record --node ZEN_LIZ --duration 10s

# 获取位置
openclaw nodes location get --node ZEN_LIZ
```

---

## 6. 故障排查

### 6.1 Node 连接问题

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| `Unexpected server response: 400` | 端口配置错误或 TLS 不匹配 | 检查 port 和 tls 配置是否与 Gateway 一致 |
| `device token mismatch` | Node 配置的 token 与 Gateway 不匹配 | 执行 `openclaw devices rotate` 重新颁发 token |
| `self-signed certificate` | 自签名证书需要信任 | 添加 `--tls --tls-fingerprint` 参数 |
| `wrong version number` | 端口不启用 TLS 或端口错误 | 检查端口，移除或添加 `--tls` |
| `pairing required` | 设备未配对 | 执行 `openclaw devices approve` |
| `NODE_BACKGROUND_UNAVAILABLE` | Node App 在后台 | 将 App 切换到前台 |

### 6.2 完整排查流程

**问题：Node 连接失败**

```bash
# 1. 检查 Gateway 是否运行（在 114 上）
openclaw gateway status

# 2. 检查 Node 配置是否正确（本地）
cat ~/.openclaw/node.json

# 3. 检查 Gateway 上的节点状态（在 114 上）
openclaw nodes status
# 确认 ZEN_LIZ 的 nodeId

# 4. 如果 nodeId 不匹配，更新本地 node.json

# 5. 如果 token 过期，重新颁发（在 114 上）
openclaw devices rotate --device <nodeId> --role node --json

# 6. 更新本地 node.json 中的 token

# 7. 重新启动 Node（本地）
$env:OPENCLAW_GATEWAY_TOKEN = "d6d7858143a8faad916306a8d3393ff0e6509f3ed29b1738"
openclaw node run
```

### 6.3 调试命令

```bash
# 查看 Node 连接状态
openclaw nodes status

# 查看设备配对状态
openclaw devices list

# 查看本地配置
cat ~/.openclaw/node.json

# 查看审批配置
cat ~/.openclaw/exec-approvals.json

# 查看 Gateway 日志（在 114 上）
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

---

## 7. 服务器信息

| 项目 | 值 |
|------|-----|
| IP 地址 | 114.55.248.111 |
| SSH 端口 | 22 |
| Gateway 端口 | 18789 (内部) / 3001 (Nginx HTTPS) |
| Nginx 配置 | `/www/server/panel/vhost/nginx/openclaw.conf` |
| OpenClaw 配置 | `/root/.openclaw/openclaw.json` |
| 配对设备 | `/root/.openclaw/devices/paired.json` |

### 已配对节点

| 节点名 | 类型 | 状态 | 能力 |
|--------|------|------|------|
| ZEN_LIZ | Win32 Headless | ✅ Connected | system, browser |
| ZEN_LIC_NODE | Win32 | ⚪ Disconnected | system |

---

*文档版本: 4.0 - 添加快速启动指南和 Token 轮换说明*
*更新时间: 2026-02-20*
