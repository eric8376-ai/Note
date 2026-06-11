# Claude Code 多智能体协作指南

> 从"单打独斗"到"团队作战"——构建开发与测试双Agent协作模式

---

## 一、核心概念

Claude Code 的最新特性（"代理团队"或"子代理"）允许创建多个具有不同职责和上下文的 AI 实例协同工作。

### 为什么需要多Agent？

| 优势 | 说明 |
|------|------|
| **专注与质量** | 每个Agent拥有独立的上下文窗口，专注自己的领域，避免信息过载 |
| **自然检查点** | 编码完成后自然移交给测试，流程清晰，结果更可靠 |
| **并行探索** | 不同Agent可以并行探索多种方案 |

---

## 二、两种实现路径

### 路径一：内置"代理团队"功能（快速上手）

适合希望快速上手的开发者。

#### 1. 开启方式

在 `settings.json` 中启用实验性标志：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

#### 2. 协作模式

通过"团队负责人"协调两个专职Agent：

```
┌─────────────────────────────────────────┐
│           团队负责人 (Orchestrator)       │
│         • 分解任务                        │
│         • 分配工作                        │
│         • 整合结果                        │
└─────────────┬───────────────────────────┘
              │
      ┌───────┴───────┐
      ▼               ▼
┌──────────┐    ┌──────────┐
│ 开发者    │    │ 测试工程师 │
│          │    │          │
│ 实现功能   │    │ 编写测试   │
│ src/目录  │    │ 运行测试   │
└──────────┘    └──────────┘
```

#### 3. 指令示例

**启动团队并分配任务：**

```
我需要为一个新的待办事项CLI工具编写代码。
请创建一个有2个队友的代理团队：
1. 一个作为"后端开发者"，负责实现核心功能，代码放在 `src/` 目录。
2. 一个作为"测试工程师"，负责为 `src/` 下的代码编写单元测试和集成测试，并指导如何运行它们。
请确保他们分工明确，避免编辑同一个文件。
```

**处理复杂任务：**

```
现在我们的"用户认证"模块需要重构。
"开发者"队友，请负责重构 `auth/` 目录下的代码，确保功能不变。
"测试"队友，请在重构前先编写测试用例来锁定现有行为，然后在重构完成后运行所有测试，确保全部通过。
```

---

### 路径二：子代理 + 钩子自定义流水线（精细控制）

适合对可靠性和流程有严格要求的项目。

#### 1. 配置方式

在项目 `.claude/agents/` 目录下创建 Agent 定义文件。

**developer-agent.md（编码代理）：**

```yaml
---
name: developer-agent
description: 负责根据需求编写功能代码，遵循项目编码规范。
tools: [Read, Edit, Write, Glob, Bash]
---
```

```markdown
# 角色与职责

你是一名资深后端开发者。你的任务是实现清晰、健壮、可维护的代码。

- **输入**：来自产品经理的需求或设计文档
- **输出**：实现功能的代码，放在 `src/` 目录下
- **规则**：代码必须包含必要的注释，并遵循项目 `CLAUDE.md` 中定义的风格
- **交接**：完成代码编写后，更新任务状态，等待测试工程师接手
```

**tester-agent.md（测试部署代理）：**

```yaml
---
name: tester-agent
description: 负责为代码编写测试、运行测试套件，并执行部署流程。
tools: [Read, Edit, Write, Glob, Bash]
---
```

```markdown
# 角色与职责

你是一名严谨的测试与运维工程师。你的任务是确保代码质量并顺利交付。

- **输入**：开发者完成的代码模块
- **输出**：测试用例（`tests/` 目录）、测试报告、部署指令
- **规则**：测试覆盖率需达到80%以上。部署前必须确保所有测试通过
- **交接**：测试和部署完成后，通知项目负责人
```

#### 2. 用钩子串联工作流

通过 Hooks 在一个 Agent 完成后自动触发下一个：

```
developer-agent 完成
        │
        ▼
on-subagent-stop.sh 读取任务队列
        │
        ▼
输出提示："tester-agent 现在可以开始测试刚完成的模块了"
        │
        ▼
用户发送指令 → tester-agent 启动
```

---

## 三、目录结构示例

```
project/
├── .claude/
│   ├── CLAUDE.md              # 团队共享手册（编码规范、测试框架、部署命令）
│   └── agents/
│       ├── developer-agent.md # 开发Agent定义
│       └── tester-agent.md    # 测试Agent定义
├── src/                       # 开发Agent负责
│   └── ...
├── tests/                     # 测试Agent负责
│   └── ...
└── pyproject.toml             # 测试脚本配置
```

---

## 四、成功协作的关键要点

### 1. 明确的职责边界

**核心原则**：避免两个Agent编辑同一个文件

| Agent | 负责目录 | 职责 |
|-------|---------|------|
| 开发Agent | `src/` | 所有 `.py` 或 `.js` 源码文件 |
| 测试Agent | `tests/` | 测试文件 + `pyproject.toml`/`package.json` 中的测试脚本 |

### 2. 任务颗粒度适中

| 颗粒度 | 问题 | 建议 |
|--------|------|------|
| 太大 | Agent长时间无反馈，风险高 | ❌ 避免 |
| 太小 | 协调成本过高 | ❌ 避免 |
| **适中** | 单个任务产出**一个完整交付物** | ✅ 推荐 |

**良好示例**：
- "实现用户登录API"
- "为用户登录API编写集成测试"

### 3. 充分利用共享上下文

`CLAUDE.md` 是所有Agent都能读取的"团队手册"，应包含：

```markdown
# 项目规范

## 架构
- 分层架构：Controller → Service → Repository

## 编码规范
- Python: 遵循 PEP 8
- TypeScript: ESLint + Prettier

## 测试框架
- Python: pytest
- Node.js: jest

## 部署命令
- npm run build && npm run deploy
```

### 4. 保持人工监督环节

关键步骤加入"计划审批"：

```
developer-agent 提交重构计划
        │
        ▼
   人工审批 ✓
        │
        ▼
developer-agent 开始编码
```

---

## 五、路径选择建议

| 场景 | 推荐路径 |
|------|---------|
| 探索阶段，快速验证 | 路径一（代理团队） |
| 流程规范性要求高 | 路径二（子代理+钩子） |
| 项目复杂度低 | 路径一 |
| 需要自动化流水线 | 路径二 |

---

## 六、参考资料

- [Claude Agent SDK 官方文档](https://docs.anthropic.com/claude-agent-sdk)
- [Claude Code 多Agent技术实现原理](https://www.langchain.cn/t/topic/842)
- [Anthropic 多Agent架构博客](https://www.anthropic.com/research/building-effective-agents)

---

> 文档版本: 1.0
> 更新日期: 2026-03-01



### 1. 官方定性：尚在“预览”的试验品

多智能体功能（官方名称 Agent Teams 或 Agent Swarms）被明确标记为 **“研究预览版”** [](https://blockchain.news/zh/ainews/claude-code-teams-launch-parallel-agent-swarms-enable-advanced-ai-collaboration-zh)[](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://news.qq.com/rain/a/20260207A02YAJ00)。既然是预览版，就意味着它还不够稳定和完善，存在一些已知的限制和问题，不适合作为所有用户的默认配置。把它设为可选，可以让感兴趣的开发者先行体验和反馈，而普通用户则不受影响[](https://blog.lightnote.com.cn/bian-pai-claude-code-de-agent-teams/#/#/portal)。

### 2. 核心痛点：成本可能“起飞”

这是最关键的实际原因。多智能体模式会**显著增加Token消耗**[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)[](https://addyosmani.com/blog/claude-code-agent-teams/)。

- **每个“人”都是独立的**：在单次会话中，你只和一个AI对话。但在多智能体模式下，一个“团队主管”（Team Lead）加上多个“队友”（Teammates），每个都是独立的Claude实例，拥有自己庞大的上下文窗口[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)[](https://addyosmani.com/blog/claude-code-agent-teams/)。
    
- **成本线性增长**：这意味着Token的消耗量会随着队友数量**线性增长**[](https://news.qq.com/rain/a/20260207A02YAJ00)。官方也明确指出，多智能体适合复杂任务，而日常的简单编辑用单会话更经济[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://blog.lightnote.com.cn/bian-pai-claude-code-de-agent-teams/#/#/portal)。如果默认开启，用户可能在不知情的情况下，为一个简单任务付出高昂的Token代价。
    

### 3. 技术复杂性：需要“精心设计”而非“一键魔法”

多智能体不是万能的“魔法按钮”，它的成功高度依赖用户的**精心设计**和**主动监控**[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)[](https://news.qq.com/rain/a/20260207A02YAJ00)。

- **上下文需手动加载**：队友们**不会继承**你与主管的历史对话[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://addyosmani.com/blog/claude-code-agent-teams/)。你必须为每个新创建的队友提供充足的初始上下文（如项目架构、编码规范），否则他们就像一群“失忆”的员工，无法有效工作[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://news.qq.com/rain/a/20260207A02YAJ00)。
    
- **管理难度增加**：多个AI并行工作，如果方向错了，纠正起来会比指挥单个AI更复杂[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)。你需要像个真正的团队领导一样，分配合适的任务、避免文件冲突、监控进度[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://addyosmani.com/blog/claude-code-agent-teams/)。
    

### 4. 对比分析：它与“子代理”有本质区别

为了让你更清楚地理解，这里对比一下多智能体（Agent Teams）和它更轻量的“同事”——子代理（Sub-agents）[](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://blog.frognew.com/2026/02/claude-code-agent-teams-notes1.html)[](https://blog.lightnote.com.cn/bian-pai-claude-code-de-agent-teams/#/#/portal)：

|特性|子代理 (Sub-agents)|多智能体 (Agent Teams)|为什么默认关闭？|
|---|---|---|---|
|**通信方式**|只能向主代理汇报结果|**队友间可直接通信**，协同讨论[](https://blog.csdn.net/weixin_44058951/article/details/157840560)[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)[](https://addyosmani.com/blog/claude-code-agent-teams/)|协同带来复杂性，需要更复杂的协调机制。|
|**协调方式**|由主代理全权管理|通过**共享任务列表**自我协调[](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://blog.frognew.com/2026/02/claude-code-agent-teams-notes1.html)[](https://developer.aliyun.com/article/1711162)|自主性更强，但出错概率和不可控性也更高。|
|**适用场景**|只需结果的聚焦任务|需要讨论和协作的**复杂工作**[](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://blog.frognew.com/2026/02/claude-code-agent-teams-notes1.html)[](https://blog.lightnote.com.cn/bian-pai-claude-code-de-agent-teams/#/#/portal)|并非人人、时时都需要处理复杂任务。|
|**Token成本**|**较低**|**较高**（每个队友独立实例）[](https://gihyo.jp/article/2026/02/get-started-claude-code-07)[](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://blog.frognew.com/2026/02/claude-code-agent-teams-notes1.html)|默认开启可能导致大量不必要的Token消耗。|

### 总结

所以，多智能体功能默认关闭，是Anthropic在产品设计上的一种**审慎和负责任**的体现。它就像一个工具箱里的精密仪器，强大但需要专业知识来操作。当你准备好应对它的复杂性和成本，并希望用它来解决真正的复杂问题时，就可以通过一个简单的配置（`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`）来“解锁”它的全部潜力。

你之前设想的“一个编码，一个测试”的场景，正是多智能体最能发挥价值的地方之一。既然你了解了它默认关闭背后的原因，就可以更有准备地去开启和驾驭它了。