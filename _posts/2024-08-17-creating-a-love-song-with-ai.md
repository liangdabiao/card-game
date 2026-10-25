---
layout: post
title: 用 AI 创作一首情歌
date: 2024-08-17 12:00:00.000000000 -00:00
permalink: creating-a-love-song-with-ai
author: Benny Cheung
artwork:
  author: iStock
tags:
- Generative AI
- AI Music
- AI Avatar
- Movie Making
category: post
comments: true
image: images/creating-a-love-song-with-ai/post_cover.jpg
images:
  cover: images/creating-a-love-song-with-ai/cover2x.jpg
  header: images/creating-a-love-song-with-ai/header.jpg
published: true
---

<!--excerpt.start-->
我一直梦想着为挚爱写一首歌，但我的音乐技能实在有限。不过，我倒确实擅长创作优美的爱情诗。得益于 AI 的进步，我的梦想如今借助几款创新工具得以实现。
在本文中，我将展示我如何组合多种 AI 技术，创作出一支真挚动人的音乐视频：
<!--excerpt.end-->

- [Stable Diffusion](https://github.com/AUTOMATIC1111/stable-diffusion-webui)：专门为我的爱人训练了一个模型，让视觉画面个性化。
- [Hedra](https://www.hedra.com/)：用这款免费 beta 工具生成演唱的虚拟形象（avatar）。虽有局限，效果却出奇地好。
- [FaceFusion](https://docs.facefusion.io/)：用这个工具对某些音乐视频片段做 deepfake 换脸，为我的爱人添上一抹魔法。（视频片段出处：Vivian Chow 周慧敏 - 《天荒愛未老》）
- [Suno](https://suno.com/) v3：把我的诗歌作为歌词提供给这个工具，它谱出曲子，甚至还完成了人声演唱。

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/creating-a-love-song-with-ai/Podcast_Creating_Love_Song_with_AI.mp3
{% include open-embed.html %}
----

![AI 音乐制作工作流]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_Making_Workflow.jpg)

_图：**AI 赋能的情歌创作工作流：**本图展示了使用多种 AI 工具组合制作个性化音乐视频的过程。Stable Diffusion 用于生成定制视觉画面，Hedra 创建演唱虚拟形象，FaceFusion 添加 deepfake 特效把这些元素融入音乐视频。Suno V3 用于根据用户的诗歌谱曲并演唱。最终的情歌影片在 iMovie 中剪辑合成，然后发布到 YouTube。_

经过最后一道视频剪辑的润色，我成功地把所有这些元素编织成一支流畅感人的音乐视频。这个项目真正是梦想成真，全靠 AI 技术不可思议的能力。 

> 以下是我的中文诗，它作为输入歌词提供给了 Suno。

```
[Verse 1]
春雨綿綿情如畫， | Spring rain, tender like a painting,
夏夜星空閃無暇。 | Summer night, stars twinkle without flaw.
秋葉紅遍庭芳秀， | Autumn leaves, red and vibrant in the garden,
冬雪飄飄暖如霞。 | Winter snow, warm and floating like rosy clouds.

[Chorus]
光陰流逝歲無憂， | Time passes without worry through the years,
共渡晨曦共牽手。 | Together, we greet the dawn, hand in hand.
風雨同舟不曾變， | Through storm and wind, our bond unchanged,
地老天荒情深厚。 | As the earth ages, our love remains deep.

[Verse 2]
佳人如玉美若花， | My beloved, as pure as jade, as beautiful as a flower,
傾心細訴枕邊話。 | Whispering sweet words by the pillow.
千言萬語送安康， | Thousands of words wish you peace,
願君笑顏伴芳華。 | May your smile accompany the blooming of youth.
```

观看这个项目的成果：[四季的情话 (Love in All Seasons) - Creating a Love Song with AI](https://www.youtube.com/watch?v=vrKwaheRz2k)

{% include youtubePlayer.html id="vrKwaheRz2k" %}


如果你对这一切是如何实现的感兴趣，让我们逐一探索工作流的每个组件。你甚至可以复现这一流程，为你的挚爱创作一首歌。

## Stable Diffusion——AI 生成虚拟形象

[Stable Diffusion](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 是一款免费的 AI 工具，可用于生成高质量的人脸图像。要使用 Stable Diffusion，你需要提供一些起始参数，如性别、年龄和发型，它就会生成一张符合这些参数的逼真人脸图像。

要生成你的个人虚拟形象图，先按照我们之前的文章 [Dreambooth Training for Personal Embedding]({{ site.baseurl }}/dreambooth-training-for-personal-embedding) 构建个人嵌入（embedding）。你可以任意组合参数，取决于你想创造什么样的虚拟形象。选好参数后，运行 [Stable Diffusion 生成你的形象图]({{ site.baseurl }}/create-personal-animated-ai-avatar)。

![AI_Song_StableDiffusion_Generate_May_Avatar]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_StableDiffusion_Generate_May_Avatar.jpg)

*图：使用 Stable Diffusion（经 AUTOMATIC1111 Web UI）从提示词生成个人形象图。*

## Suno——AI 音乐生成

[Suno](https://suno.com/) v3 对任何有兴趣创作音乐的人来说都是一件迷人的工具。它把"文本到音乐"的概念发挥得淋漓尽致：用户输入歌词或描述，就能收到一整首歌。这可不是简单的叮当小调；我们说的是完整的曲目——包含器乐、人声和歌词，涵盖流行、摇滚、爵士等各种风格。

令人印象深刻的是它的定制程度。你可以指定情绪、乐器和风格，引导 AI 创作出契合你设想的东西。如果对第一版不满意，你可以生成多个迭代版本，反复调整打磨，直到感觉恰到好处。人声合成功能更上层楼，让 AI 能演唱或说唱你的歌词。

尽管优势突出，Suno 也并非没有局限。一些用户指出了歌曲结构和歌词重复的问题。这就是为什么我们可以直接给 Suno 提供我们的歌词，而不是用它自动生成的歌词。

Suno 代表了 AI 音乐生成的重大飞跃。它让音乐创作民主化，无论资深音乐人还是普通用户都触手可及。如果你曾想把文字变成一首情歌，Suno 可能正是你需要的工具。

指定你想要的音乐风格，它就会相应地输出情歌！

```
Cantonese, alternative rock, pop, female voice, romantic
```

![AI_Song_Suno_Generate_Song_from_Poetry_Lyrics]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_Suno_Generate_Song_from_Poetry_Lyrics.jpg)

_图：这幅图展示了用 Suno 把中文爱情诗变成歌曲的过程。通过指定音乐风格——包括粤语、另类摇滚、流行和浪漫元素——Suno 从输入歌词生成了一首定制的情歌。_

## Hedra——音频到虚拟形象视频

[Hedra](https://www.hedra.com/) 是一个专注于为视频生成与叙事开发先进 AI 技术的创作实验室。他们的首要目标是赋能创作者，提供对想象世界、角色和故事的完全创作控制。

公司的旗舰产品是 Character-1，一个基础模型，让用户可以生成由富有表现力、可控的人类角色出演的视频内容。这项技术旨在通过 AI 驱动的角色动画与叙事工具，革新创作者制作视频内容的方式。

Hedra 面向那些寻求创新方式、希望借助 AI 辅助视频生成和世界构建工具把想法变成现实的创作者、电影人和叙事者。Hedra 目前只能接受 512x512 的方形图像。

![AI_Song_Hedra_Generate_Avatar_Singing_UI_02]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_Hedra_Generate_Avatar_Singing_UI_02.jpg)

_图：**用 Hedra 创建栩栩如生的演唱虚拟形象：**这幅图演示了使用 Hedra 生成一个能够表达丰富情感虚拟角色的过程。上传或生成音频后，你可以将其与角色的表情同步，创建逼真的视频内容，从而打造沉浸式、情感充沛的虚拟形象。_

Hedra 让我们的虚拟形象表达丰富的情感。这不只是让形象角色好看，更是让它感觉真实。我们可以生成任意长度的视频（可选扩展默认的 30 秒限制），这意味着我们不受时间约束。

输出质量令人印象深刻，让我们的内容在视觉上颇具吸引力。无论我们想让形象说话、唱歌还是说唱，Character-1 都能胜任。它甚至能把文本转换成逼真的音频，我们可以选择完美契合角色个性的声音。


![AI_Song_May_Avatar_Snapshot_Grid]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_May_Avatar_Snapshot_Grid.jpg)

_图：这个网格展示了 Hedra 生成的虚拟角色，呈现多样的表情与动作。丰富的面部表情和动态动作为形象赋予了逼真而引人入胜的质感，增强了角色的真实感与情感深度。_


## FaceFusion——AI 视频换脸

 [FaceFusion](https://docs.facefusion.io/) 是一款强大的工具，吸引了内容创作者和开发者的目光。它的核心功能是让用户在图像和视频中换脸，逼真程度令人惊叹。这种能力绝非噱头；它建立在先进的 AI 技术之上，让结果看起来真实可信。

FaceFusion 的独特之处在于它的开源属性。它在 GitHub 上开放社区贡献，这意味着它在不断进化。该项目维护活跃，最新版本 2.6.1 发布于 2024 年 7 月。这种持续开发的承诺确保用户受益于定期更新和新功能。

它的高级功能包括提升视频画质的帧增强器、为黑白影像上色的工具，以及基于 wave2lip 模型的唇形同步能力。它还支持多种人脸检测选项，提高换脸的准确性。

重要的是，开发者认真对待伦理考量，实施了审核系统以鼓励负责任的使用。FaceFusion 是 AI 驱动人脸处理领域的一项重大进步，无论对普通用户还是专业人士都提供高质量的成果。

![AI_Song_FaceFusion_May_Music_Video_Clip_Processing_02]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_FaceFusion_May_Music_Video_Clip_Processing_02.jpg)

_图：使用 FaceFusion 对音乐视频场景进行换脸，为我的爱人添上一抹魔法。（视频片段出处：Vivian Chow 周慧敏 - 《天荒愛未老》）_

## iMovie——视频合成

用 iMovie 制作视频合成是一个相对直接的过程，让你可以组合视频片段、添加转场、配入音乐并应用各种效果。

![AI_Song_iMovie_AI_Song_Editing_01]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_iMovie_AI_Song_Editing_01.jpg)

_图：影片片段与元素在 iMovie 中组合与剪辑。_

当你审视视频剪辑的时间线布局时，就像在窥视一个故事的骨架。时间线的上半部分是视频片段的居所。这些片段按线性顺序排列。它们在场景和主题上各不相同，形成视觉内容的马赛克。

视频片段下方，是一条贯穿时间线全长的绿色音轨。这条轨道是连续的，AI 生成的音乐在整支视频中播放。音轨中的波形显示音量与强度的变化，为叙事又添一层。

整个作品长约 2 分 03 秒。它混合了不同的场景与元素来构建叙事。用心的剪辑——片段搭配均衡、转场流畅——体现出打造一个动人音乐爱情故事的努力。


### 视频/音频同步

在歌唱音轨中让视频与音频同人声对齐，确实颇具挑战。

![AI_Song_iMovie_AV_Sync_01]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_iMovie_AV_Sync_01.jpg)

截图中的示例：
- 截图显示了视频片段，底部有一条音轨。
- 将音频波形（绿色轨道）中波峰与波谷的图案与视频片段可见波形中的相应点对齐。
- 使用播放头精确对齐视频与音频的起始点。

![AI_Song_iMovie_AV_Sync_02]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_iMovie_AV_Sync_02.jpg)

提示：
- 始终**戴着耳机监听播放**，以捕捉细微的同步问题。
- 你可以**静音视频片段的原始音轨**，以便专注于新加的人声轨道。
- 定期**保存项目**，防止编辑成果丢失。

## HandBrake——视频再压缩（可选）

用 [HandBrake](https://handbrake.fr/) 重新压缩视频，可以在几乎不牺牲质量的情况下显著减小 MP4 视频的文件体积。以下是操作的分步指南：

首先，打开 HandBrake。启动应用，准备载入视频。点击 "Open Source" 按钮，导航到你想压缩的 MP4 视频文件。选择文件并点击 "Open"。

接下来，选择预设。HandBrake 为不同设备和用途提供了多种预设。对于一般视频压缩，你可以使用 "Fast 1080p30" 预设或类似的其他适合你需求的预设。

现在，调整视频设置。确保分辨率符合你的需要。降低分辨率可以显著减小文件体积，但可能影响质量。用 "Constant Quality" 滑块调整压缩级别。一个好的起点是 RF 20-24；较低值（如 RF 18-20）产出更高质量和更大文件，较高值（如 RF 22-24）压缩更多、文件更小。帧率设为 "Same as Source"，除非你需要更改。

然后，选择目标位置。点击 "Save As" 区域的 "Browse" 按钮，选择压缩视频的目标文件夹。需要的话也可以重命名输出文件。

最后，开始压缩。点击 "Start Encode" 按钮开始压缩过程。HandBrake 会处理视频，根据视频文件的大小和复杂度，这可能需要一些时间。

HandBrake 完成编码后，前往目标文件夹查看新的压缩视频文件。播放它以确保质量达到你的预期。
    
![AI_Song_HandBrake_Video_Recompression_to_Smaller_Size]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_HandBrake_Video_Recompression_to_Smaller_Size.jpg)

高效压缩的技巧：
- **比特率：** 如果需要更多控制，你可以在 "Video" 标签下手动设置平均比特率。降低比特率会减小文件体积，但以质量为代价。
- **音频设置：** 用 AAC 之类的编解码器压缩音轨，比特率 128 kbps 左右通常足以满足大多数用途。
- **多次编码：** 考虑使用 "2-pass encoding"（二次编码）以在较低比特率下获得更好质量。该选项位于 "Video" 标签下。

## Youtube——视频发布（可选）

YouTube Studio 的布局确保了关于视频的所有必要信息——包括标题、描述、可见性和详细的编辑选项——都易于访问，让用户可以高效地优化和管理视频内容。

截图所示是 YouTube Studio 中的 "Video details"（视频详情）区块，展示了 YouTube 视频发布流程。 

- **视频详情区块：**
  - 标题：视频标题为 "四季的情话 - Creating a Love Song with AI"。
  - 描述：
    - 描述提供了视频内容的概览。它讲述了创作者写歌的梦想、借助 AI 技术实现这一目标的经过，并列出了制作这支音乐视频所用的各种工具与方法。
- **可见性与限制：**
  - 可见性：视频被标记为 "Public"（公开），意味着 YouTube 上的任何人都可以访问。
  - 限制：视频内容没有施加任何限制。

![AI_Song_Youtube_Video_Publishing]({{ site.baseurl }}/images/creating-a-love-song-with-ai/AI_Song_Youtube_Video_Publishing.jpg)

_图：这张截图展示了 YouTube Studio 中的 "Video details" 区块，用户可以在这里优化和管理视频内容。它突出了关键信息的易访问性，包括视频标题、描述、可见性设置和编辑选项——这些都是在 YouTube 上有效发布与推广内容的关键。_


## 结语

创作一首情歌和一支配套的音乐视频，一直是我的梦想。得益于 AI 技术的进步，这个梦想已成现实。Stable Diffusion、Hedra、FaceFusion 和 Suno 等工具的组合，让我得以打造一种独特的、个性化的爱的表达——把我的诗歌与最先进的 AI 生成视觉和音乐融合在一起。

这个项目向我展示了 AI 的惊人潜力：不仅在于创意事业，还在于让复杂任务对每个人都更易上手——无论其技术或音乐背景如何。通过整合这些工具，我得以克服自身的局限，创造出真正在情感层面引发共鸣的东西。

从写诗到在 YouTube 上发布成品视频的旅程，既充满挑战又颇有收获，一路上满是学习的机会与灵感的瞬间。我希望通过分享这个过程，能激励其他人在自己的创意项目中探索 AI 的可能性。
