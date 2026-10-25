---
layout: post
title: Stable Diffusion 个人嵌入训练
date: 2022-11-02 12:00:00.000000000 -00:00
permalink: stable-diffusion-training-for-embeddings/
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- AI
- Stable Diffusion
- Textural Inversion
- Python
category: post
comments: true
image: images/stable-diffusion-training-for-embeddings/cover2x.jpg
images:
  cover: images/stable-diffusion-training-for-embeddings/cover.jpg
  header: images/stable-diffusion-training-for-embeddings/header.jpg
published: true
---

<!--excerpt.start-->
我们此前介绍过[神经风格迁移（Neural Style Transfer）]({{ site.baseurl }}/deep-learning-on-windows-10)和 [Deep Dream]({{ site.baseurl }}/deep-dream-on-windows-10)，它们是 5 年前 AI 技术应用于艺术作品的第一批热门应用，但很快便让位于一种更强大、更能干的模型——文本反转（Textual Inversion）。Stable Diffusion 是一款免费工具，它使用文本反转技术借助 AI 创作艺术作品。该工具为用户提供了庞大的艺术图库，其中的作品由一个 AI 模型生成，该模型在来自 ImageNet 和 LAION 数据集的海量图像上训练而成。由此得到的扩散模型（Diffusion Model）能够以多种媒介创作出极其逼真的图像，包括数字绘画、基于照片的艺术、漫画，甚至动画。这个工具极易上手，几秒钟就能创建图像。我们可以在 [Lexica](https://lexica.art/) 网站上看到许多使用 Stable Diffusion 生成美丽艺术作品的示例提示词。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主播既风趣又有见地。

{{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/Podcast_Stable_Diffusion_Personal.mp3
{% include open-embed.html %}
----

成功训练一个个人嵌入之后，Stable Diffusion 的 AI 创造力几乎是无穷无尽的。驱动 Stable Diffusion 的算法是两个协同工作的神经网络：自编码器（autoencoder）和生成对抗网络（generative adversarial network），它们都是为学习数据中的模式而设计的神经网络。这一架构在数千小时的计算机时间里被输入数十亿张图像和文本提示，以教会它如何处理语言和理解图像。训练之后，它能够把某些词语或短语与某些视觉元素关联起来。例如，它可以学会：当你输入“forest（森林）”这个词时，它应当生成一幅包含树木和树叶的图像。

生成下面大部分肖像所用的示例提示词是
```
"[realbenny-t2 | panda] happy front portrait as pixar, disney character
 illustration by artgerm, tooth wu, studio ghibli, sharp focus, artstation".
```

![无穷无尽的头像]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/endless-profile-pictures-from-embeddings.jpg)

*Figure. 使用训练好的个人嵌入和给定的输入提示词，由 Stable Diffusion AI 生成肖像的示例。在 Stable Diffusion 中，我们有许多控制手段来引导 AI 创造力的方向。*

如果你有兴趣学习如何使用 Stable Diffusion 从文本提示生成个人头像，那么读完本文后，你将能够为 Stable Diffusion AI 训练一个个人嵌入（embeddings）模型！

## 安装 Stable Diffusion
名为 [AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 的官方 Stable Diffusion 仓库提供了在 Linux、Windows 和 Mac 上安装的分步说明。这里我们不再逐一介绍，但如果你决定在配备 M1 Pro 芯片的 Mac 上安装，我们会留下一些提示。如果你不使用 M1 Pro，可以放心跳过本节。

### 在 Mac M1 Pro 上安装
> 如果 Xcode 尚未完整安装，请运行以下命令来完成安装：
> ```
> xcodebuild -runFirstLaunch
> ```

虽然 Web UI 运行良好，但在 Apple Silicon 上运行这个 fork 仍然存在一些问题。现在所有采样器似乎都能工作，除了 “DPM fast”（它返回随机噪声），以及 DDIM 和 PLMS（二者都会立即失败并报出以下错误："AssertionError: Torch not compiled with CUDA enabled"）。

### 安装 HomeBrew
首先，你需要使用 [Homebrew](https://brew.sh/) 安装所需的依赖。

```
xcode-select --install

/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/benny.cheung/.bash_profile
eval "$(/opt/homebrew/bin/brew shellenv)"
brew -v
```

### 安装 Rosetta 2
它就是将 intel 64 位代码自动翻译为 arm64 的背后魔法！

```
sudo softwareupdate --install-rosetta --agree-to-license
```

### 安装 Stable Diffusion 依赖项
脚本可以从[这里](https://github.com/dylancl/stable-diffusion-webui-mps/blob/master/setup_mac.sh)下载，或者按照下面的说明操作。

```
brew install cmake protobuf rust python git wget
```

脚本可以从[这里](https://github.com/dylancl/stable-diffusion-webui-mps/blob/master/setup_mac.sh)下载，或者按照下面的说明操作。

1. 打开 Terminal.app
2. 运行以下命令：

```
$ cd ~/Documents/
$ curl https://raw.githubusercontent.com/dylancl/stable-diffusion-webui-mps/master/setup_mac.sh -o setup_mac.sh
$ chmod +x setup_mac.sh
$ ./setup_mac.sh
```

安装完成后，你会在 `stable-diffusion-webui` 目录中找到 `run_webui_mac.sh`。使用 `./run_webui_mac.sh` 运行该脚本即可启动 Web UI。该脚本会自动激活 conda 环境、从仓库拉取最新更改并启动 Web UI。退出时，conda 环境会被停用。

**Homebrew 安装后的注意事项：**

安装之后，我们并不希望 PostgreSQL 和 Redis 在 Mac 重启时总是自动启动，可以执行以下命令移除这些服务

```
brew services stop postgresql@14
brew services stop redis
```

## Stable Diffusion 是如何工作的？

Stable Diffusion 是一个文本到图像的生成模型，你可以输入类似这样的文本提示，
```
half (realbenny-t1 | yoda) person, star war,
art by artgerm and greg rutkowski and alphonse mucha
```
Steps: 30, Sampler: LMS, CFG scale: 7, Seed: 3711471449, Size: 512x512, Model hash: 7460a6fa, Denoising strength: 0.75, Mask blur: 4

- 组合了自训练的嵌入 `realbenny-t1` 与默认网络中的 `yoda`，呈现为一个人。
- 背景和主题如同 `star war`
- 组合了 `artgrem`、`greg rutkowski` 和 `alphonse mucha` 的艺术风格

它输出了这样一幅图像（512x512）作为结果，看起来像我自己和尤达（Yoda）的混合生物！

![个人嵌入混合示例]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/personal_embeddings_mixture_example.png)


### 高层概览
这个系统由三个部分组成：
-   一个语言模型，把你输入的文本提示转换为扩散模型可以通过交叉注意力（cross-attention）机制使用的表示。这一部分使用的是现成的（"from the shelf"）Bert 分词器（tokenizer）加 transformer。
-  扩散模型是一个[时间条件 U-Net](https://www.paepper.com/blog/posts/deep-learning-on-medical-images-with-u-net/)。它以你的文本提示表示和一份高斯噪声源作为输入，从表示中去除噪声，逐步逼近你的文本提示。这一过程会重复多次，这些变化被称为时间步（time steps）。
-   一个解码器（Decoder），它接收扩散模型的输出并将其上采样为完整图像。扩散模型在 64x64 分辨率上工作，解码器将其放大到 512x512 的分辨率，目标是在整幅结果图像中保持与 64x64 输入中所见相似的特征。

![Stable Diffusion 工作原理]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/Stable_Diffusion_How_It_Works_Diagram.png)

*Figure. 语言模型为文本提示创建嵌入。它连同一些随机噪声一起被送入扩散模型。扩散模型朝着嵌入的方向对噪声进行去噪。这一过程重复若干次。最后，解码器把图像放大到更大的尺寸。*

> 请参阅 [Marc Päpper 的文章](https://www.paepper.com/blog/posts/how-and-why-stable-diffusion-works-for-text-to-image-generation/)，它对这些技术训练概念有很好的讲解，本文不再涉及。

#### 从噪声到逼真图像
Stable Diffusion 架构的所有组成部分都在海量图像和文本上完成了训练，从而创建出覆盖人类大部分语义空间的嵌入。当新的文本提示把多个概念组合在一起时，这些概念就被合并成一个涵盖输入概念的新表示。潜在扩散模型（latent diffusion model）被训练用来从噪声中揭示出图像，而在来自自编码器和文本模型的嵌入引导下，最终得到的图像就是提示中所输入概念的组合。随后，解码器帮助把图像放大，从图像生成过程中创建出高分辨率图像。

这些随机图像是使用“噪声”创建的。噪声是一类不包含任何有用信息的数据。模型使用一个称为“扩散（diffusion）”的过程来生成输出图像。在扩散过程中，数据流经若干层噪声——它们是随机生成、不包含任何信息的图像。当模型最初收到一个文本提示时，它对目标图像应该是什么样子毫无先验知识。因此，它首先会随机生成一幅完全不含任何视觉元素的图像。

这意味着，最初的图像只是“噪声”。算法会挑选出一些与提示关键词相匹配的图像，尝试把它们拟合到“噪声”中。算法首先会捕捉输入图像中的粗略模式。例如，它可能学到某些颜色更常与某些类型的图像相关联。接着，它利用这些粗略模式来创建具有相应颜色的细致图像。在生成若干图像之后，算法会开始捕捉更具体的模式，并继续不断细化图像

妙处在于，Stable Diffusion 架构在训练过程中学到了相互重叠的视觉概念。它能够组合诸如“半人/半尤达”这样的概念并创作出完整图像，因为它在训练中学习过该概念的各个组成部分。这一架构还能够
- 通过在大量相关概念的图像上训练，来使用像 Star Wars 或尤达（Yoda）这样的特定概念
- 在此之上，把从 `realbenny-t1` 额外训练得到的嵌入用作一个新概念

随着数据流经每一层，它会从周围捕捉到越来越多的模式，从而形成更加细致的图像。随着模型从周围学到越来越多的模式，它会慢慢开始加入更多细节。这一过程持续到扩散过程的终点，此时它就能够产出一幅与用户输入完美匹配的超逼真图像。

当模型到达最后一层时，它就能生成与文本提示相符的高度逼真的图像。从文本提示创建图像的过程被称为“文本反转（textual inversion）”。通过组合所有这些图像和概念，它可以运用所学的知识创造出逼真的新图像。

在深入个人嵌入模型的训练过程之前，我们先来讨论嵌入（embedding）与超网络（hypernetwork）的区别。

## 嵌入与超网络的区别
Stable Diffusion 中的**嵌入（Embedding）**层负责把输入（例如文本提示和类别标签）编码为低维向量。这些向量帮助引导扩散模型生成与用户输入相匹配的图像。**超网络（Hypernetwork）**层是系统学习和表示自身知识的一种方式，它让 Stable Diffusion 能够基于自己以往的经验来创建图像。

### 嵌入（Embedding）
- **嵌入（Embedding）**：文本反转的结果。文本反转试图为模型找到一个特定的提示词，使其生成与你的训练数据相似的图像。模型本身保持不变，你只能得到模型本来就有能力生成的东西。所以嵌入基本上只是一个“关键词”，它会在内部被展开成一个非常精确的提示词。

嵌入是指把对象的特征映射到向量空间中。例如，在一个机器学习任务中，训练集可能由表示其想要学习的对象的特征向量组成。嵌入技术随后把每个对象的特征转换为数字向量。这个向量然后可以用作另一个机器学习算法（如神经网络）的输入。

### 超网络（Hypernetwork）
- **超网络（Hypernetwork）**：一个额外的层，会在图像通过模型渲染之后再进行处理。超网络会把模型的所有结果向你的训练数据方向偏斜，因此它实际上是以每个超网络约 80mb 的小文件体积来“改变”模型。

在 Stable Diffusion 中，超网络负责保留对系统先前生成图像的记忆。这意味着当用户提供新的输入时，系统可以利用其已有知识来创建更准确的图像。超网络让系统能够更快地学习并随时间不断改进。

### 优点与缺点
优点和缺点基本上是同一回事：任何包含描述你训练数据之内容的图像，都会看起来像你的训练数据。如果你训练的是一只特定的猫，那么想用超网络得到任何其他的猫都会非常困难。不过，它似乎依赖于模型已经认识的关键词。

我们的用例是从文本提示生成自画像。我们发现，与超网络相比，使用个人嵌入进行训练能带来更快、更好的结果。我们注意到，在尝试生成逼真图像时 AI 出现的错误更少，而且我们的测试者认为生成的图像更令人满意。 

## 训练嵌入（文本反转）流程
为了确保你的神经网络得到恰当的训练，必须提供足够数量的图像，涵盖你在不同造型、姿势和背景下的样子。如果你只给 AI 你摆一个姿势或穿一套衣服的照片，它就只能生成与该输入相匹配的图像。给你的 AI 一组多样化的图像来学习，将确保获得更广泛的选择和图像。

根据 Jame Cunliffe 的[视频]( https://www.youtube.com/watch?v=P1dfwViVOIU&ab_channel=JAMESCUNLIFFE)，我们了解到只需 20 张图像就足够了。
- 3 张不同角度的全身照
- 5 张不同角度的半身照
- 12 张不同角度和面部表情的特写
- 我有点强迫症——再多给一点

训练过程要求你提供的图像尺寸为 512x512。如果你的图像尺寸不同，可以使用 [Birme](https://www.birme.net/?target_width=512&target_height=512) 来调整和缩放图像，使其符合神经网络读取的适当要求。

> BIRME：免费将任意图像调整为 512x512；当然，如果你有 Photoshop 许可证，也可以使用它。

![自画像嵌入训练照片集]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/Stable_Diffusion_00_SelfPortrait_Capture20_01.png)

*Figure. 仅仅用于展示各种不太帅的个人肖像照。*

我们建议遵循 [Greg Rutkowski](https://www.artstation.com/rutkowski) 的风格
  - 使用以下提示词来引导超网络训练过程，以跟随其风格变化
```
A portrait of a man, trending on artstation, greg rutkowski, 4k
```

随后，我们在训练期间会使用 `txt2img` 的提示词来取样。这就是我们要先在这里输入提示词的原因，以免忘记。

![用于训练样本的 Stable Diffusion txt2img 提示词]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/Stable_Diffusion_04_txt2img_prompt_01.png)

*Figure. 在 txt2img 页面输入一个测试提示词；随后它将用于生成训练样本结果。你可以利用训练样本来监控训练进度。*

## Stable Diffusion 训练嵌入
我们发现，对于生成自画像而言，使用嵌入进行训练比使用超网络更容易。我们的训练取得了令人满意的良好结果。

### 硬件要求的重要说明
需要注意的是，训练个人嵌入模型需要一块显存（VRAM）至少 8GB 的 GPU。这是一项对生成高质量图像至关重要的硬件要求，GPU 不足可能导致图像质量差或性能低下。

如果你没有兼容的显卡，应当考虑使用云基础设施进行训练。你可以使用 Runpod.Io、Google Colab 或其他提供训练所需 GPU 硬件云端访问的平台，来创建自己的嵌入模型。

### 第 1 步 - 创建新嵌入
-   给它起一个名字——这个名字也将是你在提示词中使用的名称，例如 1 个 token 的 `realbenny-t1` 和 2 个 token 的 `realbenny-t2` 嵌入。名字必须足够独特，以便文本反转过程不会把你的个人嵌入与其他东西混淆。
-   设置一些初始化——我喜欢使用"face"或"person"，因为它们能给我相当多样的结果，而且在编写提示词时简单易记。
-   设置每个 token 的向量数（建议使用 1 或 2，因为它在可编辑性与面部准确性之间达到了很好的平衡，但最终还是取决于你的偏好）。
    - 向量越多往往需要越多的图像，所以要做好准备，为这么多向量生成更多图像

### 第 2 步 - 处理图像
这一步基本上就是把图像处理成 512x512，并可以对它们重命名
- 选择图像的源目录（例如 `/home/data/my_images`）
- 选择图像的目标目录（例如 `/home/data/my_images/preprocess`）
- 可选择启用翻转（flip）和添加说明文字（使用 BLIP 自动为图像生成说明文字）。在训练人脸时，翻转面部并不是一个好主意。
- 点击"Preprocess"

### 第 3 步 - 训练

![Stable Diffusion 嵌入训练设置标注图]({{ site.baseurl }}/images/stable-diffusion-training-for-embeddings/Stable_Diffusion_05_Training_Embeddings_Setting_annotated.png)

*Figure. 展示 Stable Diffusion 设置的一个示例界面——训练嵌入*

- 将第 2 步得到的目标目录粘贴到"Dataset directory"输入框中（例如 `/home/data/my_images/preprocess`）
- 设置学习率非常重要，这会影响神经网络的训练
  - 越高 = 学习越"快"，但要小心 NaN！（参数爆炸了）
  - 我用过低至 0.0005、高至 0.005 的值
  - 向量较少时，稍高的学习率可能有帮助。我在 1 个向量时使用 0.005，看起来没问题。
- 如果不使用风格（style），把提示词模板文件改为 `subject_filewords.txt`
  - style.txt 和 subject.txt 是一样的，但不要使用文件名作为（说明文字）
  - 我更倾向于新建一个模板文件，命名为 `filewords.txt`，其中只包含 `[name] [filewords]` 模板。
- 调低最大步数——因为 100000 步要耗费很多个小时。超过 30000 步之后差别非常小。另外，步数太大也没关系，我们可以随时中断训练。
- 按下"Train"——到你的日志目录中查看图像和嵌入。

### 第 4 步 - 将嵌入投入使用
训练嵌入完成之后，我们在 `textual_inversion/<date>/embeddings` 文件夹中找到了结果。我们可以查看样本图像，找到最适合你想生成的图像类型的那一个。一旦找到令你满意的嵌入文件，就需要复制一份，把它放入你的 `embeddings` 模型文件夹。

例如，我们观察到最好的是 `textual_inversion/<date>/embeddings/realbenny-t2-30000.pt`
- 把训练得最好的 `.pt` 嵌入文件复制到顶层文件夹 `embeddings`，并重命名为 `realbenny-t2.pt`
- 重启 Stable Diffusion UI。你可以看到嵌入被自动加载，

```
...
Loaded a total of 2 textual inversion embeddings.
Embeddings: realbenny-t2, realbenny-t1
...
```

- 嵌入关键词 `realbenny-t2` 已经可以投入使用了。在我们的例子中，我们还额外训练了一个 1 token 的嵌入 `realbenny-t1`，以便针对不同的生成参数使用不同的嵌入。

## 附加说明：提示词注意力/强调
这是 Stable Diffusion 最近新增的功能。在提示词中使用 `()` 可以提高模型对括起来词语的注意力，而 `[]` 则会降低它。你可以组合使用多个修饰符。
速查表：

-   `a (word)` - 将对 `word` 的注意力提高到原来的 1.1 倍
-   `a ((word))` - 将对 `word` 的注意力提高到原来的 1.21 倍（= 1.1 * 1.1）
-   `a [word]` - 将对 `word` 的注意力降低为原来的 1/1.1
-   `a (word:1.5)` - 将对 `word` 的注意力提高到原来的 1.5 倍
-   `a (word:0.25)` - 将对 `word` 的注意力降低为原来的 1/4（= 1 / 0.25）
-   `a \(word\)` - 在提示词中使用字面的 `()` 字符

使用 `()` 时，可以像这样指定权重：`(text:1.4)`。如果不指定权重，则默认为 1.1。只有 `()` 支持指定权重，`[]` 不支持。

如果你想在提示词中使用字面的 `()[]` 字符，请用反斜杠对它们进行转义：`anime_\(character\)`。


## 结语
希望本文提供的关于 Stable Diffusion 的信息，能让你对这个架构有足够的了解，从而创建你自己的个人嵌入模型。除了制作头像之外，你也许还能发现其他基于文本提示生成图像的实用场景！

有兴趣的读者可以继续阅读下一篇关于[用于个人嵌入的 Dreambooth 训练]({{ site.baseurl }}/dreambooth-training-for-personal-embedding)的文章，它将介绍如何训练 Stable Diffusion 的超网络嵌入。

这仅仅是个开始。去吧，训练你自己的嵌入吧！

## 参考文献

### 正式出版物
- Robin Rombach 等，[High-Resolution Image Synthesis with Latent Diffusion Models](https://arxiv.org/abs/2112.10752)（基于潜在扩散模型的高分辨率图像合成），arXiv:2112.10752v2 cs.CV，2022 年 4 月 13 日
- Nataniel Ruiz 等，[DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation](https://arxiv.org/abs/2208.12242)（DreamBooth：面向主题驱动生成的文本到图像扩散模型微调），arXiv:2208.12242v1 cs.CV，2022 年 8 月 25 日，Google Research
  - 在线 [DreamBooth 博客文章](https://dreambooth.github.io/)

### 其他相关博客文章
- Thomas Capelle，[Making My Kid a Jedi Master with Stable Diffusion and Dreambooth](https://wandb.ai/capecape/hector/reports/Making-My-Kid-a-Jedi-Master-with-Stable-Diffusion-and-Dreambooth--VmlldzoyODE0OTMy?galleryTag=stable-diffusion)（用 Stable Diffusion 和 Dreambooth 把我的孩子变成绝地武士），Weights & Biases，2022 年 10 月。
  - 最新的 Stable Diffusion 版本已经包含嵌入训练，这相当于 Dreambooth。基本上，你把某人（或某物）的一组不同图像传给模型，它就会学会描绘这个人或物。为了让模型更容易理解这个新"概念"，你还要把这个概念的类别提供给模型。
- Marc Päpper，[How and why stable diffusion works for text to image generation](https://www.paepper.com/blog/posts/how-and-why-stable-diffusion-works-for-text-to-image-generation/)（Stable Diffusion 为何以及如何实现文本到图像生成），papper.com Blog，2022 年 8 月。
  - 这篇博客文章解释了 Stable Diffusion 图像生成工作原理的更深入技术细节。

### Stable Diffusion
- [Stable Diffusion 1.4 功能特性](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features)
- Stable Diffusion AUTOMATIC1111 [仓库](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- [超网络训练指南](https://rentry.org/hypernetwork4dumdums)

### 视频教程
- James Cunliffe，[Create Art From Your Face With AI For Free Part 2 - No Google Colab - No Code - Automatic1111 Guide](https://www.youtube.com/watch?v=P1dfwViVOIU&ab_channel=JAMESCUNLIFFE)（用 AI 免费从你的面部创作艺术，第 2 部分——无需 Google Colab——无需代码——Automatic1111 指南），Youtube 视频，2022 年 10 月。
  - 在本系列的第 1 部分中，我向你展示了如何使用你自己的图像和 Google Colab 版的 [#dreambooth](https://www.youtube.com/hashtag/dreambooth) 来创作 AI 艺术。这一次我们将使用 [#stablediffusion](https://www.youtube.com/hashtag/stablediffusion) 和 [#automatic1111](https://www.youtube.com/hashtag/automatic1111)，同时利用 [#hypernetwork](https://www.youtube.com/hashtag/hypernetwork) 功能。只要你的 GPU 足够强大，就可以在自己的电脑上免费运行。按照教程操作，并使用下面的链接让自己快速上手！
  - 其中有在 Windows 上安装 Stable Diffusion 的完整说明

- Nerdy Rodent，[Textual Inversion - Make Anything In Stable Diffusion](https://www.youtube.com/watch?v=7Lxdk89W2K0&t=151s&ab_channel=NerdyRodent)（文本反转——用 Stable Diffusion 创造万物），Youtube 视频，2022 年 10 月
  - 这个视频有点长，但它很好地讲解了最新的嵌入训练。此外，它用一个直观的例子展示了如何定义 token 数量 1、2、4、16，以及这会如何影响生成图像的可编辑性。本质上，token 越多，生成的图像就越接近嵌入图像；token 越少，生成图像的可编辑性越高。

### 辅助工具
- 试用 [Lexica](https://lexica.art/)，查看许多生成美丽 Stable Diffusion 艺术作品的示例提示词。
- 使用 [Birme 批量调整图像尺寸](https://www.birme.net/?target_width=512&target_height=512)
  - 训练过程要求你提供的图像尺寸为 512x512。如果你的图像尺寸不同，可以使用该网站来调整和缩放图像，使其符合神经网络读取的适当要求。