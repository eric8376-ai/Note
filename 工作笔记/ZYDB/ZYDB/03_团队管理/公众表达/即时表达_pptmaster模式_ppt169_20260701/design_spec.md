# Design Spec — 即时表达（ppt-master 模式）

## I. Project Info
- name: 即时表达
- canvas: ppt169 (1280×720)
- page_count: 8
- audience: 内部团队
- use_case: training
- delivery_purpose: presentation
- content_divergence: balanced

## II. Canvas
- format: ppt169
- dimensions: 1280×720px
- content_area: x=60, y=110, w=1160, h=590
- margin: top=40, bottom=40, left=50, right=50

## III. Visual Theme
- mode: instructional
- visual_style: corporate_clean
- theme: light
- palette:
  - bg: "#FFFFFF"
  - bg_alt: "#F8FAFC"
  - primary: "#0F2A43"
  - secondary: "#2563EB"
  - accent: "#E8A838"
  - body_text: "#1E293B"
  - muted_text: "#64748B"
  - success: "#22C55E"
  - warning: "#EF4444"

## IV. Typography
- font_family: "Microsoft YaHei"
- roles:
  - title: 28px bold
  - subtitle: 20px regular
  - body: 16px regular
  - annotation: 14px regular
  - footnote: 12px regular
  - hero_number: 40px bold

## V. Page Roster
- P01: cover — 居中标题 + 底部渐变带
- P02: content — 左右双栏对比
- P03: content — 纵向编号列表（每项不同视觉权重）
- P04: content — 双圆 Venn 对比 + 结论框
- P05: content — 中心聚焦圆 + 左右辐射
- P06: content — 纵向阶梯 + 右侧范例
- P07: content — 三列结构 + 标签条
- P08: ending — 编号卡片 + 深色金句

## VI. Icons
- library: none

## VII. Charts
- none

## VIII. Images
- none

## IX. Outline
- P01: 封面 — 即时表达 / 公众发言的核心艺术
- P02: 表达本质 — WHAT(前端/后端/护城河) vs HOW(流畅性/场景模式)
- P03: 四大原则 — 00目的/01时间/02线性/03带宽
- P04: 权威来源 — 信任(情感) vs 权威(理性) + Venn交叠
- P05: 流畅性启发 — 认知压榨 + 轻松=正确/困难=可疑
- P06: 场景表达 — 钩子→结论→观点→行动 + 范例
- P07: 实战模板 — 结构(结论前置/三点/建议) + 风格标签
- P08: 总结 — 5要点 + 金句

## X. Speaker Notes
- 见 notes/total.md

## XI. Technical Constraints
- 纯文字 + 几何形状
- 无图片、无图标、无图表
- SVG 技术约束遵循 shared-standards.md
