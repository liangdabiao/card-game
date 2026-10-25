---
layout: post
title: 破解 Q* 的魔咒——问题求解的新方法
date: 2023-11-28 12:00:00.000000000 -00:00
permalink: q-star-new-method-in-problem-solving
author: Benny Cheung
artwork:
  author: DALL.E 3
tags:
- AI
- Algorithms
- Q-Learning
- LLM
- Problem-Solving
category: post
comments: true
image: images/q-star-new-method-in-problem-solving/cover2x.jpg
images:
  cover: images/q-star-new-method-in-problem-solving/cover.jpg
  header: images/q-star-new-method-in-problem-solving/header.jpg
published: true
series: "Philosophy of AI"
series_order: 1
---

<!--excerpt.start-->
在 AI 与计算问题求解这个动态多变的领域里，出现了一个新的推测性却引人入胜的构想：**Q\***（读作 "Q-star"）——这一概念框架被设想为位于三种不同却相互关联的技术的交汇点：A* 算法、Q-learning，以及作为压缩知识系统的大语言模型（LLM）。尽管仍处于理论阶段，Q* 的这种融合暗示了一种协同效应，有望突破 AI 问题求解效率与效果的边界，为应对复杂挑战提供一种全新的方法。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以将本文当作一期**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主播既风趣又富有洞见。

{{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Podcast_Spell_of_QStar.mp3
{% include open-embed.html %}
----

> 免责声明：虽然我们对 Q* 概念的解读很可能与最近 [路透社的新闻报道](https://www.reuters.com/technology/sam-altmans-ouster-openai-was-precipitated-by-letter-board-about-ai-breakthrough-2023-11-22/) 中提到的 OpenAI 神秘突破并不一致，但这篇调查性文章为解决我们自己的 AI 问题提供了极大的启发。


![Q-Star 技术交汇示意图]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Q-Star_intersection_of_technologies.jpg)

_图：概念 Q\* 的抽象表示，融合了 A* 搜索、LLM 记忆和强化学习中的 Q-learning 方法等元素_

A* 算法以其强大的搜索能力而闻名，长期以来一直是在复杂空间中导航的首选方法，无论是游戏环境还是机器人路径规划皆是如此。它基于启发式的方法能够对最有效的路径做出有根据的猜测，从而大幅减少到达目的地所需的时间和计算资源。然而，A* 的静态特性限制了它适应动态环境或从过往经验中学习的能力。

接下来登场的是 Q-learning，这是一种强化学习形式，特别擅长以适应性和基于经验的决策为关键的环境。Q-learning 允许智能体从其行动的后果中学习，调整自身策略以最大化累积奖励。这种学习能力使它成为 A* 的理想搭档，为该算法稳健的搜索机制带来了动态、自适应的优势。

Q\* 的第三根支柱——作为压缩知识系统的 LLM——为这个等式引入了新的维度。以 GPT-4 等系统为代表，这些模型标志着处理与加工海量信息方式上的范式转变。通过把全局知识压缩到一个概率模型中，LLM 提供了一种 Q* 可以利用的独特记忆系统形式。这种整合使 Q* 不仅能够在环境中导航并从环境中学习，还能利用丰富的知识来支撑和增强其决策过程。

Q* 在高级计算问题求解中的潜在影响是巨大的。从在复杂的数据版图中导航，到解决错综复杂的现实世界问题，Q* 有望重新定义 AI 可能性的边界。它将高效搜索、自适应学习与深度压缩知识结合起来的能力，为探索（搜索）与利用（知识）开辟了新的途径。

在本文的后续内容中，我们将探讨 Q\* 的各个组成部分、它们如何协同互动，以及这一新方法的实际应用。请与我们一起踏上深入 Q* 核心的旅程——它是智能体时代问题求解未来的一座灯塔。

> 为什么将该方法记作 Q\*？
>
> 在 Q-learning 方法中，"Q" 代表 "quality"（质量），表示在某个特定状态下采取特定行动的有效性或价值。从本质上讲，它是衡量在特定情境中采取某一行动之可取性或适宜性的指标。此外，"Q\*" 还具有象征意义，代表在所有可能搜索策略中可达到的最高 Q 值，意味着应当采取的最优行动。另一种解释是，"\*" 可以来自 A\* 算法，它为解空间提供最优搜索策略。

## 背景知识

### A* 算法

Q* 的核心是 [A* 算法](https://www.redblobgames.com/pathfinding/a-star/introduction.html)，它是搜索与导航领域一个广为人知的 AI 工具。A* 是一种基于启发式的算法，这意味着它通过有根据的猜测来寻找两点之间的最短路径。这一方法使它有别于暴力搜索方法，能够高效地解决复杂的搜索问题。

![A* 算法动画]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/A-Star_Algorithm_Animated.gif)

_图：A* 算法寻找最优路径的动画（图片来源：imgur 用户 [comacomacomacomachameleon](https://imgur.com/user/comacomacomacomachameleon "View profile of comacomacomacomachameleon")）_

要理解 A*，让我们从该算法的关键特性开始：

- **启发式函数（Heuristic Function）**：A\* 高效率的核心在于其启发式函数。该函数估计从给定点到达目标的代价，通常使用直线距离之类的度量。这一估计帮助 A* 将搜索聚焦于朝向目标的方向，显著加快了求解过程。

- **代价计算（Cost Calculation）**：A* 计算两种代价——$g$ 代价，即从起点到当前点的已知代价；以及 $h$ 代价，即由启发式给出的从当前点到目标的估计代价。总代价 $f$ 是二者之和（$f = g + h$）。这一计算确保 A* 同时掌握已走过的路程和前方的旅程。

- **路径优先级（Path Prioritization）**：A* 持续跟踪多条路径，并根据它们的 $f$ 代价排定优先级。$f$ 代价最低的路径会被优先探索（与 BFS 或束搜索非常相似），确保算法始终沿着最有希望通往目标的路径前进。

### Q-Learning 详解

在 Q* 中与 A* 算法互补的是 [Q-learning](https://www.assemblyai.com/blog/reinforcement-learning-with-deep-q-learning-explained/)，它是强化学习（Reinforcement Learning, RL）的一种。与遵循预定义规则的传统算法不同，Q-learning 使智能体能够通过与环境的试错交互来学习最优行动。

![Q-learning 与深度 Q-learning 对比]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Q-learning_vs_Deep-Q-learning.jpg)

_图：Q-Learning 与 Deep Q-Learning 的对比。（图片来源：Assembly AI, Reinforcement Learning With (Deep) Q-Learning Explained）。_

适应性与经验学习：

- **从环境中学习（Learning from the Environment）**：在 Q-learning 中，智能体通过采取行动并观察结果来学习。这一过程帮助智能体理解在不同情境下哪些行动能带来最佳结果。

- **适应（Adaptation）**：Q-learning 的关键优势在于其适应性。它可以根据环境的反馈调整策略，这使它在动态或不可预测的环境中非常有效。

我们不用 RL 方程，而是用以下实际步骤来解释 Q-learning。从本质上讲，"Q" 值衡量的是在特定情境下采取某个行动有多好。

1. 智能体从一张 Q 表开始，它就像一张随学习不断填写的备忘表。这张表帮助智能体记住哪些行动带来奖励、哪些带来惩罚。
2. 起初，智能体一无所知，因此 Q 表里全是零或随机值。它开始探索迷宫，随机选择行进方向。
3. 智能体每走一步，都会从环境中获得反馈：好的行动得到奖励，差的行动受到惩罚。
4. 智能体用这些新信息更新 Q 表。它使用一个公式，该公式综合考虑当前奖励、预测的最大未来奖励以及 Q 表中的现有值。这一更新旨在改进智能体对 "在特定状态下采取某一行动有多好" 的猜测。
5. 随着时间的推移，经过足够的探索与学习，Q 表中会填满反映每个状态下最佳行动的值。
6. 最终，智能体在决策时更多地依赖 Q 表而非随机选择。它开始根据所学内容选择奖励最高的路径。

目标是让机器人最终得到一张 Q 表，告诉它在任何情况下应采取的最佳行动以最大化奖励，也就是说，找到穿过迷宫而不撞墙的最快路径。

### 作为压缩知识系统的 LLM

Q* 的第三个组成部分是[大语言模型（LLM）](https://www.youtube.com/watch?v=zjkBMFhNj_g&ab_channel=AndrejKarpathy)的使用。GPT-4 等 LLM 代表了机器处理海量信息方式上的突破。

![LLM 神经网络预测下一个词的原理]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/LLM_Neural_Network_predict_next_word_explained.jpg)

_图：此图直观地解释了 LLM 神经网络如何通过调用预训练中压缩的知识来工作。在本例中，给定 4 个单词的上下文 "cat sat on a"，模型以 97% 的置信度预测下一个单词是 "mat"。（图片来源：Andrej Karpathy, [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g&ab_channel=AndrejKarpathy)，视频，2023 年 11 月 22 日）。_

压缩知识与概率预测：

- **编码海量信息（Encoding Vast Information）**：LLM 压缩了海量文本语料中广泛存在的信息、模式与关系。这种压缩是一种有损的信息存储形式——模型捕获的是数据的本质而非确切的细节。

- **基于概率的预测（Probability-Based Predictions）**：LLM 使用概率模型进行预测。例如在生成文本时，LLM 会根据给定上下文中下一单词出现的可能性进行预测。这种方法使 LLM 尽管其知识压缩是有损的，仍能生成连贯且符合上下文的回应。

对 A\*、Q-learning 和作为知识系统的 LLM 的这一基础性理解，为探索它们在 Q\* 中的协同整合奠定了基础。

## A\*、Q-Learning 与 LLM 的融合

在 Q* 方法中将 A\*、Q-learning 与大语言模型（LLM）整合在一起，标志着计算问题求解的一个重大飞跃。本节将解释这三个组件如何协同工作，创造出一个整体大于部分之和的系统。

### 对 Q* 的协同贡献

- **A\* 用于高效搜索**：A* 为 Q* 带来了在复杂路径中导航的无与伦比的效率。其启发式方法能快速识别最有希望的路径，显著降低计算开销。

- **Q-Learning 用于动态适应**：Q-learning 增加了一层适应性与从经验中学习的能力。当条件变化或出现新信息时，Q-learning 使 Q* 能够调整策略，随着时间推移优化出更好的结果。

- **LLM 作为知识库**：LLM 充当庞大的知识宝库。它们为 Q* 提供对世界的广泛理解，提供算法当前环境中未显式编码的背景与洞见。

### 提升问题求解效率：假设性示例

想象一辆使用 Q\* 在繁忙城市中行驶的自动驾驶汽车。A* 引导汽车穿行于街道之间，高效地计算路线。然而，城市是动态的——道路会封闭，交通模式会变化。这时 Q-learning 就派上了用场，它使汽车能够从这些变化中学习并调整其路线规划。与此同时，LLM 协助理解复杂的交通法规或解读意外出现的道路标志，提供超越即时感官输入的知识深度。

在另一个场景中，设想一个使用 Q* 进行医学诊断的医疗 AI。A* 可以快速筛选症状以形成初步诊断，而 Q-learning 则会根据患者结局和新兴医学研究对这些诊断进行调整。LLM 则提供全面的医学知识库，确保 AI 的建议植根于最新的医学文献。

### LLM：Q* 的记忆系统

LLM 在 Q* 中的角色类似于一个精密的记忆系统。与保留静态信息的传统记忆存储不同，LLM 提供动态且富含上下文的知识。LLM 的这一特性使 Q* 不再局限于单纯的数据检索；它能够生成洞见、预测结果，甚至提出创造性的解决方案。本质上，LLM 赋予 Q* 以细腻且近乎人类的方式理解和处理信息的能力，极大扩展了它的问题求解能力。

接下来，我们将讨论如何把 Q* 集成到思维树（Tree of Thoughts）规划框架中。


## 思维树（Tree of Thoughts, ToT）框架中的 Q*

### 走进思维树

思维树（Tree of Thoughts, ToT）框架代表了在大语言模型（LLM）用于复杂问题求解方面的一个重大进步。ToT 的核心是一种结构化方法，它使语言模型能够生成、评估并迭代一系列 "想法" 或 "思想"，每一个都通向给定问题的一个潜在解。这一过程类似于构建一棵树，其中每个分支代表一条不同的推理路线或探索路径。

> 有趣的是，[Tree of Thoughts: Deliberate Problem Solving with Large Language Models]([arXiv:2305.10601](https://arxiv.org/abs/2305.10601)) 论文的 "Related Work"（相关工作）一节提到，ToT 是 A* 算法的一种现代演绎，其中每个搜索节点处的启发式由 LLM 的自我评估提供。


![Tree of Thoughts 论文]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Tree_of_Thoughts_paper.jpg)

_图：示意性地展示使用 LLM 进行问题求解的多种方法。每个矩形框代表一个想法（thought），即作为问题求解中间步骤的一段连贯语言序列。（图片来源：Yao, L., et al., Tree of Thoughts: Deliberate Problem Solving with Large Language Models, 2023）_

ToT 的关键要素：

- **结构化问题求解（Structured Problem-Solving）**：ToT 将问题求解过程组织成树状框架，其中每个节点代表一个独特的想法或思路。

- **搜索算法（Search Algorithms）**：为了遍历这棵树，需要使用深度优先搜索（DFS）、广度优先搜索（BFS）和束搜索（beam search）等搜索算法。这些算法有助于系统地探索不同路径，使模型在得出解之前能够考虑各种可能性。

- **LLM 的角色（Role of LLMs）**：LLM 拥有庞大的知识库，在生成每个想法的内容方面发挥着关键作用。它们提供必要的背景与信息，引导 ToT 框架内的探索与评估过程。

### 将 Q* 集成到 ToT 中

将 Q* 集成到 ToT 框架中，为增强决策与问题求解能力开辟了令人兴奋的新可能。

在 ToT 中构思 Q*：

- **用 A* 与 Q-Learning 增强搜索**：Q* 中的 A* 组件可用于高效遍历 ToT，识别最值得探索的想法或分支。与此同时，Q-learning 让系统能够从树中先前的探索中学习，并根据不同路径的结果随时间调整策略。

- **动态适应与学习**：随着条件变化或遇到新信息，Q* 使 ToT 能够动态调整其方法，确保探索始终保持相关且有效。

LLM 在 ToT 中的贡献：

- **增强的知识库**：LLM 贡献了对问题背景的深刻而细腻的理解，丰富了 ToT 中生成的想法。这可以带来更明智的决策，以及对潜在解更广泛的探索。

- **上下文相关性与创造性**：有了 LLM，ToT 不再只是在一组静态的想法中导航，而是能够生成新颖的想法和解决方案，并针对手头问题的具体细微之处量身定制。


![Q-Star 在 ToT 下的运行]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Q-Star_Operation_Under_ToT.jpg)

_图：该图概括了在思维树（ToT）框架内应用 Q\* 方法的抽象系统，展示了一种精巧的问题求解途径。_

从图中可以看到，系统接收输入并生成一棵 ToT。A\* 启发式估计器引导在这棵树上的探索，而 LLM 组件则以深入的、上下文敏感的知识丰富这一过程。Q-learning 部分从每次迭代中动态学习，增强系统适应并随时间改进策略的能力。输出则是基于这一整合过程被认为最有效的解或决策。

- **输入（Input）**：问题或场景由此进入系统。
- **ToT 结构（ToT Structure）**：由相互连接的节点（绿色和红色）表示，该结构反映了系统生成的可能想法或行动之树。
- **A\* 启发式估计器（A\* Heuristic Estimator）**：该组件连接到 ToT 中的每个节点，评估最值得遵循的路径。它使用启发式来估计每个行动的 "代价"，优先考虑那些最有希望引向解的行动。
- **LLM 动作生成器（LLM Action Generator）**：一旦 A* 启发式估计器识别出潜在的想法或行动，LLM 动作生成器便负责基于 LLM 中蕴含的深层知识创建新的状态或行动。
- **LLM 状态评估器（LLM State Evaluator）**：它评估生成的状态或行动，同时考虑上下文以及 LLM 更广泛的知识库，判断新状态或行动是否使系统更接近解。
- **Q-learning**：由指向 "Q-value"（Q 值）框的虚线表示，这体现了 Q\* 的学习层面。它评估每个行动的有效性（状态-行动代价），并基于反馈（解的估计）更新策略，从过去的行动中学习并调整未来行动以优化决策。
- **解记忆（Solution Memory）**：这里存储 Q 值，本质上是系统的记忆，保存所学到的行动质量值，为未来的决策提供依据。

通过将 A\* 在搜索 ToT 时的高效性、Q-learning 的动态学习能力与 LLM 的广博知识整合在一起，ToT 中的 Q\* 方法为应对各领域的复杂、自适应挑战提供了一种强有力的途径。它将 ToT 的结构化探索与自适应、知识丰富的能力相结合，以应对复杂而动态的挑战。这种整合不仅提升了 ToT 框架的效率与效果，也极大地拓展了它在各领域的潜在应用。


## 实际应用

Q\* 框架通过 A\*、Q-learning 与 LLM 的融合提供了一套前景可期的能力，但同时也带来了相当大的复杂性。必须认识到，并非所有问题都需要如此复杂的解决方案——有时更简单的工具就足够了。然而，对于那些缺乏令人满意的现有方案的重大而复杂的挑战，Q\* 框架可能是一件强有力的利器。本节将探讨 Q\* 框架的一些潜在实际应用。

![Q-Star 在环境管理中的应用]({{ site.baseurl }}/images/q-star-new-method-in-problem-solving/Q-Star_Application_to_Environment_Management.jpg)

_图：本插图展示了 Q\* 在环境管理与气候建模中的应用，画面中是一个配备先进数据处理系统、气候模型和环境数据可视化的未来感控制中心。_

### Q\* 应用示例

1. **城市规划与智慧城市**：Q\* 有潜力通过优化交通流、公共交通物流和应急响应协议来变革城市规划。利用其分析并预测交通模式的能力，Q* 可以提出动态改道策略，以应对实时状况，如正在举行的活动、天气变化以及城市生活的独特脉搏。

2. **医疗诊断与治疗规划**：凭借 Q\* 个性化诊断与治疗方案的能力，医疗行业可能迎来范式转变。通过将个人健康记录与前沿医学研究相结合，Q\* 或许能够提供量身定制的治疗方案、预见并发症，并随着新数据的出现无缝调整健康建议。

3. **环境管理**：在环境科学领域，Q\* 可以协助气候建模与生态系统管理。它处理海量数据集并适应新信息的能力，对于预测气候变化影响和提出缓解策略可能至关重要。

Q* 的部署预计将在各个行业掀起波澜，改变我们应对和解决复杂问题的方式。尽管可能出现障碍与限制，Q* 框架的持续演进可能预示着 AI 发展的新篇章，重新定义我们处理复杂问题求解的方法。


## 结语

正如我们在本文中所探讨的，Q\* 凭借其对 A\* 算法、Q-learning 与大语言模型（LLM）的创新性融合，代表了计算问题求解领域的一个重大飞跃。这一框架有潜力重新定义人工智能所能达到的边界，为各领域复杂、动态的挑战提供新的解决方案。

高效搜索、自适应学习与庞大知识库的整合，使 Q* 能够以前所未有的复杂度与有效性来解决问题。从城市规划到医疗保健，从环境管理到科学研究方法，Q* 的应用预计将既多样化又富有影响力。

最后，Q\* 不仅是在近期 AI 成果之上迈出的一个推测性步骤，也是一盏照亮未来可能性的明灯。它邀请我们想象这样一个世界：复杂的问题配上同样精巧的解决方案，AI 不仅支持而且增强人类的决策，而我们所能解决的问题边界将拓展到超出我们目前的认知。随后，我们将对 Q\* 方法进行更具技术性的评估。如果我们的推测性理解被证明是准确的，我们打算设计并实现该系统，作为下一步的进展。


## 参考资料

对于希望深入研究本文所讨论概念与技术的读者，以下资源提供了关于 Q\*、A\*、Q-learning、LLM 以及思维树框架的宝贵信息与洞见：

- **A\* 算法**：
    - Amit Patel, [Amit's A* Pages](https://theory.stanford.edu/~amitp/GameProgramming/), Red Blob Games
      - 一份关于 A* 算法的详尽指南，全面介绍它的工作原理及其应用。
    - Amit Patel, [Red Blob Games: A* Pathfinding](https://www.redblobgames.com/pathfinding/a-star/introduction.html), Red Blob Games
      - 一个关于 A* 寻路的交互式教程，适合喜欢动手实践的学习者。

- **Q-Learning**：
    - Sutton, Richard S., and Andrew G. Barto. _Reinforcement Learning: An Introduction_（《强化学习导论》）. Vol. 1. MIT Press, 2018.
      - 一本奠基性教材，深入探讨强化学习，重点关注 Q-learning。
    - Patrick Loeber, [Reinforcement Learning With (Deep) Q-Learning Explained](https://www.assemblyai.com/blog/reinforcement-learning-with-deep-q-learning-explained/), AssemblyAI, 2 Feb 2022.
      - 在本教程中，我们将学习强化学习与（深度）Q-learning。Q
    - [DeepLizard: Q-Learning 系列](https://deeplizard.com/learn/video/qhRNvCVVJaA)
      - 一个以易于理解的方式讲解 Q-learning 基础的视频系列。

- **大语言模型（LLM）**：
  - Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, L., & Polosukhin, I., [Attention is all you need](https://arxiv.org/abs/1706.03762)（《注意力就是你所需要的一切》）, In Proceedings of the 31st International Conference on Neural Information Processing Systems, 2017.
    - 引入 Transformer 模型的开创性论文，它是许多现代 LLM 的基础。
   - Andrej Karpathy, [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g&ab_channel=AndrejKarpathy), video, 22 Nov 2023.
      - 这是一个面向大众的 1 小时大语言模型导论：介绍 ChatGPT、Claude 和 Bard 等系统背后的核心技术组件，它们是什么、它们的发展方向、与当今操作系统的类比，以及这一新计算范式的一些安全相关挑战。
  - [OpenAI 的语言模型研究博客](https://openai.com/research/overview)
      - OpenAI 的文章与研究论文合集，讨论 LLM 的各个方面，包括 GPT-4。

- **思维树（ToT）框架**：
    - Yao, L., et al., Tree of Thoughts: Deliberate Problem Solving with Large Language Models, [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) (2023)
      - 在决策 AI 背景下引入 ToT 概念的研究论文。
    - Long, J., Large Language Model Guided Tree-of-Thought, [arXiv:2305.08291](https://arxiv.org/abs/2305.08291) (2023)
      - 对强化学习中 ToT 框架的详细探索。

- **通用 AI 与机器学习资源**：
    - [MIT 开放课程：人工智能导论](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-034-artificial-intelligence-fall-2010/)
      - MIT 的一门课程，广泛概述 AI 的原理与技术。
    - [Coursera：吴恩达机器学习](https://www.coursera.org/learn/machine-learning)
      - 一门广受欢迎的在线课程，涵盖机器学习的各个方面，包括 A* 等算法与强化学习概念。

- **相关文章**
    - Benny Cheung, [Game Architecture for Card Game AI (Part 3)]({{ site.baseurl }}/game-architecture-card-ai-3), Benny's Mind Hack, 3 Jul 2021.

    - Benny Cheung, [Adventures in Deep Reinforcement Learning using StarCraft II]({{ site.baseurl }}/adventures-in-deep-reinforcement-learning), Benny's Mind Hack, 15 Nov 2019.


这些资源为任何希望加深对这些复杂而迷人主题之理解的人提供了一个起点。无论你是学生、专业人士，还是纯粹的 AI 爱好者，这些材料都能为你提供关于高级计算问题求解世界的宝贵洞见。
