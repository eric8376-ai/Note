## BMad
https://github.com/bmad-code-org/BMAD-METHOD
**突破性的敏捷人工智能驱动开发方法**——BMad 方法模块生态系统的人工智能驱动敏捷开发模块，BMad 方法模块生态系统是最佳、最全面的敏捷人工智能驱动开发框架，具有真正的规模自适应智能，可从错误修复调整到企业系统。

**100% 免费开源。**没有付费墙。没有内容限制。没有封闭的 Discord 服务器。我们相信应该让每个人都能受益，而不仅仅是那些能够付费加入封闭社区或参加课程的人。

## 为什么选择 BMad 方法？

[利用人工智能驱动的工作流程和专业代理，更快地构建软件，这些代理将指导您完成规划、架构和实施。](https://github.com/bmad-code-org/BMAD-METHOD#why-the-bmad-method)

传统人工智能工具替你思考，但结果平平。BMad 智能体和引导式工作流程则扮演着专家级合作者的角色，引导你完成结构化的流程，从而与人工智能协同工作，激发你的最佳思维。

- **AI智能助手**——`/bmad-help`随时询问下一步该怎么做。
- **规模域自适应**——根据项目复杂性自动调整规划深度
- **结构化工作流程**——基于敏捷最佳实践，涵盖分析、规划、架构和实施各个环节。
- **专业代理**——12 位以上领域专家（项目经理、架构师、开发人员、用户体验设计师、Scrum Master 等）
- **派对模式**——将多个代理角色放入同一个会话中进行协作和讨论
- **完整生命周期**——从头脑风暴到部署



## 如何安装
**前提条件**：[Node.js](https://nodejs.org/) v20+

```shell
npx bmad-method install
```

> 如果您获得的是过时的测试版，请使用：`npx bmad-method@6.0.1 install`

按照安装程序的提示操作，然后在项目文件夹中打开您的 AI IDE（Claude Code、Cursor 等）。

**非交互式安装**（适用于 CI/CD）：

```shell
npx bmad-method install --directory /path/to/project --modules bmm --tools claude-code --yes
```



# BMAD 帮助文档 - 下一步建议

## 项目概述
JeecgBoot v3.9.1 - 企业级 AI 低代码开发平台，采用前后端分离架构

## 当前状态
- **项目阶段**: 新项目，尚未创建任何 BMAD 工作流产物
- **运行模型**: haiku (glm-4.5-air)

## 常用的功能



## 📊 BMAD Command 完整列表

### 1. 分析阶段 (Analysis Phase)

| Command | 全称 | 描述 | 适用场景 |
|--------|------|------|----------|
| `/bmad-brainstorming` | Brainstorm Project | 专家引导的头脑风暴 | 创意阶段、突破思维限制 |
| `/bmad-bmm-market-research` | Market Research | 市场分析、竞争格局研究 | 了解市场环境 |
| `/bmad-bmm-domain-research` | Domain Research | 行业领域深度研究 | 特定领域专业知识 |
| `/bmad-bmm-technical-research` | Technical Research | 技术可行性研究 | 评估技术实现方案 |
| `/bmad-bmm-create-product-brief` | Create Brief | 创建产品简介 | 明确产品愿景 |

### 2. 规划阶段 (Planning Phase)

| Command                      | 全称               | 描述         | 必要性         |
| ---------------------------- | ---------------- | ---------- | ----------- |
| `/bmad-bmm-create-prd`       | Create PRD       | 创建产品需求文档   | ✅ 必需        |
| `/bmad-bmm-validate-prd`     | Validate PRD     | 验证 PRD 完整性 | 可选          |
| `/bmad-bmm-edit-prd`         | Edit PRD         | 编辑现有 PRD   | 修订阶段        |
| `/bmad-bmm-create-ux-design` | Create UX Design | 创建用户体验设计   | 推荐（如项目含 UI） |

### 3. 解决方案阶段 (Solutioning Phase)

| Command                                    | 全称                             | 描述        | 必要性  |
| ------------------------------------------ | ------------------------------ | --------- | ---- |
| `/bmad-bmm-create-architecture`            | Create Architecture            | 创建技术架构设计  | ✅ 必需 |
| `/bmad-bmm-create-epics-and-stories`       | Create Epics and Stories       | 创建史诗和用户故事 | ✅ 必需 |
| `/bmad-bmm-check-implementation-readiness` | Check Implementation Readiness | 检查实施准备情况  | ✅ 必需 |

### 4. 实施阶段 (Implementation Phase)

| Command                     | 全称                 | 描述     | 阶段     |
| --------------------------- | ------------------ | ------ | ------ |
| `/bmad-bmm-sprint-planning` | Sprint Planning    | 冲刺计划制定 | ✅ 开始实施 |
| `/bmad-bmm-sprint-status`   | Sprint Status      | 冲刺状态查看 | 冲刺过程中  |
| `/bmad-bmm-create-story`    | Create Story       | 创建用户故事 | 每个故事开始 |
| `/bmad-bmm-dev-story`       | Dev Story          | 开发用户故事 | 核心开发   |
| `/bmad-bmm-code-review`     | Code Review        | 代码审查   | 质量保证   |
| `/bmad-bmm-qa-automate`     | QA Automation Test | 自动化测试  | 增加测试覆盖 |
| `/bmad-bmm-retrospective`   | Retrospective      | 项目回顾   | 史诗结束后  |

### 5. 随时可用工具 (Anytime Tools)

| Command                              | 全称                       | 描述      | 特点        |
| ------------------------------------ | ------------------------ | ------- | --------- |
| `/bmad-bmm-document-project`         | Document Project         | 项目文档化   | 适用于现有项目   |
| `/bmad-bmm-generate-project-context` | Generate Project Context | 生成项目上下文 | AI 优化项目理解 |
| `/bmad-bmm-quick-spec`               | Quick Spec               | 快速技术规格  | 简单任务快速通道  |
| `/bmad-bmm-quick-dev`                | Quick Dev                | 快速开发    | 直接实现小型功能  |
| `/bmad-bmm-correct-course`           | Correct Course           | 调整方向    | 处理重大变更    |
| `/bmad-bmm-write-document`           | Write Document           | 技术文档写作  | 专业文档生成    |
| `/bmad-bmm-update-standards`         | Update Standards         | 更新标准    | 定制文档规范    |
| `/bmad-bmm-mermaid-generate`         | Mermaid Generate         | 图表生成    | 可视化需求     |
| `/bmad-bmm-validate-document`        | Validate Document        | 文档验证    | 质量检查      |

### 6. 核心工具 (Core Tools)

| Command | 全称 | 描述 | 用途 |
|--------|------|------|------|
| `/bmad-brainstorming` | Brainstorming | 生成多样化创意 | 头脑风暴 |
| `/bmad-party-mode` | Party Mode | 多代理讨论 | 获得多角度观点 |
| `/bmad-help` | BMAD Help | 获取帮助 | 紧急情况 |
| `/bmad-index-docs` | Index Docs | 文档索引 | 快速文档扫描 |
| `/bmad-shard-doc` | Shard Document | 文档分片 | 大文档管理 |
| `/bmad-editorial-review-prose` | Editorial Review - Prose | 文字编辑审查 | 语言润色 |
| `/bmad-editorial-review-structure` | Editorial Review - Structure | 结构编辑审查 | 结构优化 |
| `/bmad-review-adversarial-general` | Adversarial Review | 对抗性审查 | 质量保证 |

---

## 🎯 推荐行动路径

### 路径 1: 完整项目规划 (推荐)
1. `/bmad-brainstorming` - 头脑风暴收集创意
2. `/bmad-bmm-create-product-brief` - 明确产品愿景
3. `/bmad-bmm-create-prd` - 创建详细需求文档
4. `/bmad-bmm-create-architecture` - 设计技术架构
5. `/bmad-bmm-create-epics-and-stories` - 分解为具体任务
6. `/bmad-bmm-check-implementation-readiness` - 确保准备就绪
7. `/bmad-bmm-sprint-planning` - 开始实施

### 路径 2: 快速启动
1. `/bmad-bmm-create-product-brief` - 快速定义产品
2. `/bmad-bmm-create-prd` - 直接进入需求阶段

### 路径 3: 简单任务
1. `/bmad-bmm-quick-spec` - 快速创建规格
2. `/bmad-bmm-quick-dev` - 直接实现

### 路径 4: 现有项目
1. `/bmad-bmm-generate-project-context` - 让 AI 理解项目
2. `/bmad-bmm-document-project` - 生成项目文档

---

## 💡 使用建议

### 通用原则
- **新对话窗口**: 每个工作流建议在新的对话窗口中运行
- **保持专注**: 按顺序执行，跳过步骤可能导致后续问题
- **及时反馈**: 定期使用 `/bmad-bmm-sprint-status` 查看进展

### 特殊说明
- **必需标记** (`✅ Required`)：必须完成才能进入下一阶段
- **可选标记**：可根据项目需求选择执行
- **紧急工具**：随时可用的帮助和调整工具

### 项目定制
- 所有配置已针对你的项目 (`bus-int-mon`) 进行优化
- 通信语言：English（可从配置中修改）
- 输出位置：`_bmad-output/` 目录

---

## 🚀 开始建议

**如果你刚开始项目**：
```
/bmad-brainstorming
→ /bmad-bmm-create-product-brief
→ /bmad-bmm-create-prd
```

**如果你已经有想法**：
```
/bmad-bmm-create-prd
```

**如果你只是想快速实现一个小功能**：
```
/bmad-bmm-quick-spec
```

---

*文档生成时间：2026-02-24*