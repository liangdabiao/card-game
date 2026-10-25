---
layout: post
title: 用 YOLO 做实时食物检测
date: 2018-06-07 01:00:00.000000000 -05:00
permalink: yolo-for-real-time-food-detection
author: Benny Cheung
artwork:
  author: Style of Kandinsky's TransverseLine transfer to food images
tags:
  - Deep Neural Network
  - Object Detection
  - YOLO
category: post
comments: true
image: images/yolo-for-real-time-food-detection/cover2x.jpg
images:
  cover: images/yolo-for-real-time-food-detection/cover.jpg
  header: images/yolo-for-real-time-food-detection/header.jpg
published: true
series: "Food Detection"
series_order: 3
---

<!--excerpt.start-->
对识别零食与食物的痴迷一直是试验最新机器学习技术的有趣主题。最高目标将是一个能对常见食物做实时分类与定位的计算机视觉系统，让物联网设备可以部署在 AI 边缘，服务众多食物应用。
<!--excerpt.end-->

上一篇文章[用树莓派 3 做零食监视器]({{ site.baseurl }}/snack-watching-with-raspberry-pi)中的 `Snack Watcher`，
对提取的图像特征使用经典机器学习技术，识别结果实在谈不上出色。原因在于手工特征容易受背景物体、光照、物体在空间中的位置以及物体类别变化的影响。为了降低错误率，需要对环境做精细调整；随后，环境假设变得不切实际，无法部署到真实生活场景中。

随着卷积神经网络（CNN）的最新改进，图像分类准确率近年来（自 2014 年起）突飞猛进。在许多情形下，AI 识别人工智能物体的能力甚至超过人类专家。作为食物检测的技术人，深度学习方法是食物监视的未来。

深度学习的常见困难是需要大型数据集。与其投入大量人力收集所需的食物图像，
我找到了 Food100 数据集 [UEC FOOD 100](http://foodcam.mobi/dataset100.html)（来自日本电气通信大学的食物识别研究组），包含 100 类食物照片。每张食物照片都有一个边界框，标明食物在照片中的位置。这是把 `Snack Watcher` 升级为包罗万象的 `Food Watcher` 的完美数据集。

![UEC Food 100 图像样本]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_train_samples_700.jpg)

尽管这个数据集中的大多数食物类别是日本的大众食物，
凭借多伦多的国际美食文化，我们认出大多数类别毫不费力，例如"green salad""hot dog"和"hamburger"。训练结果在我们的西方文化中依然会很有趣。

这里是 [YOLO 实时食物检测在 720p 视频流上的结果](https://www.youtube.com/watch?v=mZ3FlCeDWlU)，运行在 Nvidia GTX TitanX 上，约 70 fps！

{% include youtubePlayer.html id="mZ3FlCeDWlU" %}

继续阅读本文，了解如何搭建并训练一个定制的 YOLO 神经网络来实现这一结果。

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/yolo-for-real-time-food-detection/Podcast_YOLO_Food_Detection.mp3
{% include open-embed.html %}
----

## YOLO 实时目标检测
在解释最新最强的 YOLO 目标检测之前，值得先了解目标检测的演化，以体会 YOLO 的贡献。

### 图像分类
图像分类是：给定输入图像，呈现给 CNN，预测一个类别标签及描述预测正确置信度的概率。概率最高的类别即预测类别。这个类别标签旨在刻画整幅图像的内容；它并不定位预测类别出现在图像中的位置。

例如，给定下图（左）的输入图像，我们的 CNN 把图像标记为 "hot-dog"。

![图像分类 vs 目标检测]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_image_classification_vs_object_detection_700.jpg)

### 目标检测
目标检测构建在图像分类之上，力求精确定位图像中每个物体出现的位置。

执行目标检测时，给定输入图像，我们希望得到：

* 一组边界框，或图像中每个物体的 (x, y) 坐标
* 与每个边界框关联的类别标签
* 与每个边界框和类别标签关联的概率/置信度分数

上图（右）演示了深度学习目标检测的一个例子。注意 "hamburger" 和 "french-fries" 是如何分别用边界框分类和定位的。

因此，目标检测是叠加在图像分类之上的额外算法复杂度。

### 实时目标检测
除了目标检测，终极挑战是检测能做得多快。要达到可接受的"实时"性能，期望是至少 15 fps（帧每秒），即每张图像约 67 ms 完成目标分类与定位。

### 你好，Darknet 的 YOLO 
在很长一段时间里，检测系统都是把分类器或定位器改造成目标检测之用。它们在多个位置和尺度上把模型应用于图像。图像中得分高的区域被视为检测结果。

感谢 Joseph Redmon 的 Darknet 实现 <https://pjreddie.com/darknet/yolo/>，**YOLO** 采用了一种完全不同的方法。它把单个神经网络应用于整幅图像。这个网络把图像划分为区域，并为每个区域预测边界框和概率。这些边界框按预测概率加权。有兴趣的读者应该研读原始论文_"You Only Look Once: Unified, Real-Time Object Detection"_ <https://arxiv.org/abs/1506.02640>。能看到用 C 语言写成的实现依然是性能之王，实在令人耳目一新。

![YOLO 工作原理]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Yolo_How_it_Works2_700.jpg)

YOLO 模型在测试时接收整幅图像，因此其预测有图像全局上下文的支撑。它用单次网络评估完成预测，不像 R-CNN 这类系统对单张图像需要数千次评估。这使得 YOLO 极快，在有算力的 GPU 上实时运行。

## 实时食物检测
我们将用 Darknet 配合日本 Food100 数据集训练一个定制的 YOLO 神经网络！`Food Watcher` 将成为能实时识别常见食物的最先进 AI。但愿 AI 会对这些美丽的碳水化合物复合物（ aka 食物）的人类需求表现出更多同情心。

### YOLO 训练
以下说明集中于描述 YOLO v2 的设置与训练。
首先，你需要下载并安装 Darknet <https://pjreddie.com/darknet/install/>。
> YOLO v3 的流程非常相似，只有一些配置差异。我们不在本文中把讨论搞混。

要让 Darknet YOLO 训练跑起来，需要

* 每张图像的目标边界框文件
* 全部类别名称的类别名文件
* 训练图像列表的训练数据集文件
* 验证图像列表的验证数据集文件
* YOLO 神经网络规范的配置文件
* 查找全部数据信息的数据位置文件

这是我的 github <https://github.com/bennycheung/Food100_YOLO_Tools> 工具集，有疑问就去看！它还包含后文描述的我的配置 `.cfg`、类别名 `.names` 和数据位置 `.data` 文件。
它们为满足 Darknet 对边界框和训练数据的要求而设计或修改。
* `food100_generate_bbox_file.py` - python 脚本，读取每个 Food100 类别图像目录的 `bb_info.txt` 并创建单独的 bbox 文件。
* `food100_split_for_yolo.py` - python 脚本，把全部 Food100 类别图像拆分为 (1) `train.txt` 训练图像列表和 (2) `test.txt` 验证图像列表。
* `food100_tk_label_bbox.py` - python 脚本，使用 UI 帮助绘制与图像关联的 bbox。

![food100_tk_label_bbox.py 运行示例]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/food100_tk_label_bbox_tool.jpg)

#### 目标边界框文件
下载并解压 Food100 数据集 [UEC FOOD 100](http://foodcam.mobi/dataset100.html) 后，
需要后处理才能做出符合 Darknet YOLO 训练要求的边界框。

Food100 的类别形如，

| food_id | name |
|----|------|
| 1	| rice
| 2	| eels_on_rice
| 3	| pilaf
| 4	| chicken-n-egg_on_rice
| 5	| pork_cutlet_on_rice
| 6	| beef_curry
| 7	| sushi
| ... | up to 100 classes

>

#### 边界框类别编号
Darknet YOLO（v2 与 v3 皆然）期望类别编号从 0 开始，即用 food_id-1 得到类别编号

```
[object-class-id] [center-X] [center-Y] [width] [height]
```

> 最初我忘了给 [object-class-id] 减 1，让我一度以为训练坏了！重新对齐为从 0 开始的 [object-class-id] 之后，检测结果就正确了。

#### 边界框描述文件
Darknet YOLO 期望每个 `.jpg` 图像文件都有一个对应的边界框 `.txt` 文件——位于同一目录、同名但以 `.txt` 为扩展名，文件内容为：该图像上的目标编号和目标坐标。

如果我们把 food100 图像目录命名为 `images`，那么 Darknet 会自动在 `labels` 目录中寻找对应的 `.txt`。例如，`images/1/2.jpg` 会去 `labels/1/2.txt` 文件中找对应的标签。我喜欢这个做法，因为我们可以把编辑的 bbox 保留在 `images/1/2.txt`，而 `labels/1/2.txt` 是 YOLO 训练要求的 bbox 格式。

#### 图像中的边界框坐标
每个食物类别目录中有一个 `bb_info.txt`，包含每个图像文件的全部 bbox。原始 bbox 的格式为，

```
[image-number] [top-left-X] [top-left-Y] [bottom-right-X] [bottom-right-Y]
```

然而，YOLO 期望每个 `.jpg` 图像文件都有一个对应的 bbox 描述 `.txt` 扩展名文件。bbox 描述文件的格式为，

```
[object-class-id] [center-X] [center-Y] [width] [height]
```

字段定义为，

* [object-class-id] - 目标的整数编号，从 0 到 (food_id-1)
* [center-X] [center-Y] [width] [height] - 相对图像宽高的浮点值，取值范围 (0.0 到 1.0]
  * 例如：[x] = [absolute_x] / [image_width]，或 [height] = [absolute_height] / [image_height]
  * 注意：[center-X] [center-Y] - 是边界框的中心（不是左上角）

#### 数据位置文件
告诉系统在哪里找训练与验证路径的数据文件，`food100.data`

```
classes = 100
train = /Users/bcheung/dev/ML/darknet/data/food100/train.txt
valid = /Users/bcheung/dev/ML/darknet/data/food100/test.txt
names = /Users/bcheung/dev/ML/darknet/data/food100/food100.names
backup = backup
```

* classes = 100 是类别数量，应等于类别名文件的行数
* train = [path] 是全部训练图像的文件列表，每行一张图像
* valid = [path] 是全部验证图像的文件列表，每行一张图像
* names = [path] 是列出类别名的文件
* backup = [path] 是中间 `.weights` 和最终 `.weights` 文件的写入位置

显然，把数据 [path] 改成你自己的数据位置。

#### 训练数据集文件
`train.txt` 文件列出全部训练图像，每行一张。例如，

```
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6164.jpg
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6170.jpg
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6158.jpg
...
```

不应有来自验证数据集的图像。

#### 验证数据集文件
`test.txt` 文件列出全部验证图像，每行一张。例如，

```
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6990.jpg
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6149.jpg
/Users/bcheung/dev/ML/darknet/data/food100/images/61/6099.jpg
...
```

不应有来自训练数据集的图像。

#### 类别名文件
类别名指定在 `food100.names` 中。
随后，类别 id 从 0 开始，即 `rice` 是类别 `0`。
这个文件没有类别 id，只有名称。

```
rice
eels-on-rice
pilaf
chicken-n-egg-on-rice
pork-cutlet-on rice
beef-curry
sushi
...
```

#### 配置文件
我们需要创建配置文件 `yolov2-food100.cfg`。
> 你可以复制 darknet 的 `cfg/yolov2-voc.cfg` 并做修改

```
[net]
# Testing
# batch=1
# subdivisions=1
# Training
batch=64
subdivisions=8
...
```

batch 和 subdivisions 的含义是你把一个 batch 拆分成多少个 mini batch。

* batch=64，为这一"迭代"载入 64 张图像。
* subdivision=8，把 batch 拆分为 8 个 "mini-batch"，即 64/8 = 每个 "minibatch" 8 张图像，送入 GPU 处理。

这会重复 8 次直到 batch 完成，然后以新的 64 张图像开始新的迭代。 batching 时你是在更多图像上取平均，目的不只是加速训练过程，也是让训练更具泛化性。
如果你的 GPU 显存充足，可以减小 subdivision 以同时向 GPU 载入更多图像处理。

进一步编辑配置文件中的 `classes` 和 `filters` 规格，

```
# number of filters calculated by (#-of-classes + 5)*5
# e.g. (100 + 5)*5 = 525
edit to line 237: filters=525
edit to line 244: classes=100
```

### Darknet 训练
训练时我们使用在 Imagenet 上预训练 YOLO 的卷积权重。我们使用 Extraction 模型的权重。
你可以直接从 YOLO 作者的网站下载卷积层权重 `darknet19_448.conv.23`。

```
wget https://pjreddie.com/media/files/darknet19_448.conv.23
```

然后，我们可以开始训练定制的 Darknet YOLO（把路径改为你的文件位置）

```
./darknet detector train data/food100/food100.data data/food100/yolov2-food100.cfg data/darknet19_448.conv.23
```

训练日志形如，

```
...
Loaded: 0.000042 seconds
Region Avg IOU: 0.833301, Class: 0.427628, Obj: 0.624505, No Obj: 0.004012, Avg Recall: 1.000000,  count: 7
Region Avg IOU: 0.704510, Class: 0.390626, Obj: 0.270502, No Obj: 0.003151, Avg Recall: 0.875000,  count: 8
Region Avg IOU: 0.749244, Class: 0.420920, Obj: 0.444735, No Obj: 0.003990, Avg Recall: 1.000000,  count: 8
Region Avg IOU: 0.748636, Class: 0.649185, Obj: 0.626328, No Obj: 0.003061, Avg Recall: 1.000000,  count: 8
Region Avg IOU: 0.718167, Class: 0.590265, Obj: 0.568300, No Obj: 0.003715, Avg Recall: 1.000000,  count: 8
Region Avg IOU: 0.798298, Class: 0.615076, Obj: 0.464930, No Obj: 0.004568, Avg Recall: 1.000000,  count: 8
Region Avg IOU: 0.727497, Class: 0.273119, Obj: 0.348449, No Obj: 0.004568, Avg Recall: 0.875000,  count: 8
Region Avg IOU: 0.767684, Class: 0.285230, Obj: 0.376508, No Obj: 0.003249, Avg Recall: 0.875000,  count: 8
9998: 5.413931, 5.206974 avg, 0.000010 rate, 2.222207 seconds, 639872 images
...
```

通常网络对每个类别（物体）需要 2000 次迭代，即 100 个食物类别需要 100 x 2000 = 200000 次迭代。
但要更精确地判断何时停止训练，使用以下启发式（经验法则）：

训练期间，你会看到各种误差指标的变化，当 0.XXXXXXX avg 不再下降时就该停止：

```
Region Avg IOU: 0.767684, Class: 0.285230, Obj: 0.376508, No Obj: 0.003249, Avg Recall: 0.875000,  count: 8
9998: 5.413931, 5.206974 avg, 0.000010 rate, 2.222207 seconds, 639872 images
```

* 9998 - 迭代编号（batch 数）
* 0.767684 avg - 平均损失（误差）——越低越好

当你看到平均损失 0.xxxxxx avg 在许多次迭代后不再下降，就应停止训练。

## 测试 YOLO 食物检测
运行测试之前，确保把配置 `yolov2-food100.cfg` 文件中的 `batch` 和 `subdivisions` 改为 1 张图像。

```
[net]
# Testing
batch=1
subdivisions=1
# Training
# batch=64
# subdivisions=8
...
```

用测试图像测试食物检测器，以 `test` 模式运行，

```
./darknet detector test data/food100/food100.data data/food100/yolov2-food100.cfg backup/yolov2-food100_final.weights My_amazing_food.jpg
```

Food100 的 YOLO v2 测试结果相当亮眼，能正确检测广泛的食物类别，

![Food100 YOLO v2 结果]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLOv2_Tests02_700.jpg)


### 用摄像头做食物检测
要运行实时食物检测器，接上摄像头并以 `demo` 模式执行 Darknet，

```
./darknet detector demo data/food100/food100.data data/food100/yolov2-food100.cfg backup/yolov2-food100_final.weights
```

### 用视频文件做食物检测
如果你有视频文件例如 `.mp4`，可以以 `demo` 模式把视频流送入实时食物检测器，

```
./darknet detector demo data/food100/food100.data data/food100/yolov2-food100.cfg backup/yolov2-food100_final.weights My_amazing_food_video.mp4
```

你也可以用 `-thresh` 选项降低检测阈值，0.3 即 30% 置信度

```
./darknet detector demo data/food100/food100.data data/food100/yolov2-food100.cfg backup/yolov2-food100_final.weights -thresh 0.3 My_amazing_food_video.mp4
```

### 关于食物检测结果的更多思考
以下是一些供消遣的有趣结果，

| 食物检测案例 | 描述 |
|---------------------|-------------|
| ![Hot Dog]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLO_HotDog_500.jpg) | 自从 HBO 剧集《硅谷》发布了一个识别"热狗——与非热狗"的真实 AI 应用以来，识别热狗就成了任何食物检测的黄金测试标准。 |
| ![Ham and Burger]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLO_Hambruger_500.jpg) | 识别汉堡中的火腿——火腿不一定在面包里，即便是特写镜头。YOLO 仍能看出那是一个汉堡 |
| ![Soupy Things]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLO_Bowl_Noodles_500.jpg) | 识别带汤的食物——当食物以汤的形式呈现时。被淹没的内容并没有迷惑 YOLO。 |
| ![Food with a Mask]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLO_Egg_Smile_and_Cut_up_500.jpg) | 带着笑脸的单面煎蛋也逃不过检测器。切成块的牛排依然被检测到。什么都躲不过 AI 的感知力。 |
| ![Shapy Food]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/Food100_YOLO_Pizza_500.jpg) | 它能理解披萨与披萨吐司的区别。也许 AI 能从形状上分辨差异。 |

>

就写到这里，AI 确实享受看食物！...

![AI 是危险的]({{ site.baseurl }}/images/yolo-for-real-time-food-detection/AI_is_dangerous.jpg)

但我直到 AI 认为我是"食物"才意识到 AI 是危险的。:)

## 参考文献
* Joseph Redmon，他用 C 语言实现的超快 Darknet
  * <https://pjreddie.com/darknet/>
* Nils Tijtgat，他关于 YOLO v2 训练的详细说明帮助巨大
  * <https://timebutt.github.io/static/how-to-train-yolov2-to-detect-custom-objects/>
* Adrain Rosebrock，他关于众多 Python 计算机视觉项目的出色教程
  * <https://www.pyimagesearch.com/2018/05/14/a-gentle-guide-to-deep-learning-object-detection/>
* 日本电气通信大学食物识别研究组，他们收集全部食物图像和边界框的辛勤工作
  * <http://foodcam.mobi/dataset100.html>
* Jonah Group，他们在 AI 创新与硬件上的支持文化
  * <http://www.jonahgroup.com>
