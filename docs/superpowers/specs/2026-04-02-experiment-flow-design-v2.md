# ExperimentFlow Component Design Spec (Static)

**Date:** 2026-04-02
**Component:** `components/ExperimentFlow.vue`
**Project:** lettuce-ppt (Slidev presentation)
**Topic:** 基于实例分割与像素回归的温室生菜鲜重无损估算

---

## 1. Concept & Vision

A static, minimalist component for PPT presentation. No animations, no interactions — purely visual. The aesthetic is "technical minimal" with strong contrast suitable for projection: dark background, bright green blocks, white text. Designed for academic presentation context where clarity and readability at a distance matter most.

---

## 2. Design Language

### Aesthetic Direction
极简技术风 — 纯色块 + 大字号文字 + 强对比，适合演讲投影

### Color Palette
| Role | Hex | Usage |
|---|---|---|
| Primary | `#2E7D32` | Green blocks, node backgrounds |
| Background | `#1a1a1a` | Component container (dark for projection) |
| Text | `#FFFFFF` | All text on colored backgrounds |
| Muted | `#AAAAAA` | Secondary text, descriptions |
| Border | `rgba(255,255,255,0.1)` | Subtle card borders |

### Typography
- Node labels: `font-bold text-xl`
- Node sublabels: `text-sm opacity-80`
- Card numbers: `text-4xl font-bold opacity-30`
- Card titles: `text-2xl font-semibold`
- Card descriptions: `text-base leading-relaxed`

### Spatial System
- Upper section: 5 nodes evenly distributed horizontally
- Lower section: 3 equal-width cards in a row
- Gap between cards: `2rem`
- Section padding: `2rem`

### Motion Philosophy
**None.** Static display only. No hover states, no transitions, no animations.

---

## 3. Layout & Structure

```
┌─────────────────────────────────────────────────────────────┐
│  [🥬输入]  →  [🔬分割]  →  [🔢统计]  →  [📊回归]  →  [⚖️预测]  │
└─────────────────────────────────────────────────────────────┘

┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  01           │  │  02           │  │  03           │
│  实例分割      │  │  像素特征构建  │  │  鲜重回归建模  │
│                │  │                │  │               │
│  YOLO26m-seg  │  │ lettuce_pixels │  │ R² = 0.9613  │
│                │  │                │  │               │
│  采用 YOLO26   │  │ 多实例掩码逐   │  │ 以像素面积为  │
│  模型对生菜    │  │ 像素取并集，   │  │ 输入，分别构建│
│  进行单类别   │  │ 统计前景像素   │  │ 线性、多项式 │
│  实例分割     │  │ 总数作为面积   │  │ 与随机森林   │
│               │  │ 代理变量       │  │ 回归模型     │
└────────────────┘  └────────────────┘  └────────────────┘
```

**Upper Section — Flow Nodes (25% height):**
- Horizontal flex layout, 5 nodes evenly spaced
- Each node: colored block with icon + label + sublabel
- SVG arrow connectors between nodes

**Lower Section — Detail Cards (75% height):**
- CSS Grid: `grid-template-columns: repeat(3, 1fr)`
- Each card: large number watermark, title, sublabel, description paragraphs
- No hover, no expand, no interaction

---

## 4. Component Inventory

### FlowNode
- Props: `icon` (emoji), `label` (string), `subLabel` (string)
- Appearance: green background block, white text, rounded corners
- States: default only (no hover)

### DetailCard
- Props: `number` (string), `title` (string), `subLabel` (string), `lines` (string[])
- Appearance: semi-transparent dark card, large watermark number, stacked text lines
- States: default only (no interaction)

### ArrowConnector
- SVG line with arrowhead
- Color: primary green with opacity

---

## 5. Technical Approach

- **Framework:** Vue 3 Composition API (Script Setup)
- **Styling:** Tailwind CSS (via Slidev)
- **No animations, no transitions, no JavaScript interactivity**
- **Icons:** Unicode emoji (lightweight)
- **State:** No reactive state needed (purely static)

### Key Implementation Notes
- Use `<script setup>` syntax
- No `ref` or reactive state needed (pure display)
- All text hardcoded as component data
- No user interaction handlers
- No CSS transitions or animations
