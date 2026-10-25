---
layout: post
title: "GameGrammar 亮相：AI 驱动的桌游设计"
date: 2026-02-03 00:00:00.000000000 -00:00
permalink: introducing-gamegrammar-ai-powered-board-game-design/
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- Game Design
- Tabletop Games
- Design Tools
- Game Architecture
- Board Games
- Co-Design
- Game Analysis
category: post
comments: true
image: images/introducing-gamegrammar-ai-powered-board-game-design/post_cover.png
images:
  cover: images/introducing-gamegrammar-ai-powered-board-game-design/cover2x.png
  header: images/introducing-gamegrammar-ai-powered-board-game-design/header.png
  animated-cover: images/introducing-gamegrammar-ai-powered-board-game-design/animated-cover.webp
published: true
unlisted: false
series: "Game Architecture"
series_order: 5
---

<!--excerpt.start-->
我在文本框里输入了十二个词，得到了一份结构化的桌游初稿。机制、组件、计分表、一张六边形地图、一个四阶段回合结构，还有一个告诉我"这游戏坏了"的评审。整个过程耗时 73 秒。这就是把一套结构化的游戏分类学交给六个专门化设计代理、让它们对你的设计进行批评时会发生的事。
<!--excerpt.end-->

![GameGrammar 落地页]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_Landing_Hero.png)

_图：GameGrammar 通过六个专门化的设计代理，把一个主题与约束转化为结构化的桌游设计。_

那十二个词是：*"Rival astronomers racing to name celestial objects before their competitors claim the glory."*（竞争的天文学家竞相命名天体，抢在对手摘走荣耀之前。）

得到的是 **Stellar Rivals: A Race to the Stars**：一款 2-4 人竞争游戏，讲述 19 世纪天文学家探索由星区组成的六边形网格、收集天体、竞相完成星座的故事。它有具体的行动点消耗、一张含五条不同胜利路径的计分表、装备升级卡，以及一份平衡性批评——标出了两个高严重度问题，而这些问题我原本需要数周的游戏测试才能发现。

我没有设计这款游戏。我没有通过二十轮与 ChatGPT 的来回对话把它"提示工程"出来。我输入了一个主题、设置了一些约束，然后按下了生成。

{% include youtubePlayer.html id="iW6g_1mnyMQ" %}

本文讲述这一切如何运作的故事、它为什么不同于让 LLM"设计一款桌游"，以及它对游戏设计未来的意义。这也是[游戏架构系列]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)的第 5 部分——自[第 4 部分]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)首次梳理桌游结构 [1] 以来，我们一直在朝着这一刻进发。如果你想理解 GameGrammar 背后的设计理论，包括其哲学基础以及人类设计师与 AI 之间的协同设计关系，请见[第 6 部分：生成式桌游设计理论]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)。

---

## 游戏测试的坟场

在看 GameGrammar 产出什么之前，我们需要理解它解决什么问题。因为问题不是"我想让 AI 替我设计游戏"。问题是空白页。

创作一款出版桌游是一段九阶段的旅程 [2]，大多数人只看到最后几个阶段：货架上的盒子、Kickstarter 活动、评测视频。他们看不到的是第 1 和第 2 阶段——设计师真正花费最多时间的地方。

| 阶段 | 时期 | 发生什么 |
|-------|-------|--------------|
| **1** | **概念与早期设计** | 核心想法、初始机制、纸面原型 |
| **2** | **迭代式游戏测试** | 砍、并、重写规则；压力测试系统 |
| 3-9 | 设计锁定到发布后 | 开发、美术、制造、营销、发行、售后 |

在一个经过测试的原型与货架上的盒子之间，还隔着六个阶段：出版商开发、美术指导、制造、营销、发行与售后支持。但坟场在第 1 和第 2 阶段。

> "大多数'酷机制'死在这里——也应该死在这里。" [2]

游戏测试的坟场人满为患。设计师花几个月开发一个资源交易机制，跑一次盲测，发现它造成一个支配性策略。机制被砍。设计师从头再来。这个循环必不可少，但它也是动力被消磨的地方——对没有团队维持势头的独立设计师尤其如此。

空白页是第一个敌人。在设计师开始游戏测试的严酷考验之前，他们需要一个值得测试的概念。不只是一个主题，而是机制、组件、玩家动态与胜利条件的一个连贯组合——经过足够的迭代，它有可能成为一款真正的游戏。

![桌游设计流水线]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Board_Game_Design_Pipeline.png)

_图：九阶段的桌游生产流水线。GameGrammar 加速第 1 和第 2 阶段——设计师花费最多时间、有前途的想法最常死掉的地方。_

我在 [Dynamind Research](https://www.dynamindresearch.com) [5] 构建了 GameGrammar——这是一家连接计算设计研究与落地实现的研究与产品工作室——就是为了攻克这个具体问题。不是为了取代游戏设计师，不是为了把创作过程自动化，而是为了消除空白页瘫痪，给设计师值得迭代的结构化起点。它运作于第 1 和第 2 阶段——设计师的挑战是生成足够多的可行概念，以找到值得打磨的那颗宝石。

---

## Stellar Rivals：看六个代理如何造出一款游戏

实际过程是这样的。打开 GameGrammar，输入一个主题，设置一些约束：

![GameGrammar 生成第 1 步]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_Generate_Step1.png)

_图：输入：一个主题、约束，以及可选的预选机制。或者只输入一句话，让系统自己选。_

主题可以是任何东西：*"中世纪商人沿丝绸之路贩运香料"*、*"深海探险家发现失落文明"*，或者在我们的例子中，*"竞争的天文学家竞相命名天体"*，约束为 *"2-4 人，竞争性，中等复杂度，45-60 分钟"*。

然后选择生成模式。我选了多代理（Multi-Agent），因为它揭示了 GameGrammar 与单提示词方法的根本区别：六个专门化的代理按顺序工作，每个代理都会阅读它之前所有代理的完整输出。

| 模式 | 速度 | 发生什么 |
|------|-------|--------------|
| **Quick** | 约 15 秒 | 单个强大模型一次生成完整设计 |
| **Multi-Agent** | 45-90 秒 | 六个专门化代理按序协作，每个都在上一个的基础上构建 |
| **RAG-Enhanced** | 约 30 秒 | 生成以 2,000+ 款已出版的 BoardGameGeek 游戏数据为依据 [4] |

![GameGrammar 生成页]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_Generate_Page.png)

_图：选择生成模式：Quick 用于快速原型，Multi-Agent 用于完整的六代理流水线，RAG-Enhanced 用于以真实出版游戏为依据的设计。_

73 秒之后：

![Stellar Rivals 标题]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Title.png)

_图：Stellar Rivals: A Race to the Stars。来自十二个词与四条约束的结构化初稿。_

![Stellar Rivals 概览]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Overview.png)

_图：远征概览：2-4 人，45-60 分钟，中等复杂度。每一次发现都把你的名字刻在天体上。_

返回的不是一段建议文字。它是一份结构化的设计文档，带具体的组件数量、具体的分值、定义好的回合阶段，以及已识别的平衡性问题。那种你可以直接递给游戏测试者并说"我们试试这个"的文档。

以下是每个代理做了什么，以及这个顺序为什么重要。

### MechanicsArchitect 搭骨架

第一个代理不想主题或组件的事。它想机制。它可以使用一个精心整理的分类学：35 种游戏机制，组织为七大类 [1][3]，它的任务是选出一套兼容的组合。

![GameGrammar 多代理流水线]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_MultiAgent_Pipeline.png)

_图：六个专门化的代理按顺序工作。每个代理都能看到并构建于之前所有代理的完整输出之上。_

为 Stellar Rivals，它选了四种：

- **行动点**（每回合 6 点）作为核心决策经济
- **集合收集**用于基于星座的计分
- **隐藏信息**：面朝下的天体瓦片
- **区域移动**：跨越由星区组成的六边形网格

为什么是这四种？因为 MechanicsArchitect 会核对一个从真实出版游戏的共现模式中构建的兼容性矩阵。行动点与集合收集常常一起出现，因为基于稀缺的经济与收集目标相得益彰。隐藏信息与区域移动创造空间探索——这正是"竞争的天文学家"所暗示的。这个代理不是在猜。它是在与数千款已出版设计做模式匹配。

![GameGrammar 机制浏览器]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_Mechanism_Browser.png)

_图：机制浏览器：七大类（回合顺序、行动选择、资源/经济、冲突/领地、卡牌/牌库、信息、其他）下的 35 种机制，附有来自真实出版游戏的兼容性数据。_

这套分类学不是拍脑袋的。它源自对数千款已出版桌游及成熟游戏设计文献的分析 [1][3]。MechanicsArchitect 从这个整理过的词汇表中选取，而不是从 LLM 训练数据里的模糊模式中选取。

### ThemeWeaver 给骨架穿衣

第二个代理读取机制选择，把它们翻译进一个 19 世纪天文发现的设定：

- 六个星区（内行星、小行星带、外行星、星云场、深空、银心）
- 天体包括类星体、星系、双星和星云
- 影响观测消耗的天气条件
- 代表科学进展的研究代币

这就是"区域移动"变成"把你的望远镜在星区之间移动"、"隐藏信息"变成"通过观测揭示的面朝下天体瓦片"的地方。机制没有变。它们被披上了皮肤。

![Stellar Rivals 六边形地图]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Hex_Map.png)

_图：绘制天球：六个六边形星区，移动消耗以行动点计。距离即消耗。_

### ComponentDesigner 让它实体化

第三个代理指定每一件物理部件：

- 6 块星区六边形瓦片
- 48 张面朝下的天体瓦片（每区 8 张）
- 装备升级卡
- 研究代币与观测点标记
- 玩家望远镜标记

这很重要，因为作为一个概念存在的游戏，与一个可以制作原型的游戏不是一回事。具体的数量、具体的卡牌类型、具体的令牌功能。设计师读完这份清单就可以开始裁卡纸了。

### DetailsArchitect 写规则

第四个代理把回合结构定义为一个四阶段循环：

| 阶段 | 活动 | 关键决策 |
|-------|----------|--------------|
| **望远镜定位** | 同时选择一个星区 | 本回合在哪里观测 |
| **观测** | 花费 6 个行动点用于移动、揭示与认领 | 稀缺下的资源分配 |
| **分析** | 为认领的天体计分、检查星座完成情况 | 集合收集进度 |
| **装备** | 花研究代币购买升级 | 引擎构筑投资 |

![Stellar Rivals 回合阶段]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Turn_Phases.png)

_图：天文学家的日常：定位、观测、分析、装备的四阶段循环。_

设计从这里开始感觉像一款真正的游戏了。看看陈博士（Dr. Sarah Chen）的一回合：

她花费 6 个观测点中的 3 个把望远镜移动到银心星区。她使用一张光谱滤光镜装备卡以 1 点的优惠成本揭示一张瓦片， uncovering 一颗类星体。她用最后的 2 点揭示了一个椭圆星系。类星体的原始分值更高，但星系完成了她的仙女座星座，净得该天体 4 分外加 8 分星座奖励。她赚取研究代币，随后获得了天文台网络卡。

这是一个有真实取舍的真实决策。现在的分数与以后的分数。单体价值与集合完成。把钱花在移动还是花在发现。正是这类张力让游戏有趣——而它们从十二个字的输入中涌现了出来。

计分系统有五条不同的路径：

| 计分类别 | 机制 | 分值 |
|-----------------|-----------|--------|
| 天体 | 按天体认领 | 按稀有度 1-6 |
| 星座奖励 | 集合完成 | 3 / 5 / 8（递增） |
| 双星系统 | 成对发现 | 印刷值的 2 倍 |
| 命名权 | 首次发现奖励 | 3-4 分 |
| 星系团 | 终局收集 | 每个星系 +2（若达 4 个以上） |

![Stellar Rivals 计分]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Scoring.png)

_图：五条通往胜利的战略路径：天体稀有度、星座集合、双星配对、命名权与星系团。_

---

## "你的游戏是坏的"

这是我最初构建它时最让我惊讶的部分。

第五和第六个代理不是设计师。它们是评审。**BalanceCritic** 评估完整设计的公平性与战略深度。**FunFactorJudge** 评估这款游戏玩起来是否真的有趣。而且它们不给你留情面。

以下是 BalanceCritic 对 Stellar Rivals 的评价：

| 问题 | 严重度 | 建议 |
|-------|----------|----------------|
| 天文台网络卡规避了移动经济 | **高** | 限制每局使用次数，或把成本提高到额外 2-3 点 |
| 星座难度断层，简单集合奖励过高 | **高** | 调整奖励结构（例如 3-6-10）以反映难度 |
| 4 人时星区过度拥挤 | 中 | 每区瓦片数量随玩家数缩放 |
| 初始 6 个观测点显得束手束脚 | 中 | 提高到 7-8 或允许 1-2 点结转 |
| 研究代币兑换率（2:1）低效 | 中 | 改善为 3:2 以保证战术可行性 |

两个高严重度问题。天文台网络卡——就是陈博士在我们的玩法示例中获得的那张——实际上破坏了让游戏其余部分得以运转的移动经济。而星座奖励结构对简单集合的奖励不成比例，制造了一个支配性策略。

FunFactorJudge 给该设计打了 7/10，指出来自隐藏瓦片的"发现快感"与紧凑的行动点经济是主要的张力来源。

![Stellar Rivals 平衡批评]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/Stellar_Rivals_Balance_Critique.png)

_图：BalanceCritic 在指出关键问题的同时，也识别了优点，并附上具体的修复建议。这不是奉承。这是一场设计评审。_

想想刚刚发生了什么。系统生成了一款游戏，然后告诉我它是坏的，再告诉我确切怎么修。这种反馈通常需要数周的游戏测试、多个测试小组，以及一位诚实到能看见自己盲区的设计师。而在这里，它发生在同一次 73 秒的生成之中。

GameGrammar 不只是生成。它就自己产出的质量与自己辩论。

---

## "我不能直接用 ChatGPT 吗？"

这个问题很合理。你当然可以让 ChatGPT 设计一款桌游。它会产出流畅、自信的文字。它会告诉你一款有"资源管理""战略深度""高重玩性"的游戏。

但试着把那份输出递给一位游戏测试者。问他们：每回合我有几个行动点？当两名玩家落在同一格时会发生什么？起始牌库里有多少张牌？答案几乎总是：LLM 生成了一件游戏设计的*外观*，却没有*实质*。

| 方面 | 原始 LLM | GameGrammar |
|--------|---------|-------------|
| **架构** | 单提示词、单次生成 | 6 个专门化代理按序工作 |
| **机制** | 取自模糊的训练数据 | 35 种机制的整理分类学 |
| **游戏引用** | 可能幻觉出游戏名与数据 | 2,000+ 款真实 BGG 游戏被索引 |
| **输出结构** | 无结构的散文 | 一致的 schema：组件、规则、计分 |
| **自我批评** | 除非显式提示，否则没有 | 内置 BalanceCritic 与 FunFactorJudge |
| **迭代** | 复制-粘贴-重新提示 | 工作台：定向展开、重评估、一致性检查 |
| **平衡分析** | 泛泛之谈 | 带严重度评级的问题与具体修复建议 |

差别不是边际的。原始 LLM 可能告诉你一款游戏有"资源管理"。GameGrammar 会告诉你：每位玩家每回合获得 6 个观测点，移动按星区距离消耗 1-3 点，揭示一张瓦片花费 2 点，研究代币以 2:1 的比例兑换观测点。一个是建议。另一个是规格说明。

![GameGrammar vs 原始 LLM]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_vs_RawLLM.png)

_图：建议与规格。GameGrammar 产出带具体数字、具体组件与自我批评评估的设计。_

---

## 按下生成之后发生什么

生成是开始，不是结束。没有哪个设计第一次就是对的——哪怕自带评审。GameGrammar 为迭代生成的设计提供了一个工作台：

**展开**任何区块以补充细节。**重新评估**——在你做出修改后再次运行 BalanceCritic 与 FunFactorJudge。**一致性检查**验证组件与规则匹配、被引用的条目确实存在、数量对得上。你可以生成**变体**、创建**封面美术**，并**导出**为 JSON、Markdown 或 PDF。

设计库存储每个设计，带搜索、过滤器与版本历史。社区画廊让设计师分享作品、浏览他人创作，发现自己可能没想到过的机制组合。

![GameGrammar 工作台]({{ site.baseurl }}/images/introducing-gamegrammar-ai-powered-board-game-design/GameGrammar_Workbench.png)

_图：设计工作台。展开、重评估、一致性检查、生成变体、创建封面美术、导出。生成是流程的开始，不是结束。_

这映照着真实的设计工作流：原型、测试、迭代。GameGrammar 压缩了那个循环中"生成"的一侧，让设计师把时间花在最重要的地方：与真实玩家同桌。

### 依然属于人类的

我想把 GameGrammar 不能做什么说清楚，因为边界比能力更重要。

它不能**测试游戏**。没有任何算法能模拟四个人围桌而坐、发现某机制枯燥或某计分路径支配性时的体验。它不能**读房间**——那些笑声、挫败、以及当连招奏效时玩家脸上的惊喜。它不能**驾驭出版之旅**——制造经济学、出版商关系、展会展台推介。它也不能**做品味判断**。一款 45 分钟的维多利亚天文学家游戏*有趣吗*？这是设计师与他们的受众的问题，不是算法的问题。

GameGrammar 是一个设计加速器，不是替代品 [2]。它把设计师从空白页瘫痪中解放出来，给他们值得迭代的结构化起点。之后发生的一切——游戏测试、打磨、出版——依然是设计师的手艺。

已经有游戏了？你不必从零生成。把你现有的设计描述给 Nova，平台会把它结构化为本体论格式，让你对*你自己*设计的游戏运行平衡分析与策略测试。

---

## 试试吧

GameGrammar 现已开放公测：[gamegrammar.dynamindresearch.com](https://gamegrammar.dynamindresearch.com)。

免费层每天包含 5 次生成。Quick 模式约 15 秒产出一份完整设计。多代理流水线在 90 秒内交付一份带内置平衡批评与趣味性评估的设计。

**生成你的第一款游戏：**
1. 在 [gamegrammar.dynamindresearch.com](https://gamegrammar.dynamindresearch.com) 创建账户
2. 输入主题：从"海盗竞逐宝藏"到"量子物理学家争夺诺贝尔奖"，什么都行
3. 设置约束：玩家人数、复杂度、时长
4. 第一次设计选择多代理模式
5. 读一读平衡批评。那是会让你惊讶的部分。

探索区提供机制浏览器（含全部 35 种机制）、展示哪些机制搭配良好的兼容性矩阵，以及用于浏览 2,000+ 款出版游戏寻找灵感的游戏探索器。

---

## 结语

这是[游戏架构系列]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)的第 5 部分。在第 4 部分，我们为理解桌游构建了结构化的词汇 [1]。现在，那套词汇变成了一台创意引擎：有名字、有界面、有生成按钮。

十二个词。73 秒。不是一款成品游戏——而是一份结构化初稿：五条计分路径、一张六边形网格、装备升级、星座奖励，还有一个告诉你它在哪里坏掉的评审。

空白页不再是你的敌人。它是你的发射台。而如果你已经亲自填满了那一页，平台也已准备好帮你测试你所构建的东西。

本文展示的是*是什么*。如果你想理解*为什么*，[第 6 部分：生成式桌游设计理论]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)将探索 GameGrammar 背后的设计理论。它涵盖把机制与主题分离的三层架构、人类设计师与计算代理之间的协同设计关系，以及"算法能否理解乐趣"这一哲学问题。如果你好奇这一方法在更深层面上有何不同，下一步就去那里。

---

## 系列

* [解锁桌游本体论的奥秘（第 4 部分）]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)
* **>>** [GameGrammar 亮相：AI 驱动的桌游设计（第 5 部分）]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)
* [GameGrammar：生成式桌游设计理论（第 6 部分）]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)

---

## 参考文献

[1] Benny Cheung. [*Unlocking the Secrets of Tabletop Games Ontology*]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology). bennycheung.github.io, Feb 2025.
  - 游戏架构系列第 4 部分，本工作的本体论基础

[2] Board Game Design Pipeline Analysis. Dynamind Research, Jan 2026.
  - 从概念到发布后的九阶段流水线，GameGrammar 定位于第 1-2 阶段

[3] Geoffrey Engelstein, Isaac Shalev. [*Building Blocks of Tabletop Game Design: An Encyclopedia of Mechanisms*](https://www.routledge.com/Building-Blocks-of-Tabletop-Game-Design-An-Encyclopedia-of-Mechanisms/Engelstein-Shalev/p/book/9781138365490). CRC Press, 2022.
  - 游戏机制分类学与分类的综合参考

[4] [BoardGameGeek](https://boardgamegeek.com/). The largest board game database and community.
  - RAG 增强生成所用的 2,000+ 游戏索引的来源

[5] [Dynamind Research](https://www.dynamindresearch.com). Research and product development studio.
  - GameGrammar 的创造者，连接计算设计研究与实际产品实现

[6] Benny Cheung. [*Generative Ontology: When Structured Knowledge Learns to Create*](https://arxiv.org/abs/2602.05636). arXiv:2602.05636, Feb 2026.
  - GameGrammar 设计理论与生成式架构背后的正式研究论文
