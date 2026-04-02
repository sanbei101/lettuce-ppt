<template>
  <div class="experiment-flow">
    <!-- Flow Diagram Section (40%) -->
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

    <!-- Detail Cards Section (60%) -->
    <div class="cards-section">
      <div class="cards-grid">
        <!-- 3 DetailCard components -->
      </div>
    </div>
  </div>
</template>

<style scoped>
.experiment-flow {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 1rem;
}

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
