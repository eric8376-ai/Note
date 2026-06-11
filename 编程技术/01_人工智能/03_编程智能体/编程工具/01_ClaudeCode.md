https://code.claude.com/docs/zh-CN/overview


## 日常操作
https://code.claude.com/docs/zh-CN/interactive-mode

### 升级claude
claude --version
npm update -g @anthropic-ai/claude-code
claude update


### 交互模式
按 **Shift+Tab** 循环切换权限模式
- Normal Mode：默认模式，每次操作需确认
- Auto-Accept Mode：自动接受编辑（底部显示 `⏵⏵ accept edits on`）
- Plan Mode：只读模式（底部显示 `⏸ plan mode on`）
### 常用命令
/clear	清除对话历史
/compact [instructions]	使用可选焦点指令压缩对话
/config	打开设置界面（配置选项卡）
/context	将当前上下文使用情况可视化为彩色网格

/doctor	检查您的 Claude Code 安装的健康状况
/exit	退出 REPL
/export [filename]	将当前对话导出到文件或剪贴板

/init	使用 CLAUDE.md 指南初始化项目
/mcp	管理 MCP server 连接和 OAuth 身份验证
/memory	编辑 CLAUDE.md 内存文件
/model	选择或更改 AI 模型
/permissions	查看或更新权限
/plan	直接从提示进入 Plan Mode
/rename name	重命名当前会话以便于识别
/resume [session]	按 ID 或名称恢复对话，或打开会话选择器
/rewind	回退对话和/或代码
/stats	可视化每日使用情况、会话历史、连胜和模型偏好
/status	打开设置界面（状态选项卡），显示版本、模型、帐户和连接性
/statusline	设置 Claude Code 的状态行 UI（麻烦）
### YOLO模式
https://chat01.ai/zh/chat/01KC1SNHWVZ5NXVDB9QCRQSDG0

claude --dangerously-skip-permissions
### 沙盒模式
https://www.hubwiz.com/blog/run-claude-code-in-a-sandbox/

###  安装记忆插件
Claude Code长期记忆插件你
https://github.com/thedotmack/claude-mem
```text
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

### 多Agent模式
#### 1、使用 Claude Code 内置的“代理团队”功能**

这是最简单直接的方式，适合希望快速上手的开发者。

**1. 如何开启**  
在 Claude Code 的设置文件（`settings.json`）中启用实验性标志：

json

{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}

之后，你只需用自然语言告诉 Claude 你想组建一个团队 [](https://addyosmani.com/blog/claude-code-agent-teams/)。

**2. 协作模式与指令示例**  
你可以通过一个“团队负责人”来协调两个专职代理。负责人负责分解任务、分配工作和整合结果，而两个代理则在自己的上下文中并行工作 [](https://addyosmani.com/blog/claude-code-agent-teams/)。

- **启动团队并分配任务**
    
    我需要为一个新的待办事项CLI工具编写代码。
    请创建一个有2个队友的代理团队：
    1. 一个作为“后端开发者”，负责实现核心功能，代码放在 `src/` 目录。
    2. 一个作为“测试工程师”，负责为 `src/` 下的代码编写单元测试和集成测试，并指导如何运行它们。
    请确保他们分工明确，避免编辑同一个文件。
    
- **处理更复杂的任务**


    现在我们的“用户认证”模块需要重构。
    “开发者”队友，请负责重构 `auth/` 目录下的代码，确保功能不变。
    “测试”队友，请在重构前先编写测试用例来锁定现有行为，然后在重构完成后运行所有测试，确保全部通过。
    

#### 2、通过“子代理”和“钩子”构建自定义流水线**

这种方式提供了更精细的控制，适合对可靠性和流程有严格要求的项目。你可以将每个角色的指令和权限固化到独立的代理配置文件中 [](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)。

**1. 如何配置**  
在项目的 `.claude/agents/` 目录下创建两个Markdown文件，为每个代理定义其“岗位职责”。

- **`developer-agent.md` (编码代理)**
    
    yaml
    
    ---
    name: developer-agent
    description: 负责根据需求编写功能代码，遵循项目编码规范。
    tools: [Read, Edit, Write, Glob, Bash] # 允许编写和运行代码
    ---
    # 角色与职责
    你是一名资深后端开发者。你的任务是实现清晰、健壮、可维护的代码。
    *   **输入**：来自产品经理的需求或设计文档。
    *   **输出**：实现功能的代码，放在 `src/` 目录下。
    *   **规则**：代码必须包含必要的注释，并遵循项目 `CLAUDE.md` 中定义的风格。
    *   **交接**：完成代码编写后，更新任务状态，等待测试工程师接手。
    
- **`tester-agent.md` (测试部署代理)**
    
    yaml
    
    ---
    name: tester-agent
    description: 负责为代码编写测试、运行测试套件，并执行部署流程。
    tools: [Read, Edit, Write, Glob, Bash] # 需要运行测试和部署命令
    ---
    # 角色与职责
    你是一名严谨的测试与运维工程师。你的任务是确保代码质量并顺利交付。
    *   **输入**：开发者完成的代码模块。
    *   **输出**：为对应模块编写的测试用例（如 `tests/` 目录下）、通过的测试报告、以及部署指令。
    *   **规则**：测试覆盖率需达到80%以上。部署前必须确保所有测试通过。
    *   **交接**：测试和部署完成后，通知项目负责人。
    

**2. 用“钩子”串联工作流**  
通过配置钩子（Hooks），可以在一个代理工作结束后，自动提示下一个代理启动，形成自动化流水线 [](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)。例如，在 `developer-agent` 完成后，一个 `on-subagent-stop.sh` 脚本可以自动读取任务队列，并输出指令：“`tester-agent` 现在可以开始测试刚完成的模块了。” 你只需复制这个指令发给 Claude，即可启动测试流程。

#### 成功协作的关键要点

无论选择哪种路径，以下几点是保证协作成功的关键：

1. **明确的职责边界**：核心是**避免两个代理编辑同一个文件** [](https://addyosmani.com/blog/claude-code-agent-teams/)。你需要清晰划分文件所有权。例如：
    
    - **编码代理**：负责 `src/` 目录下的所有 `.py` 或 `.js` 文件。
        
    - **测试代理**：负责 `tests/` 目录下的所有文件，以及 `pyproject.toml` 或 `package.json` 中的测试脚本部分。  
        在给代理的初始指令中，就明确这些边界 [](https://addyosmani.com/blog/claude-code-agent-teams/)。
        
2. **任务颗粒度适中**：任务既不能太大（导致代理长时间无反馈，风险高），也不能太小（协调成本过高）。一个良好的实践是，让每个代理在单个任务中产出 **一个完整的、可审查的交付物**，例如“实现用户登录API”或“为用户登录API编写集成测试” [](https://addyosmani.com/blog/claude-code-agent-teams/)。
    
3. **充分利用共享上下文**：项目根目录下的 `CLAUDE.md` 文件是所有代理都能读取的“团队手册”。你可以在其中写入项目的整体架构、编码规范、测试框架（如pytest, jest）和部署命令（如 `npm run deploy`），让每个代理都遵循同样的标准 [](https://platform.claude.com/cookbook/claude-agent-sdk-01-the-chief-of-staff-agent)[](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)。
    
4. **保持人工监督环节**：对于关键步骤，可以在流程中加入“计划审批”环节。例如，你可以要求 `developer-agent` 在动手前先提交一份重构计划，等你批准后再开始编码。这能有效避免方向性错误 [](https://addyosmani.com/blog/claude-code-agent-teams/)[](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)。
    

这种“开发-测试”双代理模式，能让你体验到AI驱动的、更可靠、更高效的软件交付流程。如果你的项目对流程的规范性要求特别高，可以优先考虑路径二。如果在探索阶段想快速验证效果，路径一会更合适。
###  常见问题
#### 🔍 关于"上下文窗口限制"错误分析

  你提到的 API Error: The model has reached its context window limit.
  错误，这是在多智能体并行工作时出现的。问题原因：

  问题根源

  1. 多智能体并行产生大量上下文：当同时启动多个智能体时，每个智能体都会产生大量的对话历史和代码上下文
  2. 上下文累积：随着对话进行，上下文不断累积，最终超过了模型的上下文窗口限制
  3. 没有上下文管理：之前的实现没有进行上下文清理或分段处理
  4. 
##  智谱后台
https://open.bigmodel.cn/api/anthropic
https://api.z.ai/api/anthropic

### 购买套餐
[智谱AI开放平台](https://bigmodel.cn/glm-coding?utm_source=bigModel&utm_medium=Special&utm_content=glm-code&utm_campaign=Platform_Ops&_channel_track_key=8BAeCdUS)

13606935895
41b59bba0b4841718f341a5a7e68ed38.gr9MnqBnBUIVqrm4
15980984600
90e5b8ec45764c00a732252a6061cab1.huMPezC8Jwgt2CkM

### 安装智普小助手

npm install -g @z_ai/coding-helper
coding-helper

### 版本切换
https://docs.bigmodel.cn/cn/coding-plan/tool/claude
~/.claude/settings.json
####  旗舰/深度思考
  - **Anthropic Claude**: Opus
  - **智谱 GLM**: GLM-5

  #### 均衡/主力
  - **Anthropic Claude**: Sonnet
  - **智谱 GLM**: GLM-4.7

  ####  轻量/快速
  - **Anthropic Claude**: Haiku
  - **智谱 GLM**: GLM-4.5-Air
## 安装claude code
npm install -g @anthropic-ai/claude-code
claude --version
### 配置claude

始终用中文回答我的问题，记录到配置文件中


其实解决方案直接且高效：启动工具时添加`--dangerous-skip-permissions`参数。但这绝非鲁莽的权限放开，而是建立在对项目环境和 AI 能力充分理解与信任之上的操作。

更精细化的权限控制，藏在四种可切换的权限模式里（按下 Shift+Tab 即可切换）：

- **谨慎模式**（默认）：所有敏感操作均需人工确认，安全性拉满但效率偏低；
- **自动编辑模式**：可自动接受常规文件编辑请求，适合日常编码任务；
- **规划模式**：仅允许读取和分析代码，无法执行修改操作，多用于架构梳理和代码审查；
- **YOLO 模式**：完全放开所有操作权限，仅建议在容器等隔离环境中，用于批量处理无风险任务。
### SKILL
Agent Skills 是可复用的知识模块，教会 Claude 如何完成特定任务。当你的请求匹配 Skill 的用途时，Claude 会自动应用它。
个人 Skills 存放在 `~/.claude/skills/`，跨项目可用：
Create skills in .claude/skills/ or ~/.claude/skills/
#### 通过工具添加

https://github.com/vercel-labs/add-skill

###  自定义命令
自定义命令允许你将常用的提示模板存储为可复用的命令，通过简单的斜杠命令即可调用。

###  Subagents 子代理

Subagents（子代理）是专门处理特定类型任务的 AI 助手。每个 Subagent 在独立的上下文窗口中运行，拥有自定义系统提示、特定工具访问权限和独立权限设置。

###  Hooks 系统

Hooks 允许在 Claude Code 特定事件发生时自动执行脚本或 LLM 评估，实现自动化工作流。
## 安装插件
### claude-hub
https://juejin.cn/post/7604301929121366025
/plugin marketplace add jarrodwatts/claude-hud
/plugin install claude-hud
/claude-hud:setup
##  安装skill
## 安装mcp
### 飞书mcp
https://github.com/larksuite/lark-openapi-mcp
https://zhuanlan.zhihu.com/p/1967663998402033545

## 参考
https://claudecn.com/docs/claude-code/



 - 位置: C:\Users\liz\AppData\Roaming\Claude\claude_desktop_config.json
  - 服务器名: skill-installer
  - Python 路径: C:\Users\liz\.pyenv\pyenv-win\versions\3.12.10\python.exe
  - 脚本路径: D:\AICode\MCP\add-skill\skill_installer_mcp.py