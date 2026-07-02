# spec_lock.md — 即时表达（ppt-master 模式）

## Canvas
- width: 1280
- height: 720
- content_area: { x: 60, y: 110, w: 1160, h: 590 }

## Colors
- bg: "#FFFFFF"
- bg_alt: "#F8FAFC"
- primary: "#0F2A43"
- secondary: "#2563EB"
- accent: "#E8A838"
- body_text: "#1E293B"
- muted_text: "#64748B"
- success: "#22C55E"
- warning: "#EF4444"

## Typography
- font_family: "Microsoft YaHei"
- title: { size: 28, weight: bold }
- subtitle: { size: 20, weight: regular }
- body: { size: 16, weight: regular }
- annotation: { size: 14, weight: regular }
- hero_number: { size: 40, weight: bold }

## Spacing
- card_gap: 25
- card_padding: 20
- card_radius: 8
- row_gap: 32
- bar_height: 50
- badge_diameter: 40

## Page Layouts
- P01: cover_centered
- P02: split_comparison
- P03: vertical_numbered_list
- P04: venn_comparison
- P05: center_radial
- P06: vertical_staircase
- P07: three_column_plus_tags
- P08: summary_cards

## Per-Page Design Decisions

### P01 (cover_centered)
- bg: white
- title: centered 56px primary
- subtitle: centered 28px muted
- decoration: gold line above title, gradient bar at bottom (secondary→sky)

### P02 (split_comparison)
- header: dark bar 1280×90
- left: WHAT card (bg_alt, blue top bar)
- right: HOW card (amber bg, gold top bar)

### P03 (vertical_numbered_list)
- header: dark bar 1280×90
- 4 rows, each 130px high
- last row dark bg (emphasis)

### P04 (venn_comparison)
- header: dark bar 1280×90
- two overlapping circles
- bottom conclusion boxes

### P05 (center_radial)
- header: dark bar 1280×90
- center dual circle (secondary outer, primary inner)
- left green radiation, right red radiation
- bottom amber inspiration strip + 3 strategy cards

### P06 (vertical_staircase)
- header: dark bar 1280×90
- 4 steps with progressive indent
- right panel with example

### P07 (three_column_plus_tags)
- header: dark bar 1280×90
- 3 numbered structure cards
- 5 style tag chips
- full example + dark quote bar

### P08 (summary_cards)
- bg: bg_alt (light)
- 5 cards in 2+2+1 layout
- dark quote box with gold text
