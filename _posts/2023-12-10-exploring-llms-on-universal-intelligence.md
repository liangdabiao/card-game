---
layout: post
title: 借明斯基之眼探索 LLM 与通用智能
date: 2023-12-10 12:00:00.000000000 -00:00
permalink: exploring-llms-on-universal-intelligence/
author: Benny Cheung
artwork:
  author: DALL.E 3
tags:
- Philosophy
- Universal Intelligence
- LLM
- AI
category: post
comments: true
image: images/exploring-llms-on-universal-intelligence/cover2x.jpg
images:
  cover: images/exploring-llms-on-universal-intelligence/cover.jpg
  header: images/exploring-llms-on-universal-intelligence/header.jpg
published: true
series: "Philosophy of AI"
series_order: 2
---

<!--excerpt.start-->
在最近的世界科学节上，一场题为"[AI: Grappling with a New Kind of Intelligence](https://www.youtube.com/watch?v=EGDG3hgPNp8)"（AI：与一种新智能角力）的讨论提出了一种观点：AI 可以被看作一个不断演化的"存在"，而不仅仅是一件工具。这个想法把我带回到 1985 年 4 月《Byte》杂志上一篇迷人的文章——AI 先知[马文·明斯基](https://en.wikipedia.org/wiki/Marvin_Minsky)（Marvin Minsky）撰写的["Communication with Alien Intelligence"](https://web.media.mit.edu/~minsky/papers/AlienIntelligence.html)（与外星智能通信），它探讨了接触我们世界之外的心智的可能性。
<!--excerpt.end-->

明斯基论证说：智慧生命，无论源自何处，都面临相似的约束，因而会发展出可比拟的概念和问题求解方法。他讨论了资源管理的经济学和独特想法的稀疏性等原理，并暗示这些原理是普适的。明斯基还论及理解差异巨大的智能时可能存在的局限，并强调基本问题求解要素在智能通信中的重要性。

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/Podcast_Minsky_Universal_Intelligence.mp3
{% include open-embed.html %}
----

![智能之间的对话]({{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/Conversation_between_Intelligence.jpg)

_图：一幅隐喻人类智能与人工智能之间对话的插图。_

这一视角让我们得以重新思考大语言模型（LLM）的能力，以及如何把问题求解技术应用于当前的约束。

## AI 中的通用问题求解

AI 作为一种智能形态，有潜力与人类竞争——通过通用问题求解的原理发展其现有能力。

具体而言，AI 中的通用问题求解概念——尤其就 LLM 而言——可以包括：

- **模式识别**：这种探测并吸纳模式的能力是人类智能的核心属性。其有效性依赖于健壮的记忆架构。
- **语言理解**：对语言的理解涉及对词语与符号的解读，让 AI 或人类得以构建有意义的回应。
- **适应与学习**：在机器学习中，通过行为变化来适应新信息，是一种数据驱动、算法化的进步方式。

这些例子在人工与自然的问题求解形式之间建立了平行关系。

### LLM 与资源管理的经济学

沿着明斯基的讨论展开，LLM 可以被视为管理计算资源以优化问题求解的系统。这涉及平衡计算力、内存和数据处理能力的分配，以高效地处理和生成语言。理解 LLM 使用的资源管理策略，是理解其运行机制与效率的关键。

这涉及对有限资产的分配与利用做出明智决策，以高效有效地达成特定目标。智能系统——无论生物的还是人工的——常常必须在约束之内运作：无论是能源、时间、材料还是计算力。这种能力不只是节约资源，更是最大化资源的潜在影响——这是人类智能与人工智能共同的关键侧面。

例如，LLM 处理一个复杂句子时，必须理解上下文、维持对话的连贯，并生成相关的回应。这需要一种微妙的平衡：用足够的资源有效完成这些任务，又不能多到变得低效。在训练 LLM 的语境下，资源管理涉及有效利用数据集和计算力。充足的计算资源对处理庞大数据集和试验各种模型架构至关重要。然而必须指出：避免模型过拟合或欠拟合，主要取决于训练数据的质量与多样性、模型的适当复杂度，以及有效的正则化技术。

### 独特想法的稀疏性

"稀疏性原理"（The Sparseness Principle）为理解与独特想法生成相关的智能本质提供了框架。该原理主张：可能的计算结构之宇宙浩瀚无边，但能产生有意义输出的过程寥寥无几，从而导致真正独特想法的天然稀缺。

#### 明斯基关于稀疏性原理的实验

> 稀疏性原理：当相对简单的过程产出两个相似之物时，这两物往往相同！

![可能计算结构的宇宙]({{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/Universe_of_possible_computational_structure.jpg)

_图：可能计算结构的宇宙（图片来源：Byte magazine, April 1985）。_

这幅原始图示是明斯基稀疏性原理技术实验的视觉抽象，与 Alan Turing 于 1936 年描述的图灵机概念紧密相关。图示展示了一个实验的结果：其中所有可能的计算过程——以图灵机为象征——都被探索。

在图中：

- 每个 'X' 标记一个未能产生有意义行为的过程（图灵机）。这些过程要么立即停机，要么擦除了输入数据，要么陷入无尽循环而没有执行任何有意义的计算。
- 'A' 代表这些过程中一个小小的子集，它们都收敛到了相似的非平凡行为。它们执行了明斯基所称的"计数"运算——简单却非平凡的任务，可视为算术运算的前身。
- 'B' 标记了一个更复杂机器的出现，这种情形远为罕见。这台 B 机器代表了从 A 机器出发的复杂度演化跃迁。

明斯基的实验揭示：在浩如烟海的可能计算过程中，大多数是无产出或平凡的（那些 'X'），但稀稀疏疏的少数（那些 'A'）往往表现出相同或相似的有用行为。尽管起始规则各异，A 机器之间的这种一致性表明：某些计算路径存在内在的效率，它们会导向类算术基本功能的发展。

这幅插图隐喻地表明：在计算（或认知）发展的早期阶段，更简单、更高效的过程更容易涌现并被复制。

这个实验凸显了计算机科学与认知科学的一个基础概念：尽管可能的过程或想法多如天文数字，功能性的、高效的却很稀少，而且在不同系统之间趋于相似。这些高效过程构成更复杂操作的积木——无论在计算机器中还是在人类心智中——最终孕育出我们努力想要理解的那些高级智能形态。

### LLM 的局限

当我们考虑 LLM 生成语言与概念时，常常发现这些模型产出的东西似曾相识，或是既有想法的衍生。这是因为 LLM 的运作方式是解析并重新组合海量的人类语言数据集。它们不是在真空中创造，而是从编码在训练数据中的人类知识总池中取材。正如明斯基所言，独特想法的稀少源于能导致重大且独特结果的简单过程数量有限。就 LLM 而言，这意味着这些模型表面上的创造性，往往是重新混合或挪用既有信息的结果，而非从零开始创造全新的东西。

设想一个 LLM 写诗的例子。模型生成的诗句或许契合人类的诗歌结构，但它是在遵循学到的模式，而不是凭空变出新的诗歌表达形式。类似地，如果让 LLM 解一道数学题，它会应用从既有数学知识派生的算法和启发式。它的"解法"是既有方法的重新组合，而非开创性的数学发现。

在两个例子中，稀疏性原理都在起作用。LLM 伸进自己有限的规则集——它们的程序与训练数据——并抽出那些基于人类过往努力、在统计上可能成功的组合。这种迭代的、组合式的问题求解与内容生成方式，是明斯基所述原理的数字映照。输出受制于设计与训练的边界——虽然广大，却无法囊括人类创造力的真正无限。

在思考人工智能的潜力与边界时，这一理解至关重要。它凸显了 LLM 的一个基本事实：它们是强大的工具，能够模仿类人的创造力与问题求解，但其运作局限在人类创造者定义的一组有界可能性之内。稀疏性原理由此提醒我们：人工系统在复制人类思想的深度与原创性、以及真正的创造与创新的非凡性方面，存在固有的局限。

## 原因与从句

马文·明斯基关于 CAUSES（原因）与 CLAUSES（从句）的沉思——关乎与外星智能通信的可能性——为理解 LLM 的精妙之处提供了一面迷人的镜子。明斯基认为，我们把原因归于结果、把经验分格为对象符号、差异符号和原因符号的方法，不只是一场演化的偶然，而很可能是一条认知发展的必经之路。这意味着：如果外星智能经历了认知演化，它们可能在沟通与思维过程中拥有相似的基本结构，从而有可能让我们找到共同语言。

![AI 智能通信抽象]({{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/AI_Intelligence_Communication_Abstract.jpg)

人类语言使用从句结构的概念——让我们能从基本构件搭建复杂想法——可以视为智能的一种通用机制。LLM 处理与生成语言时，起作用的正是这一机制。然而，LLM 的理解与生成受限于喂养给它们的程序与数据，它们的"从句结构"由算法预先定义。虽然它们可以通过链接较简单的构件来模仿思维的复杂性，但它们并不像我们那样从本质上"理解"这些结构。

## 在文化语境中解释

把解析式的成分拆解与更整体论的视角相对照，会敦促我们超越人类中心的视角去思考智能与认知的本质。它提出了这样一个问题：我们把世界看成部分的趋势——LLM 也在模仿这一趋势——是否真的是处理信息最有效的方式。也许，在一切可想象想法的浩瀚之中，存在着某种智能形态——无论是外星的还是人工的——它们感知与理解世界的方式，与我们、以及我们当前 AI 系统的能力有着根本的不同。这一想法拓宽了我们在开发和与 LLM 交互时的思考范围，推动我们探索常规框架之外的可能性，并可能为 AI 的革命性进步铺平道路。

### AI 的咆哮：超越语言的解释

诚然，LLM 在自我解释方面扮演着重要角色，但必须指出：可解释性并不限于语言。在 AI 语境中，可解释性指理解并解读 AI 系统所做决策的能力。这可以通过多种技术实现，包括可视化、特征重要性分析等，而不仅限于生成人类可读的解释。

语言可以是解释事物的强大工具；但对实现 AI 可解释性而言，它并非总是必要或充分的。在某些情况下，可视化或其他非语言技术可能更有效地帮助用户理解 AI 系统是如何得出其决策的。


![狮子若能言的类比]({{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/Analogy_to_If_Lion_Can_Speak_02.jpg)

_图：这幅图像反映了路德维希·维特根斯坦的概念，作为人类与人工智能之间在不同维度上对话的隐喻。_

维特根斯坦的名言"如果狮子会说话，我们也无法理解它"，常被解读为对语言局限以及理解不同视角或思维方式之困难的评论。然而，在与 AI 沟通并实现 AI 可解释性的语境中，重要的是认识到：实现可解释性有许多不同的方式，语言只是其中之一。归根结底，AI 可解释性的目标是让用户获得理解并信任 AI 系统决策所需的信息，而无论采用何种具体技术。

## 结语

马文·明斯基关于理解不同智能之局限的论述，在考虑与非人类智能——如外星心智或 LLM 这样的人工实体——沟通时尤其切题。他提出：我们的认知过程——通过名词、动词和从句在语言中反映出来——并非任意，而是源自对普适问题的演化解决方案。这些认知结构让我们能从较简单的构件创建复杂的想法，这个过程可能是智能的普遍特征。

这表明：尽管潜在智能形态的思维过程千差万别，却可能存在一个共同的基础——扎根于对复杂信息的简化与结构化。因此，如果我们遭遇一种外星智能，它可能共享某些基本的认知策略，例如对对象和动作进行分类，这可能让沟通成为可能。反过来，如果它们的认知不把经验分格为离散单元，这将对相互理解构成重大挑战。这一推测延伸到 AI 领域：LLM 或许在其程序中模仿这些认知结构，从而弥合人类理解与机器理解之间的鸿沟。然而，真正新颖的想法与概念的稀疏性依然是一个挑战——无论人类智能还是机器智能，都在与复杂性的极限以及创新组合的有限性搏斗。


## 参考资料

- Marvin Minsky, [Communication with Alien Intelligence](https://web.media.mit.edu/~minsky/papers/AlienIntelligence.html), Byte Magazine, Apr 1985.
  - 发表于 _Extraterrestrials: Science and Alien Intelligence_ (Edward Regis, Ed.) Cambridge University Press 1985。亦发表于 Byte Magazine, April 1985。

![Byte_Magazine_198504_AI_on_Alien_Intelligence]({{ site.baseurl }}/images/exploring-llms-on-universal-intelligence/Byte_Magazine_198504_AI_on_Alien_Intelligence.jpg)

_图：《Byte》杂志原始文章中的插图，优雅地捕捉了与外星智能的沟通可能如何展开的精髓。这幅图从此印在我的脑海里（图片来源：Byte magazine, April 1985）。_ 

### 图书

- Wittgenstein, Ludwig, and Gertrude Elizabeth Margaret Anscombe. _Philosophical Investigations: The English Text of the Third Edition_. Vol. 1. Prentice Hall, 1973.

- Jobst Landgrebe & Barry Smith, [Why Machines Will Never Rule the World: Artificial Intelligence without Fear](https://www.amazon.ca/Machines-Will-Never-Rule-World/dp/1032309938), Routledge, 12 Aug 2022. ISBN: 978-1032309934
  - 本书的核心论证是：一种能够匹敌或超越人类智能的人工智能——常被称为人工通用智能（AGI）——出于数学原因是不可能的。它为这一主张给出了两个具体理由：
    - 人类智能是一个复杂动态系统——人脑与中枢神经系统——的能力。
    - 这类系统无法被数学建模到能在计算机内运行的程度。


### 会议

- World Science Festival, [AI: Grappling with a New Kind of Intelligence](https://www.youtube.com/watch?v=EGDG3hgPNp8), video, 24 Nov 2023.
  - 一种新型智能呼啸着冲进主流，激起狂喜的兴奋，也激起彻底的恐惧。与站在人工智能最前沿的领袖们一起，在思考机器的美丽新世界中探索可能的未来图景。
  - 在这场由 Brian Greene 主持的振奋人心的两小时讨论中，杰出的 AI 研究者 Yann LeCun、Sébastien Bubeck 和 Tristan Harris 深入探讨了人工智能的历史与现状，以及它充满希望的未来。他们凭借专业知识和富有远见的视角，就一种新型智能的出现给出了有据可依的推测。
  
### 大语言模型（LLM）

  - Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, L., & Polosukhin, I., [Attention is all you need](https://arxiv.org/abs/1706.03762), In Proceedings of the 31st International Conference on Neural Information Processing Systems, 2017.
    - 引入 Transformer 模型的开山之作，它是许多现代 LLM 的基础。
    
   - Andrej Karpathy, [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g&ab_channel=AndrejKarpathy), video, 22 Nov 2023.
      - 这是一小时面向大众的大语言模型导论：ChatGPT、Claude、Bard 等系统背后的核心技术组件。它们是什么、走向何方、与当代操作系统的类比、以及这一新计算范式的一些安全相关挑战。
      
  - [OpenAI's Research Blog on Language Models](https://openai.com/research/overview)
      - OpenAI 的文章与研究论文合集，讨论 LLM 的方方面面，包括 GPT-4。
