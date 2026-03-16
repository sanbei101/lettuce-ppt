---
theme: seriph
background: https://cover.sli.dev
title: 基于实例分割与像素回归的温室生菜鲜重无损估算
info: |
  从 YOLO26 实例分割到像素面积驱动的鲜重拟合
drawings:
  enabled: false
transition: slide-left
comark: true
duration: 8min
layout: cover
---

# 基于实例分割与像素回归的温室生菜鲜重无损估算

## 从 YOLO26 分割到像素面积驱动的鲜重拟合

---
layout: section
---

# 研究问题

---

# 研究背景与科学问题

## 研究动机

- 传统鲜重测量依赖人工采收与称重,具有破坏性、低频、劳动成本高的缺点
- 在固定拍摄高度的温室环境中,生菜冠层的二维投影面积与鲜重具有稳定相关性
- 因此可以将**视觉分割问题**转换为**面积特征提取问题**,进一步求解**鲜重回归问题**

## 科学问题

如何构建一条从顶视 RGB 图像到生菜鲜重预测值的端到端流程,使其同时满足:

1. 分割精度足够高,能够稳定恢复叶片有效投影区域
2. 像素统计过程足够可解释,能够作为回归输入特征
3. 回归模型能够拟合像素面积与鲜重之间的非线性映射

---

# 研究假设与方法框架

## 核心假设

在拍摄高度、视角与分辨率保持一致时,生菜鲜重 $y$ 与分割掩码像素面积 $x$ 满足:

$$
y = f(x) + \epsilon
$$

其中:

- $x$ 为实例分割掩码合并后的前景像素总数
- $f(\cdot)$ 为待学习的质量映射函数
- $\epsilon$ 为测量误差、遮挡误差与模型误差

## 技术路线

```mermaid
flowchart LR
  A[温室顶视图像] --> B[YOLO26m-seg 实例分割]
  B --> C[掩码融合与像素统计]
  C --> D[构建 lettuce_pixels 特征]
  D --> E[线性回归 / 二次多项式 / 随机森林]
  E --> F[LFW 鲜重预测]
```

---
layout: section
---

# 数据与分割实验

---

# 数据采集与样本构成

## 成像条件

- 拍摄方式:固定高度、正上方俯视
- 图像输入尺寸:统一为 `640 × 640`
- 标注任务:单类别实例分割,仅分割生菜目标区域


<div class="grid grid-cols-3 gap-3 mt-4 *:w-30">
  <img src="./images/sample5.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample2.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample3.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample1.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample6.jpg" class="rounded shadow border border-gray-200" />
  <img src="./images/sample4.jpg" class="rounded shadow border border-gray-200" />
</div>

---

# 分割模型与训练设置

## 模型选型

本研究采用 `YOLO26m-seg` 作为实例分割主干网络

## 训练代码片段

```python
from ultralytics import YOLO

model = YOLO('yolo26m-seg.pt')

results = model.train(
    data='/kaggle/working/lettuce_model/data.yaml',
    epochs=20,
    imgsz=640,
    batch=16,
    workers=4,
    device='0',
    exist_ok=True
)
```

---

# 分割性能评估


```python
metrics = model.val()
print(f"验证集分割 mAP50: {metrics.seg.map50:.4f}")
print(f"验证集分割 mAP50-95: {metrics.seg.map:.4f}")
```

| 指标 | 数值 |
|---|---:|
| mAP@50 | 0.9950 |
| mAP@50:95 | 0.9297 |
![预测结果](./images/image.png)
---
layout: section
---

# 像素特征构建

---
layout: two-cols
layoutClass: gap-x-4
---

# 掩码提取像素面积特征

**核心思路**:

分割模型输出的实例掩码,最终需转化为**单值像素面积特征**输入回归器:

$$
\text{lettuce\_pixels} = \sum_{u,v}\mathbf{1}(M(u,v)=1)
$$

- $M(u,v)$:融合后掩码在像素$(u,v)$处的取值(1=生菜区域,0=背景)
- $\sum$:统计所有生菜区域的像素总数

**核心实现细节**:
- `retina_masks=True`:生成更精细的高分辨率掩码
  
::right::
- `max(dim=0)`:对多个实例掩码逐像素取并集
- `bool().sum()`:统计前景像素总数,得到面积特征

**核心实现代码**

```python
# 步骤1:提取YOLO预测结果的首个输出
first_result = results[0]
# 步骤2:获取实例掩码对象(无掩码时为None)
masks = first_result.masks

# 步骤3:初始化像素总数(兜底无掩码场景)
total_pixels = 0

# 步骤4:掩码存在时计算总像素数
if masks is not None:
    # 多实例掩码融合(逐像素取最大值)
    combined_mask = masks.data.max(dim=0)[0]
    # 统计掩码中生菜区域的像素总数
    total_pixels = combined_mask.bool().sum().item()
```

---
layout: section
---

# 鲜重回归建模

---

## 线性回归

```python
model = LinearRegression()
model.fit(X_train, y_train)
print(f"R²: {model.score(X_test, y_test):.4f}")
print(f"公式: LFW = {model.coef_[0]:.6f} * pixels + {model.intercept_:.4f}")
```

## 二次多项式回归

```python
degree = 2
model = make_pipeline(PolynomialFeatures(degree), LinearRegression())
model.fit(X_train, y_train)
print(f"R² (多项式 degree={degree}): {model.score(X_test, y_test):.4f}")
```

## 随机森林回归

```python
model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
print(f"R² (随机森林): {model.score(X_test, y_test):.4f}")
```

---

<div class="w-full h-full grid grid-cols-2 grid-rows-2 gap-4 items-center justify-items-center p-4">
  <img src="./images/image-2.png" alt="回归结果1" class="max-w-full max-h-[40vh] object-contain" />
  <img src="./images/image-1.png" alt="回归结果2" class="max-w-full max-h-[40vh] object-contain" />
  <img src="./images/image-3.png" alt="回归结果3" class="max-w-full max-h-[40vh] object-contain col-span-2" />
</div>
---

# 回归结果与统计解释

| 模型 | 输入特征 | 主要超参数 | 测试集 $R^2$ |
|---|---|---|---:|
| 线性回归 | `lettuce_pixels` | - | ~0.90 |
| 二次多项式 | `lettuce_pixels` | `degree=2` | ~0.92 |
| 随机森林 | `lettuce_pixels` | `n_estimators=100` | **0.9613** |

## 结果解读

- 单一像素面积特征已经能够解释大部分鲜重变异
- 二次多项式优于线性模型,说明像素面积与鲜重并非严格线性关系
- 随机森林进一步提升到 `R² = 0.9613`,说明局部非线性拟合更适合本任务

**在固定成像条件下,仅利用分割像素面积即可获得高精度鲜重估计,证明二维投影面积是有效的表型代理变量**

---
layout: section
---

# 结论与讨论

---

# 研究结论

## 主要结论

1. `YOLO26m-seg` 在本数据集上取得了 `mAP@50 = 0.9950`、`mAP@50:95 = 0.9297`
2. 基于分割掩码统计得到的 `lettuce_pixels` 可作为稳定且可解释的面积特征
3. 基于该单变量特征的随机森林回归实现了 `R² = 0.9613`

## 方法贡献

- 构建了"实例分割 -> 面积统计 -> 鲜重回归"的两阶段框架
- 兼顾了工程可实现性与学术可解释性
- 为温室场景下的无损表型监测提供了低成本方案

---
layout: center
---

# 谢谢
