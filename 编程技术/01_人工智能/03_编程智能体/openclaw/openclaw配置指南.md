# OpenClaw 配置指南

本文档介绍如何配置 OpenClaw 的模型、工具、Skills、插件等核心功能。

---

## 1. 配置文件基础

### 1.1 配置文件位置

```
~/.openclaw/openclaw.json    # 主配置文件
~/.openclaw/node.json        # Node 配置（Node Host）
~/.openclaw/.env             # 环境变量文件
```

### 1.2 配置修改方式

```bash
# 交互式向导（推荐新手）
openclaw onboard              # 完整设置向导
openclaw configure            # 配置向导

# CLI 命令
openclaw config get agents.defaults.model
openclaw config set agents.defaults.model.primary "zai/glm-5"
openclaw config unset tools.web.search.apiKey

# 直接编辑
# 配置文件支持热重载，修改后自动生效
```

### 1.3 配置热重载

```json
{
  "gateway": {
    "reload": {
      "mode": "hybrid",      // hybrid(默认) | hot | restart | off
      "debounceMs": 300
    }
  }
}
```

| 模式 | 行为 |
|------|------|
| `hybrid` | 安全更改立即生效，关键更改自动重启 |
| `hot` | 仅热应用安全更改，需重启时警告 |
| `restart` | 任何更改都重启 Gateway |
| `off` | 禁用文件监控，需手动重启 |

---

## 2. 模型配置

### 2.1 快速设置

```bash
# 使用向导设置模型
openclaw onboard

# 直接设置模型
openclaw models set zai/glm-5

# 查看当前模型
openclaw models status

# 查看可用模型
openclaw models list
openclaw models list --all    # 完整目录
```

### 2.2 模型选择顺序

OpenClaw 按以下顺序选择模型：
1. **Primary** 模型 (`agents.defaults.model.primary`)
2. **Fallbacks** (`agents.defaults.model.fallbacks`，按顺序)
3. **Provider auth failover**（在 provider 内部发生）

### 2.3 配置示例

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "zai/glm-5",
        "fallbacks": ["zai/glm-4.7", "openai/gpt-4o"]
      },
      "models": {
        "zai/glm-5": { "alias": "GLM" },
        "zai/glm-4.7": { "alias": "GLM-4.7" }
      }
    }
  }
}
```

### 2.4 内置 Provider

| Provider | 环境变量 | 示例模型 |
|----------|----------|----------|
| **Z.AI (GLM)** | `ZAI_API_KEY` | `zai/glm-5`, `zai/glm-4.7` |
| **OpenAI** | `OPENAI_API_KEY` | `openai/gpt-5.1-codex` |
| **Anthropic** | `ANTHROPIC_API_KEY` | `anthropic/claude-opus-4-6` |
| **OpenCode Zen** | `OPENCODE_API_KEY` | `opencode/claude-opus-4-6` |
| **Google Gemini** | `GEMINI_API_KEY` | `google/gemini-3-pro-preview` |
| **OpenRouter** | `OPENROUTER_API_KEY` | `openrouter/anthropic/claude-sonnet-4-5` |
| **Groq** | `GROQ_API_KEY` | `groq/llama-3.3-70b` |
| **Ollama** | 无需（本地） | `ollama/llama3.3` |

### 2.5 自定义 Provider（代理/本地模型）

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "moonshot": {
        "baseUrl": "https://api.moonshot.ai/v1",
        "apiKey": "${MOONSHOT_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "kimi-k2.5",
            "name": "Kimi K2.5",
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      },
      "lmstudio": {
        "baseUrl": "http://localhost:1234/v1",
        "apiKey": "LMSTUDIO_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "local-model",
            "name": "Local Model",
            "contextWindow": 128000,
            "maxTokens": 4096
          }
        ]
      }
    }
  }
}
```

### 2.6 聊天中切换模型

```
/model                    # 显示模型选择器
/model list               # 列出可用模型
/model 3                  # 选择第 3 个模型
/model zai/glm-5          # 直接指定模型
/model status             # 查看当前模型状态
```

### 2.7 CLI 命令汇总

```bash
openclaw models list                  # 列出配置的模型
openclaw models status                # 查看模型状态
openclaw models set <provider/model>  # 设置默认模型
openclaw models set-image <model>     # 设置图像模型

# Fallbacks 管理
openclaw models fallbacks list
openclaw models fallbacks add <model>
openclaw models fallbacks remove <model>
openclaw models fallbacks clear

# 别名管理
openclaw models aliases list
openclaw models aliases add <model> <alias>
openclaw models aliases remove <alias>
```

---

## 3. 工具配置

### 3.1 工具概览

OpenClaw 提供以下**内置工具**：

| 工具 | 功能 | 说明 |
|------|------|------|
| **exec** | 执行命令 | 在 workspace 运行 shell 命令 |
| **process** | 进程管理 | 管理后台进程 |
| **browser** | 浏览器控制 | 自动化浏览器操作 |
| **canvas** | Canvas 控制 | Node Canvas 操作 |
| **nodes** | 节点管理 | 管理和调用远程 Node |
| **web_search** | 网页搜索 | Brave Search API |
| **web_fetch** | 网页抓取 | 提取网页内容 |
| **image** | 图像分析 | 使用图像模型分析图片 |
| **message** | 消息发送 | 跨渠道消息发送 |
| **cron** | 定时任务 | 管理定时任务 |
| **gateway** | Gateway 管理 | 重启和配置 Gateway |

### 3.2 工具权限控制

```json
{
  "tools": {
    "allow": ["group:fs", "browser"],
    "deny": ["process"]
  }
}
```

**工具组（group:*）**：
- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: 所有内置工具

### 3.3 工具配置文件（Tool Profiles）

```json
{
  "tools": {
    "profile": "coding"    // minimal | coding | messaging | full
  }
}
```

| Profile | 包含的工具 |
|---------|-----------|
| `minimal` | `session_status` only |
| `coding` | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image` |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full` | 无限制（默认） |

### 3.4 Provider 特定工具策略

```json
{
  "tools": {
    "profile": "coding",
    "byProvider": {
      "google-antigravity": { "profile": "minimal" },
      "openai/gpt-5.2": { "allow": ["group:fs", "sessions_list"] }
    }
  }
}
```

### 3.5 Exec 工具配置

```json
{
  "tools": {
    "exec": {
      "host": "sandbox",        // sandbox | gateway | node
      "security": "ask",        // deny | allowlist | full | ask
      "node": "ZEN_LIZ",        // host=node 时指定节点
      "yieldMs": 10000,         // 后台运行超时
      "timeout": 1800,          // 最大执行时间（秒）
      "applyPatch": {
        "enabled": false,
        "workspaceOnly": true
      }
    }
  }
}
```

### 3.6 Web 工具配置

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "maxResults": 5
      },
      "fetch": {
        "enabled": true,
        "maxCharsCap": 50000
      }
    }
  }
}
```

---

## 4. Skills 配置

### 4.1 什么是 Skills？

Skills 是**预定义的能力包**，包括：
- 系统提示词指导
- 工具使用说明
- 领域知识

### 4.2 内置 Skills

```bash
# 查看可用 Skills
openclaw skills list

# 启用/禁用 Skill
openclaw skills enable <skill-id>
openclaw skills disable <skill-id>
```

### 4.3 Skills 配置

```json
{
  "skills": {
    "entries": {
      "quick-reminders": {
        "enabled": false
      }
    }
  }
}
```

### 4.4 ClawHub（Skills 市场）

ClawHub 是 OpenClaw 的 Skills 共享平台：

```bash
# 浏览 ClawHub
openclaw skills browse

# 从 ClawHub 安装
openclaw skills install <skill-id>
```

---

## 5. Plugins 插件

### 5.1 什么是 Plugins？

Plugins 扩展 OpenClaw 的核心功能：
- 添加新的**工具**
- 添加新的**Channel**（消息渠道）
- 添加新的**CLI 命令**

### 5.2 管理 Plugins

```bash
# 查看已安装插件
openclaw plugins list

# 启用插件
openclaw plugins enable <plugin-id>

# 禁用插件
openclaw plugins disable <plugin-id>

# 从本地路径加载
openclaw plugins load /path/to/plugin
```

### 5.3 配置插件

```json
{
  "plugins": {
    "load": {
      "paths": [
        "~/.openclaw/my-plugin"
      ]
    },
    "entries": {
      "dingtalk": {
        "enabled": true
      },
      "voice-call": {
        "enabled": false
      }
    }
  }
}
```

### 5.4 内置插件

| 插件 | 功能 |
|------|------|
| `dingtalk` | 钉钉消息渠道 |
| `voice-call` | 语音通话 |
| `google-antigravity-auth` | Google Antigravity OAuth |
| `google-gemini-cli-auth` | Google Gemini CLI OAuth |
| `qwen-portal-auth` | 通义千问 OAuth |
| `zalo-personal` | Zalo 个人消息 |

---

## 6. 环境变量与代理

### 6.1 环境变量来源

OpenClaw 从以下位置读取环境变量：
1. 父进程环境
2. `./env`（当前目录）
3. `~/.openclaw/.env`（全局）

### 6.2 配置中引用环境变量

```json
{
  "env": {
    "OPENROUTER_API_KEY": "sk-or-...",
    "vars": {
      "GROQ_API_KEY": "gsk-..."
    }
  },
  "models": {
    "providers": {
      "custom": {
        "apiKey": "${CUSTOM_API_KEY}"
      }
    }
  },
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

### 6.3 API Key 轮换

支持多 Key 轮换（遇到限流时自动切换）：

```bash
# 方式一：逗号分隔
export OPENAI_API_KEYS="sk-key1,sk-key2,sk-key3"

# 方式二：编号列表
export OPENAI_API_KEY_1="sk-key1"
export OPENAI_API_KEY_2="sk-key2"

# 方式三：实时覆盖
export OPENCLAW_LIVE_OPENAI_KEY="sk-emergency"
```

### 6.4 HTTP 代理配置

```bash
# 设置代理
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 或在 .env 文件中
HTTP_PROXY=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
```

---

## 7. Channel 消息渠道

### 7.1 支持的渠道

| 渠道 | 配置键 | 认证方式 |
|------|--------|----------|
| 钉钉 | `channels.dingtalk` | clientId + clientSecret |
| Telegram | `channels.telegram` | Bot Token |
| WhatsApp | `channels.whatsapp` | 扫码配对 |
| Discord | `channels.discord` | Bot Token |
| Slack | `channels.slack` | OAuth |
| Signal | `channels.signal` | 扫码配对 |
| iMessage | `channels.imessage` | Apple ID |

### 7.2 钉钉配置示例

```json
{
  "channels": {
    "dingtalk": {
      "agentId": 4274754144,
      "clientId": "dingkm2y6r3esprxkz7l",
      "clientSecret": "T4V07dzLsFeEAKnfYGLmKK4wWBMVbH8ELpJyXc2Kw7wWbBH3K3le5z1x6pvYSF4_",
      "corpId": "ding1da34973cb89f814f2c783f7214b6d69",
      "allowFrom": [],
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

### 7.3 访问控制

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": {
        "*": { "requireMention": true }
      }
    }
  },
  "messages": {
    "groupChat": {
      "mentionPatterns": ["@openclaw", "@assistant"]
    }
  }
}
```

---

## 8. 常用配置示例

### 8.1 最小配置

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace"
    }
  },
  "channels": {
    "dingtalk": {
      "clientId": "xxx",
      "clientSecret": "xxx"
    }
  }
}
```

### 8.2 开发者配置

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "zai/glm-5" },
      "workspace": "~/projects"
    }
  },
  "tools": {
    "profile": "coding",
    "exec": {
      "host": "sandbox",
      "security": "ask"
    },
    "web": {
      "search": { "enabled": true },
      "fetch": { "enabled": true }
    }
  },
  "channels": {
    "dingtalk": { "clientId": "xxx", "clientSecret": "xxx" }
  }
}
```

### 8.3 远程执行配置

```json
{
  "tools": {
    "exec": {
      "host": "node",
      "security": "allowlist",
      "node": "ZEN_LIZ"
    }
  },
  "channels": {
    "dingtalk": { "clientId": "xxx", "clientSecret": "xxx" }
  }
}
```

---

## 9. CLI 速查表

### 9.1 配置管理
```bash
openclaw onboard              # 完整设置向导
openclaw configure            # 配置向导
openclaw doctor               # 诊断问题
openclaw doctor --fix         # 自动修复

openclaw config get <path>    # 获取配置值
openclaw config set <path> <value>  # 设置配置值
openclaw config unset <path>  # 删除配置值
```

### 9.2 模型管理
```bash
openclaw models list          # 列出模型
openclaw models status        # 模型状态
openclaw models set <model>   # 设置默认模型
openclaw models auth login --provider <name>  # OAuth 登录
```

### 9.3 插件和 Skills
```bash
openclaw plugins list         # 列出插件
openclaw plugins enable <id>  # 启用插件
openclaw plugins disable <id> # 禁用插件

openclaw skills list          # 列出 Skills
openclaw skills enable <id>   # 启用 Skill
```

### 9.4 节点管理
```bash
openclaw nodes status         # 节点状态
openclaw devices list         # 设备列表
openclaw devices approve <id> # 批准配对
```

### 9.5 Gateway 管理
```bash
openclaw gateway status       # Gateway 状态
openclaw gateway install      # 安装为服务
openclaw gateway restart      # 重启 Gateway
openclaw logs                 # 查看日志
```

---

*文档版本: 1.0*
*更新时间: 2026-02-19*
*参考文档: https://docs.openclaw.ai/*
