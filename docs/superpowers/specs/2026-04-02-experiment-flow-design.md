# ExperimentFlow Component Design Spec

**Date:** 2026-04-02
**Component:** `components/ExperimentFlow.vue`
**Project:** lettuce-ppt (Slidev presentation)
**Topic:** 基于实例分割与像素回归的温室生菜鲜重无损估算

---

## 1. Concept & Vision

A visually engaging two-part component that presents the research experiment pipeline as a living, interactive diagram. The upper section shows the high-level flow as a connected node graph; the lower section reveals depth through expandable detail cards. The aesthetic should feel scientific yet approachable — clean lines, subtle animations, and a green palette that evokes the lettuce subject matter.

---

## 2. Design Language

### Aesthetic Direction
Scientific infographic meets modern dashboard — structured, data-forward, with enough visual warmth to keep it engaging across a presentation slide.

### Color Palette
| Role | Hex | Usage |
|---|---|---|
| Primary | `#4CAF50` | Active nodes, buttons, accents |
| Secondary | `#81C784` | Hover states, borders |
| Dark | `#2E7D32` | Headers, emphasis text |
| Background | transparent | Component container (Slidev provides page bg) |
| Surface | `rgba(255,255,255,0.9)` | Cards with glass-morphism |
| Text | `#1a1a1a` | Primary text |
| Muted | `#666666` | Secondary/descriptive text |

### Typography
- Node labels: `font-bold text-sm`
- Card headings: `font-semibold text-base`
- Card body: `text-sm`
- Code snippets: `font-mono text-xs`

### Spatial System
- Flow diagram height: 40% of component
- Cards section height: 60% of component
- Card gap: `1rem`
- Node spacing: `flex-1` distribute evenly
- Section padding: `p-6`

### Motion Philosophy
- Node hover: `scale(1.05)` + `box-shadow` glow, `200ms ease`
- Card expand/collapse: `max-height` transition, `300ms ease-in-out`
- Active node pulse: subtle `opacity` animation loop

---

## 3. Layout & Structure

```
┌──────────────────────────────────────────────────────┐
│              Flow Diagram Section (40%)               │
│                                                      │
│  [🥬输入] ────→ [🔬分割] ────→ [🔢统计] ────→ [📊回归] ────→ [⚖️预测]  │
│                                                      │
├──────────────────────────────────────────────────────┤
│              Detail Cards Section (60%)              │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐│
│  │ 实例分割      │  │ 像素特征构建   │  │ 鲜重回归建模  ││
│  │              │  │              │  │              ││
│  │ [代码片段]    │  │ [代码片段]    │  │ [代码片段]   ││
│  │              │  │              │  │              ││
│  └──────────────┘  └──────────────┘  └──────────────┘│
└──────────────────────────────────────────────────────┘
```

**Upper Section — Flow Diagram:**
- Horizontal flex layout with 5 nodes
- Nodes connected by SVG/CSS arrow lines
- Each node: icon + Chinese label + brief English sub-label
- Active/highlighted node has primary color highlight

**Lower Section — Detail Cards:**
- CSS Grid: `grid-cols-3`, responsive gap
- Each card: glass-morphism surface, rounded corners, shadow
- Card contents:
  - Header: stage number badge + Chinese title
  - Body: one-line brief description + expanded technical details
  - Expandable: click header to toggle technical description

**卡片内容（三阶段）：**
1. **实例分割阶段** — YOLO26m-seg 模型对顶视图像进行单类别实例分割，输出生菜叶片的边界掩码。关键在于 retina_masks=True 生成高分辨率掩码，保证面积统计精度。
2. **像素特征构建** — 将多实例掩码逐像素取并集融合，统计前景像素总数作为面积代理变量。该特征消除了实例数量的差异，直接反映叶片投影面积。
3. **鲜重回归建模** — 以像素面积为输入，分别尝试线性、二次多项式与随机森林回归。通过测试集 R² 评估各模型拟合能力，选出最优方案。

---

## 4. Features & Interactions

### Flow Diagram
- **Default state:** All nodes visible, first node (输入) highlighted as "current"
- **Hover:** Node scales up slightly with glow shadow
- **Active indicator:** Primary color border + subtle pulse animation

### Detail Cards
- **Default:** Collapsed, showing only title and one-line description
- **Expanded:** Reveals code snippet and key parameters
- **Toggle:** Click card header to expand/collapse
- **Only one expanded at a time** (accordion behavior)

### Responsive
- Cards stack to `grid-cols-1` on narrow viewports
- Flow diagram nodes wrap or scroll horizontally if needed

---

## 5. Component Inventory

### FlowNode
- **Props:** `label` (string), `subLabel` (string), `icon` (emoji), `active` (boolean)
- **States:** default (muted), hover (scaled + glow), active (green highlight + pulse)

### DetailCard
- **Props:** `title` (string), `description` (string), `details` (string, technical natural language)
- **States:** collapsed (default), expanded, hover (lift shadow)

### FlowConnector (arrows between nodes)
- SVG line or CSS pseudo-element with arrowhead
- Color: secondary green, fades on inactive

---

## 6. Technical Approach

- **Framework:** Vue 3 Composition API (Script Setup)
- **Styling:** Tailwind CSS (via Slidev)
- **Icons:** Unicode emoji (lightweight, no external deps)
- **State:** `ref()` for expanded card index, `computed()` for active node
- **No external component libraries** — custom-built for minimal bundle size

### Data Flow
```
slides.md content → Hardcoded stage data in component
                  ↓
            FlowNode[] + DetailCard[]
                  ↓
            Vue reactivity handles expand/collapse
```

### Key Implementation Notes
- Use `<script setup>` syntax
- Cards use `v-for` with index tracking for accordion
- Stage details stored as template literals containing natural language descriptions
- No build-time dependencies beyond what's in package.json
