# 🏠 Dashboard
---
## 📊 我的分析
```dataview 
table 
file.tags
from "03_知识库/10_数据源" 

```


---

## 📥 缓冲区

```dataviewjs
const pages = dv.pages('"02_缓冲区"');
const stats = {};

pages.forEach(p => {
  const parts = p.file.folder.split("/");
  const l1 = parts[1] || "根目录";
  const l2 = parts[2] || " ";
  const l3 = parts[3] || " ";
  const key = `${l1}|${l2}|${l3}`;
  if (!stats[key]) stats[key] = { l1, l2, l3, count: 0 };
  stats[key].count++;
});

const rows = Object.values(stats)
  .sort((a, b) => a.l1.localeCompare(b.l1) || a.l2.localeCompare(b.l2) || a.l3.localeCompare(b.l3))
  .map(s => [s.l1, s.l2, s.l3, s.count]);

dv.table(["一级目录", "二级目录", "三级目录", "文件数"], rows);
```

---

## 💡 主题卡片

```dataview
TABLE WITHOUT ID
  file.link AS "卡片",
  file.folder AS "分类",
  dateformat(file.ctime, "MM-dd") AS "创建"
FROM "03_知识库/30_主题卡片"
SORT file.ctime DESC
```

---

## 🚀 快速入口

| 入口 | 说明 |
|:-----|:-----|
| [[MOC_索引]] | 📑 内容地图 |
| [[01_配置/知识库设计方案]] | 📋 设计方案 |
| [[01_配置/模板库/模板_概念卡片]] | 📝 概念卡片模板 |

---

*✨ 知识在于积累，智慧在于连接 ✨*
