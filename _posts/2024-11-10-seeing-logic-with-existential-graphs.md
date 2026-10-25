---
layout: post
title: 看见逻辑——存在图与视觉思维的力量
date: 2024-11-10 12:00:00.000000000 -00:00
permalink: seeing-logic-with-existential-graphs/
author: Benny Cheung
artwork:
  author: DALLE-3
tags:
- Visual Logic
- Knowledge Graph
- Existential Graph
- AI Education
category: post
comments: true
image: images/seeing-logic-with-existential-graphs/post_cover.jpg
images:
  cover: images/seeing-logic-with-existential-graphs/cover2x.jpg
  header: images/seeing-logic-with-existential-graphs/header.jpg
published: true
series: "Logic Puzzles"
series_order: 3
---

<!--excerpt.start-->
逻辑天生就难——不仅难以做对，甚至常常难以把握。抽象概念对大多数人来说并不自然，这正是视觉逻辑能够带来转变的原因。视觉逻辑让逻辑变得可见，架起抽象推理与直觉理解之间的桥梁。当逻辑以视觉方式呈现时，我们开始以更直觉的方式理解它，就像我们理解椅子或门这些物理对象一样。这很重要，因为理解逻辑不只是数学家或计算机科学家的小众技能；它从根本上决定了我们如何感知世界、与世界互动。
<!--excerpt.end-->

![视觉逻辑理解的循环]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_01_Cycle_of_Visual_Logic_Understanding.png)

*图：这个循环强调了视觉逻辑让抽象思想更易接近、更直觉的力量。*

----
> 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/seeing-logic-with-existential-graphs/Podcast_Seeing_Logic.mp3
{% include open-embed.html %}
----

## 查尔斯·桑德斯·皮尔士与存在图

把逻辑可视化的想法并不新鲜。19 世纪，哲学家与逻辑学家[查尔斯·桑德斯·皮尔士](https://plato.stanford.edu/entries/peirce/)（Charles Sanders Peirce）试图让抽象概念变得具体。他开发了一套名为**存在图**（existential graphs）的系统——用图形化格式描绘逻辑关系的图。通过在一张白纸上画逻辑，皮尔士致力于把逻辑变成可见之物，让抽象关系变得有形、可及。例如，在一个陈述周围画一个圈表示它的否定——"并非如此"。这种简单的视觉表示让人们能够*看见*逻辑运算，而不只是在头脑中解读符号。

![皮尔士的存在图]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_02_Peirce_Logic_Introduction.png)

*图：由皮尔士开发的存在图，通过逻辑的可视化让抽象概念得到具体的表示。*

皮尔士的存在图提供了以视觉方式表示逻辑陈述的途径。例如，要表达"天空不是蓝的"，可以在一个圈内写下"天空是蓝的"，圈代表否定。这种视觉方法让抽象逻辑变得具体，让人们看见逻辑运算，而不必在头脑中解读抽象符号。

虽然事后看来这显而易见，但皮尔士的做法在他的时代是创新的。通过把逻辑可视化，他证明了抽象概念在变得可见时会更加直觉。他的存在图把逻辑不可见的运作变成了大脑可以轻松理解的有形图示。

皮尔士强调*图像表示*（iconic representations）的力量：符号的结构与其所代表的概念相似，使其在直觉上更易理解。存在图体现了这一原则：以视觉格式结构化逻辑陈述，用不同类型的图表示逻辑的不同层级，即 Alpha 图与 Beta 图。

### Alpha 图：命题逻辑的可视化

Alpha 图是皮尔士存在图系统中最简单的形式，对应**命题逻辑**。在命题逻辑中，陈述被视为简单单元（命题），可以用"与""或""非"等逻辑运算组合，而不涉及具体实体或变量。Alpha 图用直截了当的约定在视觉上表示这些陈述及其组合：

- **字母或符号**代表单个命题。
- **称为"切"（cut）的封闭区域**用于表示否定。把一个命题放入一个切内即否定该命题，类似于传统符号逻辑中的"非"。

例如，考虑陈述"天没有下雨"。在 Alpha 图中，写下命题"天在下雨"，然后在它周围画一个切（闭合曲线）表示否定。

![Alpha 图记法]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_Alpha_Graph.jpg)

*图：此图展示了皮尔士 Alpha 图记法，它以图形格式表示命题逻辑。左栏是常见的逻辑陈述英文表述，中栏（F）显示它们在传统符号逻辑中的表示，右栏（EG）演示每个陈述如何用 Alpha 图表达。*

通过使用 Alpha 图，皮尔士提供了一种无需文本或符号操作即可将基本逻辑运算可视化的途径，让用户能够"看见"命题之间的关系。这使 Alpha 图成为表示命题逻辑的一种图像化、直觉化的方式。

### Beta 图：扩展到谓词逻辑

在 Alpha 图的基础上，Beta 图纳入了**谓词逻辑**（也称一阶逻辑），引入变量与对象之间的关系。谓词逻辑允许涉及具体实体及其属性或关系的更复杂陈述，例如"所有人都会死"或"苏格拉底是人"。在 Beta 图中，皮尔士引入了附加特性：

- **身份线**（lines of identity）代表特定的个体或对象，连接图的各个部分，表明两个命题指向同一实体。
- **谓词与量词**允许把"有些""所有"之类的表达以图形方式表示，使图能捕捉不同变量或个体之间的关系。

![1911 年解释的 EG 记法]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_notation_explained_1911.png)

_图：来源：John F. Sowa, [Existential Graphs - The simplest notation for logic ever invented](https://jfsowa.com/talks/egintro.pdf), 10 Jan 2020._

以下是一些例子：

1. **基本存在与关系**：
   - **陈述**："一只猫在垫子上。"
   - **表示**："Cat""On""Mat"三个词由线连接，构成一个简单图示，其中"Cat"经由关系"On"连接到"Mat"。
   - **解释**：这在视觉上表示一只猫的存在及其与垫子的关系。

2. **存在陈述**：
   - **陈述**："某个东西在垫子下面。"
   - **表示**：一条线把"Under"连接到"Mat"，不指明垫子下面是什么。
   - **解释**：这表示垫子下面存在某个东西，与谓词逻辑中的存在量词一致。

3. **否定**：
   - **陈述**："有些猫不在垫子上。"
   - **表示**："Cat On Mat"关系被围在一个切内以表示否定。
   - **解释**：这表示存在一只不在垫子上的猫。

4. **条件陈述**：
   - **陈述**："如果一只猫在垫子上，那么它是一只快乐的宠物。"
   - **表示**：前件"Cat On Mat"被围在一个切内，引向后件"Pet Happy"。
   - **解释**：围住的切表示如果条件满足（一只猫在垫子上），那么结果或后果（这只猫是快乐的宠物）随之成立。

![Beta 图示例]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_Beta_Graph_Existential_Quantifier.png)

*图：此图展示存在量词与全称量词在皮尔士 Beta 图中的表示——Beta 图是其面向谓词逻辑的存在图系统的一部分。*

通过这些视觉约定，Beta 图能够以图形格式表示复杂的逻辑结构，包括否定、存在量词和条件句。这一系统让用户无需完全依赖符号记法即可将逻辑关系可视化，使逻辑推理与关系更加直觉。

### 把存在图与现代知识图谱连接起来

皮尔士在其图中对互联性的强调表明：命题天然是关系性的，构成一张意义之网。这一思路与现代知识图谱的结构遥相呼应——在知识图谱中，意义派生自实体之间关系的网络。知识图谱——例如搜索引擎或 AI 应用中使用的那些——通过节点和边表示信息，与皮尔士的存在图非常相似。

他的影响延伸到 AI 与认知心理学等领域，启发了早期的语义网络以及 John F. Sowa 等研究者。Sowa 的概念图——语义网络的前身——反映了皮尔士的视觉逻辑。这些早期网络为今天的知识图谱铺平了道路，说明了皮尔士的思想如何影响了现代知识表示。

此外，皮尔士设计存在图就是为了以视觉方式支持逻辑推理。这一思想在现代知识图谱的推理能力中得到回响：知识图谱使用计算机制来促进推理与演绎，其根源于皮尔士的愿景。皮尔士融合哲学、逻辑与符号学的工作，预示了现代知识图谱的跨学科性质——后者汲取语言学、数据科学与计算机科学，以表示跨领域的知识。


## 人脑：为视觉理解而生

### 视觉思维的演化根源

我们的视觉思维能力深植于演化史，赋予我们处理视觉信息的先天优势。人类为视觉地理解世界而演化；我们不是为了阅读符号串或文本行而演化的。人类视觉皮层与其他灵长类相比显著更大，表明处理复杂视觉信息的能力更强。这一扩张促进了高级认知功能，包括语言与抽象思维。

例如，当走过森林、看到一根像蛇的树枝时，大脑迅速解读视觉图案，触发即时反应。这种快速的视觉处理凸显了大脑是如何被接来检测模式、理解形状、感知物体之间关系的。

### 视觉理解的应用

视觉表示是跨领域学习与理解的有效工具。例如，学习电流如何在电路中流动时，一张带电池、导线和灯泡的图让人直观看到电流如何流动、组件如何连接，比抽象方程更容易把握。

类似地，用图示解释家谱树，能让人立即理解那些仅用文字难以传达的关系。视觉逻辑正是利用这种视觉处理的天然禀赋，让抽象推理更具体。

[图片优势效应](https://thevisualcommunicationguy.com/2018/11/06/use-pictures-always-why-the-picture-superiority-effect-pse-matters/)是认知心理学中一个有据可查的现象：人们对图像的记忆比文字更有效。研究表明，人们即使在短暂 exposure 之后也能高精度地回忆图像，凸显了视觉记忆的力量。在教育环境中，图表等视觉辅助通过让复杂信息更易接近、更吸引人，从而增强学习。

## 拥抱复杂性：视觉逻辑的"杂乱"

虽然皮尔士在存在图上的工作具有创新性，但它并未立即获得广泛采用，部分原因是视觉逻辑可能显得杂乱而复杂。传统符号逻辑呈现的公式整齐排列，而视觉逻辑涉及绘制与连接元素，可能像繁复的蛛网。

然而，这种复杂性可以是特性而非缺陷。视觉逻辑的"杂乱"反映了真实世界问题的本性——它们常常复杂且相互关联。能够看到全景，连同所有细微之处与关系，可以带来更深的理解与新洞见。

例如，思维导图就是一种拥抱复杂性的视觉逻辑。在头脑风暴或规划项目时，人们常常创建向多个方向分支的思维导图。虽然这些图可能看起来混乱，但它们允许探索那些在单线格式中可能不明显的概念之间的连接。这种视觉探索可以带来线性思维可能错过的突破与创新解法。

通过拥抱视觉逻辑固有的复杂性，我们可以更好地驾驭复杂系统、体会思想之间的互联，最终增强问题求解与创造性思维。

## 教育中的视觉逻辑

视觉逻辑的潜在收益是巨大的，尤其是在教育中。抽象概念仅通过文本或符号呈现时，学生可能难以把握。通过让逻辑变得具体而可视，我们可以增强理解与参与度。

![让逻辑可及]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_03_Making_Logic_Accessable.png)

*图：让逻辑更易接近、更吸引人的不同视觉工具，包括课堂场景、视觉谜题、卡片与图示。*

想象一间教室：学生们用卡片和图示（而非黑板上的符号）解以视觉方式呈现的逻辑谜题。教师可以视觉化地呈现场景，例如展示电流在电路中的流动，或梳理一个故事中的关系。通过操纵视觉元素，学生可以直觉地理解否定或合取等逻辑运算。

例如，在计算机科学教育中，视觉逻辑图可以帮助学生理解算法如何工作。流程图把算法的步骤可视化，让人更容易跟随逻辑、发现潜在问题。

类似地，排查计算机网络故障时，把网络布局可视化可以帮助定位连接失败之处。网络图显示路由器、交换机和连接，让技术员看清组件之间的关系，更高效地隔离问题。


## 视觉语言的标准化

视觉逻辑的挑战之一是建立共同的视觉语言。如果每个人画存在图的方式不同，沟通的效率就会降低。不过，这个问题可以通过标准化解决——在保留视觉思维收益的同时，实现共享的理解。

纵观历史，人们发展了各种视觉语言来表示逻辑、数学关系或复杂系统。以下是一些突出的例子（更多信息见历史参考文献）：

| 视觉语言                     | 开发者                                         | 目的                                                     | 常见用法                                                        |
| ----------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------- |
| **Venn 图**                   | John Venn (1880)                                     | 表示集合论与逻辑关系              | 广泛用于集合论、概率与逻辑可视化     |
| **流程图与框图**   | 工程师、程序员（1940-50 年代）                   | 表示过程、工作流或算法               | 建模业务流程、编程、工程工作流     |
| **概念图**               | John F. Sowa（1970 年代）                                 | 在知识表示中表示复杂关系 | 用于 AI 与知识表示                             |
| **统一建模语言（UML）** | Grady Booch、Ivar Jacobson、James Rumbaugh（1990 年代）   | 为面向对象软件建模                              | 软件工程中为系统架构与交互建模  |
| **Petri 网**                      | Carl Adam Petri (1962)                               | 为分布式系统与并行过程建模            | 分析系统与计算机科学中的并发事件         |
| **语义网络**               | 多人，心理学与计算机科学（1960-70 年代） | 表示知识网络                                | AI、语言学与知识表示的基础       |
| **真值表与卡诺图**  | 多人                                              | 展示并化简布尔逻辑结果与表达式 | 数字逻辑设计、电路化简与逻辑演示 |
| **思维导图与概念图**      | Joseph D. Novak（1970 年代）                              | 组织联想性与层次性想法                 | 教育、头脑风暴、知识组织                    |

这些视觉语言说明了人类为让复杂系统可理解而持续的努力，每一种都为更丰富的视觉表示图景做出了贡献。它们在科学、教育与商业场景中被广泛采用，证明了视觉工具在化繁为简、增进理解方面的效力。

![表达"一只猫在垫子上"]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_How_to_say_cat_is_on_a_mat.png)

*图：随时间发展的、用形式逻辑表达简单陈述"一只猫在垫子上"的多种方法。展示了 Gottlob Frege (1879)、Charles Sanders Peirce (1885)、Giuseppe Peano (1895)、皮尔士的存在图（1897）、概念图（1976）以及 Common Logic 的 CLIP 方言的表示。*


## AI 在推进视觉逻辑中的角色

人工智能（AI）凭借其在模式识别与系统翻译方面的专长，有潜力革新视觉逻辑，让抽象概念比以往任何时候都更易接近。AI 可以协助生成逻辑陈述的视觉表示，把传统符号逻辑翻译成更易理解的视觉格式。

![AI 在视觉逻辑中的角色]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_04_AI_Role_in_Visual_Logic.png)

*图：总结了 AI 通过支持学习与表示的各个方面来增强视觉逻辑的角色。*

想象一个学生试图理解不同生物过程之间的关系。AI 可以生成一幅视觉图示，展示光合作用与呼吸作用如何关联，突出依赖与交互。AI 可以把这些复杂关系变得可见，帮助学生理解正在发生什么，而不会迷失在术语里。这正是视觉逻辑可以变得真正强大的地方——不只是理解既有知识的工具，更是探索新想法的途径。

但 AI 的问题在于：它有多大用处，取决于使用它的人。AI 可以帮助让视觉逻辑更易接近，但前提是我们教会人们如何视觉地思考。这要从学校开始——那里通常以最抽象的方式教授逻辑。

如果反过来，我们像教孩子画画那样教他们逻辑呢？从简单的开始：基本的形状与连接，然后逐步搭建。把它变成他们学习思考世界的方式的一部分。想象一间教室：孩子们用彩色图示梳理逻辑论证，就像拼一个故事。这样，逻辑就变得像搭积木或玩乐高一样自然。而且不只是孩子——成年人也一样。能有更多人视觉地思考，他们就能更清晰地看清究竟发生了什么。


## 教育之外的视觉逻辑

视觉逻辑在教育之外的潜力巨大。它可以改变我们沟通的总体方式。大多数争论难以跟随，是因为难以看见。如果我们能看见不同论证背后的逻辑，我们也许会有更好的对话——少一些自说自话，多一些理解对方真正在说什么。

![教育之外的视觉逻辑]({{ site.baseurl }}/images/seeing-logic-with-existential-graphs/EG_05_Visual_Logic_Beyond_Education.png)

*图：探索视觉逻辑如何延伸到教育场景之外，增强理解与沟通的各个方面。*

视觉逻辑可以延伸到我们的日常生活。**改进的理解**让人们看见论证的结构，带来更好的理解，减少误解。**视觉表示**用图示与 visuals 传达复杂想法，提供整体视图。**增强的沟通**转变复杂信息的分享方式，使其更易接近、更清晰。**结构化的论证**让前提与逻辑连接得以可视化，支持更有条理、更有说服力的推理。

想象一场商务会议：有人展示一个想法，不只是描述它，而是把它画出来——展示项目各部分如何连接、存在哪些依赖。突然，房间里的每个人都能看到全景，而不只是零碎片断。想象这样一个世界：你不只是听到某人的论证，还能看见它的结构——前提、连接、跳跃。

## 结语：让不可见变得可见

视觉逻辑把抽象推理转化为有形的表示，架起复杂思想与直觉理解之间的桥梁。通过让不可见变得可见，它让我们看见逻辑的底层结构，促进更好的理解、沟通与创新。

是时候驾驭视觉逻辑的力量，在所有部门增进理解了。通过采纳视觉化的逻辑与推理方法，我们可以赋能人们更清晰地思考、更有效地沟通、做出更有依据的决策。让我们把视觉逻辑当作重要的工具，在一个日益复杂的世界中培育清晰与洞见的文化。


## 参考文献

- [Charles Sanders Peirce](https://plato.stanford.edu/entries/peirce/), Stanford Encyclopedia of Philosophy, revision 11 Feb 2021.

- John F. Sowa, [Conceptual Graphs, Chapter 5 of the Handbook of Knowledge Representation](https://www.jfsowa.com/cg/cg_hbook.pdf), Elsevier, 2008, pp 213-237.

- John F. Sowa, [Existential Graphs - The simplest notation for logic ever invented](https://jfsowa.com/talks/egintro.pdf), 10 Jan 2020.
  - 这是一份演示幻灯片。

- John F. Sowa, [Language, Ontology, And the Semantic Web](https://jfsowa.com/talks/eswc.pdf), eswc20 slides, 1 Sep 2020.

- John F. Sowa, [Knowledge Graphs for Language, Logic, Data, Reasoning](https://www.youtube.com/watch?v=J9_kXpZAcQY), Ontolog Forum video, 12 Feb 2020.
  - John Sowa 关于知识图谱历史的演示，涉及其在语言、逻辑、数据、推理等方面的应用。Ontology Summit 2020 的一部分。幻灯片见 https://go.aws/2SkRNc3。

- Mark P. Mattson, [Superior pattern processing is the essence of the evolved human brain](https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2014.00265/full), Frontiers in Neuroscience, Vol.8, 21 Aug 2014. [https://doi.org/10.3389/fnins.2014.00265](https://doi.org/10.3389/fnins.2014.00265)

- Curtis Newbold, [Use Pictures. Always. Why the Picture Superiority Effect (PSE) Matters](https://thevisualcommunicationguy.com/2018/11/06/use-pictures-always-why-the-picture-superiority-effect-pse-matters/), The Visual Communication Guy, 6 Nov 2018.

- Team Varthana, [Importance of Visual Aids in Teaching and Why it is Preferred](https://varthana.com/school/why-visual-aids-are-the-most-effective-teaching-tool-today/), Varthana, 29 Dec 2022.


### 历史参考文献

- J. Venn M.A. (1880) I. [On the diagrammatic and mechanical representation of propositions and reasonings](https://www.cis.upenn.edu/~bhusnur4/cit592_fall2014/venn%20diagrams.pdf) , Philosophical Magazine Series 5, 10:59, 1-18, DOI: 10.1080/14786448008626877 To link to this article: http://dx.doi.org/10.1080/14786448008626877

- John F. Sowa, [Conceptual Structures: Information Processing in Mind and Machine](https://www.amazon.ca/Conceptual-Structures-Information-Processing-Machine/dp/0201144727), Addison-Wesley, 1984. ISBN: 978-0201144727

- Grady Booch, Ivar Jacobson, and James Rumbaugh, [The Unified Modeling Language User Guide](https://www.amazon.ca/Unified-Modeling-Language-User-Guide/dp/0321267974), Addison-Wesley, 2005. ISBN: 978-0321267979

- M. Quillian, [Semantic Memory](https://www.jimdavies.org/summaries/quillian1968.html), in M. Minsky (ed.), Semantic Information Processing, pp 227-270, MIT Press, 1968.

- Maurice Karnaugh, [The Map Method for Synthesis of Combinational Logic Circuits](https://web.archive.org/web/20170416232229/http://philectrosophy.com/documents/The%20Map%20Method%20For%20Synthesis%20of.pdf) (PDF). Transactions of the American Institute of Electrical Engineers, Part I: Communication and Electronics, 72 (5): 593–599. doi: [10.1109/TCE.1953.6371932](https://doi.org/10.1109%2FTCE.1953.6371932).
