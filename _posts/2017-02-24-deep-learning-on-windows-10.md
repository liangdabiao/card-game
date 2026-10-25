---
layout: post
title: 在 Windows 10 上用 GPU 做深度学习
date: 2017-02-24 12:00:00.000000000 -05:00
permalink: deep-learning-on-windows-10/
author: Benny Cheung
artwork:
  author: Vincent van Gogh "Starry Night"
tags:
  - Deep Learning
  - Neural Network
  - Python
  - GPU
category: post
comments: true
image: images/deep-learning-on-windows-10/cover2x.jpg
images:
  cover: images/deep-learning-on-windows-10/cover.jpg
  header: images/deep-learning-on-windows-10/header.jpg
published: true
---

<!--excerpt.start-->
你刚把最新的 NVidia GPU 装进了自己的 Windows 10 机器。
除了用超高画质跑最新游戏来享受这笔新投资之外，
我们应该停下来意识到：我们实际上拥有了一台能做严肃计算的超级计算机。
深度学习算法正是能吃掉这些 GPU 算力的饥饿巨兽之一。
<!--excerpt.end-->

遗憾的是，深度学习工具通常对类 Unix 环境更友好。
当你试图在 Windows 上整合工具链时，会遇到许多困难。
我花了好几天才敲定一套能在 Windows 10 上成功运行的深度学习工具链。
以下是我的选型与安装过程总结。如果你有毅力坚持到最后，
就能在本文结尾运行神经风格迁移，生成"深刻"而震撼的图像
（原始论文《A Neural Algorithm of Artistic Style》见 <https://arxiv.org/abs/1508.06576>）

![艺术风格迁移结果网格]({{ site.baseurl }}/images/deep-learning-on-windows-10/Benny_DeepArt_Grid.jpg)

*图：对一幅自画像运行神经风格迁移算法后的样例结果网格*

先快速概览一下我们要安装的前置依赖，
* Scipy + PIL（通过 Anaconda Python 安装）
* Numpy（通过 Anaconda Python 安装）
* CUDA (GPU)（通过 NVidia 安装包安装）
* CUDNN (GPU)（可选，通过 NVidia 安装包安装）
* PyCUDA（通过预编译二进制安装）
* Theano（通过 pip 安装）
* Keras（通过 pip 安装）

## 在 Anaconda Python（Windows 10）下安装 Theano
<http://deeplearning.net/software/theano/>

Theano 是流行的深度学习框架之一，
它提供一个 Python 库，让你能够高效地定义、优化和求值涉及多维数组的数学表达式。

**注意**：本文假设你在 Windows 上使用 `bash` shell。
推荐的 `bash` shell 来自 Git for Windows <https://git-for-windows.github.io> 的安装。
在你的"mind hacking"生涯中，你需要 `git` 来检出有趣的项目和源码。
所以，现在是时候给自己装备上称手的利器了。


### Anaconda Python
<https://www.continuum.io/downloads>

Anaconda 是一个易于安装的免费包管理器、环境管理器和 Python 发行版，收录了 720 多个享有免费社区支持的开源包。

你可以在 Windows 10 上安装 Anaconda Python 2.7 或 Anaconda Python 3.5。稍后我们会创建一个虚拟环境来隔离深度学习工具的安装。下文把你的 Anaconda 安装位置记作 `{path_to_anaconda_location}`。

#### 定义 Anaconda 虚拟环境
经验之谈：折腾一套新工具通常是一团糟且充满冲突。使用隔离的 Python 虚拟环境可以让你免于安装带来的头痛与灾难。
在 `bash` shell 中输入以下命令，其中 `theano`（或你自己起的名字）是虚拟环境的名称，`python=2.7` 是你想使用的 Python 版本。

```bash
conda create -n theano python=2.7 anaconda
```

按 y 继续。这会把 Python 版本及所有相关的 `anaconda` 打包库安装到 `{path_to_anaconda_location}/envs/theano`

### 在虚拟环境中安装 Theano
`theano` 虚拟环境安装完成后，用以下命令 `activate` 该 virtualenv：

```bash
source activate theano
```

继续安装全部 shell 脚本命令所需的包，

```bash
conda install boost
conda install mingw libpython
```

然后，安装 theano

```bash
pip install theano
```

把以下几行加入 `$HOME/.theanorc` 文件

```
[global]
floatX = float32
device = gpu

[nvcc]
flags=-L{path_to_anaconda_location}\envs\theano\libs
compiler_bindir=C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin
```

因为我已经为 Theano 创建了虚拟环境，可以看到 `flags` 指向的是该虚拟环境的 libs。
我读到过必须使用 VS 12.0 才能编译 CUDA。这个配置在我的 CUDA 8.0 和 Visual Studio 2015 community 安装下工作正常，我没有再费心验证它是否适用于其他 Visual Studio 版本。

### 安装 PyCUDA
CUDA 是 NVidia 发明的并行计算平台和编程模型。
它借助图形处理器（GPU）的算力，带来计算性能的显著提升。
你需要从 NVidia <https://developer.nvidia.com/cuda-downloads> 下载并安装 CUDA 开发工具包。

PyCUDA 是 CUDA API 的 Python 封装，让你可以用 Python 编写 CUDA 代码。
在我们的场景中，安装 PyCUDA 最省事的方式是使用预编译的 Windows 10 二进制。通过以下链接下载：

<http://www.lfd.uci.edu/~gohlke/pythonlibs/?cm_mc_uid=67672146725714876308558&cm_mc_sid_50200000=1487630855#pycuda>

对应我的 CUDA 8.0、Windows 10 64 位下的 Python 2.7，所需的包是：
`pycuda‑2016.1.2+cuda8044‑cp27‑cp27m‑win_amd64.whl`

然后，用 `pip` 安装这个包

```bash
pip install pycuda‑2016.1.2+cuda8044‑cp27‑cp27m‑win_amd64.whl
```

### 测试安装
以交互 shell 模式 `-i` 启动 Python。

```bash
python -i

>>> import theano
Using gpu device 0: GeForce GTX 1070 (CNMeM is disabled, cuDNN not available)
>>>
```

如果所有 GPU CUDA 库都与 Theano 协同正常，你应该能看到你的 GPU 设备被报告出来。


### 安装 Keras
<https://keras.io>

Keras 是一个高级神经网络库，用 Python 编写，可以运行在 TensorFlow 或 Theano 之上。它的开发重点是支持快速实验，这会让你的深度学习编程更加高效。

用 `pip` 安装 keras，

```bash
pip install keras
```

安装 keras 之后，我们可以把 keras 后端改为 `theano`。
打开 `~/.keras/keras.json`

只需把 backend 字段改为 `theano` 或 `tensorflow`，Keras 就会在你下次运行任何 Keras 代码时使用新配置。
就我而言，我希望 `theano` 作为我的后端。

```json
{
    "image_dim_ordering": "th",
    "epsilon": 1e-07,
    "floatx": "float32",
    "backend": "theano"
}
```

### 在 Windows 上运行神经风格迁移
<https://github.com/bennycheung/PyDeepStyle>

到这里，我们已经安装好一套完整的深度学习工具链，足以完成一些有趣的事情。
我们来尝试运行一个算法，利用预训练卷积神经网络把一张图像的内容与另一张图像的风格结合起来。下面的例子把梵高《星夜》的艺术风格映射到一幅自画像上，生成一幅独特的艺术图像。显然，这样的发型注定与梵高的曲线笔触相得益彰。

![艺术风格迁移算法示例]({{ site.baseurl }}/images/deep-learning-on-windows-10/Benny_StarryNight_DeepArt.jpg)

确保启动配置齐全的 `theano` 虚拟环境
```bash
source activate theano
```

> 该脚本已更新，可配合最新 Keras 2.1.6 工作 [2018/05/08]

Python 脚本 `deepstyle.py` 是神经风格迁移算法的 Keras 实现，使用预训练卷积神经网络（VGG19）。

`run.sh` bash 脚本接收你的输入 {content_image}、{style_image} 和 {output_directory} 来生成结果。


```bash
# Shell scrhipt to drive deepstyle.py
# Usage: ./run.sh <content_image> <style_image> <output_dir>
#

if [ "$#" -ne 3 ]; then
  echo "need to specify <content_image> <style_image> <output_dir>"
  exit
fi

mkdir -p $3

python deepstyle.py "$1" "$2" "$3/$3" \
  --image_size 600 \
  --tv_weight 8.5E-05 \
  --iter 25 \
  --content_layer "block5_conv2" \
  --min_improvement 0
```

例如，运行脚本 `run.sh`

```bash
sh run.sh self_portrait.jpg starry_night.jpg result_image
```

每个 epoch 都会有一张迭代图像保存到 `result_image` 目录，文件名为 `result_image_at_iteration_{N}.png`（其中 {N} 是迭代序号）。借此追踪神经网络如何感知并构建图像，非常有趣。

![Benny 星夜 Epoch]({{ site.baseurl }}/images/deep-learning-on-windows-10/Benny_StarryNight_Epoch.jpg)

*图：自画像与星夜风格迁移的迭代 epoch 过程*

在 GTX 1070 GPU（1,920 核）上，每个 epoch 所需时间取决于图像尺寸（gram 矩阵大小）：

* 400x400 gram 矩阵，每个 epoch 约需 13 秒。
* 512x512 gram 矩阵，每个 epoch 约需 20 秒。

大约 25 个 epoch 后，大部分风格就已收敛。你可以尝试更高的迭代次数，让风格迁移收敛出更高的质量。

为方便读者，这里再次列出脚本的输入参数：

* *--image_size*：设置 Gram 矩阵尺寸。默认 400（宽度），因为它出结果又快又好。
* *--iter*：迭代次数。默认 10。先用 10 次迭代测试输出，再增大以改进结果。
* *--content_weight*：内容损失的权重（默认 0.025）
* *--style_weight*：风格损失的权重（默认 1.0）
* *--tv_weight*：正则化因子。取值越小图像越锐利，但 0 没有用处。默认 = 1E-5
* *--content_layer*：选择内容层。论文建议 block4_conv2，但 block5_conv2 能得到更好的结果。默认为 block5_conv2。
* *--min_improvement*：定义继续运行脚本所需的最小改进量。默认 0.0

要复现视频中的结果，请尝试以下输入参数
（在 NVidia GTX TitanX 上测试）：

```
...
python deepstyle.py "$1" "$2" "$3/$3" \
  --image_size 600 \
  --content_weight 1E-05 \
  --style_weight 1.0 \
  --tv_weight 8.5E-05 \
  --iter 300 \
  --content_layer "block5_conv1" \
  --min_improvement 0
```

## 抽象艺术的神经风格迁移
你可以观看[抽象艺术的神经风格迁移](https://www.youtube.com/watch?v=TLwqG5kh8Wc)。这段视频记录了从一幅图像到相应艺术家抽象风格的深度神经风格迁移过程。观察机器如何"思考"——这个人工艺术创作过程——着实迷人！

{% include youtubePlayer.html id="TLwqG5kh8Wc" %}

我们已经准备好在下一篇博文中*更深入*地潜入深度学习！
