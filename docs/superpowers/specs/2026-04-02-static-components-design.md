# Static PPT Components Design Spec

**Date:** 2026-04-02
**Components:** `components/ResearchChallenges.vue`, `components/MethodFramework.vue`
**Project:** lettuce-ppt (Slidev presentation)
**Topic:** 基于实例分割与像素回归的温室生菜鲜重无损估算

---

## 1. Concept & Vision

Two static, minimalist components for PPT presentation. No animations, no interactions — purely visual. Follow the same style as `ExperimentFlow.vue`:
- Light green background (#f0f9f0)
- Dark green accent blocks (#2E7D32)
- White text on colored backgrounds
- Strong contrast suitable for projection

---

## 2. Design Language

### Color Palette
| Role | Hex | Usage |
|---|---|---|
| Primary | `#2E7D32` | Green blocks, headers |
| Background | `#f0f9f0` | Component container |
| Text Light | `#FFFFFF` | Text on green backgrounds |
| Text Dark | `#333333` | Text on light backgrounds |
| Muted | `#4CAF50` | Sublabels, accents |
| Border | `rgba(76, 175, 80, 0.3)` | Card borders |

### Typography
- Section titles: `text-2xl font-bold`
- Card titles: `text-lg font-semibold`
- Body text: `text-sm leading-relaxed`

### Motion Philosophy
**None.** Static display only.

---

## 3. Component 1: ResearchChallenges.vue

### Layout Structure
```
┌─────────────────────────────────────────────────────────────┐
│                          实验难点                           │
├──────────────────────┬──────────────────────────────────┤
│  01 固定视角依赖     │  02 YOLO任务边界      03 叶片遮挡│
│                      │                                   │
├──────────────────────┴──────────────────────────────────┤
│                        应对策略                           │
│   [分割侧]    [特征侧]    [回归侧]    [流程侧]          │
└─────────────────────────────────────────────────────────────┘
```

### Content

**难点 (6 items in 2-column grid):**
1. 固定视角依赖 — 跨视角条件下面积-质量关系可能失稳
2. YOLO任务边界 — 常规YOLO仅输出检测/分割结果，无法直接给出鲜重
3. 叶片遮挡与边界粘连 — 多叶片重叠导致实例边界模糊
4. 光照与反光扰动 — 温室环境中的亮斑与阴影干扰前景-背景区分
5. 面积到质量的非线性映射 — 相同像素面积可能对应不同鲜重
6. 流程误差传递 — 分割阶段的小误差在后续阶段被放大

**应对策略 (4 items in row):**
- 分割侧: `retina_masks=True` + 低置信度阈值
- 特征侧: 掩码并集 + 空掩码兜底
- 回归侧: 对比线性、多项式与随机森林
- 流程侧: 两阶段方案弥补YOLO能力缺口

---

## 4. Component 2: MethodFramework.vue

### Layout Structure
```
┌─────────────────────────────────────────────────────────────┐
│                    研究假设与方法框架                        │
├─────────────────────────────────────────────────────────────┤
│                      y = f(x) + ε                          │
│        在拍摄高度、视角与分辨率保持一致时                    │
│        生菜鲜重与分割掩码像素面积满足...                    │
├─────────────────────────────────────────────────────────────┤
│  [🥬输入] → [🔬分割] → [🔢统计] → [📊回归] → [⚖️预测]  │
└─────────────────────────────────────────────────────────────┘
```

### Content

**核心公式:**
y = f(x) + ε

- x: 实例分割掩码合并后的前景像素总数
- f(·): 待学习的质量映射函数
- ε: 测量误差、遮挡误差与模型误差

**技术路线 (5 nodes):**
1. 🥬 输入 — 温室顶视图像
2. 🔬 分割 — YOLO26m-seg 实例分割
3. 🔢 统计 — 掩码融合与像素统计
4. 📊 回归 — 线性/多项式/随机森林
5. ⚖️ 预测 — LFW 鲜重

---

## 5. Technical Approach

- **Framework:** Vue 3 with `<script setup>`
- **Styling:** Tailwind CSS (via Slidev)
- **No animations, no transitions, no JavaScript interactivity**
- **Icons:** Unicode emoji
- **No reactive state** (purely static display)

---

## 6. File Structure

- Create: `components/ResearchChallenges.vue`
- Create: `components/MethodFramework.vue`
