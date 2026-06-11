# OpenClaw 模型配置说明

> 生成日期: 2026-02-19

## 一、配置文件位置

### 本地 Windows
- 主配置文件: `C:\Users\liz\.openclaw\openclaw.json`
- 模型配置文件: `C:\Users\liz\.openclaw\agents\main\agent\models.json`

### 远程 114 服务器 (Linux)
- 主配置文件: `~/.openclaw/openclaw.json`
- 模型配置文件: `~/.openclaw/agents/main/agent/models.json`

---

## 二、关键配置项

### 1. 当前使用的主模型

**文件**: `openclaw.json`
**路径**: `agents.defaults.model.primary`

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "zai/glm-5"
      }
    }
  }
}
```

**说明**: 修改此值可切换主模型，例如切换到 `zai/glm-4.7-flash`

---

### 2. 可用模型列表

**文件**: `openclaw.json`
**路径**: `models.providers.zai.models`

```json
{
  "models": {
    "providers": {
      "zai": {
        "baseUrl": "https://api.z.ai/api/coding/paas/v4",
        "api": "openai-completions",
        "models": [
          {
            "id": "glm-5",
            "name": "GLM-5",
            "reasoning": true,
            "contextWindow": 204800,
            "maxTokens": 131072
          },
          {
            "id": "glm-4.7",
            "name": "GLM-4.7",
            "reasoning": true,
            "contextWindow": 204800,
            "maxTokens": 131072
          },
          {
            "id": "glm-4.7-flash",
            "name": "GLM-4.7 Flash",
            "reasoning": true,
            "contextWindow": 204800,
            "maxTokens": 131072
          },
          {
            "id": "glm-4.7-flashx",
            "name": "GLM-4.7 FlashX",
            "reasoning": true,
            "contextWindow": 204800,
            "maxTokens": 131072
          }
        ]
      }
    }
  }
}
```

---

### 3. 模型详细配置（含 API Key）

**文件**: `agents/main/agent/models.json`
**路径**: `providers.zai`

```json
{
  "providers": {
    "zai": {
      "baseUrl": "https://api.z.ai/api/coding/paas/v4",
      "api": "openai-completions",
      "models": [...],
      "apiKey": "90e5b8ec45764c00a732252a6061cab1.huMPezC8Jwgt2CkM"
    }
  }
}
```

---

## 三、当前配置汇总

| 项目 | 本地 | 114 服务器 |
|------|------|------------|
| 提供商 | zai (智谱AI) | zai (智谱AI) |
| API 端点 | `https://api.z.ai/api/coding/paas/v4` | 相同 |
| 主模型 | GLM-5 (`zai/glm-5`) | 相同 |
| 上下文窗口 | 204,800 tokens | 相同 |
| 最大输出 | 131,072 tokens | 相同 |

### 可用模型

| 模型 ID | 名称 | 说明 |
|---------|------|------|
| `glm-5` | GLM-5 | 当前使用，最新版本 |
| `glm-4.7` | GLM-4.7 | 稳定版本 |
| `glm-4.7-flash` | GLM-4.7 Flash | 快速版本 |
| `glm-4.7-flashx` | GLM-4.7 FlashX | 极速版本 |

---

## 四、切换模型方法

### 方法 1: 修改配置文件

编辑 `openclaw.json`，修改 `agents.defaults.model.primary` 值：

```json
"primary": "zai/glm-4.7-flash"  // 切换到 Flash 版本
```

### 方法 2: 使用命令行

```bash
# 查看当前配置
openclaw config list

# 设置主模型
openclaw config set agents.defaults.model.primary "zai/glm-4.7-flash"
```

---

## 五、配置项对照表

| 配置文件 | JSON 路径 | 作用 |
|----------|-----------|------|
| `openclaw.json` | `agents.defaults.model.primary` | 当前使用的主模型 |
| `openclaw.json` | `agents.defaults.models` | 模型别名映射 |
| `openclaw.json` | `models.providers.zai.baseUrl` | API 端点地址 |
| `openclaw.json` | `models.providers.zai.models` | 可用模型列表 |
| `models.json` | `providers.zai.apiKey` | API 密钥 |
| `models.json` | `providers.zai.models` | 模型详细参数 |
