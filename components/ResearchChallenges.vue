<template>
  <div class="research-challenges">
    <!-- Title -->
    <div class="section-title">实验难点</div>

    <!-- Challenges Grid -->
    <div class="challenges-grid">
      <div class="challenge-item">
        <div class="challenge-number">01</div>
        <div class="challenge-title">固定视角依赖</div>
        <div class="challenge-desc">跨视角条件下面积-质量关系可能失稳</div>
      </div>

      <div class="challenge-item">
        <div class="challenge-number">02</div>
        <div class="challenge-title">YOLO任务边界</div>
        <div class="challenge-desc">常规YOLO仅输出检测/分割结果，无法直接给出鲜重</div>
      </div>

      <div class="challenge-item">
        <div class="challenge-number">03</div>
        <div class="challenge-title">叶片遮挡与边界粘连</div>
        <div class="challenge-desc">多叶片重叠导致实例边界模糊，影响掩码完整性</div>
      </div>

      <div class="challenge-item">
        <div class="challenge-number">04</div>
        <div class="challenge-title">光照与反光扰动</div>
        <div class="challenge-desc">温室环境中的亮斑与阴影干扰前景-背景区分</div>
      </div>

      <div class="challenge-item">
        <div class="challenge-number">05</div>
        <div class="challenge-title">面积到质量的非线性映射</div>
        <div class="challenge-desc">相同像素面积可能对应不同鲜重，需更强回归器建模</div>
      </div>

      <div class="challenge-item">
        <div class="challenge-number">06</div>
        <div class="challenge-title">流程误差传递</div>
        <div class="challenge-desc">分割阶段的小误差在后续阶段被放大</div>
      </div>
    </div>

    <!-- Strategies Section -->
    <div class="strategies-section">
      <div class="strategies-title">应对策略</div>
      <div class="strategies-grid">
        <div class="strategy-item">
          <div class="strategy-label">分割侧</div>
          <div class="strategy-desc">
            采用 YOLO26m-seg 模型进行20 epochs训练<br>
            设置 retina_masks=True 生成高分辨率掩码<br>
            使用低置信度阈值(conf=0.01)保留目标区域
          </div>
        </div>
        <div class="strategy-item">
          <div class="strategy-label">特征侧</div>
          <div class="strategy-desc">
            对多实例掩码取并集融合<br>
            masks.max(dim=0) 逐像素合并<br>
            bool().sum() 统计前景像素总数
          </div>
        </div>
        <div class="strategy-item">
          <div class="strategy-label">回归侧</div>
          <div class="strategy-desc">
            对比线性、多项式与随机森林回归模型<br>
            以像素面积为输入进行鲜重预测<br>
            随机森林效果最优(R²=0.9613)
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.research-challenges {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 1rem;
  background: #f0f9f0;
}

.section-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: #2E7D32;
  text-align: center;
  margin-bottom: 0.75rem;
}

.challenges-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0.75rem;
  margin-bottom: 0.75rem;
}

.challenge-item {
  position: relative;
  padding: 0.75rem 1rem;
  background: white;
  border: 1px solid rgba(76, 175, 80, 0.3);
}

.challenge-number {
  position: absolute;
  top: 0.3rem;
  right: 0.75rem;
  font-size: 1.5rem;
  font-weight: 700;
  color: rgba(46, 125, 50, 0.15);
  line-height: 1;
}

.challenge-title {
  font-size: 0.9rem;
  font-weight: 600;
  color: #2E7D32;
  margin-bottom: 0.3rem;
}

.challenge-desc {
  font-size: 0.8rem;
  color: #555;
  line-height: 1.4;
}

.strategies-section {
  background: #2E7D32;
  padding: 1rem;
  border-radius: 6px;
  flex: 1;
}

.strategies-title {
  font-size: 1rem;
  font-weight: 600;
  color: white;
  text-align: center;
  margin-bottom: 0.75rem;
}

.strategies-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
}

.strategy-item {
  text-align: center;
}

.strategy-label {
  font-size: 0.9rem;
  font-weight: 600;
  color: #4CAF50;
  margin-bottom: 0.5rem;
}

.strategy-desc {
  font-size: 0.75rem;
  color: white;
  line-height: 1.6;
}
</style>
