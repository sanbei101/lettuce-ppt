# ExperimentFlow Component Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create `ExperimentFlow.vue` component for Slidev presentation showing the lettuce fresh weight estimation pipeline.

**Architecture:** Two-section layout — upper flow diagram (5 horizontal nodes with connectors) and lower accordion cards (3 stages with technical descriptions). Vue 3 Composition API with Tailwind CSS.

**Tech Stack:** Vue 3, Tailwind CSS (via Slidev), pure CSS animations

---

## File Structure

- Create: `components/ExperimentFlow.vue` — single-file component with all logic and styles

---

## Task 1: Scaffold Component Structure

**Files:**
- Create: `components/ExperimentFlow.vue`

- [ ] **Step 1: Create component file with template structure**

```vue
<template>
  <div class="experiment-flow">
    <!-- Flow Diagram Section (40%) -->
    <div class="flow-section">
      <div class="flow-nodes">
        <!-- 5 FlowNode components -->
      </div>
    </div>

    <!-- Detail Cards Section (60%) -->
    <div class="cards-section">
      <div class="cards-grid">
        <!-- 3 DetailCard components -->
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Styles here */
</style>
```

- [ ] **Step 2: Add script setup with stage data**

```vue
<script setup>
import { ref } from 'vue'

const stages = [
  {
    id: 1,
    title: '实例分割',
    label: 'YOLO26m-seg',
    description: '单类别实例分割',
    details: 'YOLO26m-seg 模型对顶视图像进行单类别实例分割，输出生菜叶片的边界掩码。关键在于 retina_masks=True 生成高分辨率掩码，保证面积统计精度。'
  },
  {
    id: 2,
    title: '像素特征构建',
    label: 'lettuce_pixels',
    description: '掩码融合与统计',
    details: '将多实例掩码逐像素取并集融合，统计前景像素总数作为面积代理变量。该特征消除了实例数量的差异，直接反映叶片投影面积。'
  },
  {
    id: 3,
    title: '鲜重回归建模',
    label: 'LFW Prediction',
    description: 'R² = 0.9613',
    details: '以像素面积为输入，分别尝试线性、二次多项式与随机森林回归。通过测试集 R² 评估各模型拟合能力，选出最优方案。'
  }
]

const expandedIndex = ref(null)

const flowNodes = [
  { icon: '🥬', label: '输入', subLabel: '温室顶视图像' },
  { icon: '🔬', label: '分割', subLabel: 'YOLO26m-seg' },
  { icon: '🔢', label: '统计', subLabel: '掩码融合' },
  { icon: '📊', label: '回归', subLabel: 'RF/Poly/Linear' },
  { icon: '⚖️', label: '预测', subLabel: 'LFW 鲜重' }
]
</script>
```

- [ ] **Step 3: Commit**

```bash
git add components/ExperimentFlow.vue
git commit -m "feat: scaffold ExperimentFlow component structure"
```

---

## Task 2: Implement Flow Diagram Section

**Files:**
- Modify: `components/ExperimentFlow.vue:1-80`

- [ ] **Step 1: Add flow node HTML with connector arrows**

Replace placeholder in template with:

```vue
<div class="flow-section">
  <div class="flow-nodes">
    <div
      v-for="(node, index) in flowNodes"
      :key="index"
      class="flow-node"
      :class="{ 'active': index === 0 }"
    >
      <div class="node-icon">{{ node.icon }}</div>
      <div class="node-label">{{ node.label }}</div>
      <div class="node-sublabel">{{ node.subLabel }}</div>
    </div>

    <!-- Arrow connectors -->
    <div class="flow-arrows">
      <div v-for="i in 4" :key="i" class="arrow">→</div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add flow section styles**

Add to `<style scoped>`:

```css
.flow-section {
  height: 40%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
  background: linear-gradient(135deg, #f0f9f0 0%, #e8f5e8 100%);
  border-radius: 12px;
  margin-bottom: 1.5rem;
}

.flow-nodes {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  max-width: 900px;
  position: relative;
}

.flow-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 1.5rem;
  background: white;
  border-radius: 12px;
  border: 2px solid #e0e0e0;
  transition: all 0.2s ease;
  cursor: pointer;
  z-index: 1;
}

.flow-node:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 20px rgba(76, 175, 80, 0.3);
  border-color: #81C784;
}

.flow-node.active {
  border-color: #4CAF50;
  background: linear-gradient(135deg, #e8f5e8 0%, #c8e6c9 100%);
  box-shadow: 0 0 20px rgba(76, 175, 80, 0.4);
}

.node-icon {
  font-size: 2rem;
}

.node-label {
  font-weight: 600;
  color: #2E7D32;
  font-size: 0.95rem;
}

.node-sublabel {
  font-size: 0.75rem;
  color: #666;
}

.flow-arrows {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  transform: translateY(-50%);
  display: flex;
  justify-content: space-around;
  pointer-events: none;
  z-index: 0;
}

.arrow {
  color: #4CAF50;
  font-size: 1.5rem;
  font-weight: bold;
  opacity: 0.6;
}
```

- [ ] **Step 3: Commit**

```bash
git add components/ExperimentFlow.vue
git commit -m "feat: implement flow diagram section with 5 nodes"
```

---

## Task 3: Implement Detail Cards Section

**Files:**
- Modify: `components/ExperimentFlow.vue:80-150`

- [ ] **Step 1: Add cards section HTML**

Add below flow-section in template:

```vue
<div class="cards-section">
  <div class="cards-grid">
    <div
      v-for="(stage, index) in stages"
      :key="stage.id"
      class="detail-card"
      :class="{ 'expanded': expandedIndex === index }"
    >
      <div class="card-header" @click="expandedIndex = expandedIndex === index ? null : index">
        <div class="card-badge">{{ stage.id }}</div>
        <div class="card-title-group">
          <div class="card-title">{{ stage.title }}</div>
          <div class="card-label">{{ stage.label }}</div>
        </div>
        <div class="card-toggle">{{ expandedIndex === index ? '▲' : '▼' }}</div>
      </div>

      <div class="card-body">
        <p class="card-description">{{ stage.description }}</p>
        <div class="card-details" :class="{ 'visible': expandedIndex === index }">
          <p>{{ stage.details }}</p>
        </div>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Add cards section styles**

Add to `<style scoped>`:

```css
.cards-section {
  height: 60%;
  padding: 0 0.5rem;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
  height: 100%;
}

.detail-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 16px;
  border: 1px solid rgba(76, 175, 80, 0.2);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
  transition: all 0.3s ease;
  overflow: hidden;
}

.detail-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(76, 175, 80, 0.2);
}

.detail-card.expanded {
  border-color: #4CAF50;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.25rem;
  cursor: pointer;
  background: linear-gradient(135deg, #f8fff8 0%, #f0f7f0 100%);
  border-bottom: 1px solid #e8e8e8;
}

.card-badge {
  width: 2rem;
  height: 2rem;
  background: #4CAF50;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.875rem;
  flex-shrink: 0;
}

.card-title-group {
  flex: 1;
}

.card-title {
  font-weight: 600;
  color: #2E7D32;
  font-size: 1rem;
}

.card-label {
  font-size: 0.75rem;
  color: #81C784;
  font-family: monospace;
}

.card-toggle {
  color: #4CAF50;
  font-size: 0.75rem;
  transition: transform 0.3s ease;
}

.card-body {
  padding: 1.25rem;
  flex: 1;
}

.card-description {
  color: #666;
  font-size: 0.875rem;
  margin-bottom: 0.5rem;
}

.card-details {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease-in-out;
  color: #333;
  font-size: 0.85rem;
  line-height: 1.6;
}

.card-details.visible {
  max-height: 200px;
}
```

- [ ] **Step 3: Commit**

```bash
git add components/ExperimentFlow.vue
git commit -m "feat: implement detail cards section with accordion"
```

---

## Task 4: Add Animations and Polish

**Files:**
- Modify: `components/ExperimentFlow.vue:150-200`

- [ ] **Step 1: Add pulse animation for active node**

Add to `<style scoped>`:

```css
@keyframes pulse {
  0%, 100% { box-shadow: 0 0 20px rgba(76, 175, 80, 0.4); }
  50% { box-shadow: 0 0 30px rgba(76, 175, 80, 0.6); }
}

.flow-node.active {
  animation: pulse 2s ease-in-out infinite;
}
```

- [ ] **Step 2: Adjust responsive layout for narrow viewports**

Update `.cards-grid` style:

```css
@media (max-width: 768px) {
  .cards-grid {
    grid-template-columns: 1fr;
    overflow-y: auto;
  }

  .flow-nodes {
    flex-wrap: wrap;
    gap: 1rem;
    justify-content: center;
  }

  .flow-arrows {
    display: none;
  }
}
```

- [ ] **Step 3: Commit**

```bash
git add components/ExperimentFlow.vue
git commit -m "feat: add animations and responsive styles"
```

---

## Task 5: Final Integration

- [ ] **Step 1: Verify component file is complete and syntactically correct**

Check that:
- All tags are properly closed
- Vue directives (`v-for`, `v-if`, `:class`) are correct
- No syntax errors in styles

- [ ] **Step 2: Verify imports in slides.md work**

Add to slides.md where needed:
```
<!-- slides.md -->
<!-- To use the component: -->
<ExperimentFlow />
```

- [ ] **Step 3: Final commit**

```bash
git add -A
git commit -m "feat: complete ExperimentFlow component

- Flow diagram with 5 stages (input → segmentation → pixel stats → regression → prediction)
- 3 detail cards with technical descriptions
- Accordion behavior for card expansion
- Responsive design
- Animations and hover effects"
```

---

## Spec Coverage Check

| Spec Section | Task |
|---|---|
| Flow diagram (5 nodes) | Task 2 |
| Detail cards (3 stages) | Task 3 |
| Accordion behavior | Task 3 |
| Technical descriptions (no code) | Task 1 |
| Color palette (#4CAF50 etc.) | Tasks 2, 3 |
| Hover/active states | Tasks 2, 3, 4 |
| Animations | Task 4 |
| Responsive design | Task 4 |

**All spec items covered.**
