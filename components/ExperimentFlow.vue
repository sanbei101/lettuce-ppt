<template>
  <div class="experiment-flow">
    <!-- Flow Diagram Section -->
    <div class="flow-section">
      <div class="flow-nodes">
        <div class="flow-node">
          <span class="node-icon">🥬</span>
          <span class="node-label">输入</span>
          <span class="node-sublabel">温室顶视图像</span>
        </div>

        <svg class="flow-arrow" viewBox="0 0 50 20">
          <path d="M0 10 L40 10 L35 5 M40 10 L35 15" stroke="#4CAF50" stroke-width="2" fill="none" />
        </svg>

        <div class="flow-node">
          <span class="node-icon">🔬</span>
          <span class="node-label">分割</span>
          <span class="node-sublabel">YOLO26m-seg</span>
        </div>

        <svg class="flow-arrow" viewBox="0 0 50 20">
          <path d="M0 10 L40 10 L35 5 M40 10 L35 15" stroke="#4CAF50" stroke-width="2" fill="none" />
        </svg>

        <div class="flow-node">
          <span class="node-icon">🔢</span>
          <span class="node-label">统计</span>
          <span class="node-sublabel">掩码融合</span>
        </div>

        <svg class="flow-arrow" viewBox="0 0 50 20">
          <path d="M0 10 L40 10 L35 5 M40 10 L35 15" stroke="#4CAF50" stroke-width="2" fill="none" />
        </svg>

        <div class="flow-node">
          <span class="node-icon">📊</span>
          <span class="node-label">回归</span>
          <span class="node-sublabel">RF/Poly/Linear</span>
        </div>

        <svg class="flow-arrow" viewBox="0 0 50 20">
          <path d="M0 10 L40 10 L35 5 M40 10 L35 15" stroke="#4CAF50" stroke-width="2" fill="none" />
        </svg>

        <div class="flow-node">
          <span class="node-icon">⚖️</span>
          <span class="node-label">预测</span>
          <span class="node-sublabel">LFW 鲜重</span>
        </div>
      </div>
    </div>

    <!-- Detail Cards Section -->
    <div class="cards-section">
      <div class="cards-grid">
        <!-- Card 1: 实例分割 -->
        <div class="detail-card">
          <div class="card-number">01</div>
          <div class="card-title">实例分割</div>
          <div class="card-sublable">YOLO26m-seg</div>
          <div class="card-lines">
            <p>采用 YOLO26 模型对生菜进行单类别实例分割</p>
            <p>输出生菜叶片的边界掩码</p>
          </div>
        </div>

        <!-- Card 2: 像素特征构建 -->
        <div class="detail-card">
          <div class="card-number">02</div>
          <div class="card-title">像素特征构建</div>
          <div class="card-sublable">lettuce_pixels</div>
          <div class="card-lines">
            <p>多实例掩码逐像素取并集</p>
            <p>统计前景像素总数作为面积代理变量</p>
          </div>
        </div>

        <!-- Card 3: 鲜重回归建模 -->
        <div class="detail-card">
          <div class="card-number">03</div>
          <div class="card-title">鲜重回归建模</div>
          <div class="card-lines">
            <p>以像素面积为输入，分别构建线性、多项式与随机森林回归模型</p>
            <p>分别评估各模型性能</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.experiment-flow {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 1.5rem;
  background: #f0f9f0;
}

/* Flow Section */
.flow-section {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1.5rem 0;
  margin-bottom: 2rem;
}

.flow-nodes {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
}

.flow-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  padding: 1rem 1.5rem;
  background: #2E7D32;
  color: white;
}

.node-icon {
  font-size: 1.75rem;
}

.node-label {
  font-size: 1rem;
  font-weight: 600;
}

.node-sublabel {
  font-size: 0.75rem;
  opacity: 0.8;
}

.flow-arrow {
  width: 3rem;
  height: 1.25rem;
  flex-shrink: 0;
}

/* Cards Section */
.cards-section {
  margin-top: -30px;
  flex: 1;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
  height: 80%;
}

.detail-card {
  position: relative;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(76, 175, 80, 0.3);
  color: #1a1a1a;
  display: flex;
  flex-direction: column;
}

.card-number {
  position: absolute;
  top: 0.5rem;
  right: 1rem;
  font-size: 3rem;
  font-weight: 700;
  color: rgba(46, 125, 50, 0.15);
  line-height: 1;
}

.card-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.card-sublable {
  font-size: 0.875rem;
  color: #4CAF50;
  font-family: monospace;
  margin-bottom: 1rem;
}

.card-lines p {
  font-size: 0.875rem;
  line-height: 1.6;
  color: #333;
  margin-bottom: 0.5rem;
}

.card-lines p:last-child {
  margin-bottom: 0;
}
</style>
