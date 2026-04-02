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
