---
layout: post
title: AGI 中的空间推理——来自哲学视角的洞见
date: 2023-03-26 12:00:00.000000000 -00:00
permalink: spatial-reasoning-in-agi/
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- AI
- AGI
- Philosophy
- Spatial Reasoning
category: post
comments: true
image: images/spatial-reasoning-in-agi/cover2x.jpg
images:
  cover: images/spatial-reasoning-in-agi/cover.jpg
  header: images/spatial-reasoning-in-agi/header.jpg
published: true
series:
  - name: "Spatial Reasoning"
    order: 3
  - name: "Philosophy of AI"
    order: 6
---

<!--excerpt.start-->
空间理解确实是实现人工通用智能（AGI）的重要侧面。AGI 指的是机器在广泛的任务与领域上具备人类水平的智能。尽管人们相信高级大语言模型（LLM）——例如 [GPT-4 展现了某些 AGI 能力](https://arxiv.org/abs/2303.12712)——这些模型在解释需要空间推理技能的概念时可能遇到困难。
<!--excerpt.end-->

LLM 擅长在大规模上处理与生成文本；然而，当要表达那些涉及理解与操纵三维空间中物体之间视觉与空间关系的想法时，它们可能力不从心。这一局限的根源在于：空间推理技能很难单靠语言来传达。

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/spatial-reasoning-in-agi/Podcast_Spatial_Reasoning_AGI.mp3
{% include open-embed.html %}
----

![AGI 空间理解 SD 插图 01]({{ site.baseurl }}/images/spatial-reasoning-in-agi/AGI_Spatial_Understanding_SD_Illustration_01.jpg)

_图：一幅受 M.C. Escher、Salvador Dalí 和 Wassily Kandinsky 艺术风格启发的插图，代表 AGI（人工通用智能）需要空间推理与认知能力的概念。（图片来源：Stable Diffusion）。_

本文中，我们对现有 AI 系统在 AGI 上的局限表达不满，并提出一个纳入哲学思想的更广阔视角。通过考察那些研究过人类认知与空间推理的哲学家的著作，我们旨在探索新的研究方向，使我们更接近 AGI 的目标。我们主张：理解人类智能的底层原理，并把这些洞见纳入 AI 系统，可以带来更通用、更具适应性、空间推理能力更强的模型，最终推进 AGI 的事业。

### 为什么是空间推理？

空间推理是人工智能的一个子领域，让计算机能够基于自身位置理解周围环境。这包括识别环境中的物体，然后以实用的方式熟练地操纵它们。空间推理的应用包括导航、物体操纵和环境解读。它目前应用于 GIS、机器人和游戏等领域。

从个人体验角度理解，想象一个简单的谜题（见插图），它需要空间推理技能，我们可能很容易解开。我们能够回答这些问题：(1) 是否存在把所有零散拼块放进空间的解？(2) 我们如何最优地把所有拼块装进空间？

没有类人的、理解空间关系的先天能力，这类谜题对机器构成重大挑战。这类谜题可能涉及操纵物体、想象旋转或变换，或在复杂环境中导航。虽然人类通常能直觉地把握这些概念，但如果没有专门设计的能处理空间信息的表示、算法或方法，机器可能难以找到有效的途径来解决这些问题。

![空间推理实验宇宙]({{ site.baseurl }}/images/spatial-reasoning-in-agi/Spatial_Reasoning_Experimental_Universe.jpg)

*图：图示一个空间推理领域的示例。待回答的问题 (1) 是否存在解？(2) 我们如何填充空间？*

对工程而言，开发空间推理算法面临若干挑战，例如管理高维数据、处理带噪的传感器输入、以及达成实时性能。空间推理研究需要设计出更可靠、更高效的、适合机器使用的表示与算法。

开发具备更强空间推理能力的 AI 系统很重要：它们才能理解周围环境并对未来结果做出有依据的预测。然而，在深入这类空间推理系统工程的相关技术挑战之前，有必要先探索能为设计提供指引的哲学洞见。下一节中，我们将考察多位哲学家的贡献——他们的工作触及了人类空间推理与认知的本质。站在空间推理的哲学基础上，我们可以更有效地识别应当指导先进 AGI 系统开发的关键考量。

## 哲学与空间推理

理解我们的认知能力至关重要——正是这些能力让我们能够感知、加工空间信息并据以行动。虽然哲学可能不直接涉及工程，但它的变革性思想可以影响具备类人空间推理能力的 AGI 系统的开发原则。

![AGI 空间理解 SD 插图 02]({{ site.baseurl }}/images/spatial-reasoning-in-agi/AGI_Spatial_Understanding_SD_Illustration_02.jpg)

_图：受 M.C. Escher、Salvador Dalí 和 Wassily Kandinsky 风格启发——破碎的风景、不可能的结构、梦幻般的场景（图片来源：Stable Diffusion）。_

### 哲学家关于空间认知与推理的思考

以下是一些写过空间认知与推理的著名哲学家的例子：
> 他们的哲学著作见参考文献。

![关于空间认知与推理的哲学家（标注版）]({{ site.baseurl }}/images/spatial-reasoning-in-agi/Philosopher_on_Spatial_Cognition_and_Reasoning_annotated.jpg)

- 伊曼努尔·康德（Immanuel Kant）：在《纯粹理性批判》中，康德论证空间是一种先验直观，即它并非源于经验——而是塑造了我们的经验。他的思想强调：空间推理是人类认知的核心成分。

- 戈特弗里德·威廉·莱布尼茨（Gottfried Wilhelm Leibniz）：他提出了空间的关系观，主张空间不是独立实体，而是对象之间关系的集合。

- 昂利·庞加莱（Henri Poincaré）：对空间与几何哲学做出了重大贡献。他主张我们的空间理解深深扎根于我们与世界经验和互动，强调直觉与实践知识的作用。

- 昂利·柏格森（Henri Bergson）：他的"绵延"（duration）概念凸显了时间的重要性及其与空间认知的关系。通过理解人类如何感知与加工时间和空间，AGI 开发者可以更好地设计能处理动态环境、随时间适应变化的系统。

- 莫里斯·梅洛-庞蒂（Maurice Merleau-Ponty）：认为我们的身体在场及与世界的互动塑造了我们对空间的理解。这一视角可以启发开发纳入具身认知的 AGI 系统，让它们通过与环境的直接交互更好地进行空间推理。

- 阿尔弗雷德·诺斯·怀特海（Alfred North Whitehead）：以发展过程哲学最负盛名。他强调经验与知觉的重要性，表明我们对空间的理解是通过与世界的互动不断精炼的。

- 埃德蒙德·胡塞尔（Edmund Husserl）：现象学创始人。他探索了空间对象的本质，以及意识在感知与建构空间关系中的作用。

- 安迪·克拉克（Andy Clark）：以认知科学领域的工作著称，尤其是具身认知与延展认知。他对空间推理的贡献强调心智延展于大脑之外，纳入工具与外部资源（例如身体在与环境互动中的作用）。


### AGI 设计中有影响力的哲学概念

综合这些哲学家关于空间推理的哲学洞见，我们可以识别出 AGI 设计与架构的若干关键原则与考量：

| 原则 | 哲学家 | 描述 |
|-----------|----------------|-------------|
| 强调具身认知 | 莫里斯·梅洛-庞蒂、安迪·克拉克 | 设计 AGI 系统时，纳入身体及其与环境的互动在塑造空间理解与推理中的作用。
| 聚焦空间关系 | 戈特弗里德·威廉·莱布尼茨、埃德蒙德·胡塞尔 | 在 AGI 设计中优先理解对象之间的空间关系，承认空间是关系的集合而非独立实体。
| 发展直觉与实践知识 | 昂利·庞加莱、伊曼努尔·康德 | 设计能通过与世界经验和互动获得空间推理实践知识与直觉的 AGI 系统。
| 纳入知觉与经验 | 阿尔弗雷德·诺斯·怀特海、埃德蒙德·胡塞尔 | 构建 AGI 系统，通过与环境的持续互动、纳入知觉与经验，精炼其对空间和空间推理的理解。
| 培育适应性与动态性 | 昂利·柏格森、昂利·庞加莱 | 创建能处理动态环境、通过理解时间与空间认知的关系随时间适应变化的 AGI 系统。

以上是关于空间理解的哲学思想如何应用于 AGI 设计的概览。

这些洞见帮助我们体会空间推理的复杂性及其在认知过程中的核心地位。它强调具身认知、空间关系、直觉与实践知识、知觉与经验，以及适应性的重要性。把这些原则纳入 AGI 系统的设计，我们就能朝着开发更贴近人类空间推理能力之复杂性与丰富性的人工通用智能迈进。


## 改进 AGI 空间推理的途径

AGI 专家强调空间推理对实现人工通用智能的重要性，并指出：克服基于文本的 LLM 的局限需要多种方法的组合。这些方法包括：整合不同数据源的多模态学习、让系统能与环境交互的具身 AI、模拟环境中的强化学习、受人类认知启发的认知架构，以及实现跨空间任务泛化的迁移学习。采纳这些方法，AI 系统可以更好地表示和操纵空间信息，更接近实现 AGI。

![AGI 空间认知迁移 SD 插图 01]({{ site.baseurl }}/images/spatial-reasoning-in-agi/AGI_Spatial_Cognitive_Transfer_SD_Illustration_01.jpg)

_图：图示空间认知与空间推理能力从人类向 AI 的迁移，受 M.C. Escher、Salvador Dalí 和 Wassily Kandinsky 风格启发——没有空间能力，AGI 就不完整（图片来源：Stable Diffusion）。_

虽然目前没有任何 AI 系统完美体现空间推理哲学著作所述的全部原则，但有一些 AI 项目与系统在不同程度上纳入了其中一些原则。举几个例子：

- 机器人系统：许多机器人系统通过与环境交互、处理感知信息来执行导航、物体操纵和场景理解等任务，从而践行具身认知。这类系统的例子包括自动驾驶车辆、无人机，以及制造或研究环境中使用的机械臂。这些系统通常使用 [SLAM（同步定位与建图）](https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping)等技术及其他感知算法来处理传感数据并理解空间关系。

- 强化学习代理：强化学习（RL）是一种 AI 范式，让代理通过试错从与环境的交互中学习。RL 代理在学习导航和与环境交互的过程中，往往发展出直觉与实践知识。[OpenAI 的 Dactyl](https://openai.com/research/learning-dexterity) 和 [DeepMind 的 AlphaGo](https://www.bing.com/search?q=deepmind+alphago&cvid=d13cb6e8bd20484185fec61eb1a86428&aqs=edge.0.0l5.4738j0j4&FORM=ANAB01&PC=U531) 等项目分别把强化学习用于发展机器人操纵与对弈技能。

- 面向空间推理的神经网络架构：[卷积神经网络（CNN）](https://en.wikipedia.org/wiki/Convolutional_neural_network)和[图神经网络（GNN）](https://en.wikipedia.org/wiki/Graph_neural_network)是为处理空间关系与结构而设计的 AI 架构示例。CNN 已被广泛用于图像识别与计算机视觉任务，其设计本身就以分层方式处理空间信息。GNN 则能以图的形式显式建模空间关系，使其能够对复杂空间结构进行推理。

- 多模态感知：纳入多模态感知的 AI 系统旨在处理并整合来自不同来源——如视觉、听觉和触觉——的感知信息。这些系统可以融合来自多种模态的信息，形成对环境更全面的理解。例子包括自动驾驶车辆的 AI 系统，它们把摄像头的视觉数据与 LIDAR 及其他传感器的数据相结合。

尽管这些 AI 系统与项目纳入了部分所述原则并取得了重大进展，它们仍是专门化的、能力有限。或者，我们不禁要问：为什么不聚焦于直接增强 LLM 的空间推理能力？我们将在下一节深入探讨这一路径的细节。

### 增强 LLM 的空间推理能力

以下是通过提升大语言模型的能力来获得 AGI 空间推理能力的一些可能策略：

#### 增广训练数据

大语言模型在海量文本数据上训练，这些数据的质量与数量会显著影响其性能。

增广训练数据是向机器学习模型的既有训练集添加更多数据，以提升其在特定任务上表现的过程。就大语言模型与空间推理而言，增广训练数据可以包括：添加更多包含空间取向信息的文本数据，或其他空间相关的数据源，例如图像、地图和图表。

例如，如果要训练一个大语言模型理解城市中物体之间的空间关系，训练数据可以用描述该城市布局的文本数据增广，也可以加入城市的图像或地图。增广数据还可以包含空间取向事件或活动的描述，例如游行或建筑项目，帮助模型学习如何在动态环境中推理空间关系。

为空间推理增广训练数据的一个挑战是：空间关系可能很复杂，难以在文本中准确表示。不过，自然语言处理与计算机视觉的最新进展，已经使自动从文本和图像中提取空间相关信息的技术成为可能。另一个挑战是：某些任务可用的空间取向数据源可能有限，尤其是冷门或专门的领域。在这些情况下，研究者可能需要自建数据源，例如生成合成文本和图像，或向人类专家众包标注。

#### 纳入多模态输入

空间推理既涉及视觉输入也涉及语言输入，纳入图像或视频等多模态输入，可以帮助大语言模型更好地理解和推理空间关系。

纳入多模态输入是一种组合多种输入形式——文本、图像、视频或音频——以提升大语言模型在空间推理任务上表现的方法。

举例来说，当被要求回答一个空间推理问题时，纳入多模态输入的大语言模型可能获得一张图示两物体空间关系的图像，以及同一关系的文字描述。通过同时分析图像与文本，模型可以更有效地推理两物体之间的空间关系。

把多模态输入纳入大语言模型有不同的方法。一种是使用能处理多种输入形式的模型架构，例如带多个输入层的神经网络。另一种是使用称为注意力（attention）的技术，让模型根据与任务的相关性选择性地聚焦于输入的某些部分。它对空间推理任务尤其有用，因为它允许模型选择性地聚焦于最相关的视觉与文本信息。例如，根据任务，注意力可以聚焦于物体之间的距离或它们的朝向。

#### 在空间推理任务上微调

微调（fine-tuning）——把预训练模型适配到特定任务——可以提升大语言模型在该任务上的表现。

微调是一种技术：通过在新的标注样本集上更新参数，把预训练的机器学习模型适配到特定任务或领域。

该流程通常包括以下步骤：

1.  预训练：大语言模型在大规模文本语料（如 Wikipedia 或网页）上训练，学习一般的语言模式与结构。
    
2.  任务特定数据收集：收集或整理一组较小的、针对所关注空间推理任务的标注数据。该数据集包含任务的示例，例如空间关系的描述、空间相关的问答，或带有空间相关信息的图像与视频。
    
3.  微调：在任务特定数据上更新预训练模型，优化其参数以最小化损失函数——它度量模型预测输出与任务特定数据中真实标签之间的差异。微调过程调整模型参数，以更好地捕捉任务特定数据中的空间关系，从而提升其在空间推理任务上的表现。

微调的优点之一是：它无需大量重训就能提升模型在特定任务上的表现。不必从零开始，微调通过调整参数使预训练模型更好地捕捉空间推理任务的特定特征。微调还可以与其他技术结合，例如数据增广或纳入多模态输入，进一步提升模型在空间推理任务上的表现。

#### 开发新型架构

把语言处理与视觉注意机制相结合的模型架构，可以更好地整合语言输入与视觉输入。

一个把语言处理与视觉注意机制相结合的新型架构示例是 [VisualBERT](https://arxiv.org/abs/1908.03557) 模型。VisualBERT 基于 BERT 架构——一种流行于大规模文本语料上训练的预训练语言模型。VisualBERT 扩展了 BERT 架构以纳入视觉注意机制，允许模型选择性地关注输入图像的相关部分。

VisualBERT 的视觉注意机制在不同的粒度层级上运作，让模型既能关注图像的高层特征（如整体场景），也能关注低层特征（如特定物体或感兴趣区域）。这使模型能够在不同尺度上推理物体之间的空间关系。

另一个把语言处理与视觉注意机制相结合的新型架构示例是[视觉问答（VQA）](https://arxiv.org/abs/1903.00366)模型。VQA 模型旨在通过关注图像的相关部分并生成这些部分的自然语言描述来回答关于图像的问题。

### 结语

总之，空间推理是人类认知的一个关键侧面，在我们与周围复杂动态世界互动、理解并导航的能力中扮演重要角色。要开发展现类人智能的 AGI 系统，必须纳入能反映人类空间认知之丰富性与复杂性的空间推理能力。

借鉴那些为我们理解空间推理做出贡献的哲学家的洞见，以下几项关键原则可以为 AGI 设计与系统架构提供指引：

1.  强调具身认知，让 AGI 系统与环境交互并处理感知信息。
2.  聚焦空间关系，开发能够表示、学习并推理空间结构的算法。
3.  鼓励通过经验与环境互动发展直觉与实践知识，可能纳入无监督学习或强化学习技术。
4.  纳入知觉与经验，设计能处理并整合来自多种来源感知信息的 AGI 系统，形成对环境的全面理解。
5.  培育适应性与动态性，使 AGI 系统能够应对变化的环境与演进的空间关系。

尽管包括 LLM 在内的现有 AI 系统尚无一完全涵盖所有这些原则，但众多项目与系统已经通过采纳其中一些原则取得了进展。这些研究展示了把空间推理能力整合进 AI 系统的前景，最终让我们更接近人工通用智能的目标。未来，我们将撰写更多文章，深入探讨构建这样一个系统的技术细节。

## 参考文献

### 人工通用智能（AGI）

微软研究院最近发表的论文展示了 GPT-4 _令人惊讶的_ AGI 潜力：
-  Sebastien Bubeck, et. al., [Sparks of Artificial General Intelligence: Early experiments with GPT-4](https://arxiv.org/pdf/2303.12712.pdf), [arXiv:2303.12712](https://arxiv.org/abs/2303.12712), Microsoft Research, Mar 22, 2023.

有几本关于人工通用智能（AGI）的书值得一读。以下是一些最好的：
-   Nick Bostrom, [Superintelligence: Paths, Dangers, Strategies](https://www.amazon.ca/Superintelligence-Dangers-Strategies-Nick-Bostrom/dp/0198739834), Oxford University Press, Apr 2016. ISBN:978-0198739838 
-   Stuart Russell, [Human Compatible: Artificial Intelligence and the Problem of Control](https://www.amazon.ca/Human-Compatible-Artificial-Intelligence-Problem/dp/0525558632), Penguin Books, Nov 2020. ISBN:978-0525558637

AGI 的经典图书（虽然非常老了）：
-  Goertzel, Ben, and Cassio Pennachin, [Artificial General Intelligence (Cognitive Technologies)](https://www.amazon.com/Artificial-General-Intelligence-Cognitive-Technologies/dp/3642062679), Vol. 1. Springer, 2007. ISBN:978-3642062674

### 空间推理
- Sabine Hossenfelder, [I believe chatbots understand part of what they say. Let me explain](https://youtu.be/cP5zGh2fui0?t=641), video, Mar 2023.
  - 当她描述"What's goes up must come down"的例子，以及"Toronto CA vs Windsor UK 纬度"位置的纬度例子时，两个例子都图示了 ChatGPT 缺失空间关系理解这一重要侧面。它没有世界的空间模型。

- Renz, Jochen. [Qualitative Spatial Reasoning with Topological Information](https://www.amazon.ca/Qualitative-Spatial-Reasoning-Topological-Information/dp/3540433465). Vol. 1. Springer, 2002. ISBN: 978-3540433460
- Aiello, Marco, Ian Pratt-Hartmann, and J. F. A. K. van Benthem. [Handbook of Spatial Logics](https://www.amazon.ca/Handbook-Spatial-Logics-Marco-Aiello/dp/1402055862). Vol. 1. Springer, 2007. ISBN: 978-1-4020-5586-7

### 关于空间认知与推理的哲学

伊曼努尔·康德的哲学著作：
-   [Critique of Pure Reason - Wikipedia](https://en.wikipedia.org/wiki/Critique_of_Pure_Reason)
-   [Kant's Account of Reason - Stanford Encyclopedia of Philosophy](https://plato.stanford.edu/entries/kant-reason/)

戈特弗里德·威廉·莱布尼茨的哲学著作：
-   [Gottfried Wilhelm Leibniz (Stanford Encyclopedia of Philosophy)](https://plato.stanford.edu/entries/leibniz/)
-   [Gottfried Wilhelm Leibniz - Wikipedia](https://en.wikipedia.org/wiki/Gottfried_Wilhelm_Leibniz)

昂利·庞加莱与空间推理相关的哲学著作：
-   [Science and Hypothesis](https://en.wikipedia.org/wiki/Science_and_Hypothesis)
-   [The Foundations of Science](https://en.wikipedia.org/wiki/The_Foundations_of_Science)
-   [The Value of Science](https://en.wikipedia.org/wiki/The_Value_of_Science)

昂利·柏格森与空间推理相关的哲学：
- [Matter and Memory](https://literariness.org/2018/01/06/the-philosophy-of-henri-bergson/)
- [Creative Evolution](https://en.wikipedia.org/wiki/Henri_Bergson)

莫里斯·梅洛-庞蒂与空间认知和推理相关的哲学著作：
-   [Phenomenology of Perception](https://en.wikipedia.org/wiki/Phenomenology_of_Perception)
-   [The Visible and the Invisible](https://en.wikipedia.org/wiki/The_Visible_and_the_Invisible)
-   [The Structure of Behavior](https://en.wikipedia.org/wiki/The_Structure_of_Behavior)

阿尔弗雷德·诺斯·怀特海与空间认知和推理相关的哲学著作：
-   [The Concept of Nature](https://en.wikipedia.org/wiki/The_Concept_of_Nature)
-   [Process and Reality](https://en.wikipedia.org/wiki/Process_and_Reality)
-   [Modes of Thought](https://en.wikipedia.org/wiki/Modes_of_Thought)

埃德蒙德·胡塞尔与空间推理相关的哲学著作：
-   [The Phenomenology of Internal Time-Consciousness](https://en.wikipedia.org/wiki/The_Phenomenology_of_Internal_Time-Consciousness)
-   [Ideas Pertaining to a Pure Phenomenology and to a Phenomenological Philosophy](https://en.wikipedia.org/wiki/Ideas_Pertaining_to_a_Pure_Phenomenology_and_to_a_Phenomenological_Philosophy)
-   [The Crisis of European Sciences and Transcendental Phenomenology](https://en.wikipedia.org/wiki/The_Crisis_of_European_Sciences_and_Transcendental_Phenomenology)

安迪·克拉克与空间认知和推理相关的哲学著作：
-   [Embodied Cognition and the Magical Future of Interaction Design](https://www.youtube.com/watch?v=QXvoYGrp-lA)
-   [Natural-Born Cyborgs: Minds, Technologies, and the Future of Human Intelligence](https://mitpress.mit.edu/books/natural-born-cyborgs)
-   [Being There: Putting Brain, Body, and World Together Again](https://mitpress.mit.edu/books/being-there)

### 空间推理相关文章

- Benny Cheung, [Spatial Reasoning Explained]({{ site.baseurl }}/spatial-reasoning-explained), Benny's Mind Hack, Jun 2016.
- Benny Cheung, [Model of Spatial Construction]({{ site.baseurl }}/model-of-spatial-construction), Benny's Mind Hack, Jul 2016.
- Benny Cheung, [Geospatial Granular Computing]({{ site.baseurl }}/geospatial-granular-computing), Benny's Mind Hack, Dec 2018.
