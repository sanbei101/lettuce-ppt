---
theme: seriph
background: https://cover.sli.dev
title: 基于YOLO 26的生菜重量预测
drawings:
  enabled: false
transition: slide-left
comark: true
duration: 5min
---
# 基于YOLO 26的生菜重量预测


---

# 研究背景与意义
传统农作物重量测量面临耗时费力等痛点。探索**无损、高效、低成本**的生物量估算方法是智慧农业的重要研究方向。
* **研究目标**：利用计算机视觉与机器学习技术，通过顶视图像实现温室生菜重量的无损预测。
* **核心假设**：在固定拍摄高度下，生菜的**二维投影像素面积**与其**实际重量** 存在高度的物理相关性

**技术路线**：
  1. 硬件部署与数据采集 (固定高度摄像头)
  2. 图像实例分割 (YOLO 26)
  3. 机器学习回归拟合 (Random Forest)

---

# 数据采集与预处理
* 在栽培面上方部署**固定高度**的摄像头。
* 采用**正上方俯视拍摄**,最大限度消除因相机倾斜带来的透视畸变。
* **统一预处理**:所有原始图像规范化为 $640 \times 640$ 分辨率输入。

<div class="grid grid-cols-3 gap-3 mt-2 *:w-30">
  <img src="./images/sample5.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample2.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample3.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample1.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample6.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample4.jpg" class="rounded shadow border border-gray-200" />
</div>
<div class="text-center text-sm text-gray-500 mt-2">固定高度俯视拍摄的生菜图像样本</div>



---

# 图像分割数据集构建:
* **数据标注**: 对生菜图像进行实例分割标注，生成对应的掩码

![数据标注示例](./images/image.png)


---

# 图像分割模型训练

本研究引入 YOLO 26 实例分割模型进行核心特征提取

* **实例分割推理**：
  * 将采集到的生菜图像输入 YOLO 模型。
  * 提取高精度的对象掩码 (Mask),过滤低置信度噪点 (conf=0.5)。
* **面积特征计算**：
  * 通过最近邻插值 (`INTER_NEAREST`) 将内部降采样后的 Mask 严格还原至原图分辨率。
  * 统计二值化掩码中响应值 $> 0.5$ 的像素点总数，作为描述生菜形态的唯一输入特征 (Area)。

```python
# 核心特征提取代码演示
results = yolo_model.predict(source=img_path, conf=0.5)[0]

if results.masks is not None:
    # 提取分割掩码并还原至 640x640 原始物理基准分辨率
    mask = results.masks.data.cpu().numpy()[0]
    mask_resized = cv2.resize(mask, (orig_w, orig_h), interpolation=cv2.INTER_NEAREST)
    
    # 计算目标二维投影的像素面积
    area_pixel = np.sum(mask_resized > 0.5)
```


---
layout: two-cols
---

模型核心性能指标：
- 平均精度均值 $\text{mAP}_{50} = 99.5\%$
- 精确率 $\text{Precision} = 99.1\%$
- 召回率 $\text{Recall} = 100\%$

---

# 回归模型训练与评估

在获取 YOLO 提取的生菜掩码后，我们需要一个机器学习算法将“二维像素点数量”转化为“实际物理重量”。本研究选用**随机森林回归** 来完成这一特征映射。

* **从像素到重量的转化原理**：
  * **特征输入 ($X$)**：YOLO 掩码经过原图比例还原后，统计出的有效像素总面积。
  * **目标输出 ($Y$)**：生菜真实的物理称重数据。
  * **算法映射**：随机森林通过构建多棵决策树，学习像素面积与重量之间的内在规律。

* **模型评估表现**：
  * **决定系数 ($R^2$)**: 0.9613
  * **均方根误差 (RMSE)**: 8.71 g