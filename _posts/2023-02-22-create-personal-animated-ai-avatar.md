---
layout: post
title: 创建个人动画 AI 形象
date: 2023-02-22 12:00:00.000000000 -00:00
permalink: create-personal-animated-ai-avatar
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- Stable Diffusion
- AI Creativity
- AI Avatar
category: post
comments: true
image: images/create-personal-animated-ai-avatar/cover2x.jpg
images:
  cover: images/create-personal-animated-ai-avatar/cover.jpg
  header: images/create-personal-animated-ai-avatar/header.jpg
published: true
---

<!--excerpt.start-->
创建你的个人 AI 形象（avatar）可以是一种有趣又激动的方式，去探索尖端 AI 技术的能力。AI 形象是用人工智能（AI）工具和技术创建的某个人的数字表示。它可以呈现多种形态，例如 3D 模型、动画角色或虚拟助手。AI 形象在游戏、娱乐、虚拟助手和社交媒体等领域正日益流行。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/create-personal-animated-ai-avatar/Podcast_Personal_AI_Avatar.mp3
{% include open-embed.html %}
----

我们可能想创建 AI 形象的潜在理由包括：

- 游戏：我们可以创建一个形象在电子游戏中代表用户，给用户更个性化的游戏体验。
- 社交媒体：AI 形象可以充当社交媒体平台上独特而吸睛的头像。
- 虚拟助手：我们可以创建一个形象充当虚拟助手，响应语音指令，帮助用户处理预约安排或客户支持等任务。

本文将带你走完用最前沿的 AI 工具——Stable Diffusion、ChatGPT、ElevenLabs 和 D-ID——创建个人 AI 形象的流程。随后，我们还会考察创建动画聊天机器人系统所需的数据模型与类结构，以及用户输入如何被处理和分类以生成恰当的回应。

![动画 AI 形象创建流程]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/Avatar_AI_Animation_Process.jpg)
*图：图示动画 AI 形象的创建工作流。(1) 用 Stable Diffusion 生成形象图片 (2) 用 ChatGPT 生成文本 (3) 用 ElevenLabs 生成语音 (4) 用 D-ID 把形象图片与生成的语音组合起来。*

> 例如，看看我的个人 AI 形象实况：[Animated Personal AI Avatar Demo](https://youtube.com/shorts/OVKUMkwTTdI)

{% include youtubePlayer.html id="OVKUMkwTTdI" %}

*图：动画个人 AI 形象演示。*

要学习如何为 Stable Diffusion 训练个人模型，请阅读我们之前的主题文章：
- [Stable Diffusion Training for Personal Embedding]({{ site.baseurl }}/stable-diffusion-training-for-embeddings)
- [Dreambooth Training for Personal Embedding]({{ site.baseurl }}/dreambooth-training-for-personal-embedding) 

### 第 1 步：用 Stable Diffusion 生成形象图片

[Stable Diffusion](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 是一款最先进的 AI 工具，可用于生成高质量的人脸图像。要使用 Stable Diffusion，你需要提供一些起始参数，例如性别、年龄和发型，它会生成一张符合这些参数的逼真人脸图像。

要生成你的个人形象图，先按我们之前文章 [Dreambooth Training for Personal Embedding]({{ site.baseurl }}/dreambooth-training-for-personal-embedding) 所述构建个人嵌入。你可以自由组合参数，取决于你想创建什么样的形象。选好参数后，运行 Stable Diffusion 生成你的形象图。

提示词：`bennycheung` person 是我们个人嵌入模型的 token
```
bennycheung person as a fantasy character, ultra realistic, intricate, elegant,
highly detailed, 8k, digital painting, detailed background, trending on artstation,
smooth, sharp focus, illustration, in the style of wlop, greg rutkowski
```

负面提示词（Negative Prompt）：没错，我们需要告诉 Stable Diffusion 一大堆"不要做"！
```
(((wrinkle))), bread, hat, disfigured, kitsch, ugly, oversaturated, grain,
low-res, Deformed, burry, bad anatomy, disfigured, poorly drawn face, mutation,
mutated, extra limb, ugly, poorly drawn hands, missing limb, blurry, floating limbs,
disconnected limbs, malformed hands, bur, out of focus, long neck, long body, ugly,
disgusting, poorly drawn, childish, mutilated, mangled, old, surreal, text, blurry,
b&w, monochrome, conjoined twins, multiple heads, extra legs, extra arms,
fashion photos (collage:1.25), meme, deformed, elongated, twisted, fingers,
strabismus, heterochromia, closed eyes, blurred, watermark, wedding, group
```

![Stable Diffusion 生成形象示例]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/Stable_Diffusion_Generate_Avatar_Example_x700.jpg)

*图：使用 Stable Diffusion（经 AUTOMATIC1111 Web UI）从提示词生成形象图像。*

### 第 2 步：用 ChatGPT 生成文本

[ChatGPT](https://chat.openai.com/chat) 是一款强大的自然语言处理（NLP）工具，可基于给定提示词生成文本。要使用 ChatGPT，你需要提供提示词，它会生成承接该提示词的一段文字。

要为你的 AI 形象生成台词脚本，先选择一个有助于创建你想要对话的提示词。例如，你可以选"Introduce yourself"（自我介绍）或"Tell me about your interests"（说说你的兴趣）之类的提示词。选好提示词后，运行 ChatGPT 生成脚本。

![ChatGTP 写脚本示例]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/ChatGTP_Write_Script_Example.jpg)

*图：使用 ChatGTP 为形象演讲写脚本。*

### 第 3 步：用 ElevenLabs 生成语音

[ElevenLabs](https://beta.elevenlabs.io) 是一款尖端的 AI 工具，可基于给定文本生成逼真的语音。要使用 ElevenLabs，我们需要提供文本脚本，它会以自然、逼真的方式生成朗读文本的语音。我们可以依靠 ElevenLabs 提供的免费额度起步。

要为你的 AI 形象生成语音，先把我们用 ChatGPT 生成的脚本提供给 ElevenLabs。ElevenLabs 将用先进的深度学习技术生成与脚本匹配的语音。

![ElevenLabs 语音合成示例]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/ElevenLabs_Speech_Synthesis_Example.jpg)

*图：使用 ElevenLabs 设置声音和输入脚本，生成 MP3 音频文件*

#### 语音生成过程详解
生成逼真语音的过程涉及几个技术步骤：

**第 1 步：文本分析** - 首先用自然语言处理（NLP）算法分析输入文本，提取语法、语义和韵律等关键特征。这一分析有助于理解文本的结构与内容，包括语气、风格和要表达的情绪。

**第 2 步：声学建模** - 输入文本分析完成后，ElevenLabs 用深度学习技术基于文本训练一个声学模型。声学模型是一个神经网络，学习把文本映射到语音的相应声学特征，例如音高、音量和时长。

**第 3 步：语音合成** - 声学模型训练完成后，ElevenLabs 用合成算法生成相应的语音波形。合成算法用声学模型生成声学特征序列，随后用声码器（vocoder）转换为连续波形。声码器是一个神经网络，把声学特征转换为高度接近自然人声的高质量音频信号。

**第 4 步：后处理** - 最后，ElevenLabs 施加后处理技术，进一步提升生成语音的质量与自然度。这些技术包括滤波、降噪和变调等。

ElevenLabs 把先进的 NLP 算法、深度学习技术与信号处理相结合，从输入文本生成高质量、逼真的语音。该工具应用广泛，包括虚拟助手、有声书朗读、视频配音等。

### 第 4 步：用 D-ID 把形象图片与生成的语音组合起来

[D-ID](https://studio.d-id.com/) 是一款精密的 AI 工具，可用于把图像与语音组合，创建一个说话者的逼真视频。要使用 D-ID，我们需要提供图像和语音，它会生成图像中的人用给定语音说话的视频。我们可以依靠 D-ID 提供的免费额度起步。

要创建个人 AI 形象，先把你的个人形象图和我们用 ElevenLabs 生成的语音提供给 D-ID。D-ID 将用先进的计算机视觉与 NLP 技术，创建我们的 AI 形象用逼真声音说话的视频。

![D-ID 创建视频示例]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/D-ID_Create_Video_Example.jpg)

*图：使用 D-ID 上传生成的音频文件，并添加定制形象图像以生成视频*

#### 视频生成过程详解
把图像与语音组合创建逼真视频的过程涉及几个技术步骤：

**第 1 步：人脸检测与对齐** - D-ID 首先检测输入图像中的人脸并把它对齐到中性位置。该工具使用人脸关键点检测算法准确定位关键面部特征，如眼睛、鼻子和嘴——它们对在说话时生成逼真的面部运动至关重要。

**第 2 步：唇形同步** - 人脸对齐后，D-ID 生成逼真的唇部运动以与给定语音同步。该工具结合深度学习技术与计算机视觉算法，分析语音并生成与声音匹配的唇部运动。

**第 3 步：面部表情生成** - D-ID 还基于输入语音生成逼真的面部表情。它用 NLP 算法分析语音内容，生成与语音内容一致的情绪表达，例如微笑、皱眉或扬眉。

**第 4 步：视频渲染** - 最后，D-ID 把全部生成的元素——唇部运动、面部表情和语音——组合起来，创建一个说话者的逼真视频。该工具用先进的计算机图形学技术渲染视频，带来高质量画面与无缝的唇形同步。

D-ID 结合精密的 AI 算法与技术，基于图像和语音生成说话者的逼真视频。该工具可用于广泛的应用，例如创建个性化视频消息、为虚拟助手生成形象，或增强视频会议体验。

--------------
## 动画聊天机器人设计

如果我们想设计一个结合形象生成与语音生成技术与用户交互的聊天机器人，可以通过包含以下组件的系统设计来实现：

1.  **用户界面**：系统设计的第一个组件是用户界面，它允许用户与聊天机器人交互。可以是 Web 或移动应用，甚至可以是嵌入网站的聊天挂件。用户通过用户界面输入文本消息或语音指令，聊天机器人将处理并回应。
    
2.  **自然语言处理（NLP）**：系统设计的第二个组件是自然语言处理，用于理解用户的输入消息。NLP 组件将使用文本分析、情感分析和意图识别等技术，理解用户输入背后的含义。
    
3.  **形象生成**：系统设计的第三个组件是形象生成，它使用 Stable Diffusion 之类的工具创建聊天机器人的逼真图像。形象可以定制以契合聊天机器人的品牌或个性，甚至可以包含由 D-ID 生成的面部表情和唇部运动。
    
4.  **语音生成**：系统设计的第四个组件是语音生成，它使用 ElevenLabs 为聊天机器人生成逼真的语音。语音可以定制以契合聊天机器人的品牌或个性，甚至可以根据输入文本包含情绪语调和抑扬顿挫。
    
5.  **对话管理**：系统设计的第五个组件是对话管理，负责管理用户与聊天机器人之间对话的流程。该组件使用 ChatGPT 之类的工具，基于用户输入和当前对话上下文生成回应。
    
6.  **集成与部署**：系统设计的最后一个组件是聊天机器人的集成与部署。包括把所有组件集成为一个连贯的系统、测试和验证聊天机器人，并将其部署到生产环境。
    
一个结合形象与语音生成的聊天机器人的系统设计，会包含用户界面、NLP、形象与语音生成、对话管理以及集成与部署等组件。这些组件的组合将为用户创造高度个性化、引人入胜的体验，提升整体用户体验与满意度。

### 动画聊天机器人

进一步把聊天机器人功能做成用动画形象提供回应，其工作方式与纯文本（或纯语音）聊天机器人类似，但增加了聊天机器人动画化视觉表示这一元素。

当用户输入消息或指令时，聊天机器人会用自然语言处理（NLP）理解用户意图并生成恰当回应。回应随后被翻译为文本，作为生成聊天机器人形象动画回应的基础。

为实现这一点，聊天机器人会用 Stable Diffusion 之类的形象生成工具创建聊天机器人的逼真动画图像。形象可以定制以匹配聊天机器人的品牌或个性，并可包含一系列面部表情、动作和手势。

形象生成之后，它将被集成进聊天机器人的界面，用户在接收回应时会看到它。形象的动作、手势和表情将与聊天机器人生成的回应文本同步，为用户创造更引人入胜、更个性化的体验。

### 系统设计

从用户界面开始，消息或指令被传递给自然语言处理（NLP）组件，它分析输入以理解用户意图并生成回应。NLP 组件生成的回应被传递给形象生成组件，后者创建聊天机器人响应用户消息或指令的动画图像。

同时，NLP 组件生成的回应文本被传递给语音生成组件，后者用 ElevenLabs 生成聊天机器人的语音回应。形象与语音回应都被传回用户界面，用户可以在那里看到并听到聊天机器人的回应。

对话管理组件负责管理对话流程，确保聊天机器人的回应恰当且与当前上下文相关。最后，整个系统被集成并部署到生产环境，在那里它可以实时与用户交互。

![动画聊天机器人系统设计]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/Animated_Chat_Bot_System_Design.jpg)

*图：在此图中，用户界面（A）连接到 NLP 组件（B），后者又同时连接到形象生成组件（C）与语音生成组件（D）。形象生成组件与语音生成组件都把输出送回用户界面，对话管理组件（E）负责管理各组件之间对话的流程。最后，整个系统被集成并部署到生产环境（F）。*

### 系统数据模型

系统数据模型可以概括为下表，

| 数据模型 | 描述 |
| -----------|-------------|
| User Input |  用户输入的消息或指令
| Text Preprocessing |  NLP 组件处理用户输入：去除停用词、词干化或词形还原
| Intent Classification |  NLP 组件把用户输入分类为一个或多个意图类别
| Dialogue State |  对话的当前状态，包括上下文与历史
| Avatar Pose |  形象的当前姿态，反映其情绪与物理状态
| Dialogue Response |  聊天机器人生成的回应，用于答复用户请求
| Voice Output |  语音生成组件基于对话回应生成的音频回应
| Facial Expression |  形象的面部表情，反映其情绪与语调
| Lip Sync |  语音输出与形象唇部运动之间的同步
| Response |  聊天机器人的最终回应，包含形象姿态与语音输出

这一系统数据模型表示了聊天机器人创建与回应生成所涉及的各种组件和数据结构。用户输入是交互的起点，由文本预处理组件处理，后者对输入施加各种 NLP 技术，使其更易分析。

随后，意图分类组件把用户输入分类为一个或多个意图类别，用于确定恰当的回应。对话状态组件存储对话的当前状态——包括上下文与历史——并随每条新用户输入更新。

形象姿态组件存储形象的当前姿态，反映其情绪与物理状态。面部表情组件利用这些信息生成与语音输出的语调和情绪相匹配的面部表情。

对话回应组件生成对用户输入的回应，然后传递给语音输出组件。该组件基于对话回应生成音频回应，并与唇形同步组件保持同步，确保形象的唇部运动与音频匹配。

最后，回应组件把形象姿态与语音输出组合起来，创建展示给用户的最终回应。这些组件与数据结构共同创造了引人入胜、能响应用户需求的对话体验。

#### 类设计
下面是每个组件数据模型的类数据结构布局示例：

![动画聊天机器人系统类设计]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/Animated_Chat_Bot_System_Class_Design.jpg)

下面是一个时序图示例，描绘了用户输入如何影响上述类：

![动画聊天机器人系统时序图]({{ site.baseurl }}/images/create-personal-animated-ai-avatar/Animated_Chat_Bot_System_Seq_Diagram.jpg)

## 结语
本文中，我们探索了 AI 形象与聊天机器人的精彩世界。我们讨论了如何用 Stable Diffusion、ChatGPT、ElevenLabs 和 D-ID 等尖端 AI 工具与技术创建逼真且可交互的 AI 形象。我们探究了这些工具与组件如何协同工作，为用户创造引人入胜的对话体验。

我们还考察了创建动画聊天机器人系统所涉及的数据模型与类结构，以及用户输入如何被处理和分类以生成恰当回应。我们审视了生成形象面部表情与唇形同步所涉及的各个组件，以及与形象动作同步的语音输出。

我们已经看到，AI 形象与聊天机器人拥有彻底改变我们与技术交互方式的巨大潜力。随着 AI 技术的进步，我们可以创建更逼真、更引人入胜的形象，提升用户体验，提供与他人连接的新方式。可能性无穷无尽，我们可以期待未来几年这一领域出现更激动人心的发展。

## 参考文献

- Prompt Engineering, [Create Your Own AI Animated Avatar: A Step-by-Step Guide](https://www.youtube.com/watch?v=V2efVSXSlqc&ab_channel=PromptEngineering), Youtube, Feb 2023
  - 我们在这里学习了如何用尖端 AI 工具与技术创建 AI 形象。该 AI 形象使用了 MidJourney、ChatGPT、ElevenLabs 和 D-ID。

### 其他博文

- Benny Cheung, [Stable Diffusion Training for Personal Embedding]({{ site.baseurl }}/stable-diffusion-training-for-embeddings), Benny's Mind Hack, Nov 2022.
- Benny Cheung, [Dreambooth Training for Personal Embedding]({{ site.baseurl }}/dreambooth-training-for-personal-embedding), Benny's Mind Hack, Nov 2022.

### AI 工具

- 用于脚本创作：[ChatGPT](https://chat.openai.com/chat)
- 使用 Automatic1111 做图像生成：[Stable Diffusion](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- 用于音频生成：[Elevenlabs](https://beta.elevenlabs.io)
- 用于视频生成：[D-ID](https://studio.d-id.com/)
