---
layout: post
title: 小数据集上的深度迁移学习
date: 2017-03-14 01:00:00.000000000 -05:00
permalink: deep-transfer-learning-on-small-dataset/
author: Benny Cheung
tags:
  - Deep Learning
  - Neural Network
  - Transfer Learning
  - Keras
category: post
comments: true
image: images/deep-transfer-learning-on-small-dataset/cover2x.jpg
images:
  cover: images/deep-transfer-learning-on-small-dataset/cover.jpg
  header: images/deep-transfer-learning-on-small-dataset/header.jpg
published: true
---

<!--excerpt.start-->
卷积神经网络（ConvNet）在图像分类上取得成功依赖两个因素：(1) 有大量数据，(2) 有大量算力；其中 *(1) 有数据* 似乎是更难的问题。数据获取通常是任何实际项目的主要成本。但如果我们只负担得起一个小数据集，还能有效使用 ConvNet 吗？
<!--excerpt.end-->

**迁移学习**（Transfer Learning）技术是利用现有的 ConvNet 特征提取器和相应的训练好的网络权重，迁移到另一个不同的领域使用。研究表明，ConvNet 利用的是层次化的分布式表示。ConvNet 的较低层包含更通用的特征（如边缘检测器或色块检测器），应对许多任务都有用；而 ConvNet 的较高层则变得更加特化，针对原始数据集中领域类别的细节。

----
> [2024/10/07] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/Podcast_Deep_Transfer_Learning.mp3
{% include open-embed.html %}
----

Francois Chollet 在 Keras 博客上的 "[Building powerful image classification models using very little data](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html)"（用极少数据构建强大的图像分类模型）是一篇鼓舞人心的文章，讲解了如何借助向现有 ConvNet 的迁移学习来克服小数据集问题。由于现代 ConvNet 在 ImageNet（含 120 万张图像、1000 个类别）上用多块 GPU 训练需要 2-3 周，人们常常发布最终训练好的 ConvNet 权重，供他人微调（fine-tuning）使用。例如，Caffe 库有一个 [Model Zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)，人们在那里分享网络权重。不过，较高层的权重可能需要微调。以使用 ImageNet 数据集（包含许多猫狗）的网络为例，网络权重的相当一部分可能被投入到高层中用于区分狗的品种的特征上，而这些特征对其他领域就没那么有用。迁移学习策略必须把这些考虑进去。

![VGG16 ConvNet 微调技术]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/VGG16_ConvNet_700.jpg)

*图：用于适配不同领域的 VGG16 ConvNet 微调技术*

一个好的迁移学习策略可以概括为以下步骤：
1. 冻结较低的 ConvNet 块（`蓝色`）作为固定特征提取器。取一个在 ImageNet 上预训练的 ConvNet，移除最后的全连接层，然后把 ConvNet 的其余部分当作新数据集的固定特征提取器。在 VGG16 网络中，这会为每张图像计算一个 4096 维向量，其中包含紧邻分类器之前的隐藏层激活。这些特征被称为 CNN codes。
2. 训练新的全连接层（`绿色`，又称瓶颈层）。为所有图像提取 CNN codes，为新数据集训练一个线性分类器（如 Linear SVM 或 Softmax 分类器）。
3. 微调 ConvNet。在新数据集上替换并重新训练 ConvNet 顶部的分类器，同时通过继续向较高的部分层（`黄色`+`绿色`）反向传播来微调预训练网络的权重。

## 迁移学习实验
本节报告应用上述迁移学习策略的 3 个实验。每个实验都有一个我们必须弄明白的具体问题。
* Kaggle 的 *Dogs vs Cats*——学习如何使用这些技术。
* 牛津大学的 *102 Category Flower*——回答大量类别能否适配。
* UEC 的 *Food 100*——回答零食/食物监视能否适配。

### 实验环境
之前的博文[深度风格迁移]({{ site.baseurl }}/deep-learning-on-windows-10)和[Deep Dream]({{ site.baseurl }}/deep-dream-on-windows-10) 已讲解了如何在 Windows 10 上搭建环境。
这些实验使用 NVidia GTX 1070 GPU、CUDA 8.0，运行于 Windows 10。
所有软件用 Python 编写，使用配置为 Theano 后端的 Keras。

> 由于最新 CUDA 8.0 与最新 Keras 版本不兼容，如有需要，以下步骤可把 Keras 降级到 1.2.0。

#### 降级到 Keras 1.2.0
为了沿用"图像分类"博文的[这里的代码](https://gist.github.com/fchollet/7eb39b44eb9e16e59632d25fb3119975)，
只需降级到 Keras 1.2.0 就解决了运行问题。

查询 Keras 版本，
```
python -c "import keras; print keras.__version__"
```

如果报告的 Keras 版本高于 1.2.0，按以下步骤降级。

```
pip uninstall keras
pip install keras==1.2.0
```

--------------------------------------
### 实验 1：Dogs vs Cats 数据集
按照文章
"[Building powerful image classification models using very little data](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html)"，
两组图片[从 Kaggle 下载](https://www.kaggle.com/c/dogs-vs-cats/data)：1000 只猫和 1000 只狗（从包含 12,500 只猫和 12,500 只狗的原始数据集中提取，每个类别只用前 1000 张图像）。我们再从每个类别各取 400 个额外样本作为验证数据，用于评估模型。

以下是"Dog vs Cat"数据集的一些样本图像。有些图像绝对很有挑战性，例如动物被部分遮挡，或场景不理想。

![Dogs vs Cats 图像样本]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/catdog_train_samples_700.jpg)

*图：Dogs vs Cats 数据集图像样本*

在 NVidia GTX 1070 GPU 上，每个训练 epoch 耗时 20 秒。

```
$ python classifier_from_little_data_script_3.py

Found 2000 images belonging to 2 classes.
Found 800 images belonging to 2 classes.
Epoch 1/50
2000/2000 [==============================] - 20s - loss: 0.4330 - acc: 0.8995 - val_loss: 0.3401 - val_acc: 0.8900
Epoch 2/50
2000/2000 [==============================] - 19s - loss: 0.2412 - acc: 0.9140 - val_loss: 0.3244 - val_acc: 0.9113
...
Epoch 49/50
2000/2000 [==============================] - 21s - loss: 0.0116 - acc: 0.9955 - val_loss: 0.4171 - val_acc: 0.9375
Epoch 50/50
2000/2000 [==============================] - 21s - loss: 0.0190 - acc: 0.9935 - val_loss: 0.3553 - val_acc: 0.9300
```

微调过程显示，准确率随训练 epoch 数量提升，

![Dogs vs Cats 随 Epoch 数的微调准确率曲线]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/catdog_fine_tuning_accuracy.jpg)

整体验证准确率达到 93%，相当亮眼。这要归功于原始 VGG16 ImageNet 网络权重在基础特征提取中见过那么多猫狗。即使只有 1000 张狗和 1000 张猫图片的小数据集，我们仍然能取得很高的准确率。


--------------------------------------
### 实验 2：牛津 102 类花卉
沿用 Alexander Lazarev 的 [Github 代码](
<https://github.com/Arsey/keras-transfer-learning-for-oxford102>) 的编码改进——它让数据集设置和类别数量设置更灵活——我们可以看看 ConvNet 迁移学习策略能否轻松应用到花卉这个不同领域。[牛津 102 类花卉数据集](http://www.robots.ox.ac.uk/~vgg/data/flowers/102/index.html)收录的是英国常见的花卉。每个类别包含 40 到 258 张图像。

![牛津 102 种花卉图像样本]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/flower_train_samples_700.jpg)

*图：牛津 102 类花卉数据集图像样本*

以下是数据集中花卉类别的列表，
<http://www.robots.ox.ac.uk/~vgg/data/flowers/102/categories.html>

> Windows 10 注意事项：`model/vgg16.py` 需要把 NumPy 数组的保存改为 'wb'（二进制写模式）、读取改为 'rb'（二进制读模式）。

Alex 改进的代码可用于任何数据集，只需在运行 `bootstrap.py` 后按 data/sorted 目录中的原始文件结构组织。如果你想把数据集存到别处，也可以，运行 `train.py` 时用特殊参数 `--data_dir==path/to/your/sorted/data` 设置数据集路径即可。
训练过程完全自动化，模型的最佳权重会被保存。

在 NVidia GTX 1070 GPU 上每个 epoch 耗时 55-65 秒。相比之前的 Dogs vs Cats 实验，这是更大的数据集。日志显示模型重新加载后开始微调。

```
$ python train.py --model=vgg16 --data_dir=flower/sorted
....
Weights for "fc1" are loaded
Weights for "fc2" are loaded
Weights for "predictions" are loaded
Found 6149 images belonging to 102 classes.
Found 1020 images belonging to 102 classes.
Epoch 1/300
6149/6149 [==============================] - 92s - loss: 11.5339 - acc: 0.4261 - val_loss: 7.1121 - val_acc: 0.4627
....
Epoch 112/300
6149/6149 [==============================] - 88s - loss: 0.1837 - acc: 0.9572 - val_loss: 0.5032 - val_acc: 0.8853
Epoch 113/300
6149/6149 [==============================] - 88s - loss: 0.1947 - acc: 0.9563 - val_loss: 0.5058 - val_acc: 0.8961
```

微调过程显示，准确率随训练 epoch 数量提升，

![牛津 102 花卉随 Epoch 数的微调曲线]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/flower_fine_tuning_accuracy.jpg)

整体验证准确率达到 89%——在没有调整任何 VGG16 模型超参数的情况下，这依然相当亮眼。若有更深入的洞见和努力，达到 >95% 应该不难。

现在，我们可以对测试数据做预测。

```
$ python predict.py --data_dir=flower/sorted --path=flower/sorted/test/*/*.jpg --accuracy
...
[1018]flower/sorted/test/99/image_07934.jpg : 101 (99) ---> 101 (99)
top-5 [101  37  10   5  46] hit True
[1019]flower/sorted/test/99/image_07941.jpg : 101 (99) ---> 101 (99)
top-5 [101  70  10  37   5] hit True
accuracy 0.85
top-5 0.967647058824
```

整体测试准确率为，

| 指标 | 准确率 |
|---------|----------|
| top-1 准确率 | 85%
| top-5 准确率 | 96.8%

如果我们看 top-5 准确率——即正确类别出现在前 5 个预测中——我们在花卉分类上达到了 96.8%！

--------------------------------------
## 实验 3：UEC Food 100 数据集
在 [Jonah Group](https://www.jonahgroup.com)，我们曾把零食监视作为 PoC，参见之前的文章[用树莓派 3 做零食监视器]({{ site.baseurl }}/snack-watching-with-raspberry-pi)。
那个零食分类器对提取的图像特征使用经典机器学习技术；准确率实在谈不上出色。随着 ConvNet 的到来，我们希望为 Snack Watcher**+** 升级一颗新的 AI 大脑。

为确认升级潜力，我们找到了数据集 [UEC FOOD 100](http://foodcam.mobi/dataset100.htm)（来自日本电气通信大学食物识别研究组），包含 100 种食物的照片。每张食物照片都有一个边界框（bounding box）标明食物在照片中的位置，但我们的 ConvNet 训练实验并未利用该边界框（参见实验后文的混淆分析）。

这个数据集中的大多数食物类别是日本的大众食物。
因此，非日本人士可能对某些类别不熟悉。
好在多伦多的国际美食文化让我们认出大多数类别毫不费力。

![UEC Food 100 图像样本]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/food_train_samples_700.jpg)

*图：UEC Food 100 数据集图像样本*

类别样例，

| id | name |
|----|------|
| 1	| rice
| 2	| eels on rice
| 3	| pilaf
| 4	| chicken-'n'-egg on rice
| 5	| pork cutlet on rice
| 6	| beef curry
| 7	| sushi
| ... | up to 100 categories

> 食物识别研究组的最新进展：他们使用在 ILSVRC2010 数据集上训练的 DCNN（深度卷积神经网络）特征，取得了 72.26% 的 top-1 准确率和 92.00% 的 top-5 准确率。

在 NVidia GTX 1070 GPU 上每个 epoch 耗时 180-200 秒。

```
$ python train.py --model=vgg16 --data_dir=food/sorted

...
Weights for "fc1" are loaded
Weights for "fc2" are loaded
Weights for "predictions" are loaded
Found 10006 images belonging to 100 classes.
Found 4355 images belonging to 100 classes.
Epoch 1/100
10006/10006 [==============================] - 200s - loss: 14.9965 - acc: 0.2187 - val_loss: 3.1983 - val_acc: 0.2774
Epoch 2/100
10006/10006 [==============================] - 192s - loss: 5.6558 - acc: 0.2231 - val_loss: 2.6880 - val_acc: 0.3637
...
Epoch 52/100
10006/10006 [==============================] - 189s - loss: 1.2442 - acc: 0.7556 - val_loss: 2.0172 - val_acc: 0.5449
Epoch 53/100
10006/10006 [==============================] - 189s - loss: 1.1892 - acc: 0.7574 - val_loss: 1.9635 - val_acc: 0.5483
```

微调过程显示，准确率随训练 epoch 数量提升，

![UEC Food 100 随 Epoch 数的微调准确率曲线]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/food_fine_tuning_accuracy.jpg)

整体验证准确率只有 55%。与其他实验相比，结果不算出色。仔细检查预测失败的案例后，我们意识到：食物照片中包含多个食物类别，却被归类为某一个特定类别。更添挑战的是，有些图片很难说清汤里装的是什么，例如乌冬面。此外，有些食物类别天生就是模糊的，例如中式汤。

![UEC Food 100 混淆样本]({{ site.baseurl }}/images/deep-transfer-learning-on-small-dataset/food_category_confusion_700.jpg)

*图：食物类别混淆与挑战的图示*

| Id | Category | Source of Confusion |
|----|----------|-----------|
| 1 | rice | has a mixes of side diskes
| 14 | roll bread | has accompanied with other foods
| 20 | udon noodles | are completely invisible
| 91 | chinese soup | is ill defined

如果我们看 top-5 准确率——即正确类别出现在前 5 个预测中——我们可以宣称 81% 的准确率。当然，我们没有超过 UEC 研究组报告的 92% top-5 准确率。但他们使用了更复杂的组合技术。相比之下，我们只是使用了基础的 ConvNet 迁移学习技术，几乎没有调超参数。

| 指标 | 准确率 |
|---------|----------|
| top-1 准确率 | 54.8%
| top-5 准确率 | 80.8%

有了这个食物分类结果，我们有信心继续升级 Snack Watcher 的识别能力。
