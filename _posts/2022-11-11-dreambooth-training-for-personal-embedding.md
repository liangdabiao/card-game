---
layout: post
title: 用 Dreambooth 训练个人嵌入
date: 2022-11-11 12:00:00.000000000 -00:00
permalink: dreambooth-training-for-personal-embedding/
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- Dreambooth
- Stable Diffusion
- AI Creativity
- Python
category: post
comments: true
image: images/dreambooth-training-for-personal-embedding/cover2x.jpg
images:
  cover: images/dreambooth-training-for-personal-embedding/cover.jpg
  header: images/dreambooth-training-for-personal-embedding/header.jpg
published: true
---

<!--excerpt.start-->
本文将聚焦训练一个比上一篇文章——[Stable Diffusion Training for Personal Embedding]({{ site.baseurl }}/stable-diffusion-training-for-embeddings)——所描述的原始 Stable Diffusion 软件"更深、走得更远"的嵌入。那篇文章已经有一个扎实的嵌入可以用来生成艺术。我们想把它训练得更个性化，让它能穿越更大、更复杂的语境。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Podcast_Dreambooth_Training_Personal.mp3
{% include open-embed.html %}
----

训练这个更泛化的 checkpoint 模型将借助 Dreambooth 的超网络（hypernetwork）训练技术。这会把个人图像与其中的嵌入结合起来，产出比仅从基础 Stable Diffusion 嵌入模型生成的作品质量更高、更准确的新艺术。这是下一步，将提升艺术的保真度与准确性，以及 AI 对给定提示词的理解。

![Dreambooth Stable Diffusion 个人嵌入示例]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Benny_SD_Profile_Gridx16_02.jpg)

*图：个人嵌入的全部创意想象，化作玩具与材料！从左上开始：人偶（action figure）、土豆先生、毛绒娃娃、辛普森、培乐多、乐高小人、加菲猫、迪士尼、low-poly、funko-pop、皮克斯、塑料、电子游戏、狗、毛线娃娃、乐高积木。*

学会使用 Dreambooth 训练个人嵌入之后，我们还将探索 AI 创造力的话题，学习如何用 img2img 的 inpainting 改进生成的图像。

### 使用 Dreambooth
Dreambooth 和 Stable Diffusion 都能产出伟大的艺术作品。关键差别在于：Dreambooth 更面向想创建看起来像特定人物的图像的用户，而 Stable Diffusion 是更通用的图像生成。
- 用 Stable Diffusion，艺术家创建一个提示词，然后把提示词跑过 AI 系统看生成什么图像。这个系统能产出更逼真的图像，但可能要更久才能生成用户喜欢的东西。
- 用 Dreambooth，艺术家训练一个 AI 系统来创建特定艺术家或艺术风格。产出的图像更可预测，生成也更快。

Dreambooth 能生成更精确的结果，但只能生成特定的个体。Stable Diffusion 能生成多种图像，但结果可能不那么精确。一般用途下，Stable Diffusion 是更好的选择——而精确的个体用途下，Dreambooth 更胜一筹。

![Dreambooth Stable Diffusion 训练个人图像]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Dreambooth_Stable_Diffusion_Training_Personal_Images.jpg)

*图：输入一组个人图像，Dreambooth 训练将产出一个可在各种语境中使用的 Stable Diffusion 超网络模型。参见 Nataniel Ruiz 等人的原始论文 [DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation](https://arxiv.org/abs/2208.12242)*

跟随 "Aitrepreneur" 在 YouTube 上的[视频](https://www.youtube.com/watch?v=7m__xadX0z0&ab_channel=Aitrepreneur)——详解如何在 RunPod.io 或 Google 的 Colab 环境上用 Dreambooth 训练定制的 Stable Diffusion 模型——他一步步解释了流程，甚至在视频描述里附上了必要的代码参考。我们看到训练出的定制模型表现出色，这激励我们亲自试一试这项技术！

### 如何找到训练硬件？
用 Dreambooth 算法训练的最大障碍之一是硬件要求。该算法需要至少 24GB 显存的 GPU。NVidia RTX 3090 是个好选择——如果你弄得到的话——但全新一块要价高达 1,500 加元。我们寻找更便宜的替代方案，例如能以更低价格提供这种级别显存的云服务。

在搜索了具备所需硬件的合适服务器之后，我们决定使用 [RunPod.io](https://www.runpod.io/)。它提供租用安全的云端机器的选项，包含一块 24GB 显存的 RTX A5000。租金只要每小时 0.49 美元，比买 NVidia RTX 3090 划算多了！

![RunPod Dreambooth 训练：分配带 24GB GPU 的 pod]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_allocate_pod_with_GPU_24G.jpg)

> **重要提示！！！启动 Pod 时，确保分配了更多磁盘空间。就我们而言，把容器磁盘和卷磁盘空间都从 20 改为 40 GB；否则可能冒着磁盘空间耗尽的风险。

![RunPod Dreambooth 训练：更改磁盘空间]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_disk_sizes.jpg)

### 选哪个 Dreambooth-Stable-Diffusion？
[Google 最初的 Dreambooth-Stable-Diffusion](https://github.com/XavierXiao/Dreambooth-Stable-Diffusion) 在 GitHub 上有数百个分支。不过，"Aitrepreneur" 的视频指出，[Joe Penna 分支的 Dreambooth-Stable-Diffusion](https://github.com/JoePenna/Dreambooth-Stable-Diffusion/) 包含专门设计的 Jupyter notebook，帮助训练你的个人嵌入。其中有为 Google Colab 或 RunPod.io 设计的 notebook。这让我们可以训练一个能与 Stable Diffusion 配合工作的超网络模型。目标是创建一个 checkpoint 模型，基于文本提示词生成个人头像图片。

## 训练过程
现在进入训练细节的"干货"：训练超网络是为 Stable Diffusion 软件生成个性化嵌入的过程；它创建的嵌入可用于生成你自己风格的图像。Stable Diffusion 训练需要时间，完成后你将得到一个定制嵌入。你可以训练任意多独特的对象和风格。

用 Dreambooth 训练只需要有限的训练数据。例如：20 张图像就足以创建一个嵌入，而且数据不必高质量。光线差或不完整的图像也能正常工作。在 24G 显存的 RTX3090 上，训练时间通常约一小时。

![RunPod Dreambooth 训练 notebook 03 进度说明]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_notebook_03_progress_notes.jpg)

*图：展示了在 RunPod.io 上成功运行训练 notebook 所需步骤的参考。每个训练步骤的说明，都可以回到这张图对照。*

Jupyter notebook 是一个基于网页的代码执行程序，可用于交互式地编写和运行代码。这个程序让你可以在浏览器窗口运行 Python 脚本及其他编程语言。有许多可用的实用模块，可以执行各种任务，例如创建图形图表或可视化。

### 1. git clone 到 RunPod 的工作区
在新的 Jupyter notebook 中，执行这条 `git` 命令，把代码仓库克隆进 pod 的工作区。

再强调一次，[Joe Penna 分支的 Dreambooth-Stable-Diffusion](https://github.com/JoePenna/Dreambooth-Stable-Diffusion/) 包含帮助训练个人嵌入的 Jupyter notebook。它可以在 RunPod.io 上运行。

```
!git clone https://github.com/JoePenna/Dreambooth-Stable-Diffusion/
```

### 2. 把 sd-v1-4.ckpt 上传到 pod 的工作区
为了加速 Stable Diffusion 模型的上传速度（直接用 RunPod 上传太慢）。我们建议使用 Google Drive。在 notebook 中执行这个安装：
```
%pip install gdown
```

然后用 `gdown` 从 Google Drive 链接下载文件。要找到你的 hugging-face checkpoint 文件的链接，你需要找到共享 Google Drive 链接的 ID。

```
!gdown https://drive.google.com/uc?id=<link_to_file_location_id>
```

### 3. 在 pod 上运行 notebook
克隆完成后，我们关闭下载 notebook，切换到使用 pod 工作区中检出的 `Dreambooth-Stable_Diffusion`：
- 使用 `Dreambooth_runpod_joepenna.ipynb` notebook。

> **重要提示！！！截至 [2023/01/09]，JeoPenna 的 RunPod.io Jupyter notebook 缺少一些 Python 模块，无法成功运行。请在执行 `pip install` 的 `BUILD DEV` 单元格中加入以下缺失模块：

```
!pip install taming-transformers-rom1504
!pip install clip
```

### 4. 运行 notebook 单元格生成正则化图像
不必自己生成一组正则化图像，我们可以跳到 notebook 单元格运行 "Download pre-generated regularization images"。

正则化图像是一类帮助神经网络检测模式、提升准确性的数据。正则化图像的创建方式是：取一组数据点（如像素或矢量图像）并随机修改它们以加入噪声。这些图像随后在训练中使用，以防止过拟合——即神经网络开始"看到"数据中并不存在的模式。过拟合会让神经网络在特定训练集上过度精确，却无法把知识用于对新数据做出准确预测。

![RunPod Dreambooth 训练 notebook 03 正则化图像]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_notebook_03_reg_images.jpg)

*图：展示下载给定类别正则化图像的 notebook 单元格。*

运行该单元格后，它会抓取 1500 张这样的正则化图像，放入 `./regularization_images` 文件夹，供后续训练步骤使用。

### 5. 把你的图像放入训练图像文件夹
> **重要提示**！！！训练期望 PNG 图像，务必确保训练图像是 PNG 格式！

要训练你的嵌入，你需要一组图像。图像应为 PNG 格式，尺寸应为 512 x 512 像素。你还应为图像提供文字描述，让模型能够学习。图像应清晰、光线良好，且图像之间应有足够的变化，给模型足够的信息可供学习。

notebook 单元格要求我们把每个文件作为 URL 位置提供。但实际上，单元格从 URL 位置下载图像之后，这些图像会存入 `./training_images` 文件夹。换句话说，我们可以直接把训练图像放入 `./training_images` 文件夹，而不用费劲上传到云服务。

> 我们不需要运行下面的 notebook 单元格。它使用一个手工输入的 URL 列表从网络下载图像。

![RunPod Dreambooth 训练 notebook 01 进度]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_notebook_01_progress.jpg)

*图：展示的 notebook 单元格正在遍历 URL 列表并把图像下载到 "./training_images" 文件夹。理解了这个单元格之后，我们可以直接把训练图像放入 "./training_image" 文件夹。能看到将要用于训练的图像作为单元格执行输出，这一点很棒。*


### 6. 训练模型

> **重要提示**！！！这是 Dreambooth 的要求，务必把 `sd-v1-4.ckpt` 文件重命名为 `model.ckpt`

全部设置配置完成、正则化图像与训练图像上传就位之后，对 notebook 的训练单元格做以下修改，我们就几乎可以开始训练了：
- 定义项目名，例如 `bennycheung`，它将用于 checkpoint 模型的文件名。
- 定义唯一 token，例如 `bennycheung`，以及类别词 `person`。记住这一点很重要："token 类别"的组合，例如 "bennycheung person"，将用于在超网络中触发定制嵌入。
- 定义 `max_training_steps` 数量，每张训练图像应有 100 步；本例我们用 30 张训练图像——即需要 3000 步。

![RunPod Dreambooth 训练 notebook 04 训练（标注版）]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_notebook_04_training-annotated.jpg)

*图：图示 notebook 的训练单元格。我们需要修改高亮的参数，如本节所述。*

然后，训练 3000 步约需 80 分钟。

训练完成后，运行单元格把最终的 checkpoint 模型复制到 `./trained_models` 文件夹。然后，你可以下载最终的 checkpoint 模型来使用。文件名包含训练的全部细节，

```
2022-11-10T00-46-29_bennycheung_30_training_images_3000_max_training_steps_bennycheung_token_person_class_word.ckpt
```

我们建议重命名为更短的名字，例如 `bennycheung-v1-3000.ckpt`。

![RunPod Dreambooth 训练 notebook 05 checkpoint 模型]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Training_notebook_05_checkpoint_model.jpg)

*图：图示 notebook 的单元格：把最后的 checkpoint 模型保存到 "trained_models" 文件夹。*


## 模型使用
使用你的嵌入的方式是 `<token> <class>` 模式对，例如——图示的训练好的模型是 `bennycheung person` 而不只是 `bennycheung`。

```
bennycheung person as a Warhammer 40K Space Marine, movie still,
  cinematic lighting, dramatic, octane render, long lens,
  shallow depth of field, 8k, hyper detailed
```
- `bennycheung person` 有了这个组合，它将触发嵌入影响图像结果。

这里有几个示例结果：按照文本提示词，嵌入高保真、非常准确地融入了生成的图像。

![Stable Diffusion Dreambooth bennycheung 模型生成的星际战士]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/bennycheung_model_generated_space_marine.jpg)

*图：把自己想象成战锤 40k 星际战士（Astartes）的梦想终于成真！*

许多人可能认为 AI 艺术最重要的部分是 AI 本身。但生成艺术品的质量也在很大程度上取决于给 AI 的文本提示词，因为文本提示词会在创作新艺术时强烈影响 AI 的选择。OpenArts 是该领域的领军专家之一，他们发布了一本出色的 [Stable Diffusion Prompt Book PDF 图书](https://openart.ai/promptbook)，讲解如何提升文本提示词的准确性。这会在你创作 AI 艺术时提升文本提示词的准确性，从而带来更好、更准确的艺术！

## 论 AI 创造力
关于 AI 是否真的具有创造力的争论，有人可能主张它只是在组合既有的想法和艺术来创造新东西。这个主张可以理解。然而，对人类创造力也可以这么说。复制与创造新东西是有区别的，正如人类艺术家自己也从其他艺术风格中学习，以创造独特的东西。这是否意味着人类创造力不是真的？

可以说，AI 并非简单地复刻既有作品，而是在探索新的可能性与艺术风格。具体到 Stable Diffusion，它以独特而有趣的方式创造了与人类作品不同的新艺术与风格。该技术能够在既有知识之间建立新的连接，找出更多的变化与有效的组合方式，从而孕育更独特的艺术风格。它还能寻找不寻常的方法与策略来创造更有趣、更令人惊喜的艺术——以人类受限的方式之外的方式。

以更积极的视角看，AI 的进步非常迅速。变得更大、更高效、拥有更大神经连接的神经网络，能够在不同的想法与记忆之间建立连接。这种层次的模拟正是神经网络试图重现的。随着这一能力被进一步模拟，AI 得以具备创造力——它可以在不同的想法、记忆与艺术风格之间做出更复杂的连接，也能从既有想法中创造新想法。

### 定制模型艺术作品画廊
下面的画廊展示了 Stable Diffusion 产出的创意变体。深度与创造力显而易见，你可以体会到神经网络连接是如何被创建以形成新艺术风格的。这位 AI 艺术家确实才华横溢，能从给定的提示词生成如此多的变化！

```
bennycheung person, anime, warhammer 40k, space marine, meditative look,
  trending on artstation, watercolor pen by brad mesina, greg rutkowski
```
- 使用各种 Sampler 与 CFG scale，Seed: 769704955

![SD Benny 星际战士水彩 01]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/SD_Benny_SpaceMarine_Watercolor_01.jpg)

![SD Benny 星际战士水彩 03]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/SD_Benny_SpaceMarine_Watercolor_03.jpg)

*图：这些 AI 创意作品展示了通过定制训练模型创造出的丰富多样与风格。Stable Diffusion 能够从单一提示词产出多样的图像，每件作品在风格与细节上都独一无二。*

这位 AI 真正触及了创造力。虽然展示的艺术品是用 AI 创作的，它仍是创造力的产物。AI 被编程了许多东西，但正是神经网络把这一切组合起来，创造出原创的作品或概念。它有能力同时使用新的与既有的记忆，创作出真正美丽的艺术。

## 论图像生成的修复

### 修复眼睛
Stable Diffusion 不擅长准确生成眼睛。幸运的是，我们可以用内置的 `img2img` Inpainting 功能修正生成图像中的怪异眼睛。

例如，我的眼球比例失调，看起来很怪。我们来展示如何修复。

我们可以把图像送入 `img2img`（图到图）标签页的 `Inpaint` 子标签功能。
- 我们应保持与原图相似的提示词
- 图像尺寸应与原图完全一致
- 用画笔工具（可调整大小）涂掉（涂黑）我们想重新生成的那部分图像。本例中，我们想涂掉眼睛，让 `Inpaint` 替换那些像素。
- 我们通常会多生成几张，看运气（纯随机概率），算法会产出我们喜欢的完美眼睛。然后，我们保留那张图像。未涂掉的其他像素保持原样——没有像素被重新生成。

![Stable Diffusion img2img Inpaint 修复眼睛]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Stable_Diffusion_img2img_Inpaint_Fixing_Eyes.jpg)

眼睛被改进到令人满意的水平。这就是我们选择的那张。
下面的图像展示了 `Inpaint` 眼睛修复操作的前后对比。

![Stable Diffusion img2img Inpaint 修复眼睛对比 01（标注版）]({{ site.baseurl }}/images/dreambooth-training-for-personal-embedding/Stable_Diffusion_img2img_Inpaint_Fixing_Eyes_Compare_01_annotated.jpg)

*图：展示眼睛被 "inpaint" 以生成更平衡、更逼真的我的眼睛，我知道。*

### 修复手
Stable Diffusion 图像生成的另一个弱点是：看运气，手也会画乱。我们可以套用类似的 `Inpaint` 技术重新生成手，提示词用 "correct hands"，强调算法注意生成"正确的手"。


## 结语
我们正在见证艺术与技术的一次重大突破：AI 如今能够理解并模仿人类艺术，甚至创造自己的艺术风格！这是两全其美——人类创造力与机器精确性融合，产出真正美丽的艺术作品。这一切都通过 Dreambooth 技术训练超网络、以个人嵌入打造最佳艺术风格来完成。

就本文而言，RunPod.io 上的总训练成本是 - 0.608 美元，换来这么多乐趣，太值了！！

## 参考文献

-  Aitrepreneur, [Dreambooth: Train Stable Diffusion With Your Images Using Google's AI!](https://www.youtube.com/watch?v=7m__xadX0z0&ab_channel=Aitrepreneur), Youtube video, Oct 2022
   - 很棒的教程，一步步走完用 RunPod.io 配合 [Dreambooth-Stable-Diffusion 仓库](https://github.com/JoePenna/Dreambooth-Stable-Diffusion/) Jupyter notebook 的训练流程。

### 正式发表
- Robin Rombach, et. al., [High-Resolution Image Synthesis with Latent Diffusion Models](https://arxiv.org/abs/2112.10752), arXiv:2112.10752v2 cs.CV, 13 Apr 2022
- Nataniel Ruiz, et. al., [DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation](https://arxiv.org/abs/2208.12242), arXiv:2208.12242v1 cs.CV, 25 Aug 2022, Google Research
  - 在线 [DreamBooth 博客](https://dreambooth.github.io/)

### 其他相关博文
- Suraj Patil, [Training Stable Diffusion with Dreambooth](https://wandb.ai/psuraj/Dreambooth/reports/Training-Stable-Diffusion-with-Dreambooth--VmlldzoyNzk0NDc3), Weights & Biases, Oct 2022.
  - 对其用 Dreambooth 训练 Stable Diffusion 实验的分析，对训练参数有极详细的讲解。 
- OpenArts, [Stable Diffusion Prompt Book](https://openart.ai/promptbook), Oct 2022
  - [下载 PDF](https://cdn.openart.ai/assets/Stable%20Diffusion%20Prompt%20Book%20From%20OpenArt%2010-28.pdf)
  - OpenArts 新近发布的 Stable Diffusion 提示词书，汇集了提示词的技巧与诀窍。

### 支持工具
- 玩玩 [Lexica](https://lexica.art/)，看看生成精美 Stable Diffusion 艺术作品的大量示例提示词。
- 用 [Birme 批量调整图像尺寸](https://www.birme.net/?target_width=512&target_height=512)
  - 训练流程期望你提供的图像尺寸为 512x512。如果你的图像尺寸不同，可以用这个网站调整并缩放图像，使其符合神经网络读取的正确要求。
