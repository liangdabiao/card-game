---
layout: post
title: "You Only Look Once：八年食物检测演进"
date: 2026-01-11 12:00:00.000000000 -05:00
permalink: from-yolo-v2-to-yolo11-food-detection-evolution/
author: Benny Cheung
artwork:
  author: Gemini 3 Pro (Nano Banana Pro)
tags:
  - Deep Neural Network
  - Object Detection
  - YOLO
  - Food Detection
  - Machine Learning
category: post
comments: true
image: images/from-yolo-v2-to-yolo11-food-detection-evolution/cover.jpg
images:
  cover: images/from-yolo-v2-to-yolo11-food-detection-evolution/cover2x.jpg
  header: images/from-yolo-v2-to-yolo11-food-detection-evolution/header.jpg
  animated-cover: images/from-yolo-v2-to-yolo11-food-detection-evolution/animated-cover.webp
published: true
series: "FoodInsight"
series_order: 1
---

<!--excerpt.start-->
八年前，我们训练了一个 YOLO v2 模型来实时检测 100 种日本食物。它需要编译 C 代码、编写定制 Python 脚本、小心地调配置文件。今天，同样的任务只需三条命令和一个下午。这就是食物检测——以及更广义的深度学习工具链——从 2018 到 2026 的演进故事。
<!--excerpt.end-->

2018 年，我们写过 [YOLO for Real-Time Food Detection]({{ site.baseurl }}/yolo-for-real-time-food-detection) [6]，记录了我们用 Joseph Redmon 的 Darknet 框架 [1] 和 UEC FOOD 100 数据集 [2] 构建"Food Watcher"的历程。就当时而言，结果令人印象深刻：GTX TitanX 上 70 fps 的食物检测，从寿司到汉堡无所不识。

> "对识别零食与食物的痴迷一直是试验最新机器学习技术的有趣主题。"

那份痴迷从未消退。但整个图景已天翻地覆。曾经需要专用 NVIDIA GPU、手工编译 C、数小时配置的工作，如今在一台 Apple Silicon 的 MacBook 上运行，pip 一装即用，睡觉时就能训练。

本文以 YOLO11 [3] 和 FoodInsight 项目 [4] 为镜重访食物检测：什么变了，什么没变，以及为什么 2026 年是训练你自己的目标检测模型的最佳时机。

![YOLO 食物检测流水线]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/YOLO_Food_Detection_Hero.png)

_图：YOLO11 食物检测流水线：来自 UEC FOOD 100 数据集 [2] 的输入图像流经卷积层和检测头，为多样的日式菜肴输出带类别标签和置信度分数的边界框。_

---

## YOLO 的演进

在深入技术细节之前，值得先体味 YOLO 走过了多远。2015 年的原始论文 "You Only Look Once" [5] 引入了一个激进的想法：不再把图像扫描数千遍来寻找物体，而是让整幅图像在单次神经网络前向中处理完毕。

![YOLO 的演进]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/YOLO_Evolution_Timeline.png)

_图：YOLO 从 2015 到 2024 的演进：从 Joseph Redmon 革命性的单次前向检测概念，经 Ultralytics 对目标检测的民主化，到今天以效率为重的 YOLO11。橙色高亮标记关键突破（v1、v5、v11），青色标注重大架构创新。_

这个故事的发展弧线引人注目。Joseph Redmon 的原始 Darknet 实现用纯 C 编写，快，但需要仔细编译和平台特定的调整 [1]。2020 年 Redmon 退出计算机视觉研究后，社区分叉并演进着这套代码。Ultralytics 成为了事实标准，把一切封装进一个干净、"开箱即用"的 Python API [3]。

从研究代码到生产就绪库。从 C 编译到 `pip install`。从 NVIDIA 专属到 Apple Silicon、Intel 和边缘设备。

---

## 昔与今：环境搭建对比

2018 与 2026 之间最 striking 的差别不是模型架构，而是开发者体验。

### 2018：Darknet 之路

2018 年训练 YOLO v2 需要：

```
1. Download and compile Darknet (C code, Makefile tweaks for GPU)
2. Download pre-trained weights manually (darknet19_448.conv.23)
3. Write custom Python scripts for bounding box conversion
4. Create food100.names file (class names, one per line)
5. Create food100.data file (paths to train/test lists)
6. Create yolov2-food100.cfg (50+ lines, manual filter calculation)
7. Run training with precise command syntax
8. Edit config again for testing (batch=1, subdivisions=1)
```

光配置文件就要求手工计算 filter 数量：

```
# number of filters calculated by (#-of-classes + 5)*5
# e.g. (100 + 5)*5 = 525
filters=525
classes=100
```

算错了呢？静默失败，或输出垃圾预测。

### 2026：Ultralytics 之路

现代等价流程：

```bash
# Install (once)
pip install ultralytics

# Convert dataset (once)
python convert_uec_to_yolo.py datasets/UECFOOD100 --output datasets/UECFOOD100_yolo

# Train
yolo detect train data=data.yaml model=yolo11s.pt epochs=150 device=mps
```

就这些。转换脚本自动生成 `data.yaml` 文件：

```yaml
path: /path/to/UECFOOD100_yolo
train: images/train
val: images/val
test: images/test
nc: 100
names: ['rice', 'eels on rice', 'pilaf', ...]
```

不用手工计算 filter。不用分开的训练/测试配置。不用编译 C。框架搞定一切。

### 被消除的痛点

| 2018 痛点 | 2026 解法 |
|-----------------|---------------|
| 跨平台 C 编译 | 纯 Python，pip install |
| 手工计算 filter | 自动架构 |
| 分开的训练/测试配置 | 单一模型对象 |
| 无内置可视化 | 集成图表、指标 |
| 必须 NVIDIA GPU | Apple Silicon、CPU、Intel |
| 手工下载权重 | 从 hub 自动下载 |

---

## 在 Apple Silicon 上训练

也许最令人惊讶的变化：你不再需要 NVIDIA GPU。

2018 年，CUDA 是获得合理训练速度的唯一途径。一块 GTX TitanX 是 1200 美元的投资，还得配一台主机。今天，Apple 的 Metal Performance Shaders（MPS）后端让任何 M 系列的 Mac 都能进行 GPU 加速训练。

```bash
python datasets/train.py \
    --data /path/to/UECFOOD100_yolo/data.yaml \
    --model yolo11s.pt \
    --epochs 150 \
    --batch 32 \
    --device mps \
    --name food100_best
```

只需改动 `--device mps` 标志。其余完全一样。

### 硬件对比

| 指标 | 2018 (GTX TitanX) | 2026 (M4 Pro) |
|--------|-------------------|---------------|
| 内存 | 12GB VRAM（独占） | 48GB 统一内存 |
| 功耗 | 250W | 40W |
| 搭建 | CUDA、cuDNN、驱动地狱 | 开箱即用 |
| 成本 | 单 GPU 约 $1200 | 集成 |
| 噪音 | 喷气发动机 | 静音 |
| 形态 | 台式塔式机 | 笔记本 |

在 M4 Pro 上，在 UEC FOOD 100 [2] 上训练 YOLO11s 每个 epoch 约需 15-18 分钟。我们 80 epoch 的训练约 22 小时完成，模型收敛后可提前停止。

---

## 数据集：UEC FOOD 100（依旧是标杆）

有些东西不变。东京电气通信大学的 UEC FOOD 100 数据集依然是食物检测研究的黄金标准 [2]。

| 属性 | 值 |
|----------|-------|
| 图像 | 14,361 |
| 类别 | 100 个日本食物类别 |
| 标注 | 每个食物条目的边界框 |
| 许可 | 非商业研究 |

这个数据集的长处在于它的真实世界属性：照片摄于真实的用餐场景，不是棚拍。单帧中常出现多种食物。光照多变。摆盘各异。

现在还有 UEC FOOD 256 [2]，含 31,395 张图像、256 个类别，加入了国际食物。不过，原始的 100 类版本训练更快，且单类别准确率更高，所以它仍是最佳起点。

### 数据集转换

原始 UEC 格式把边界框存在每个类别目录的 `bb_info.txt` 文件中：

```
img x1 y1 x2 y2
1 0 143 370 486
2 20 208 582 559
```

YOLO 期望每张图像的标签文件中是归一化的中心坐标：

```
class_id x_center y_center width height
0 0.231250 0.524167 0.462500 0.571667
```

2018 年，我们写了三个独立的 Python 脚本来处理这个转换。今天，一个统一脚本搞定一切，包括自动划分 train/val/test：

```bash
python convert_uec_to_yolo.py datasets/UECFOOD100 \
    --output datasets/UECFOOD100_yolo \
    --train-ratio 0.8 \
    --val-ratio 0.1 \
    --test-ratio 0.1
```

---

## 看懂训练输出

2026 大幅超越 2018 的一个方面：指标与可观测性。

### 2018 年我们看到的

```
Region Avg IOU: 0.767684, Class: 0.285230, Obj: 0.376508
9998: 5.413931, 5.206974 avg, 0.000010 rate, 2.222207 seconds
```

指引很简单："平均损失不再下降时停止。"没有 precision/recall 拆解。没有逐类分析。只是盯着一个数字往下走，然后祈祷。

### 2026 年我们看到的

每个 epoch 产出全面的指标：

```csv
epoch,mAP50,mAP50-95,precision,recall,box_loss,cls_loss,dfl_loss
48,0.780,0.596,0.737,0.704,0.833,1.383,1.344
49,0.782,0.594,0.725,0.721,0.833,1.381,1.342
50,0.787,0.598,0.762,0.697,0.820,1.359,1.335
```

让我们拆解这些指标的含义：

![理解检测指标]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/Detection_Metrics_Explained.png)

_图：目标检测指标的可视化解释：IoU 度量边界框重叠，Precision 追踪误报，Recall 追踪漏检，mAP 是跨 IoU 阈值的平均精度（0.50 为宽松，0.50-0.95 为严格）。_

**mAP50-95（主指标）**：跨 0.50 至 0.95 IoU 阈值的平均精度均值。这是最严格的度量，因为高分同时要求分类正确**且**边界框精准。

**mAP50（宽松指标）**：IoU=0.50 时的精度。检测与真值至少 50% 重叠即算正确。总是高于 mAP50-95。

**Precision（查准率）**：所有检测中，正确的比例是多少？高 precision 意味着误报少。

**Recall（查全率）**：图像中所有真实食物中，我们检测到的比例是多少？高 recall 意味着漏检少。

**IoU（交并比）**：度量边界框的准确性：

```
IoU = Area of Overlap / Area of Union
```

IoU 0.50 意味着预测框与真值重叠 50%。IoU 0.95 则要求近乎完美的对齐。

### 当前训练结果

我们的 YOLO11s 模型在 UEC FOOD 100 [2] 上训练 80 epoch 后达到以下指标：

| 指标 | 值 | 解读 |
|--------|-------|----------------|
| mAP50-95 | 0.60 | 整体检测准确率良好 |
| mAP50 | 0.79 | IoU 0.50 下检测强劲 |
| Precision | 0.75 | 误报少 |
| Recall | 0.71 | 能找到图像中的大多数食物 |

对一个包含多种面条和米饭料理等视觉相似类别的 100 类食物检测问题，这些数字代表扎实的性能。模型使用了从 COCO 预训练权重 [9] 出发的迁移学习，这帮助我们在相对较小的 UEC FOOD 100 数据集上也取得了好结果。

---

## 经典测试：重访

在 2018 年的文章中，我们突出过几个令人难忘的测试用例。现在用 YOLO11 重访它们。

### 测试 1：热狗测试

> "自从 HBO 剧集《硅谷》发布了一个识别'热狗与非热狗'的真实 AI 应用以来，识别热狗就成了任何食物检测的黄金测试标准。"

![热狗检测]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/hot_dog_collage.jpg)

_图：YOLO11 检测各种风格的热狗：经典芥末款（0.89）、芝麻面包成对款（0.93）、堆料款（0.83）、淋番茄酱款（0.95）。《硅谷》的"非热狗"测试？漂亮通过。_

### 测试 2：汉堡挑战

从快餐经典到美食创作，汉堡的呈现千变万化。模型能应付巨无霸、自制汉堡以及介于其间的一切吗？

![汉堡检测]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/hamburger_collage.jpg)

_图：YOLO11 检测形形色色的汉堡：经典肉饼（0.84）、巨无霸式堆叠（0.88）、配新鲜番茄的 gourmet 款（0.72），以及麦当劳配薯条套餐（0.56）。注意同帧中多类别检测抓到了炸薯条（0.41）。_

### 测试 3：带汤的食物

汤面带来独特挑战：食物部分浸没、被遮挡。

![拉面检测]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/ramen_noodle_collage.jpg)

_图：YOLO11 检测拉面菜肴，置信度从 0.51 到 0.95，成功处理汤中部分被遮挡的食物。_

### 测试 4：百变鸡蛋

鸡蛋以无数形态出现：煎的、卷的、包的、折进蛋包饭的。模型能区分视觉差异明显的蛋类菜肴吗？

![鸡蛋检测]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/egg_collage.jpg)

_图：YOLO11 检测多样的蛋类料理：蛋包饭式煎蛋卷（0.81）、日式厚蛋烧 tamagoyaki（0.77）、酥脆蛋卷（0.85）、经典单面煎蛋（0.56）。_

### 测试 5：万般披萨

从单片到整张，从外卖盒到餐厅餐盘。模型能识别各种呈现与配料下的披萨吗？

![披萨检测]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/pizza_collage.jpg)

_图：YOLO11 检测多样风格的披萨：纸盘上的一片（0.88）、外卖盒里的玛格丽特（0.87）、蘑菇披萨（0.62）、鲜虾披萨（0.80）。_

---

## 部署：超越桌面

2018 年，部署意味着在一台带 NVIDIA GPU 的机器上运行 Darknet [1]。仅此而已。

```bash
./darknet detector demo data/food100.data cfg/yolov2-food100.cfg weights/final.weights
```

没 GPU？没检测。

### 2026：部署到一切

YOLO11 可以导出到几乎任何运行时 [3]：

| 目标 | 格式 | 命令 |
|--------|--------|---------|
| Raspberry Pi | NCNN [8] | `yolo export model=best.pt format=ncnn` |
| iOS | CoreML | `yolo export model=best.pt format=coreml` |
| Android | TFLite | `yolo export model=best.pt format=tflite` |
| Web | ONNX.js | `yolo export model=best.pt format=onnx` |
| Intel CPUs | OpenVINO | `yolo export model=best.pt format=openvino` |
| 任意平台 | ONNX | `yolo export model=best.pt format=onnx` |

FoodInsight 项目瞄准树莓派部署，让一台 50 美元的电脑在边缘运行食物检测 [4]：

```bash
# Export for Raspberry Pi (NCNN format) [8]
yolo export model=runs/detect/food100_best/weights/best.pt format=ncnn

# Deploy the NCNN model folder to Pi
# Run inference on-device, no cloud required
```

隐私保护、随时可用的食物检测，成本约等于一顿餐厅晚餐。

---

## 性能对比

给这些进步配上数字：

| 指标 | YOLO v2 (2018) [1] | YOLO11 (2026) [3] |
|--------|----------------|---------------|
| 框架 | Darknet (C) | Ultralytics (Python) |
| 基础模型大小 | ~200MB | 22MB (yolo11s) |
| 推理速度 | ~70 fps (TitanX) | 120+ fps (M4 Pro) |
| mAP50 | ~0.65-0.70（估计） | 0.79 |
| 搭建时间 | 数小时 | 数分钟 |
| 配置行数 | 50+ | <10 |
| 部署目标 | 仅 NVIDIA GPU | 一切设备 |

模型更小、更快、更准确，而且好用无穷倍。

---

## 下一步：FoodInsight 愿景

食物检测是达到目的的手段。我们正在构建 **FoodInsight**，一个展示 YOLO11 食物检测两类实际应用的平台。

### 零食监视：FoodInsight

第一个应用是面向办公室茶水间的智能零食库存监测系统。想象一台装有摄像头的树莓派，架在零食架上方：

![FoodInsight 部署场景]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/FoodInsight_Deployment_Scene.png)

_图：FoodInsight 部署构想：一台带摄像头的树莓派监视零食架，实时检测被取走或添加的物品。边界框显示检测到的物品及数量，警告指示器标记低库存。_

**FoodInsight 架构：**

- **边缘 AI 检测**：YOLO11n 以 NCNN 格式 [8] 直接运行在树莓派上，达到 4-6 FPS，零云依赖
- **隐私内建**：只存储库存计数，永不存储图像。ROI 蒙版确保只处理零食架区域
- **实时追踪**：ByteTrack [7] 跨帧维持目标持续存在，检测物品被取走或补货
- **消费端访问**：轻量级 PWA 让任何人用手机查询当前零食可用性
- **零云成本**：SQLite 数据库本地运行，整个系统自给自足

完整方案的硬件成本不到 140 美元（树莓派 5 + Camera Module 3），展示了边缘 AI 如何把简单的目标检测变成实用的职场自动化。曾经需要昂贵云基础设施的工作，如今完全跑在一台信用卡大小的电脑上。

### 营养追踪流水线：FoodInsight

第二个应用把食物检测延伸到自动化营养追踪：给你的餐食拍张照，获取热量与宏量营养估算，记录到你的健康应用。

![FoodInsight 流水线]({{ site.baseurl }}/images/from-yolo-v2-to-yolo11-food-detection-evolution/FoodInsight_Pipeline_Diagram.png)

_图：FoodInsight 营养追踪流水线：从相机拍摄，经 YOLO11 检测、分量估算、营养查询，到健康仪表盘可视化。_

YOLO11 漂亮地完成了第一步。剩余的部分：

1. **分量估算**：通过深度传感、参照物，或学习到的大小估算
2. **营养数据库**：把检测到的类别映射到营养信息
3. **用户界面**：移动应用或挂在厨房的显示屏

硬件已经有了。模型已经能用了。零食监视应用已经完成，营养追踪流水线是下一个前沿。

---

## 结语

八年，对深度学习而言是一段漫长的时间。曾经需要专用 GPU 工作站、C 编译专长和精心手工配置的工作，如今在一台笔记本上三条命令搞定。

**关键收获：**

1. **简化**：Ultralytics 生态让 YOLO 触手可及
2. **民主化**：Apple Silicon 与 CPU 训练移除了 NVIDIA 壁垒
3. **更好的结果**：现代架构以更少的努力获得更高的准确率
4. **新的部署目标**：边缘设备、移动与 Web 现在都是一等公民
5. **同样出色的数据**：UEC FOOD 100 依然是食物检测的黄金标准

2018 年的原文如此作结：

> "AI 确实享受看食物！"

八年后，AI 比以往任何时候都更擅长这件事，而且现在任何人都能在一个下午训练出自己的食物检测器。

痴迷仍在继续。

---

## 附录：快速上手指南

想训练你自己的食物检测模型？这是最短路径：

### 前置要求

- Python 3.10+
- Apple Silicon 的 macOS，或带 NVIDIA GPU 的 Linux

### 步骤

```bash
# 1. Install Ultralytics
pip install ultralytics

# 2. Download UEC FOOD 100
wget http://foodcam.mobi/dataset100.zip
unzip dataset100.zip

# 3. Convert to YOLO format (use FoodInsight conversion script)
git clone https://github.com/bennycheung/FoodInsight
python FoodInsight/datasets/convert_uec_to_yolo.py UECFOOD100 --output UECFOOD100_yolo

# 4. Train
yolo detect train \
    data=UECFOOD100_yolo/data.yaml \
    model=yolo11s.pt \
    epochs=100 \
    device=mps  # or 'cuda' for NVIDIA, 'cpu' for CPU-only

# 5. Test
yolo detect predict \
    model=runs/detect/train/weights/best.pt \
    source=your_food_photo.jpg
```

就这些。欢迎来到 2026。

---

## 参考文献

[1] Joseph Redmon. [*Darknet: Open Source Neural Networks in C*](https://pjreddie.com/darknet/). PJReddie.com, 2016.
  - 用 C 编写的原始 YOLO 实现框架，为实时目标检测研究奠定基础。

[2] Matsuda, Y., Hoashi, H., and Yanai, K. [*Recognition of Multiple-Food Images by Detecting Candidate Regions*](http://foodcam.mobi/dataset100.html). IEEE ICME, 2012.
  - 电气通信大学的 UEC FOOD 100 数据集，含 100 个日本食物类别、14,361 张图像。

[3] Ultralytics. [*YOLO11 Documentation*](https://docs.ultralytics.com/). Ultralytics, 2024.
  - 现代 Python 实现的 YOLO，以简单 API 和丰富的部署选择提供最先进的目标检测。

[4] Benny Cheung. [*FoodInsight Project*](https://github.com/bennycheung/FoodInsight). GitHub, 2026.
  - FoodInsight 项目仓库，含数据集转换脚本、训练流水线和食物检测的部署配置。

[5] Redmon, J., Divvala, S., Girshick, R., and Farhadi, A. [*You Only Look Once: Unified, Real-Time Object Detection*](https://arxiv.org/abs/1506.02640). CVPR, 2016.
  - 引入 YOLO 目标检测方法的开山论文，把检测当作单一回归问题处理，实现实时处理。

[6] Benny Cheung. [*YOLO for Real-Time Food Detection*]({{ site.baseurl }}/yolo-for-real-time-food-detection). Benny's Mind Hack, 2018.
  - 记录用 YOLO v2 和 Darknet 做食物检测的原始博文，是这篇八年回顾的基线。

[7] Zhang, Y., Sun, P., Jiang, Y., et al. [*ByteTrack: Multi-Object Tracking by Associating Every Detection Box*](https://arxiv.org/abs/2110.06864). ECCV, 2022.
  - 高性能多目标跟踪算法，跨帧关联检测框，实现视频流中的持续目标识别。

[8] Tencent. [*NCNN: High-Performance Neural Network Inference Framework*](https://github.com/Tencent/ncnn). GitHub, 2017.
  - 面向移动与嵌入式平台优化的神经网络推理框架，让树莓派等 ARM 设备能高效进行基于 CPU 的推理。

[9] Lin, T.Y., Maire, M., Belongie, S., et al. [*Microsoft COCO: Common Objects in Context*](https://arxiv.org/abs/1405.0312). ECCV, 2014.
  - 用于预训练 YOLO 模型的大规模目标检测数据集，为食物检测等领域特定应用提供迁移学习收益。
