---
title: MOC 索引
type: moc
created: 2026-03-16
updated: 2026-03-24
---

# MOC：知识库总索引

> 内容地图 | 最后更新：`$= dv.current().file.mtime`

---

## 目录结构

```
知识笔记/
├── 01_配置/              # 配置层
├── 02_缓冲区/            # 入库缓冲
├── 03_知识库/            # 四层知识
│   ├── 10_数据源/
│   ├── 20_摘要/
│   ├── 30_主题卡片/
│   └── 40_自我笔记/
└── 04_视图/              # Dashboard + MOC
```

---

## 学科索引

```dataview
TABLE without id
  file.link as "学科MOC",
  length(rows) as "卡片数"
FROM "04_视图"
WHERE file.name != "Dashboard" AND file.name != "MOC_索引" AND file.name.contains("MOC_")
GROUP BY file.link
```

| 学科 | 说明 | 链接 |
|------|------|------|
| 心理学 | 认知偏见、决策科学 | [[MOC_心理学]] |

---

## 类型索引

```dataview
TABLE without id
  choice(file.folder.contains("31_概念"), "💡 概念",
    choice(file.folder.contains("32_定义"), "📖 定义",
    choice(file.folder.contains("34_案例"), "📝 案例",
    choice(file.folder.contains("35_引文"), "💬 引文",
    choice(file.folder.contains("37_方法"), "⚙️ 方法", "📁 其他"))))) as "类型",
  length(rows) as "数量"
FROM "03_知识库/30_主题卡片"
WHERE !file.name.contains("MOC")
GROUP BY file.folder
```

### 按类型浏览

| 类型 | 说明 | 目录 |
|------|------|------|
| 概念 | 核心知识单元 | [[03_知识库/30_主题卡片/31_概念]] |
| 方法 | 可操作流程 | [[03_知识库/30_主题卡片/37_方法]] |
| 案例 | 经典实例 | [[03_知识库/30_主题卡片/34_案例]] |
| 引文 | 精辟金句 | [[03_知识库/30_主题卡片/35_引文]] |

---

## 数据源索引

```dataview
TABLE without id
  choice(file.folder.contains("11_书籍"), "📚 书籍",
    choice(file.folder.contains("12_视频"), "🎬 视频",
    choice(file.folder.contains("13_文章"), "📄 文章",
    choice(file.folder.contains("14_AI对话"), "🤖 AI对话", "📁 其他")))) as "类型",
  file.link as "标题",
  dateformat(file.ctime, "MM-dd") as "添加日期"
FROM "03_知识库/10_数据源"
WHERE !file.name.contains("MOC")
SORT file.ctime DESC
LIMIT 10
```

---

## 项目索引

```dataview
TABLE without id
  file.link as "项目",
  status as "状态",
  due as "截止日期"
FROM "03_知识库/40_自我笔记/41_Projects"
WHERE !file.name.contains("MOC")
SORT due ASC
```

---

## 最近更新

### 最近创建的卡片

```dataview
LIST file.link + " (" + dateformat(file.ctime, "MM-dd") + ")"
FROM "03_知识库/30_主题卡片"
WHERE !file.name.contains("MOC")
SORT file.ctime DESC
LIMIT 5
```

### 最近添加的数据源

```dataview
LIST file.link + " (" + dateformat(file.ctime, "MM-dd") + ")"
FROM "03_知识库/10_数据源"
WHERE !file.name.contains("MOC")
SORT file.ctime DESC
LIMIT 5
```

---

## 快速入口

| 入口 | 说明 |
|------|------|
| [[Dashboard]] | 📊 知识库仪表盘 |
| [[01_配置/知识库设计方案]] | 📋 完整设计方案 |
| [[01_配置/模板库/模板_概念卡片]] | 📝 概念卡片模板 |

---

## 知识流向

```
02_缓冲区 → 03_知识库/10_数据源 → 03_知识库/20_摘要 → 03_知识库/30_主题卡片 → 03_知识库/40_自我笔记
   ↑                                                                              ↓
   └──────────────────── 归档 ←──────────────────────────────────────────────────┘
```
