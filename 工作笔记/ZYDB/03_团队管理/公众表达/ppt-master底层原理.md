# PPT-Master 底层原理

> 基于 [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) 2026-06-28 版本
> 本文是对上游架构的完整解析，帮助你理解"AI 生成 PPT"到底是怎么回事。

---

## 一、核心问题：谁在设计 PPT？

**答案是：AI（大语言模型）在设计，不是程序。**

```
传统理解（错误）：  内容 → 程序自动排版 → PPT
PPT-Master（实际）：内容 → AI 逐页设计 SVG → 程序转换 → PPT
```

程序只做两件事：
1. **后处理 SVG**（嵌入图标、裁剪图片、优化圆角）
2. **SVG → PPTX 格式转换**（DrawingML 原生形状）

所有"设计决策"——这页用四宫格还是流程图、卡片放左边还是右边、标题多大字号——都是 AI 做的。

---

## 二、完整管道（7 步）

```
源文件（PDF/DOCX/MD/URL）
    │
    ├─ Step 1: 源文件 → Markdown
    │   （pdf_to_md / doc_to_md / ppt_to_md / web_to_md）
    │
    ├─ Step 2: 创建项目目录
    │   （project_manager.py init）
    │
    ├─ Step 3: 模板选项（可选，用户主动提供才触发）
    │
    ├─ Step 4: ★ Strategist（战略家）★
    │   AI 分析内容 → 八项确认 → 输出 design_spec.md + spec_lock.md
    │   ⛔ BLOCKING：需要用户确认才能继续
    │
    ├─ Step 5: 图片获取（可选）
    │   （AI 生图 / 网络搜图 / 用户提供）
    │
    ├─ Step 6: ★ Executor（执行器）★
    │   AI 逐页手写 SVG → svg_output/01_封面.svg, 02_xxx.svg ...
    │
    └─ Step 7: 后处理 + 导出
        finalize_svg.py → svg_to_pptx.py → exports/*.pptx
```

---

## 三、两个核心角色

### Strategist（战略家）— Step 4

**做什么**：读取源内容，规划整个 PPT 的设计方案。

**输入**：源文件 Markdown

**输出**：两个文件
- `design_spec.md` — 人类可读的设计规范（11 个章节）
- `spec_lock.md` — 机器可读的执行契约（配色/字体/布局锁定值）

**八项确认**（唯一的阻塞门控，用户必须拍板）：

| 层级 | 确认项 | 说明 |
|------|--------|------|
| **第一层（锚点）** | a. 画布尺寸 | 16:9 还是 4:3 |
| | c. 受众 + 内容策略 | 忠实原文还是自由发挥 |
| | d. 叙事模式 + 视觉风格 | 金字塔式/叙事式/教学式... |
| **第二层（推导）** | b. 页数 | 根据内容量和目的推导，不是拍脑袋 |
| | e. 配色方案 | 3 个候选供选择 |
| | f. 图标库 | Tabler / Lucide / Phosphor |
| | g. 排版 | 标题/正文/注释的字号 |
| | h. 图片策略 | AI 生图 / 网络搜图 / 无图 |

关键原则：**第一层确认后，第二层自动推导**。用户只需确认 3 个核心选择，其余自动计算。

### Executor（执行器）— Step 6

**做什么**：根据 spec_lock.md，**逐页手写 SVG 代码**。

**输入**：
- `spec_lock.md`（每页生成前必须重新读取，防止长文档记忆漂移）
- `design_spec.md §IX`（页面大纲）
- 模板 SVG（如果有，批量预读一次）

**输出**：`svg_output/<序号>_<页面名>.svg`，每页一个文件

**核心规则**：
1. **逐页生成**：一次写一页，写完再写下一页
2. **禁止脚本批量生成**：不允许写 Python 循环出 SVG（规则第 9 条）
3. **spec_lock 是唯一真相**：配色/字体从文件读取，不准凭记忆
4. **模板提供结构不提供皮肤**：即使有模板，颜色和字号也要换成 spec_lock 的值

---

## 四、模板是什么角色？

**模板是可选的参考，不是必须的框架。**

### 有模板时

```
Strategist 选择模板 → spec_lock.md 记录每页用哪个布局
Executor 生成时 → 继承模板的几何结构（卡片位置、装饰线条）
                → 替换为演示文稿自己的配色和字体
```

模板提供：
- 页面骨架（封面/目录/内容页/结束页的布局）
- 装饰元素位置（几何图形、线条）
- 图表结构（坐标轴、图例位置）

模板**不**提供：
- 配色（始终从 spec_lock 重新应用）
- 字体大小（始终从 spec_lock 重新应用）
- 具体文字内容

### 没有模板时（Free Design）

```
Strategist 不指定模板 → spec_lock.md 中 page_layouts 为空
Executor 自由设计 → 每页布局由 AI 根据内容自行决定
```

**我刚才给你做的那版"上游模式"就是 Free Design** — 没有使用任何模板，8 页布局全部是 AI 根据内容逻辑逐页设计的。

### 三种模板复制模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **standard** | 继承结构，替换皮肤和内容 | 大多数情况 |
| **fidelity** | 严格复制 + AI 清理 | 参考已有 PPT 风格 |
| **mirror** | 逐字复制所有视觉元素，只改文字 | 1:1 还原已有 PPT |

---

## 五、SVG 是格式控制的唯一真相

```
          SVG（人类/AI 可读可编辑）
           │
     ┌─────┴─────┐
     ▼           ▼
 finalize    svg_to_pptx
 (后处理)     (格式转换)
     │           │
     ▼           ▼
 svg_final   exports/*.pptx
 (优化后)     (原生 DrawingML)
```

**为什么不直接生成 PPTX？**

| | 直接操作 PPTX | SVG → PPTX |
|---|---|---|
| AI 生成难度 | 极高（OOXML 极其复杂） | 低（SVG 是标准矢量格式，AI 熟悉） |
| 可编辑性 | 差（XML 嵌套深） | 好（SVG 直观可读） |
| 视觉保真度 | 容易出错 | 高（所见即所得） |
| 工具链 | 需要专门的 PPTX 解析库 | finalize_svg + svg_to_pptx 自动处理 |

所以整个架构的核心选择是：**让 AI 写它最擅长的东西（SVG），然后用程序精确地翻译成 PPTX**。

---

## 六、关键架构原则

### 1. spec_lock.md 胜出原则
当 design_spec.md（人类可读）和 spec_lock.md（机器可读）冲突时，**spec_lock.md 胜出**。执行器在每一页生成前都重新读取它，抵抗长文档中的"记忆漂移"。

### 2. 字号是皮肤不是几何
模板里写的 `font-size="12"` 永远不会被继承。每个文字角色（标题/正文/注释）都映射到 spec_lock 中锁定的字号。如果锁定的字号比模板大，就**放大容器**而不是缩小字体。

### 3. 纯像素（px-only）
整个系统不使用 pt（磅），所有尺寸都是 px。spec_lock、SVG、导出全程统一。

### 4. 路由是机械的
PPTX 处理决策是确定性的：
- 原始 PPTX + "生成" → template-fill 路径（直接 OOXML 编辑，不经过 SVG）
- 源文档 + "重新构建" → 主管道（SVG 路径）
- 源文档 + "保留布局" → beautify 路径（1:1 重做）

AI 不做路由解释，按规则走。

### 5. 禁止脚本批量生成 SVG
SKILL.md 第 9 条规则明确禁止。原因：跨页视觉一致性需要逐页根据上下文调整，脚本做不到。

---

## 七、文件结构

```
项目目录/
├── sources/              # 源文件 Markdown
├── analysis/             # 分析产物（source_profile.json 等）
├── templates/            # 选用的模板（可选）
│   └── design_spec.md    # 模板的设计规范
├── design_spec.md        # ★ Strategist 输出：人类可读设计规范
├── spec_lock.md          # ★ Strategist 输出：机器可读执行契约
├── images/               # 图片资源
├── icons/                # 图标资源
├── notes/
│   ├── total.md          # 全部演讲稿
│   └── 01_*.md ~ NN_*.md # 按页拆分的演讲稿
├── svg_output/           # ★ Executor 输出：逐页 SVG（原始）
│   ├── 01_封面.svg
│   ├── 02_表达的本质.svg
│   └── ...
├── svg_final/            # 后处理后的 SVG（导出源）
├── backup/               # svg_output 的自动备份
├── exports/              # ★ 最终 PPTX 输出
│   └── *.pptx
└── README.md
```

---

## 八、与我们之前做法的对比

| | 我们的做法（脚本参数化） | 上游做法（逐页手写） |
|---|---|---|
| **设计决策** | AI 做（在 MD 草稿阶段） | AI 做（在 Strategist 阶段） |
| **SVG 生成** | Python 脚本批量 | AI 逐页手写 |
| **布局灵活性** | 低（一个模板套所有页） | 高（每页根据内容选布局） |
| **可复现性** | 高（同输入 = 同输出） | 中（同输入可能微调） |
| **质量** | 一般（布局雷同） | 高（布局多样、贴合内容） |
| **效率** | 高（脚本秒出） | 低（逐页写 SVG 耗时长） |
| **稳定性控制** | 天然稳定 | 靠 spec_lock.md 强制锁定 |

### 混合方案（我们的改进方向）

保留我们 `AI → MD草稿 → 程序 → SVG` 的稳定流程，但让程序支持**多种布局渲染器**：

```
AI 设计 MD 草稿（每页指定布局类型）
    │
    ▼
程序根据草稿选择对应渲染器：
  四宫格 → render_card_grid()
  对比框 → render_comparison()
  流程图 → render_flowchart()
  金字塔 → render_pyramid()
    │
    ▼
程序输出 SVG（确定性渲染，可复现）
```

这样既有逐页手写的**布局灵活性**，又有脚本生成的**可复现性**。

---

## 九、总结

PPT-Master 的本质是一个 **AI 驱动的 SVG 逐页生成管道**：

1. AI 读取内容，规划设计方案（Strategist）
2. AI 逐页手写 SVG 代码（Executor）
3. 程序后处理 SVG（嵌入图标/裁剪图片）
4. 程序将 SVG 转为 PPTX（原生可编辑形状）

模板是可选的布局参考，配色和字体始终从锁定的设计规范中读取。整个系统的核心设计选择是：**让 AI 写 SVG（它擅长的），让程序做格式转换（它擅长的）**。

---

## 十、我们的流程与 ppt-master 的关系

### ppt-master 是什么角色？

ppt-master 是我们使用的**底层引擎**，不是我们的全部流程。

```
我们的 PPT 生成体系
├── ppt-design skill（我们的，上层编排者）
│   ├── 模板管理（templates/*.md 配置文件）
│   ├── 工作流程（SKILL.md 定义的步骤）
│   ├── 排版参数表（字号/间距/元素尺寸）
│   └── 设计稿 A/B 双部分（创意+锁定）
│
└── ppt-master（上游工具，底层引擎）
    ├── project_manager.py  ← 我们用
    ├── finalize_svg.py     ← 我们用
    ├── svg_to_pptx.py      ← 我们用
    ├── Strategist 工作流   ← 我们不用（用自己的替代）
    └── Executor 工作流     ← 我们不用（用自己的替代）
```

### 用了什么 vs 没用什么

| ppt-master 组件 | 我们是否使用 | 原因 |
|----------------|-------------|------|
| `project_manager.py` | ✅ 用 | 项目初始化，无替代品 |
| `finalize_svg.py` | ✅ 用 | SVG 后处理（圆角转Path等），无替代品 |
| `svg_to_pptx.py` | ✅ 用 | SVG → PPTX 格式转换，核心引擎 |
| layout templates | ✅ 用（参考） | 7种布局模板作为设计参考 |
| Strategist（八项确认） | ❌ 不用 | 太复杂，我们用"页面规划+设计稿确认"替代 |
| Executor（逐页手写） | ❌ 不用 | 太慢且不稳定，我们用"A/B 设计稿锁定后执行"替代 |
| spec_lock.md 机制 | ✅ 吸收理念 | 融入设计稿 B 部分，不单独建文件 |
| confirm-ui / live-preview | ❌ 不用 | 交互式 UI，我们是 CLI/对话模式 |
| image-search / AI 插画 | ❌ 不用 | 依赖外部 API，暂时不需要 |

### 两种工作流对比

```
ppt-master 原始流程：
  源文件 → Strategist（八项确认 → design_spec + spec_lock）
       → Executor（逐页手写 SVG，每页重读 spec_lock）
       → finalize → export

我们的流程：
  源文件 → 选模板（读排版参数）
       → 页面规划（内容逻辑分配）
       → 设计稿 A 部分（AI 发挥创意，ASCII 布局图）
       → 设计稿 B 部分（从 A 提取精确参数锁定）
       → 用户确认 A 部分
       → SVG 生成（严格按 B 部分 + 排版参数执行）
       → finalize → export
```

### 核心差异：创意与执行的分离

| | ppt-master | 我们的方案 |
|---|---|---|
| 创意在哪 | Strategist 阶段 | 设计稿 A 部分 |
| 锁定在哪 | spec_lock.md（单独文件） | 设计稿 B 部分（同文件下半部分） |
| 执行在哪 | Executor 逐页手写 | SVG 生成严格按 B 部分取值 |
| 稳定性保障 | 每页重读 spec_lock | 每页重读 B 部分 |
| 模板参数 | spec_lock 中包含 | 模板配置文件（排版参数表） |

**我们的方案吸取了 ppt-master 的核心理念（创意→锁定→执行），但简化了流程、增加了模板管理。**

### 我们做的改进（超出 ppt-master 的部分）

| 改进 | 说明 |
|------|------|
| **模板配置文件** | 每个模板有独立的 .md 配置（配色/排版参数/内容区/页面规范），ppt-master 没有 |
| **排版参数表** | 按比例压缩（通用版100% → 金圆版~50%），解决小画布适配，ppt-master 没有这个机制 |
| **设计稿 A/B 双部分** | 一个文件同时给人看（ASCII 图）和给 AI 看（精确参数），ppt-master 分成两个文件 |
| **金圆版直接操作 .pptx** | 金圆模板用 python-pptx 直接在真实底版上操作，不走 SVG 路径，ppt-master 不支持 |
| **模板可扩展** | 新增模板 = 复制 .md 配置文件 + 填参数，ppt-master 的模板系统更复杂 |

### 一句话总结关系

**ppt-master 是我们的底层引擎（用它的工具链），ppt-design skill 是我们的上层大脑（定义流程和模板管理）。我们吸收了 ppt-master 的"创意→锁定→执行"理念，但用自己的方式实现（设计稿 A/B 替代 Strategist + spec_lock），同时增加了模板管理和排版参数压缩机制。**
