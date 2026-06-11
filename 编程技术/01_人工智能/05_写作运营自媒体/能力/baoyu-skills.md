# baoyu-skills - Claude Code 技能集

> 来源：https://github.com/JimLiu/baoyu-skills

---

## 0. 整体介绍

baoyu-skills 是宝玉分享的 Claude Code 技能集，旨在提升日常工作效率。这个技能集提供了丰富的内容生成、发布和工具类功能，特别适合内容创作者、自媒体运营者使用。

**主要特性：**
- 支持小红书信息图、封面图、幻灯片、知识漫画等多种内容生成
- 支持自动发布到 X (Twitter) 和微信公众号
- 多种视觉风格可选（19+种风格）
- 支持自定义扩展

**前置要求：**
- 已安装 Node.js 环境
- 能够运行 `npx bun` 命令

---

## 1. 安装步骤

### 方式一：快速安装（推荐）

```bash
npx add-skill jimliu/baoyu-skills
```

### 方式二：注册插件市场

1. 在 Claude Code 中运行：
   ```bash
   /plugin marketplace add jimliu/baoyu-skills
   ```

2. 通过浏览界面安装：
   - 选择 **Browse and install plugins**
   - 选择 **baoyu-skills**
   - 选择要安装的插件
   - 选择 **Install now**

### 方式三：直接安装指定插件

```bash
# 安装内容生成插件
/plugin install content-skills@baoyu-skills

# 安装 AI 生成插件
/plugin install ai-generation-skills@baoyu-skills

# 安装工具插件
/plugin install utility-skills@baoyu-skills
```

### 方式四：告诉 Agent

直接告诉 Claude Code：
> 请帮我安装 github.com/JimLiu/baoyu-skills 中的 Skills

### 更新技能

1. 运行 `/plugin`
2. 切换到 **Marketplaces** 标签页
3. 选择 **baoyu-skills**
4. 选择 **Update marketplace**
5. 或选择 **Enable auto-update** 启用自动更新

---

## 2. 可用技能详解

### 插件包概览

| 插件包 | 说明 | 包含技能数量 |
|--------|------|-------------|
| **content-skills** | 内容生成和发布 | 7 个 |
| **ai-generation-skills** | AI 生成后端 | 1 个 |
| **utility-skills** | 内容处理工具 | 2 个 |

---

### 2.1 内容技能 (Content Skills)

#### baoyu-xhs-images - 小红书信息图生成器

**功能**：将内容拆解为 1-10 张卡通风格信息图，支持 **风格 × 布局** 二维系统。

**用法**：
```bash
# 自动选择风格和布局
/baoyu-xhs-images posts/ai-future/article.md

# 指定风格
/baoyu-xhs-images posts/ai-future/article.md --style notion

# 指定布局
/baoyu-xhs-images posts/ai-future/article.md --layout dense

# 组合风格和布局
/baoyu-xhs-images posts/ai-future/article.md --style tech --layout list

# 直接输入内容
/baoyu-xhs-images 今日星座运势
```

**风格选项**：
- `cute`（默认）、`fresh`、`tech`、`warm`、`bold`、`minimal`、`retro`、`pop`、`notion`

**布局选项**：
| 布局 | 密度 | 适用场景 |
|------|------|----------|
| `sparse` | 1-2 点 | 封面、金句 |
| `balanced` | 3-4 点 | 常规内容 |
| `dense` | 5-8 点 | 知识卡片、干货总结 |
| `list` | 4-7 项 | 清单、排行 |
| `comparison` | 双栏 | 对比、优劣 |
| `flow` | 3-6 步 | 流程、时间线 |

---

#### baoyu-cover-image - 文章封面图生成器

**功能**：为文章生成手绘风格封面图，支持 19 种风格。

**用法**：
```bash
# 从 markdown 文件生成（自动选择风格）
/baoyu-cover-image path/to/article.md

# 指定风格
/baoyu-cover-image path/to/article.md --style tech

# 不包含标题文字
/baoyu-cover-image path/to/article.md --no-title
```

**可用风格**：
`elegant`（默认）、`blueprint`、`bold-editorial`、`chalkboard`、`dark-atmospheric`、`editorial-infographic`、`fantasy-animation`、`intuition-machine`、`minimal`、`nature`、`notion`、`pixel-art`、`playful`、`retro`、`sketch-notes`、`vector-illustration`、`vintage`、`warm`、`watercolor`

---

#### baoyu-slide-deck - 幻灯片生成器

**功能**：从内容生成专业的幻灯片图片，自动合并为 `.pptx` 文件。

**用法**：
```bash
# 从 markdown 文件生成
/baoyu-slide-deck path/to/article.md

# 指定风格和受众
/baoyu-slide-deck path/to/article.md --style corporate
/baoyu-slide-deck path/to/article.md --audience executives

# 仅生成大纲（不生成图片）
/baoyu-slide-deck path/to/article.md --outline-only

# 指定语言
/baoyu-slide-deck path/to/article.md --lang zh
```

**风格选项**：
| 风格 | 描述 | 适用场景 |
|------|------|----------|
| `blueprint`（默认） | 技术蓝图风格 | 架构设计、系统设计 |
| `notion` | SaaS 仪表盘美学 | 产品演示、SaaS、B2B |
| `bold-editorial` | 杂志社论风格 | 产品发布、主题演讲 |
| `corporate` | 海军蓝/金色配色 | 投资者演示、客户提案 |
| `dark-atmospheric` | 电影级暗色调 | 娱乐、游戏、创意 |
| `minimal` | 极简风格 | 高管简报、高端品牌 |
| `pixel-art` | 复古像素风 | 游戏、开发者分享 |
| `scientific` | 学术图表 | 生物、化学、医学 |
| 等更多... | | |

---

#### baoyu-comic - 知识漫画创作器

**功能**：创作带有详细分镜布局的原创教育漫画。

**用法**：
```bash
# 从素材文件生成
/baoyu-comic posts/turing-story/source.md

# 指定风格
/baoyu-comic posts/turing-story/source.md --style dramatic
/baoyu-comic posts/turing-story/source.md --style ohmsha

# 自定义风格（自然语言描述）
/baoyu-comic posts/turing-story/source.md --style "水彩风格，边缘柔和"

# 指定布局和比例
/baoyu-comic posts/turing-story/source.md --layout cinematic
/baoyu-comic posts/turing-story/source.md --aspect 16:9

# 指定语言
/baoyu-comic posts/turing-story/source.md --lang zh

# 直接输入内容
/baoyu-comic "图灵的故事与计算机科学的诞生"
```

**选项**：
| 选项 | 取值 |
|------|------|
| `--style` | `classic`（默认）、`dramatic`、`warm`、`sepia`、`vibrant`、`ohmsha`、`realistic`、`wuxia`，或自然语言描述 |
| `--layout` | `standard`（默认）、`cinematic`、`dense`、`splash`、`mixed`、`webtoon` |
| `--aspect` | `3:4`（默认，竖版）、`4:3`（横版）、`16:9`（宽屏） |
| `--lang` | `auto`（默认）、`zh`、`en`、`ja` 等 |

**风格说明**：
- `classic` - 传统清线风格（传记、教育内容）
- `dramatic` - 高对比度（冲突、高潮场景）
- `ohmsha` - 欧姆社漫画风格（技术教程）
- `wuxia` - 港漫武侠风格（武侠、仙侠）

---

#### baoyu-article-illustrator - 文章插图生成器

**功能**：智能分析文章内容，在需要视觉辅助的位置生成插图。

**用法**：
```bash
# 根据内容自动选择风格
/baoyu-article-illustrator path/to/article.md

# 指定风格
/baoyu-article-illustrator path/to/article.md --style warm
/baoyu-article-illustrator path/to/article.md --style watercolor
```

**可用风格**（20 种）：
`notion`（默认）、`elegant`、`warm`、`minimal`、`playful`、`nature`、`sketch`、`watercolor`、`vintage`、`scientific`、`chalkboard`、`editorial`、`flat`、`retro`、`blueprint`、`vector-illustration`、`sketch-notes`、`pixel-art`、`intuition-machine`、`fantasy-animation`

---

#### baoyu-post-to-x - 发布到 X (Twitter)

**功能**：发布内容和文章到 X (Twitter)，支持带图片的普通帖子和 X 文章（长篇 Markdown）。使用真实 Chrome + CDP 绕过反自动化检测。

**用法**：
```bash
# 发布文字
/baoyu-post-to-x "Hello from Claude Code!"

# 发布带图片
/baoyu-post-to-x "看看这个" --image photo.png

# 发布 X 文章
/baoyu-post-to-x --article path/to/article.md
```

---

#### baoyu-post-to-wechat - 发布到微信公众号

**功能**：发布内容到微信公众号，支持图文和文章两种模式。

**前置要求**：已安装 Google Chrome，首次运行需扫码登录（登录状态会保存）

**图文模式** - 多图配短标题和正文：
```bash
/baoyu-post-to-wechat 图文 --markdown article.md --images ./photos/
/baoyu-post-to-wechat 图文 --markdown article.md --image img1.png --image img2.png
/baoyu-post-to-wechat 图文 --title "标题" --content "内容" --image img1.png --submit
```

**文章模式** - 完整 markdown/HTML 富文本格式：
```bash
/baoyu-post-to-wechat 文章 --markdown article.md
/baoyu-post-to-wechat 文章 --markdown article.md --theme grace
/baoyu-post-to-wechat 文章 --html article.html
```

---

### 2.2 AI 生成技能 (AI Generation Skills)

#### baoyu-danger-gemini-web - Gemini Web 交互

**功能**：与 Gemini Web 交互，生成文本和图片。

> ⚠️ **警告**：此技能使用逆向工程的 Gemini Web API，非官方 API，使用风险自负。

**文本生成**：
```bash
/baoyu-danger-gemini-web "你好，Gemini"
/baoyu-danger-gemini-web --prompt "解释量子计算"
```

**图片生成**：
```bash
/baoyu-danger-gemini-web --prompt "一只可爱的猫" --image cat.png
/baoyu-danger-gemini-web --promptfiles system.md content.md --image out.png
```

**代理配置**（中国大陆用户）：
```bash
HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 /baoyu-danger-gemini-web "你好"
```

---

### 2.3 工具技能 (Utility Skills)

#### baoyu-danger-x-to-markdown - X 内容转 Markdown

**功能**：将 X (Twitter) 内容转换为 markdown 格式，支持推文串和 X 文章。

> ⚠️ **警告**：此技能使用逆向工程的 X API，非官方 API，使用风险自负。

**用法**：
```bash
# 将推文转换为 markdown
/baoyu-danger-x-to-markdown https://x.com/username/status/123456

# 保存到指定文件
/baoyu-danger-x-to-markdown https://x.com/username/status/123456 -o output.md

# JSON 输出
/baoyu-danger-x-to-markdown https://x.com/username/status/123456 --json
```

**支持的 URL**：
- `https://x.com/*/status/*`
- `https://twitter.com/*/status/*`
- `https://x.com/i/article/*`

**身份验证**：使用环境变量（`X_AUTH_TOKEN`、`X_CT0`）或 Chrome 登录进行 cookie 认证。

---

#### baoyu-compress-image - 图片压缩

**功能**：压缩图片以减小文件大小，同时保持质量。

**用法**：
```bash
/baoyu-compress-image path/to/image.png
/baoyu-compress-image path/to/images/ --quality 80
```

---

### 2.4 自定义扩展配置

所有技能支持通过 `EXTEND.md` 文件自定义。可覆盖默认样式、添加自定义配置或定义个人预设。

**扩展路径**（按优先级检查）：
1. `.baoyu-skills/<skill-name>/EXTEND.md` - 项目级（团队/项目特定设置）
2. `~/.baoyu-skills/<skill-name>/EXTEND.md` - 用户级（个人偏好设置）

**示例**：为 `baoyu-cover-image` 自定义品牌配色：

```bash
mkdir -p .baoyu-skills/baoyu-cover-image
```

创建 `.baoyu-skills/baoyu-cover-image/EXTEND.md`：

```markdown
## 自定义风格

### brand
- 主色：#1a73e8
- 辅色：#34a853
- 字体风格：现代无衬线
- 始终包含公司 logo 水印
```

---

## 3. 总结

### 核心价值

baoyu-skills 是一套功能强大的 Claude Code 技能集，特别适合：

1. **内容创作者**：快速生成小红书信息图、文章封面、幻灯片、知识漫画
2. **自媒体运营**：一键发布到微信公众号和 X (Twitter)
3. **知识分享者**：将复杂概念可视化，生成教育漫画

### 主要优势

- **风格丰富**：19+ 种视觉风格可选，满足不同场景需求
- **一键生成**：从 Markdown 文件直接生成多种格式的内容
- **自动化发布**：支持自动发布到主流社交媒体平台
- **可扩展**：通过 EXTEND.md 自定义个人风格和配置

### 注意事项

1. 带有 `danger-` 前缀的技能使用逆向工程 API，存在风险
2. 发布功能需要 Chrome 浏览器支持
3. 部分功能可能需要代理访问（中国大陆用户）

### 许可证

MIT License

---

> 文档整理时间：2026-02-21
>
> 原始文档：https://github.com/JimLiu/baoyu-skills/blob/main/README.zh.md
