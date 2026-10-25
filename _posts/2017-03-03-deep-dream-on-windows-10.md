---
layout: post
title: 在 Windows 10 上用 Caffe 运行 Deep Dream
date: 2017-03-03 12:00:00.000000000 -05:00
permalink: deep-dream-on-windows-10
author: Benny Cheung
artwork:
  author: Style of Kandinsky transfer to Ralph McQuarrie "Robot Dream"
tags:
  - Deep Dream
  - Neural Network
  - Python
  - GPU
category: post
comments: true
image: images/deep-dream-on-windows-10/cover2x.jpg
images:
  cover: images/deep-dream-on-windows-10/cover.jpg
  header: images/deep-dream-on-windows-10/header.jpg
published: true
---

<!--excerpt.start-->
Deep Dream 是一种让模式检测算法"过度解读"模式的算法。Deep Dream 算法是一个经过改造的神经网络：它不再是识别输入图像中的物体，而是把图像朝着其训练数据集的方向改变，从而产生令人惊叹的超现实主义、如梦似幻的图像。
<!--excerpt.end-->
（阅读 Google 原始博客 <https://research.googleblog.com/2015/06/inceptionism-going-deeper-into-neural.html>）

其结果是美丽的幻觉图像，如下图所示。神经网络把它被训练去识别的感知对象放大了。我不禁想：这与某些药物作用于我们视觉皮层的效果是否如出一辙？

![Deep Dream 变换]({{ site.baseurl }}/images/deep-dream-on-windows-10/Dream_transform_result_annotate_650.jpg)

*图：梵高《星夜》的 Deep Dream 变换*

本文是我上一篇博客 [Windows 10 上的 GPU 深度学习]({{ site.baseurl }}/deep-learning-on-windows-10)的续篇。你可能想先了解如何设置 NVidia CUDA 8.0，以利用 GPU 加速 Deep Dream 计算。

## Caffe
<http://caffe.berkeleyvision.org>

Caffe 或许是最早的主流工业级深度学习工具包，始于 2013 年底，凭借其（当时）出色的卷积神经网络实现而流行。它至今仍是计算机视觉社区中最流行的工具包，许多扩展仍在积极增加。尤其是，众多流行的预训练神经网络（即 Caffe Model Zoo）可以轻松下载和使用。

Deep Dream 脚本使用的是 Google 在 ILSVRC 2014 中的获奖作品 **GoogLeNet**，一个用于图像识别的 22 层深层网络（原理说明见 <http://joelouismarino.github.io/blog_posts/blog_googlenet_keras.html>）。
GoogLeNet 完成了 ImageNet 数据集（各种动物、家居用品、车辆等）的分类任务：GoogLeNet 集成模型的前五个预测中有 93.33% 的概率包含正确类别。

![GoogLeNet CNN 结构图]({{ site.baseurl }}/images/deep-dream-on-windows-10/GoogLeNet_diagram_650.png)

*图：GoogLeNet CNN 22 层深层网络*

我们下载并使用的 Caffe 预训练模型，是使用 quick_solver.prototxt 迭代 2,400,000 次（60 个 epoch）的快照。

| name | caffemodel | caffemodel_url |
|------|------------|----------------|
| BVLC GoogleNet Model | bvlc_googlenet.caffemodel | <http://dl.caffe.berkeleyvision.org/bvlc_googlenet.caffemodel>

更多 Caffe 信息见：
* [Caffe 教程文档](http://caffe.berkeleyvision.org/tutorial/)
* [BVLC 参考模型](http://caffe.berkeleyvision.org/model_zoo.html)与[社区 Model Zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)

### 准备 Python 虚拟环境
<https://www.continuum.io/downloads>

**注意**：本文假设你在 Windows 上使用 `bash` shell。
推荐的 `bash` shell 来自 Git for Windows <https://git-for-windows.github.io> 的安装。

你可以在 Windows 10 上安装 Anaconda Python 2.7 或 Anaconda Python 3.5。稍后我们会创建一个虚拟环境来隔离 Deep Dream 工具的安装。下文把你的 Anaconda 安装位置记作 `{path_to_anaconda_location}`。

#### 定义 Anaconda 虚拟环境
经验之谈：折腾一套新工具通常是一团糟且充满冲突。使用隔离的 Python 虚拟环境可以让你免于安装带来的头痛与灾难。
在 `bash` shell 中输入以下命令，其中 `caffe`（或你自己起的名字）是虚拟环境的名称，`python=2.7` 是你想使用的 Python 版本。

```bash
conda create -n caffe python=2.7 anaconda
```

按 y 继续。这会把 Python 版本及所有相关的 `anaconda` 打包库安装到 `{path_to_anaconda_location}/envs/caffe`

### 在虚拟环境中安装支持包
`caffe` 虚拟环境安装完成后，用以下命令 `activate` 该 virtualenv：

```bash
source activate caffe
```

继续安装全部 shell 脚本命令所需的包，

```bash
conda install boost
conda install mingw libpython
```

然后安装 Caffe 的依赖

```bash
conda install --yes numpy scipy matplotlib scikit-image pip six
```

你还需要一个与预编译依赖兼容的 Google protobuf Python 包。该包可以这样安装：

```bash
conda install --yes --channel willyd protobuf==3.1.0
```

### 在 Windows 10 上安装 Caffe
在 Windows 10 上安装 Caffe 的偷懒方式，是从 Github 上 Caffe 的 Windows 分支下载预编译二进制：
<https://github.com/BVLC/caffe/tree/windows>

就我的 Windows 10 环境而言，可以在以下两者中任选：
* Visual Studio 2015, CUDA 8.0, Python 3.5: Caffe Release（64 位）
* Visual Studio 2015, CUDA 8.0, Python 2.7: Caffe Release（64 位）

由于我们已经创建了 Anaconda Python 2.7 虚拟环境来承载实验，我们选择安装 `Visual Studio 2015, CUDA 8.0, Python 2.7: Caffe Release` 包。

安装完成后，务必把以下内容加入 Windows 的环境变量，其中 `{path_to_caffe}` 指 Caffe 的安装位置。

```bash
PYTHONPATH={path_to_caffe}\caffe\python
```

## Deep Dream Python 脚本

### 安装 Deep Dream 脚本
你可以从 GitHub 仓库克隆 Deep Dream 脚本
<https://github.com/bennycheung/PyDeepDream>

```bash
git clone https://github.com/bennycheung/PyDeepDream.git
```
*（[2017/08/02] 修正 git 仓库路径为 `PyDeepDream.git`）*

然后按 `requirements.txt` 清单安装 Deep Dream 脚本的依赖

```bash
cd PyDeepDream
conda install --yes --file requirements.txt
```

查看命令行选项，

```bash
python deepdream.py --help
```

安装 GoogLeNet Caffe 预训练模型

```bash
cd PyDeepDream
mkdir bvlc_googlenet
cd bvlc_googlenet
```

从 <http://dl.caffe.berkeleyvision.org/bvlc_googlenet.caffemodel> 下载（约 54 MB）
并把模型保存为 `bvlc_googlenet/bvlc_googlenet.caffemodel`。`deepdream.py` 默认在该位置查找模型。

#### 使用方法
别忘了激活你准备好的 `caffe` 虚拟环境。

```bash
source activate caffe
```

用 `-i` 指定输入的内容图像。它会在一个随机层上进行 deep dream。

```bash
python deepdream.py -i {your_image}.jpg
```

如果你想手动指定 Deep Dream 起始的层深度、类型和八度（octave）：

```bash
python deepdream.py -d 1 -t 1 -o 6 -i Style_StarryNight.jpg
```

#### 跑遍所有层的实验
我们用 `-d` 定义在哪一层执行 Deep Dream。层级数字会被映射为一个 GoogLeNet 层名。

```bash
# Level 1 - inception_3a_1x1
python deepdream.py -d 1 -t 1 -o 6 -i Style_StarryNight.jpg

# Level 2 - inception_3b_1x1
python deepdream.py -d 2 -t 1 -o 6 -i Style_StarryNight.jpg

# Level 3 - inception_4a_1x1
python deepdream.py -d 3 -t 1 -o 6 -i Style_StarryNight.jpg

# Level 5 - inception_4c_1x1
python deepdream.py -d 5 -t 1 -o 6 -i Style_StarryNight.jpg

# Level 7 - inception_4e_1x1
python deepdream.py -d 7 -t 1 -o 6 -i Style_StarryNight.jpg

# Level 9 - inception_5b_1x1
python deepdream.py -d 9 -t 1 -o 6 -i Style_StarryNight.jpg
```

输出的 dream 图像与原图存放在一起，并带有所用 inception 层名的标记。例如，`Style_StarryNight.jpg` 配合 `-d 1` 将产生 Deep Dream 结果 `Style_StrarryNight_inception_3a_1x1_dream.jpg`。

以下是 Deep Dreaming 的图像，

![Deep Dream 各层结果]({{ site.baseurl }}/images/deep-dream-on-windows-10/Dream_layer_results_annotate_650.jpg)

*图：从神经网络不同深度的 inception 层得到的 Deep Dream 结果。浅层放大神经网络的纹理模式，深层放大神经网络识别的物体*


## 树莓派 3 上的 Deep Dream
Deep Dream 实际上也可以运行在树莓派 3 和 Raspbian Jessie 上；具体方法见这篇出色的教程：

<http://www.knight-of-pi.org/deepdream-on-the-raspberry-pi-3-with-raspbian-jessie/>

**注意：**
在树莓派 3 上运行并不快（意料之中，因为 CPU 较弱且没有 GPU 加速）。每次快照需要 5 到 20 分钟。另外受内存限制，它无法进行第 6 层之后的 Deep Dream（即 inception_4d_1x1 是极限）。
