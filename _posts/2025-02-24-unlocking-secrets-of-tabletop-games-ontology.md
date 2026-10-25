---
layout: post
title: 解锁桌游本体论的奥秘
date: 2025-02-24 12:00:00.000000000 -00:00
permalink: unlocking-secrets-of-tabletop-games-ontology/
author: Benny Cheung
artwork:
  author: Catan
tags:
- Ontology
- Tabletop Games
- Board Games
- Game Design
category: post
comments: true
image: images/unlocking-secrets-of-tabletop-games-ontology/post_cover.jpg
images:
  cover: images/unlocking-secrets-of-tabletop-games-ontology/cover2x.jpg
  header: images/unlocking-secrets-of-tabletop-games-ontology/header.jpg
published: true
series: "Game Architecture"
series_order: 4
---

<!--excerpt.start-->
桌游的世界里有一个迷人的概念，像是一种秘密语言。如果你能破解这套密码，也许就能开始理解为什么有些游戏登上传奇之位，而另一些则湮没无闻。这不只是背规则或精通策略；而是理解**游戏的本体论**（Ontology of Games）。
<!--excerpt.end-->

你可能会问，本体论到底是什么？最简单地说，它是一种组织知识的结构化方式。把它想象成游戏宇宙的地图：不只是罗列"工人放置"或"牌库构筑"这样的关键术语，还展示这些术语如何彼此关联（桌游的完整分类体系参见 Engelstein, Geoffrey, and Isaac Shalev 的 _Building Blocks of Tabletop Game Design: An Encyclopedia of Mechanisms_）。它不只是词汇表；它关乎理解连接。而一旦看见了那些连接，你就能做一些相当酷的事。

![Dune-Imperium-Board-Game]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/Dune-Imperium-Board-Game.jpg)

_图：《沙丘：帝国》是一款桌游，其灵感来自《沙丘》的元素与角色，冲突的中心是厄拉科斯。这将是我们案例研究之一。_

例如，当本体论把"资源管理"这一概念与掷骰子和交易等机制关联起来时，模式就浮现了。你开始看清这些元素如何创造不同的玩家体验。这就像拥有一张理解游戏运转机理的蓝图。

这种做法不只是理论。像 [Palantir](https://www.palantir.com/) 这样的公司构建在[本体论驱动的业务流程](https://www.palantir.com/platforms/foundry/)之上，用以组织和分析海量数据，做出原本不可能的连接与预测。随着 AI 的兴起，本体论变得更加关键。它们提供 AI 系统学习与推理所需的结构化信息。如果我们想训练一个 AI 理解桌游，就需要给它一个坚实的框架。这正是本体论登场的地方。

----
> 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。

{{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/Podcast_Tabletop_Games_Ontology.mp3
{% include open-embed.html %}

-----

准备好探索**作为桌游知识结构化方法的本体论**了吗？本文不止于知道规则，而是聚焦理解游戏元素之间的关系：游戏类型、机制、组件与玩家。借助一个**简化桌游本体**，设计师与玩家都可以系统地分析和设计游戏。通过两款流行桌游——**《卡坦岛》（Catan）**与**《沙丘：帝国》（Dune: Imperium）**——我们展示本体论如何帮助理解它们更深层的结构与机制。

## 游戏本体论是什么？

> 本体论是对共享概念化的形式化、显式的规范说明。

在计算领域——撇开哲学传统的复杂性不谈——本体论是一种结构化的知识表示，定义特定领域（此处即桌游）内的概念、关系与类别。本质上，它充当_共享词汇，让智能信息系统能够更有效地理解和整合数据_。与传统数据库不同，本体论支持更动态、更具上下文感知的数据管理，促进不同应用之间的互操作。本体论已被广泛用于解决数据集成难题、增强人工智能应用，例如问答系统、科学发现和自动推理，使其成为现代知识驱动技术的关键组件。

把关系数据库与作为知识库的本体论加以比较可以发现：与 RDBMS 不同，本体论（知识库）显式地包含知识的表示——纳入规则，使用自动推理（超越普通查询）来推断隐式知识并检测知识库的不一致，而且它们通常在开放世界假设（Open World Assumption）下运作。

同样的灵活性可以用于桌游设计：规则、机制与玩家交互的精巧系统需要一个结构化而可适应的框架。通过使用简化桌游本体，设计师可以系统地剖析并重新想象游戏玩法的核心组件，孕育在结构、策略与玩家投入之间取得平衡的创新设计。

### 通过简化本体探索桌游设计

我们知道，一款引人入胜的桌游需要在结构、策略与玩家交互之间精心平衡。借助简化桌游本体，我们可以拆解让游戏既有效又有趣的本质元素。这一模型强调核心组件——**游戏类型**、**机制**、**组件**与**玩家**——让我们聚焦这些元素如何交互以创造引人入胜的玩法体验。

1. **游戏类型** - 游戏的基本分类（例如合作、竞争）。
2. **核心机制** - 基本游戏机制，如回合结构、行动与结算。
3. **游戏组件** - 物理元素，如卡牌、令牌和棋盘。
4. **玩家** - 参与者及其交互的表示。

通过梳理机制、组件与玩家交互等不同元素如何相互关联，设计师可以获得宝贵的洞见，理解什么让玩法引人入胜、令人难忘。**简化桌游本体**充当结构化的蓝图，让我们通过拆解核心元素来系统地分析和设计游戏。

![Tabletop_Game_Simplified_Ontology]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/Tabletop_Game_Simplified_Ontology.png)

_图 1. 应用简化桌游本体，我们可以获得对领域的结构化视角，让我们在本文中有效地讨论和分析流行游戏。这一方法凸显了用本体论发现底层模式、加深理解、更系统地比较不同桌游的力量。_

透过这面镜子，我们不仅能理解游戏如何运转，还能理解为什么某些设计能在更深层次上引起玩家共鸣。下图把这些关系可视化，提供了一个切实的模型，说明这些概念在设计过程中如何相互关联。

以下小节将回顾简化桌游本体（如 _图 1_ 所示）中的全部核心概念与关系。

#### 定义游戏框架

每款桌游的核心是 **Game** 实体，它概述了游戏的**目标**与**结束条件**。定义良好的目标给玩家清晰的目标：无论是积分、征服领地还是完成任务。结束条件规定游戏何时结束——例如达到某个分数或完成全部游戏轮次——为体验带来结构与节奏。

游戏可以归类为**棋盘游戏**（BoardGames）和**卡牌游戏**（CardGames）等类型，各自带来独特的动态。棋盘游戏可能聚焦空间策略与棋盘上的移动，而卡牌游戏可能强调概率、虚张声势或手牌管理。

#### 打造引人入胜的机制

机制驱动玩家交互与策略深度。简化本体突出了基本机制：

- **回合结构**：确立玩家何时以及如何行动。结构化的回合顺序确保游戏性与节奏的公平。
- **行动**：决定玩家在回合中能做什么，从放置令牌到抽卡。提供有意义的选择带来策略深度。
- **结算**：解决玩家行动的结果，例如赢得挑战或完成任务。
- **不确定性**：引入运气或隐藏信息元素，让玩法保持动态与吸引力。

例如，纳入一个回合制系统让玩家按固定顺序行动，可以进行策略规划。加入不确定性——例如抽取随机卡牌——可以制造悬念与惊喜，增强整体兴奋感。

#### 构建物理与概念组件

**组件**是让游戏活起来的有形元素。棋盘、卡牌和令牌充当游戏机制与叙事的物理表示。设计良好的棋盘增强沉浸感，而定制的令牌或独特的卡牌设计可以增添主题深度与触觉乐趣。

玩家与这些组件交互，塑造游戏的叙事与流程。**Player** 类强调个体角色，每位玩家都把自己的策略与决策带上桌。通过设计允许协作、竞争或角色扮演的交互，设计师可以创造丰富的社交体验。

#### 创建引人入胜的游戏概念

设想设计一款竞争性棋盘游戏：玩家扮演探索神秘岛屿的寻宝者。**目标**是在游戏结束前收集最有价值的宝藏。玩家轮流在棋盘上移动（**回合结构**），执行挖掘宝藏或破坏对手等行动（**行动**）。随机抽取的事件卡引入**不确定性**，带来新的挑战或奖励。

结算机制决定行动是否成功——例如掷骰子看陷阱是否被成功拆除（**结算**）。当所有宝藏格都被探索完毕时游戏结束（**结束条件**）。

## 案例研究：透过本体论揭示洞见

本体构建完成后，它帮助定义游戏结构、机制与玩家角色等核心概念。这些基础观念随后与骰子、卡牌、棋盘、玩家交互空间等物理与概念游戏元素相连接。

让我们透过本体论的镜头考察具体的游戏，揭示促成其成功的精巧机制：(1)《卡坦岛》与 (2)《沙丘：帝国》。这有助于我们理解把本体论作为统一视图的效用。

### 应用桌游本体：《卡坦岛》案例研究

用简化桌游本体分析 [Catan](https://boardgamegeek.com/boardgame/13/catan)（1995）可以揭示其游戏设计如何有效整合多种机制与组件，创造动态、竞争且富有策略的体验。聚焦核心元素——游戏框架、机制、组件与玩家交互——设计师可以理解为什么**《卡坦岛》**至今仍是桌游圈深受喜爱、影响深远的棋盘游戏。

![BGG_Catan_Illustration]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/BGG_Catan_Illustration.png)

_图：在这款现代经典中收集并交易资源，建设卡坦岛。（来源：BoradGameGeek.com [Catan](https://boardgamegeek.com/boardgame/13/catan) (1995)）_

#### 在 OntoUML 中可视化

为了更好地理解简化桌游本体的应用方式，让我们分析流行的棋盘游戏**《卡坦岛》**（前称*卡坦岛拓荒者*）。这个例子将说明本体中的不同元素如何共同营造丰富而引人入胜的玩法体验。

![Tabletop_Game_Simplified_Ontology_Example_Catan]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/Tabletop_Game_Simplified_Ontology_Example_Catan.png)

_图：以 OntoUML 格式可视化的《卡坦岛》简化桌游本体。_

#### 定义游戏框架

**《卡坦岛》**在本体中被归类为**棋盘游戏**。游戏的首要**目标**是成为第一个达到 10 分胜利点的玩家，胜利点通过建造定居点、城市和发展卡获得。当任意玩家达成该目标时触发**结束条件**，宣告游戏结束。

游戏支持 3 到 4 名玩家，每人扮演一名**玩家**，通过策略、资源管理与谈判争夺主导权。

#### 《卡坦岛》的核心机制

**《卡坦岛》**整合了本体中的若干关键机制，成就其深度与重玩价值：

- **回合结构**：游戏遵循顺序回合结构。每位玩家按固定顺序行动：掷骰子、收集资源，并执行修路、交易或购买发展卡等行动。
- **行动**：在自己的回合中，玩家可以执行与他人交易资源、修建道路和定居点、购买发展卡等行动。这些行动极具策略性，迫使玩家基于自己的资源与潜在未来收益进行规划。
- **结算**：行动的结果常常由资源可用性决定。例如，骰子点数决定生成哪些资源，影响每位玩家的策略与资源管理。
- **不确定性**：骰子把随机性引入游戏，模拟资源稀缺与不可预测性。隐藏的发展卡也添入不确定性，因为玩家无法确定对手可能握有什么优势。

#### 《卡坦岛》的组件

**《卡坦岛》**的有形元素与本体中的**Component**类对应：

- **棋盘**：模块化的六边形棋盘代表卡坦岛，每个六边形产出不同资源（木材、砖块、麦子、绵羊和矿石）。
- **卡牌**：玩家收集并策略性使用的资源卡与发展卡。
- **令牌**：定居点、城市和道路令牌代表玩家在棋盘上的进展。

这些物理组件让游戏活了起来，创造出视觉上引人入胜、可交互的环境，增强了策略玩法。

#### 玩家交互与策略

在**《卡坦岛》**中，每位玩家通过交易与谈判积极与他人互动，玩家交互是游戏乐趣的核心。玩家必须在竞争与合作之间取得平衡——交易可以互利，但过多的帮助可能壮大对手。

策略性思维必不可少：玩家必须决定在哪里建设、优先获取哪些资源、何时用强盗机制封锁对手。策略、谈判与运气的结合，使**《卡坦岛》**成为引人入胜且平衡的桌游的典范。


### 应用桌游本体：《沙丘：帝国》案例研究

另一个本体论案例研究 [Dune: Imperium](https://boardgamegeek.com/boardgame/316554/dune-imperium)（2020）揭示了它的游戏设计如何精巧地编织策略机制、主题组件与动态玩家交互。聚焦这些核心元素——游戏框架、机制、组件与玩家投入——设计师可以理解为什么**《沙丘：帝国》**带来深度沉浸、策略回报丰厚的棋盘游戏体验。

![BGG_Dune_Imperium_Illustration]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/BGG_Dune_Imperium_Illustration.png)

_图：沙丘宇宙中的影响力、阴谋与战斗（来源：BoradGameGeek.com [Dune: Imperium](https://boardgamegeek.com/boardgame/316554/dune-imperium) (2020)）_

#### 在 OntoUML 中可视化

让我们用简化本体分析**《沙丘：帝国》**。

![Tabletop_Game_Simplified_Ontology_Example_Dune]({{ site.baseurl }}/images/unlocking-secrets-of-tabletop-games-ontology/Tabletop_Game_Simplified_Ontology_Example_Dune.png)

_图：以 OntoUML 格式可视化的《沙丘：帝国》简化桌游本体。_

#### 定义游戏框架

**《沙丘：帝国》**在本体中被归类为**棋盘游戏**。游戏的首要**目标**是通过影响力、战斗与策略性资源管理积累最多的胜利点。玩家通过控制关键区域、赢得冲突、结盟和积累资源来获得分数。当玩家达到 10 分胜利点或冲突牌库耗尽时触发**结束条件**，宣告游戏的最后一轮。

游戏支持 1 到 4 名玩家，每人扮演一名独特的**玩家**，代表沙丘宇宙中的一位派系领袖。每位领袖拥有影响其策略与交互的特殊能力。

#### 《沙丘：帝国》的核心机制

**《沙丘：帝国》**整合了本体中的若干关键机制，成就其深度与重玩价值：

- **回合结构**：游戏采用顺序回合制，玩家按轮行动。每轮包括玩家轮流派出代理人前往各地点，随后是揭示阶段与冲突结算。
- **行动**：玩家打出手牌，把代理人部署到棋盘各处，以获取资源、提升影响力或备战。策略性的出牌与资源管理是成功的关键。
- **结算**：冲突通过战斗机制结算，玩家投入部队与资源争夺奖励。结果取决于部署的部队数量与卡牌加成，使战斗成为玩法中高风险的环节。
- **不确定性**：牌库构筑元素引入不确定性——玩家构筑并洗切牌库，每轮的抽牌难以预料。此外，冲突的结果可能受隐藏的阴谋卡影响，叠加惊喜与策略。

#### 《沙丘：帝国》的组件

**《沙丘：帝国》**的物理与概念组件与本体中的**Component**类对应：

- **棋盘**：代表厄拉科斯上的关键地点，玩家部署代理人以获取资源、影响派系并为冲突做准备。
- **卡牌**：牌库构筑卡代表行动、角色与事件，为玩家提供新能力与策略。
- **令牌**：部队与代理人令牌代表军事力量与影响力，是解决冲突与占据棋盘位置的关键。

这些组件主题丰富，让玩家沉浸在沙丘宇宙的政治与战略动态之中。

#### 玩家交互与策略

**《沙丘：帝国》**中的玩家交互紧张而多面。玩家争夺棋盘位置、影响派系轨道，并展开直接冲突。游戏鼓励策略性封锁、谈判与细致规划。

每位玩家必须在牌库构筑、棋盘控制与资源管理之间取得平衡。冲突与揭牌的时机增添张力与深度，让每个决策都举足轻重。与宇航公会或贝尼·杰瑟里特等派系结盟提供强力加成，鼓励多样化的策略。

### 用本体论做游戏设计

游戏设计常常像迷宫。机制相互交织，目标容易迷失。这正是本体论派上用场的地方。它们提供一个框架，帮助设计师做出有意图的选择。

- **做出有意图的选择：** 本体论澄清游戏机制之间的连接。当你理解每个元素如何影响玩家体验，你就能带着目的设计。每个机制都成为塑造玩家旅程的工具，确保你的选择与目标对齐。
- **汲取灵感：** 通过研究既有的游戏框架，设计师可以找到与自己创意愿景共鸣的元素。你也许会在一款游戏中发现一个机制，点燃自己作品的灵感。灵感常常只是回望他人已做过的事。
- **打造引人入胜的循环：** 本体论帮助打磨引人入胜的循环。好的游戏设计平衡短期奖励与长期策略。借助这一框架，设计师可以构建让玩家欲罢不能的系统。
- **识别潜在陷阱：** 它们充当预警系统。本体论让设计师在早期发现潜在陷阱。你可以在概念阶段识别平衡或机制上的缺陷，避免在开发后期付出高昂代价。

在游戏设计中使用本体论不只是关于结构；它关乎清晰、创造力，并最终打造更好的玩家体验。

## 结语

本体论就像地图，但它们不是告诉我们去哪里，而是揭示事物如何连接。本文中，我们理解了如何应用本体论，通过把元素拆解为类别与关系，帮助玩家和创作者理解那些让游戏不仅有趣而且令人难忘的精巧机制。

但这个想法不止于游戏。想象把本体论应用于商业：你可以梳理不同角色、流程与结果之间的关系，更容易发现低效或机会。音乐的本体论可能揭示流派之间如何重叠，帮助听众基于既有品味发现新艺术家。在叙事中，本体论可以帮助作家识别原型与情节结构，创作更丰富的故事。

甚至在人际关系中，本体论也可以澄清连接与动态，帮助我们驾驭社交的复杂性。它是一个鼓励我们看见更大图景的框架，让我们理解任何给定系统中各元素如何交互。

本体论之美在于其适应性。随着新游戏的诞生、我们对人类互动认识的加深，这些框架也在演进。它们不只赋能我们分析，更赋能我们创新。通过拥抱本体论，我们解锁对世界更深层的理解，让我们以从未想象过的方式进行创造与连接。

## 参考文献

### 理解桌游

- [BGG - Board Game Geek](https://boardgamegeek.com/)
    - **BoardGameGeek.com** 网站是一个包含海量游戏数据库的资源。它提供图片与规则书，有助于进一步查询游戏。该网站被提及为深入学习资料中所含游戏机制示例的好资源。
    - 案例研究：[Catan](https://boardgamegeek.com/boardgame/13/catan) (1995)
    - 案例研究：[Dune: Imperium](https://boardgamegeek.com/boardgame/316554/dune-imperium) (2020) 

- Engelstein, Geoffrey, and Isaac Shalev. _Building Blocks of Tabletop Game Design: An Encyclopedia of Mechanisms_. Vol. 1. CRC Press / Taylor & Francis Group, 2020.
    - 汇总了 Geoffrey Engelstein《Building Blocks of Tabletop Game Design》节选所讨论的关键主题、概念与机制。该书旨在全面探索构成桌游设计的各种元素，细致审视单个机制及其在游戏中的交互。它强调游戏设计中的广泛可能与复杂性，并不自称是创造成功游戏的灵丹妙药。

- Domain of Science, [The Fascinating Map of Board Games](https://www.youtube.com/watch?v=fH4MaSJ8gXU), video, 14 Nov 2024.
  - 该视频探索棋盘游戏的迷人世界，洞见其历史、机制与文化意义。它概述了棋盘游戏随时间的演化，并highlighted该类型中的各种流派与主题。


### 理解本体论开发

- Natalya F. Noy and Deborah L. McGuinness, [Ontology Development 101: A Guide to Creating Your First Ontology](https://protege.stanford.edu/publications/ontology_development/ontology101.pdf), Stanford University.

- Shaimaa Haridy, et al., [An Ontology Development Methodology Based on Ontology-Driven Conceptual Modeling and Natural Language Processing: Tourism Case Study](https://www.mdpi.com/2504-2289/7/2/101?type=check_update&version=1), _Big Data Cogn. Comput._ **2023**, _7_(2), 101; [https://doi.org/10.3390/bdcc7020101](https://doi.org/10.3390/bdcc7020101)

### 桌游本体论研究

- Joshua Kritz and Geraldo Xexéo, [A Vocabulary of Board Game Dynamics](https://arxiv.org/pdf/2403.10267v1), [arXiv:2403.10267](https://arxiv.org/abs/2403.10267) [cs.SE], 15 Mar 2024. 

- J. Kritz, E. Mangeli, G. Xexéo, [Building an Ontology of Boardgame Mechanics based on the BoardGameGeek Database and the MDA Framework](https://www.semanticscholar.org/paper/Building-an-Ontology-of-Boardgame-Mechanics-based-Kritz-Mangeli/373c770ab468bb06b29030c34427eafae399cc42), 2017. 

### 本体论商业

- Palantir, [Palantir Ontology Documentations](https://www.palantir.com/docs/foundry/ontology/overview/)
  - Palantir **本体论**是组织的运营层。本体论位于集成到 Palantir 平台的数字资产（数据集与模型）之上，把它们与其现实世界对应物连接起来——从工厂、设备和产品等物理资产，到客户订单或金融交易等概念。在许多场景中，本体论充当组织的数字孪生，包含启用各类用例所需的语义元素（对象、属性、链接）与动态元素（动作、函数、动态安全）。

- J. Neil Otte , John Beverley , and Alan Ruttenberg, [Basic Formal Ontology: Case Studies](https://philarchive.org/archive/OTTBBF), 11 Aug 2023.
    - 基本形式本体论（BFO）是一个包含三十八个类的顶层本体，旨在支持所有科学研究领域的信息集成、检索与分析，目前被全球 350 多个本体项目采用。BFO 是真正的顶层本体，不含任何特定于物质领域的术语，如物理、医学或心理学。本文演示如何用 BFO 框架，以共相（universal）、定义类与关系来表示一系列说明常见变化类型的案例。我们对这些案例的讨论为其他使用 BFO 的本体学者提供了模板，并便于与使用不同顶层本体的本体学者所提策略进行比较。

### 卡牌游戏游戏架构系列

Benny's Mind Hack 博客的"卡牌游戏游戏架构"系列，有助于启发更好的卡牌游戏设计，产出更强大的卡牌游戏 AI！

- [卡牌游戏模型的游戏架构（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)
- [卡牌游戏行动的游戏架构（第 2 部分）]({{ site.baseurl }}/game-architecture-card-ai-2)
- [卡牌游戏 AI 的游戏架构（第 3 部分）]({{ site.baseurl }}/game-architecture-card-ai-3)
