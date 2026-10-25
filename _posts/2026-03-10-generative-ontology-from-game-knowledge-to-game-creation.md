---
layout: post
title: "生成式本体：从游戏知识到游戏创造"
date: 2026-03-10 12:00:00.000000000 -00:00
permalink: generative-ontology-from-game-knowledge-to-game-creation
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- Generative AI
- Ontology
- Game Design
- Tabletop Games
- AI
- Context Engineering
category: post
comments: true
image: images/generative-ontology-from-game-knowledge-to-game-creation/post_cover.jpg
images:
  cover: images/generative-ontology-from-game-knowledge-to-game-creation/cover2x.jpg
  header: images/generative-ontology-from-game-knowledge-to-game-creation/header.jpg
  animated-cover: images/generative-ontology-from-game-knowledge-to-game-creation/animated-cover.webp
published: true
unlisted: false
series: "Game Architecture"
series_order: 8
---

<!--excerpt.start-->
2025 年 2 月，我们探讨了本体如何揭示桌游的隐藏结构。但理解游戏并不等于创造游戏。如果同样的结构化知识能够变成一台创造引擎呢？这就是生成式本体（Generative Ontology）的承诺——当知识表示学会想象。
<!--excerpt.end-->

![生成式本体——创造的语法]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_01.png)

_图示。结构与想象相遇——生成式本体核心的二元性。_

本文是游戏架构系列的收官之作。在[第 4 部分]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology) [8]中，我们为桌游构建了一个本体，将 CATAN 分解为机制（资源交易、模块化棋盘、骰子驱动的生产）、组件（六边形地块、资源卡、定居点）与玩家动态（竞争、谈判密集、可变玩家人数）。这个本体给了我们一套理解游戏的词汇，一门用于分析的精确语言。在[第 5 部分]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)中，我们演示了该本体如何驱动一条多智能体生成流水线。在[第 6 部分]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)中，我们探索了结构化创意生成背后的理论。而在[第 7 部分]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)中，我们展示了一个对话式 AI 伙伴如何学习设计师的品味。

![桌游本体示例]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Board_Game_Ontology_Examples.jpg)

_图示。像 CATAN 与《沙丘：帝国》这样的游戏，在迥然不同的主题之下共享同一个本体结构。_

现在，在这最后一篇文章中，我们来处理那个仅靠分析无法回答的问题：帮助我们*理解* CATAN 的同一个本体，能否帮助我们*创造*出 CATAN 的设计师们从未想象过的游戏？

我们把这种综合称为**生成式本体（Generative Ontology）**：把领域知识编码为可执行 schema，以此约束并引导 AI 生成的实践——它把静态的知识表示转变成一台创造引擎。本文给出理论框架，完整走通一次从主题到可玩设计的游戏生成，并提供它能奏效的实验证据。

---

## 从描述到创造

我们的游戏本体 [4] 能告诉我们：工人放置类游戏通常包含行动格、工人标记和阻挡机制 [3]。但它无法生成一个全新的工人放置游戏。大语言模型则有着相反的问题 [6]。让 LLM"设计一个以鬼屋为背景的牌库构筑游戏"，它会流畅地描述玩家探索 Ravenshollow 庄园、收集幽灵卡、管理一种"恐惧机制"。听起来头头是道。但起始牌库里到底有哪些牌？玩家如何获得新牌？什么触发游戏结束？LLM 生成的是游戏设计的*外观*，而非*实质*。

![创造的悖论]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_02.png)

_图示。传统本体（地图）与纯 LLM（梦想家）：懂得国际象棋的规则，并不等于你能成为特级大师。_

| 方法 | 优势 | 弱点 |
|----------|----------|----------|
| **传统本体** | 精确、结构化、经过验证 | 无法生成新颖输出 |
| **纯 LLM 生成** | 有创意、流畅、产出丰富 | 无结构、无效、产生幻觉 |

这些局限恰好互补 [5]。本体所缺的，LLM 能提供；LLM 所缺的，本体来补足。

![定义生成式本体]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_03.png)

_图示。LLM 潜能 + 本体约束 = 有效的游戏设计：从被动词汇到主动语法。_

### 游戏的语法

诗人从不把语法体验为一种限制。语法不是妨碍诗歌的东西，而是让诗歌成为*可能*的东西。没有句法、语义和形式，就不会有十四行诗，不会有俳句，也不会有冲撞惯例的自由诗。

同样的原则适用于游戏设计。当我们把游戏本体编码为 schema 时，我们不是在限制 AI 的创造力，而是在给它一套结构化的词汇，让它能够*连贯地*创造。schema 规定：每个游戏必须有目标、有结束条件、有创造玩家选择的机制、有把这些机制实例化的组件。在这些约束之内，无限的游戏成为可能；没有它们，任何有效的游戏都无法成形。

语法写不出诗。但没有语法，就没有诗可写。

### 与怀特海的联结

![游戏的语法]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_04.png)

_图示。永恒客体（本体）结晶为现实事态（生成）——怀特海过程哲学的计算化。_

在[第 6 部分]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)以及我们早前对[面向 AI 智能体设计的过程哲学]({{ site.baseurl }}/process-philosophy-for-ai-agent-design) [9]的探讨中，我们把怀特海的形而上学与结构化生成联系了起来。怀特海区分了**永恒客体（eternal objects）**（作为潜能存在的纯粹形式）与**现实事态（actual occasions）**（形式得以表达的具体事件）[1]。我们的游戏本体就是一组永恒客体：工人放置、牌库构筑、区域控制的抽象模式。

让这一切变得精确的，是怀特海的**摄受（concrescence）**概念：现实事态从可用的永恒客体中做出选择、并将它们综合成一个新统一体的过程 [2]。这正是生成流水线所做的事情。本体呈现全部可用模式构成的空间；LLM 在 schema 的约束下进行摄受：从这些模式中挑选，将它们与主题结合，产出一个此前从未存在过的具体游戏。这种创造力是真实的，但它是*结构化的*创造力。

---

## 从本体类到生成 Schema

![作为可执行 Schema 的本体]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_05.png)

_图示。schema 迫使 LLM 输出符合本体要求的合法结构化数据，相当于面向模型的自描述文档。_

哲学照亮路径，工程修筑道路。[第 4 部分]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology) [8]中的四个本体概念（游戏、机制、组件、玩家）可以自然地映射为带类型的 schema 定义：

| 本体概念 | Schema 角色 | 用途 |
|------------------|-------------|---------|
| 游戏类型 | 受约束的枚举 | 限定为合法的游戏模式（合作、竞争、半合作） |
| 机制 | 来自分类法的带类型列表 | 确保只引用已识别的机制 |
| 组件 | 结构化嵌套对象 | 以必填字段描述实体游戏元素 |
| 目标与结束条件 | 带最小长度要求的必填字符串字段 | 保证可玩性标准绝不含糊 |

schema 把本体从用于*分析*的工具（把 CATAN 分解成各个部分）翻转成用于*综合*的工具。它不告诉 LLM 创造*什么*游戏，而是告诉 LLM 一个游戏*必须是什么样*才算有效。但仅有 schema 并不够。单次 LLM 调用必须同时兼顾机制、主题、组件、平衡与玩家体验——人类设计团队可不是这样工作的。

---

## 为每个本体领域配备专职智能体

![专职智能体及其职业焦虑]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_08.png)

_图示。我们拆分任务以制造创造性张力，防止浅薄而随声附和的输出。_

游戏设计师勾勒机制，美术师发展视觉语言，游戏测试员找出失效的交互，各司其职，每位专家都带来专注的专业能力。

生成式本体实现了同样的分工。正如我们在[第 5 部分]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)中所演示的，我们把本体分解为多个领域，并为每个领域指派专职智能体。其结果受益于每一层上的专注投入。

### 智能体名册

我们的本体自然地暗示出专门化的边界：

| 智能体 | 本体领域 | 专长 | 职业焦虑 |
|-------|-----------------|-----------|---------|
| **机制建筑师**（Mechanics Architect） | 机制 | 回合结构、行动经济、结算系统 | "玩家是否拥有有意义的自主权？" |
| **主题织造者**（Theme Weaver） | 叙事 | 背景、风味、主题融合 | "主题是否在每条机制中都鲜活起来？" |
| **组件设计师**（Component Designer） | 组件 | 卡牌、代币、棋盘布局、实体操作体验 | "玩家能否顺畅地实体操作这些内容？" |
| **平衡批评家**（Balance Critic） | 跨领域 | 交互分析、优势策略检测 | "什么会被玩坏？优化到极致时什么会变得无趣？" |
| **趣味评委**（Fun Factor Judge） | 玩家体验 | 投入循环、张力、满足感 | "我还想再玩一局吗？" |

"职业焦虑"一栏是关键的设计创新。每个智能体都带着一种*职业忧虑*，它塑造该智能体的生成与批评，防止 LLM 那种"好好先生"式倾向——产出看似合理却肤浅的输出。机制建筑师想要优雅的系统；趣味评委想要刺激。这种内置张力映照了真实设计团队的动态，而信息则通过带类型的 schema 以显式交接的方式流动。

但仅靠协作并不能保证正确性。智能体们仍可能对那些看起来有效、实则不然的输出达成一致。我们需要最后一层保证。

---

## 验证与精炼：作为契约的本体

![作为奖励函数的本体]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_07.png)

_图示。系统携带具体的错误信息不断重试，直到达成结构上的连贯一致。_

生成还不够。LLM 是出了名的好好先生：它们会产出*看起来*有效的输出，却不确保它*确实*有效。Schema 校验能抓住类型错误和缺失字段，但语义连贯需要更深入的检查。LLM 可能声明"牌库构筑"是一种机制，组件里却没有一张牌。*听起来*有效，结构上却不连贯。

### 本体约束检查

我们编码了超越 schema 校验的跨字段一致性规则：

- **机制-组件连贯性**：牌库构筑需要卡牌。区域控制需要棋盘。工人放置需要工人标记。声明了某种机制，对应的组件就必须存在。
- **游戏类型一致性**：合作游戏不能以直接冲突作为主要交互模式；竞争游戏不应声明合作式交互。
- **可玩性要求**：目标必须具体（不能只是"获胜"或"得分"）；结束条件、回合结构和不确定性来源都必须有定义。

### 精炼循环

当验证失败时，系统会以具体的约束违例作为反馈进行重试："声明了牌库构筑机制，但组件中没有卡牌。"这个生成-验证-精炼循环持续进行，直到设计通过全部本体约束，或达到最大尝试次数。

其结果是，本体发挥出*契约*的功能。下游消费者（人类设计师、平衡工具或游戏引擎）可以依赖有保证的结构连贯性，而不仅仅是语法上的有效性。

---

## 案例研究：一场竞逐 AGI（通用人工智能）的游戏

![案例研究：输入]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_10.png)

_图示。一个应景而高风险、且没有直接桌游对应物的主题——本体能否处理好带隐藏信息的竞争策略中的种种微妙之处？_

来一款关于 AGI 的游戏怎么样？理论有说服力，演示才让人信服。让我们沿着生成式本体流水线，完整追踪一次从初始主题到可玩游戏设计的生成过程。

### 输入

我们提供一个主题和一组约束：

- **主题**："相互竞争的 AI 实验室竞相开发通用人工智能"
- **约束**：2-4 名玩家、中等复杂度、竞争型、45-60 分钟

这个主题是刻意挑选的：它足够应景、能引发共鸣，却没有直接的桌游对应物，迫使系统去综合而非模仿。

### 设计对话

![设计对话]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Generative-Ontology-Grammar_of_Creation_11.png)

_图示。一步步走来：建筑师到织造者到批评家再到精炼者——通过专职智能体实现的系统自我纠错。_

**机制建筑师**分析主题，识别出相互竞争的 AI 实验室意味着并行发展、资源竞争与技术突破。"竞逐 AGI"暗示一条带有终点线的进度轨道。在竞争约束下，玩家们各自独立地朝一个共享的胜利阈值冲刺。它选定行动点分配（秘密的部门预算分配）、引擎构筑（基础设施与研究协同）、市场/拍卖（竞争性人才争夺）和隐藏信息（未发表的突破）作为核心机制。回合结构将同时进行的秘密规划与按序结算结合在一起。

**主题织造者**接手这些机制并融入叙事。行动点分配变成实验室在研究、人才、基础设施与情报部门之间的预算决策。隐藏信息变成自有研究突破，保密到战略性地发表为止。市场变成一场人才战争，各实验室竞拍顶尖 AI 研究者。

引擎构筑变成对横跨五大研究领域的协同突破的追求：神经网络、机器人学、量子计算、伦理与安全、硬件。

一副 24 张牌的事件牌组确保没有两局游戏会完全相同，其中包括同时影响所有玩家的法规、危机与突破。

标题就此浮现：**Neural Race**。

**组件设计师**随后把这些机制转化为实体形态（棋盘、卡牌、代币和游戏垫板），确保每条机制都有具体的实体实例化（详见下文的最终设计表）。

### 批评家的眼光

**平衡批评家**识别出三个问题。第一，经由基础设施的滚雪球式领先：平坦的升级成本让领先玩家快速扩张行动点，形成难以逾越的优势。建议：将行动点上限设为 10，或采用指数递增的升级成本（3、5、8 点）。第二，人才市场中的"强者愈强"动态：竞标中的声望加成让领先实验室招揽到最优秀的研究者，进一步巩固领先。建议：给落后玩家竞标加成，或引入对弱势位置特别有价值的研究者。第三，囤积并集中发表协同突破会带来出人意料的得分暴涨。建议：实施更严格的手牌上限，或加入强制部分信息公开的机制。

![Neural Race 设计评估]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Neural_Race_Assessment.png)

_图示。附带按优先级排列、可立即执行的修复路线图的设计评估。_

精炼智能体处理了高优先级的声望问题：给竞标加成设 +2 的硬上限，并加入声望衰减机制。中等优先级的问题（基础设施扩张与隐藏信息不对称）获得了针对性修复：把行动点上限降到 8，并增加展示全部协同树的玩家参考卡。**趣味评委**给精炼后的设计打出 8/10。关键的吸引力钩子在于：玩家逼近 AGI 胜利阈值时的竞速动态、对手隐藏突破牌带来的不确定性，以及"何时发表"的风险/收益权衡。

### 最终设计

{% include youtubePlayer.html id="efG_E5v7HCc" %}

_图示。Neural Race 一览：从秘密规划到进度评估的五个游戏阶段，协同加成奖励在研究上的深度投入。_

完整输出如下：

| 字段 | 值 |
|-------|-------|
| **标题** | Neural Race |
| **主题** | 相互竞争的 AI 实验室在全球高风险竞赛中竞相开发 AGI |
| **游戏类型** | 竞争 |
| **目标** | 通过发表研究突破在 AGI 进度轨道上前进；率先达到 20 点进度者获胜 |
| **结束条件** | 冲刺局：率先达到 20 点及以上 AGI 进度的玩家立即获胜。持久局：若无玩家达到阈值，则 AGI 进度 + 协同加成 + 声望总和最高者获胜。 |
| **机制** | 行动点分配、引擎构筑、市场/拍卖、隐藏信息 |
| **回合结构** | 1) 向各部门秘密分配行动点，2) 执行研究与抽取突破，3) 人才市场竞标，4) 外部事件，5) 进度评估 |
| **玩家人数** | 2-4 |
| **交互** | 竞争性竞标、隐藏突破、声望竞赛、可变的实验室专精 |
| **核心循环** | 规划（分配）、研究（抽牌）、市场（竞标）、事件（应变）、评估（计分）、循环往复 |

每条机制都对应一个主题化行动，每个组件都服务于机制目的。从 60 张牌的研究突破牌组到定制木制机器人棋子（meeple），完整的组件规格详见 [GameGrammar 上的交互式本体](https://gamegrammar.dynamindresearch.com/s/JHEYCWpyE6DvbCnBOVRPr0iuyroWIdZrO1qjvw3EECs) [11]。

![Neural Race：想象中的游戏棋盘]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Neural_Race_Game_Board.png)

_图示。Neural Race 想象中的实体呈现：AGI 进度轨道螺旋伸向中心，两侧是按颜色区分的研究牌组、研究者专家与定制机器人棋子。_

### 玩起来是什么感觉？

数字和表格描述了设计，但玩起来像不像一款游戏？看第 5 回合：Sarah 在 AGI 轨道上以 12 比 14 落后于 Tom。Tom 一直在激进地招揽人才。Sarah 把 4 点行动点分配给研究，抽到了"递归神经架构搜索"。结合她的两张隐藏牌，她现在握有一套总值 9 点 AGI 进度的毁灭性组合。人才市场翻出了 Martinez 博士，一位至关重要的量子领域专家。Sarah 出价 4（3 行动点 + 1 声望）。Tom 惊煞全场地叫出 6 点人才点数，孤注一掷要封堵她的量子优势。Sarah 被迫转而争取 Liu 博士。

![Neural Race：第 5 回合：竞逐进入白热化]({{ site.baseurl }}/images/generative-ontology-from-game-knowledge-to-game-creation/Neural_Race_Sarah_Combo.png)

_图示。一个具体的游玩时刻，展现规划、抽牌与市场竞争之间的相互作用。每张牌的成本、每个协同引用、每个竞标数值，都在生成完成前经过了本体验证。_

虽然输掉了这场市场争夺，Sarah 的隐藏组合依然完好。这一刻捕捉到 Neural Race 核心的"发表还是囤积"困境：是攥住你的突破去构筑一条庞大的连锁，还是尽早发表、在对手搅局之前把点数和声望落袋为安？本体保证了协同引用有效、成本与收益相抵、牌与牌之间的交互内在一致。它无法保证的，是这个时刻带给人的究竟是凯旋还是廉价。这个判断属于人类设计师。

### 生成式本体提供了什么

如果没有本体 schema，一个生成"关于 AI 实验室竞争的竞争型游戏"的 LLM 很可能产出含糊的胜利条件（"以某种方式率先开发出 AGI"）、不一致的机制（提到拍卖却不指明竞标货币）、缺失的组件（没有定义任何实际游戏配件）和脱节的主题（机制与研究或竞争毫无关系）。

生成式本体框架确保了：

| 要求 | 如何强制执行 |
|-------------|---------------------|
| 完整的目标规格 | 目标字段的最小长度约束 |
| 机制与组件的一致对齐 | 验证函数检查已声明的机制是否都有匹配组件 |
| 主题融合 | 主题织造者智能体把每条机制显式连接到叙事 |
| 可玩性基础 | 回合结构、不确定性来源、结束条件为必填字段 |
| 平衡审查 | 平衡批评家智能体带着"把它玩坏"的职业焦虑 |

这个输出是*可玩的*。设计师可以拿着它做出原型，开始游戏测试。本体语法保证了游戏的所有要素都齐备且连贯。

但单一而精彩的例子并不能证明该框架普遍有效。生成式本体能否稳定地产出比无约束 LLM 生成更好的设计？

---

## 它有效吗？证据在此

在我们的正式研究 [12] 中，我们进行了三项实验，以衡量生成式本体是否能稳定地改进 AI 生成的游戏设计。

### 研究 1：消融实验。每一层各自贡献了什么？

我们在四种条件下生成了 120 个游戏设计，逐步叠加生成式本体框架的各个层次：

| 条件 | 配置 | 结构错误 | 创意质量 |
|-----------|---------------|-------------------|-----------------|
| **C1** 基线 | 原始 LLM，无 schema | 每设计 5.03 个错误 | 低（趣味：基线） |
| **C2** Schema | 仅 Pydantic 校验 | 每设计 0.10 个错误 | 低 |
| **C3** 本体 | Schema + 本体，单智能体 | 0 个错误 | 中等 |
| **C4** 流水线 | 完整：schema + 本体 + 多智能体 | 0 个错误 | 高 |

结果非常醒目。仅 schema 校验就几乎消除了全部结构错误（Cohen's *d* = 4.78）。但结构上的有效并不能保证创意质量。从 C3 到 C4 的跃升——加入带着专属职业焦虑的多智能体流水线——才是创意质量涌现的地方：

- **趣味评分**：*d* = 1.12（p < .001）
- **策略深度**：*d* = 1.59（p < .001）
- **优雅度**：*d* = 1.14（p < .001）
- **张力与戏剧性**：*d* = 0.79（p < .05）

直白地说：本体提供结构有效性，多智能体流水线提供创意质量，单独任何一个都不够。这个框架需要两个层次兼备。

### 研究 2：基准对比。与已出版游戏有多接近？

随后我们把 30 个流水线生成的设计与 20 款已出版桌游（CATAN、《沙丘：帝国》、《展翅翱翔》等）进行比较，在同样的创意维度上评估：

| 维度 | 已出版游戏 | 生成设计 | 差距 |
|-----------|----------------|-------------------|-----|
| 趣味评分 | 8.9 | 8.1 | 中等 |
| 策略深度 | 8.9 | 8.1 | 中等 |
| 张力与戏剧性 | 8.5 | 8.2 | **接近持平** |
| 社交互动 | 7.2 | 6.9 | **接近持平** |
| 优雅度 | 9.3 | 8.0 | 明显 |
| 重玩价值 | 9.1 | 7.6 | 明显 |

生成设计稳定地落在 7-8 分区间（良好、可玩的初稿），而已出版游戏得分在 8-9（经过打磨与游戏测试的成品）。差距真实存在，但也在意料之中：已出版游戏经历了数月甚至数年的人工迭代。重要的是，生成设计在张力/戏剧性和社交互动上已接近持平——那正是让游戏*感觉*引人入胜的体验性品质。

一个出人意料的发现：生成设计的结构一致性错误（1.27）反而*少于*已出版游戏（2.80，*d* = 0.76）。本体强制实现了一种连职业设计师有时都会忽视的内在连贯。

### 约束悖论

消融结果揭示了一个反直觉的事实。仅添加约束（C2、C3）能消除结构错误，却不提升创意质量；约束作用在单个智能体上时甚至会*压制*丰富度。但约束与架构专门化相结合（C4）则产生了最大的创意收益。这就是我们所说的**约束-架构交互模型**：创意质量是约束表达力*乘以*架构专门化程度的函数。约束定下地板，专门化抬高天花板，单独任何一个都不够。

### 作为特性的差距

生成设计与已出版游戏之间那一分的创意差距，吊诡之处在于它反而令人鼓舞。已出版游戏代表着多年迭代的游戏测试、社区反馈与设计师打磨，任何单次生成都无法复制这一过程。流水线产出的是 7-8 分区间（"良好、可玩、有趣的初稿"）而非 8-9 分区间（"精致、优雅、耐玩"）的设计，这表明差距反映的不是该方法论的根本局限，而是迭代精炼的缺席。已接近持平的维度（张力/戏剧性与社交互动）可能正是最能在规格设计阶段就做好的；而有差距的维度（优雅度与重玩价值）则很可能需要迭代式的游戏测试来优化。

另一项重测信度研究（跨 50 次评估的 ICC 分析）验证了基于 LLM 的评估器本身：9 项创意指标中有 7 项达到良好至优秀的信度（ICC 0.836-0.989）。评估方法可信，也就意味着差距的测量可信。

### 证据告诉我们什么

实验证实了核心论点：**结构使创造力成为可能**。原始 LLM 产出流畅但结构破碎的输出；schema 校验修复结构却修不了质量；而完整的生成式本体流水线——本体约束、多智能体专门化与验证循环——产出结构健全且创意动人的设计。含完整统计分析的正式论述见我们的论文 [12]。

---

## 超越游戏：作为一般框架的生成式本体

桌游领域是我们的试验场，但这个模式并不专属于游戏。只要以下三个条件成立，生成式本体就适用：

1. **领域已有成熟结构。**存在一套由类型、关系和约束构成的词汇，专家们靠它在领域中推理。
2. **有效输出必须满足跨字段约束。**正确性不仅要求单个字段格式良好，还要求字段之间的关系连贯一致。
3. **结构内的创意生成有价值。**目标是产出符合领域规则的新颖内容。

医疗摘要要求诊断与报告的症状及处方治疗相符 [5]；法律合同要求被定义的术语出现在可执行条款中；软件架构要求声明的接口与实现相匹配；菜谱生成要求食材用量能真正做出一道可行的菜。

在每一种情形中，无约束的 LLM 都会产出可能违反领域约束的流畅输出。同样在每一种情形中，编码为可执行 schema 的本体都能提供让有效生成成为可能的语法。多智能体模式也可以自然扩展：一个医疗本体可以分解为诊断、治疗和禁忌智能体，各自带着自己的职业焦虑。

这个洞见是普适的：**领域本体就是创造的语法。**任何具备足够结构的领域都可以被赋予生成能力。

---

## 结语

纵观整个系列，我们勾勒出一条弧线：从分析游戏（第 4 部分）到生成游戏（第 5 部分），到理解结构化生成为何有效（第 6 部分），再到与设计师实时协作（第 7 部分）。这篇收官之作提供了理论综合与实证证据。

用怀特海的话说 [1]，我们为永恒客体——游戏设计的抽象模式——提供了一种实现摄受的计算机制，即把熟悉的形式综合为新颖现实的过程 [2]。生成式本体就是把"创造性前进"（creative advance）变成可操作的东西。

本文首次发表时，尚有两个问题悬而未决：对话式 AI 伙伴能否与人类设计师一起迭代生成的设计？生成的设计能否被自动游戏测试？

如今这两个问题都已有答案。[Nova]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)（第 7 部分）是一个对话式 AI 联合设计师：它读取完整的游戏本体，跨会话记住每一个设计决策，并以前后对比的方式提出修改建议。一套自动化游戏测试引擎使用蒙特卡洛树搜索智能体和基于 LLM 的玩家原型来模拟对局，在印出第一份原型之前就暴露出平衡问题、退化策略与得分缺口。论文中指出的、可能是创意差距之源的迭代精炼循环，如今已部分闭合。

更深的问题依然存在。AI 系统能否从成功案例的语料中*归纳*出本体，从范例中学习生成语法而非手工编码？创意差距能否通过自动化迭代完全弥合，还是最后一公里永远需要人类的品味？这些是开放的研究问题。但地基已经打下：被赋予生命的结构化知识，使结构化的创造成为可能。

语法写不出诗。但没有语法，就没有诗可写。而现在，我们有证据表明：这套语法行得通。

---

* [解锁桌游本体的奥秘（第 4 部分）]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)
* [GameGrammar 简介：AI 驱动的桌游设计（第 5 部分）]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)
* [GameGrammar：生成式桌游设计理论（第 6 部分）]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)
* [Nova：学习你品味的 AI 联合设计师（第 7 部分）]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)
* **>>** [生成式本体：从游戏知识到游戏创造（第 8 部分）]({{ site.baseurl }}/generative-ontology-from-game-knowledge-to-game-creation)

---

## 参考文献

[1] Alfred North Whitehead. [*Process and Reality*](https://archive.org/details/processreality0000alfr)（《过程与实在》）. Free Press, 1929/1978.
  - 永恒客体 / 现实事态框架的理论基石

[2] Timothy Barker. [*Artificial Creativity: A Process Philosophy of Technology Perspective*](https://eprints.gla.ac.uk/327708/1/327708.pdf)（人工创造力：一种技术的过程哲学视角）. Journal of Continental Philosophy, 2024.
  - 将怀特海的过程哲学与生成式 AI 创造力联系起来

[3] Geoffrey Engelstein and Isaac Shalev. [*Building Blocks of Tabletop Game Design: An Encyclopedia of Mechanisms*](https://www.routledge.com/Building-Blocks-of-Tabletop-Game-Design/Engelstein-Shalev/p/book/9781138365490)（桌游设计的积木：机制百科全书）. CRC Press, 2020.
  - 我们游戏本体所依托的全面机制分类法

[4] Natalya F. Noy and Deborah L. McGuinness. [*Ontology Development 101*](https://protege.stanford.edu/publications/ontology_development/ontology101.pdf). Stanford University.
  - 本体设计原则的基础

[5] Jorge Martínez-Gil, et al. [*Ontology-Constrained Generation of Domain-Specific Clinical Summaries*](https://arxiv.org/abs/2411.15666)（本体约束下的领域特定临床摘要生成）. arXiv:2411.15666, 2024 年 11 月.
  - 方法论上最接近的先例：本体引导的约束生成

[6] Roberto Gallotta, et al. [*Large Language Models and Games: A Survey and Roadmap*](https://arxiv.org/html/2402.18659v1)（大语言模型与游戏：综述与路线图）. arXiv:2402.18659, 2024 年 2 月.
  - LLM 在游戏中应用的全面综述

[7] Matthew Guzdial, et al. [*Boardwalk: Towards a Framework for Creating Board Games with LLMs*](https://arxiv.org/abs/2508.16447)（Boardwalk：用 LLM 创造桌游的框架初探）. arXiv:2508.16447, 2025.
  - 从规则生成桌游代码（与我们的设计生成方法形成对照）

[8] Benny Cheung. [*Unlocking the Secrets of Tabletop Games Ontology*]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)（解锁桌游本体的奥秘）. 2025 年 2 月.
  - 游戏架构系列第 4 部分，本文的基础

[9] Benny Cheung. [*Process Philosophy for AI Agent Design*]({{ site.baseurl }}/process-philosophy-for-ai-agent-design)（面向 AI 智能体设计的过程哲学）. 2026 年 1 月.
  - 与"创造性前进"相衔接的怀特海框架

[10] [*GameGrammar*](https://gamegrammar.com). Dynamind Research, 2026.
  - 建立在生成式本体之上的 AI 桌游设计平台

[11] [*Neural Race on GameGrammar*](https://gamegrammar.dynamindresearch.com/s/neural-race)（GameGrammar 上的 Neural Race）. Dynamind Research, 2026.
  - 本文案例研究所生成完整游戏本体的交互式展示

[12] Benny Cheung. [*Generative Ontology: When Structured Knowledge Learns to Create*](https://arxiv.org/abs/2602.05636)（生成式本体：当结构化知识学会创造）. arXiv:2602.05636, 2026 年 2 月.
  - 包含消融研究（120 个设计）、基准对比与评估器可靠性分析的正式论文
