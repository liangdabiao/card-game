---
layout: post
title: "AI 究竟如何设计一张卡牌"
date: 2026-03-24 00:00:00.000000000 -00:00
permalink: how-ai-actually-designs-a-card
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- Game Design
- Card Games
- Design Tools
- Tabletop Games
- Prototyping
- Game Architecture
- Race for the Galaxy
category: post
comments: true
image: images/how-ai-actually-designs-a-card/post_cover.jpg
images:
  cover: images/how-ai-actually-designs-a-card/cover2x.jpg
  header: images/how-ai-actually-designs-a-card/header.jpg
  animated-cover: images/how-ai-actually-designs-a-card/animated-cover.webp
published: true
series: "Card Architecture"
series_order: 2
---

<!--excerpt.start-->
2021 年，我花了一个月时间对《银河竞逐》（Race for the Galaxy）进行逆向工程。我解析了 Keldon Jones 的 C 源代码，把整套卡牌库转换成 Python，并梳理出 114 张独特卡牌之间的每一次阶段交互、每一项卡牌能力、每一条生产链。我这样做，是因为这款游戏的 AI 总是把我打得落花流水，而我想弄清楚原因。我发现，RFTG 中每张卡牌都携带一个结构化数据模型，其复杂程度远超卡面文字给人的印象：类型、费用、VP 分值、货物类型、军事标记，以及一份跨五个不同游戏阶段相互作用的阶段专属能力清单。五年后，当我开始构建一个能生成卡牌游戏的系统时，我意识到所需的流水线正是我当年手工完成之事的镜像。AI 并没有取代设计师的流程，它是在把这套流程形式化。
<!--excerpt.end-->

![AI 究竟如何设计一张卡牌]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/post_cover.jpg)

这是 [Card Architecture 系列]({{ site.baseurl }}/three-waves-of-card-game-design-tools)的第二篇。在第一篇中，我追溯了卡牌游戏工具从脚本编写到 AI 原生流水线的演进历程。本文则要深入流水线内部。但我不只想描述流水线如何运作，还想勾勒一个改变了我对工具辅助设计之看法的类比：在每一个阶段，AI 所做的都是人类设计师已经在做的事情的机械化版本。问题不在于 AI 能否设计卡牌，而在于卡牌设计中哪些部分是机械化的、哪些不是，以及这对人类设计师的角色意味着什么。

---

## 人类设计师如何设计一张卡牌

在审视 AI 之前，先让我描述一下人类设计师坐下来创作卡牌游戏时究竟会发生什么。我将以 Race for the Galaxy 为参照，一是因为我曾花一个月时间深入其架构内部，二是因为它代表了严肃卡牌游戏所要求的复杂度水平。

![Race for the Galaxy]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/RaceForTheGalaxy.gif)

*图 1. Tom Lehmann 的 Race for the Galaxy（2007）——114 张独特卡牌、五个同时执行的阶段、四种生产类型、军事与民用定居点之分。隐藏在每张卡牌内部的复杂性，正是它既成为设计杰作、又构成 AI 挑战的原因。*

当 Tom Lehmann 设计 RFTG 时，流程大致如下：

**第一，世界观。** 游戏需要一个能支撑 114 张独特卡牌的主题：银河文明建设——有待定居的世界、待研发的科技、待生产与交易的货物。主题不是装饰，它约束着设计空间。你不可能在一款关于中世纪农耕的游戏里放一张名为"Corporate Restructuring（企业重组）"的卡，也不可能在一款关于太空殖民的游戏里放一张"Harvest Festival（丰收节）"。主题是第一道过滤器。

**第二，机制。** RFTG 的标志性创新是同时角色选择：所有玩家秘密选择一个阶段，只有被选中的阶段才会执行，选择者还能获得特权加成。这一机制并非事后补丁，而是游戏中每张卡牌悬挂其上的骨架。每张卡牌都带有阶段专属能力：New Vinland 在阶段 5 生产新奇货物，在阶段 4 消耗任意货物抽 2 张牌。这种跨阶段交互绝非偶然，它源于设计师先定义了机制骨架，再设计出利用骨架缝隙的卡牌。

**第三，卡牌本身。** 当我解析 `cards.txt` 文件时，我发现每张 RFTG 卡牌都携带一个结构化数据模型：

| 字段 | 示例（New Vinland） | 用途 |
|-------|----------------------|---------|
| 名称 | New Vinland | 身份标识 |
| 类型 | World（类型 1） | 机械类别 |
| 费用 | 2 | 你需要支付的（从手牌弃置） |
| VP | 1 | 终局计分 |
| 货物类型 | Novelty（新奇） | 它生产什么 |
| 阶段 4 能力 | 消耗任意货物，抽 2 张牌 | 贸易/消耗交互 |
| 阶段 5 能力 | 生产与本世界类型相同的货物 | 生产引擎 |

*表 1. 单张 RFTG 卡牌背后的结构化数据模型：七个字段、两项阶段专属能力、一条生产链。这正是基础模式（schema）必须捕捉的复杂性。*

单张卡牌上就有七个结构化字段。Replicant Robots 是一张发展卡，形状则不同：费用 4、VP 2，外加一项把定居费用减少 2 的阶段 3 能力。Contact Specialist 则在你每定居一个世界时抽一张牌。每张卡都是一个小型程序，有输入、输出和条件行为。

![RFTG New Vinland 卡牌设计]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/RFTG_New_Vinland.png)

*图 2. New Vinland 的卡牌设计数据（左）与实际卡牌（右）。`cards.txt` 的编码——N:名称、T:类型:费用:VP、G:货物类型、P:阶段:能力——把七个结构化字段压进六行文字。阶段 IV 消耗任意货物抽 2 张牌，阶段 V 生产新奇货物。这就是隐藏在每张 RFTG 卡牌背后的结构化数据模型。*

人类设计师把这一切都装在脑子里。他们对生态系统需要哪些卡牌、存在哪些策略空白、哪些组合能造就令人满足的回合，都拥有直觉。他们凭经验知道：一副满是廉价进攻卡的牌库需要一张昂贵的防御反制卡；一条生产链既需要生产者也需要消费者；一局游戏若结束得太快，就意味着后期投资不值得去建。

**第四，迭代。** Lehmann 不可能一次性把 114 张卡全部设计到位。他反复试玩，发现破坏平衡的组合，删掉卡牌，重新调整费用，再添加新卡填补策略空白。RFTG 的 AI 早在 DeepMind 让强化学习声名鹊起之前，就已经使用时序差分学习（temporal difference learning）在超过 3 万局模拟对局中完成训练。AI 通过纯粹的重复学会了哪些卡能赢、哪些卡会输。正是在迭代之中，好卡才蜕变为伟大的卡。

这个四阶段流程——先主题、再机制、再卡牌、最后迭代——是每位经验丰富的卡牌游戏设计师都会做的事。有些人用设计文档把它正规化，有些人靠电子表格完成，但认知结构是相同的。

AI 流水线精确地映照了这一流程。

---

## 多智能体流水线

系统没有让一个 AI 包揽一切，而是把工作分给多个专职智能体（agent），每个智能体负责人类设计师凭本能完成的流程中的一个环节。

![多智能体流水线]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/Multi_Agent_Pipeline.jpg)

*图 3. 多智能体卡牌生成流水线。四个专职智能体对应人类卡牌设计的四个阶段：主题、机制、卡牌和迭代。*

### 阶段 1：主题编织者（Theme Weaver）

*设计师写在概念文档里的内容。*

人类设计师从一个概念出发："一款关于银河文明竞相探索、定居和开发银河系的游戏。"他们会勾画叙事边界：词汇表、基调，以及这个宇宙中存在哪些事物——世界、发展、货物、贸易路线。

Theme Weaver 智能体做的正是同样的事。它从设计师那里取一句话，生成一份详细的主题设计文档，把叙事现实锁定下来。如果游戏关乎银河扩张，AI 就不会生成一张名叫"Wheat Field（麦田）"或"Village Smithy（乡村铁匠铺）"的卡。这份文档约束了词汇，使后续每个智能体都说同一种语言。

人类是在无意识中完成这一步的，而 AI 需要把它明确写下来。

### 阶段 2：机制架构师（Mechanics Architect）

*设计师勾画回合结构时的草稿。*

主题之后，人类设计师要定义这个卡牌宇宙的物理法则：一个回合有几个阶段？存在哪些资源？玩家如何互动？什么条件触发游戏结束？

当我逆向工程 RFTG 的游戏引擎时，我发现整个游戏可以归结为一台状态机：五个阶段，每个阶段有一组合法动作，每个动作都在修改共享的游戏状态。抽牌、发展、定居世界、贸易或消耗货物、生产。同时角色选择是外层循环，阶段专属的卡牌能力是内层循环，其余一切都只是记账。

Mechanics Architect 智能体生成的正是这个骨架。它接收主题设计，产出一份机制文档，定义回合结构、资源类型、胜利条件和行动经济——即玩家在回合中实际能做什么。这就是游戏世界的引力，AI 之后生成的每张卡牌都将服从这个引力。

我反复用到的一个比喻是：机制架构师先构建物理引擎，然后才让 AI 去设计桥梁。

### 阶段 3：组件设计师（Component Designer）

*设计师敲进电子表格里的内容。*

有了主题和机制，Component Designer 智能体便开始生成实际的卡牌。正是在这个阶段，人类与 AI 之间的平行对应最为惊人。

AI 生成的每张卡都必须符合一个严格的模式（schema）：

| 字段 | 用途 | RFTG 对应物 |
|-------|---------|-----------------|
| **卡牌名称** | 身份标识，必须契合主题 | "New Vinland"、"Contact Specialist" |
| **卡牌类型** | 机械类别 | 世界（World）与发展（Development） |
| **效果文字** | 卡牌的功能，如卡面所印 | 阶段专属能力 |
| **费用** | 玩家需支付的东西 | 从手牌弃置卡牌 |
| **战略角色** | 这张卡为何存在（至少 20 个字符） | 设计师的心智模型 |

*表 2. 五字段卡牌模式。战略角色字段把人类设计师凭直觉携带的东西外化了出来。*

最后一个字段——战略角色——是最重要的。它有硬性的 20 字符下限。AI 必须为它生成的每张卡写一段说明，论证这张卡为何存在于游戏生态之中。

关键在于：每位经验丰富的卡牌游戏设计师脑中都携带着这套论证。他们知道 New Vinland 的存在意义是作为新奇货物生产链的廉价入口；他们知道 Contact Specialist 的存在是为了奖励那些投资定居世界的玩家；他们知道 Galactic Federation 的存在是为重发展（development）策略提供计分回报。

区别在于，人类设计师是隐式地持有这个心智模型的，有时到第 47 张卡时就会跟丢。战略角色字段迫使 AI 把它显式写出来。而事实证明，强迫任何人——无论人类还是机器——把每张卡存在的原因讲清楚，都会让设计变得更好。

### New Vinland 测试

要看这一点在实践中如何体现，不妨设想一款具有 RFTG 风格机制的游戏跑完整个流水线时会发生什么。AI 不会生成一张千篇一律的"世界卡：价值 1 VP"。它拥有关于主题（银河文明）、机制（带生产链的五阶段同时选择）和策略生态的完整上下文。

于是它会生成类似这样的卡："Mining World（采矿世界）。民用世界。生产珍稀货物。费用 3 张牌。战略角色：中等费用的生产型世界。珍稀货物在贸易中价值更高，与更便宜的新奇货物世界相比，为更高的投入提供了回报。"

这段战略角色陈述证明了 AI 理解资源层级：珍稀货物的贸易价值高于新奇货物；一张生产珍稀货物的 3 费世界被正确地定位在一张生产新奇货物的 2 费世界之上。AI 所推理的成本曲线，与人类设计师在电子表格里勾画的是同一条。

它不是千篇一律的糊状产物，而是一张契合游戏经济结构的卡。

### 超越五个字段：卡牌语法（Card Grammar）

五字段模式适用于许多卡牌游戏。但当我回到 RFTG 的 `cards.txt`，逐卡清点字段数量时，我发现是七个、八个，有时更多：类型、费用、VP、货物类型、标记，以及多项阶段专属能力。五字段模式是地板，不是天花板。

对于复杂的游戏，AI 流水线支持一种卡牌语法（Card Grammar）：一份针对每款游戏的解剖结构声明，明确告诉每个智能体每张卡携带哪些结构化字段。Card Grammar 声明的是带类型的字段，而不是需要系统自行解析的自由格式效果文字：

- 标签（枚举列表）：Science（科学）、Military（军事）、Production（生产）、Alien（外星）
- 生产效果（资源增量）：每回合生产 2 矿石
- 资源费用（资源映射）：花费 3 矿石 + 2 能量
- 触发时机（枚举）：打出时、激活时、回合之间、游戏结束时
- 计分公式（公式）：桌面（tableau）中每个 Military 标记 3 VP

这正是我在 RFTG 卡牌数据内部发现的结构，被泛化为可用于任何游戏。卡牌语法告诉 AI："在这款游戏中，卡牌拥有标签、生产速率、资源费用和触发时机。生成卡牌来填充这些字段。"其结果是结构化数据而非散文，这意味着模拟引擎可以直接读取而无须猜测。

![Card Grammar 模式]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/Card_Grammar_Schema.jpg)

*图 4. Card Grammar 用针对每款游戏的类型化解剖字段扩展了五字段模式。这个我当年从 RFTG 手工提取的结构，如今由系统自动声明并强制执行。*

### 阶段 4：细节扩展者（Detail Expander）

*设计师在首次试玩测试后要做的事。*

有一个问题每位卡牌游戏设计师都再熟悉不过：你设计了 30 张卡，拿去试玩，它们都没问题——平衡、可用，却也彻底无聊。

语言模型也有同样的倾向。放任不管时，它会向均值回归，产出安全、统计意义上中庸的卡牌——强度大致相同、费用区间相同、复杂度相同。能用，但过目即忘。

人类设计师会在首次试玩后修正这一点。他们发现快攻（aggro）策略太强，于是设计一张陷阱卡；他们注意到后期陷入停滞，于是加入一张奖励耐心的高费炸弹卡；他们发现没人走军事路线，因为回报不够高，于是加入一张 6 费发展卡，桌面（tableau）中每个 Military 标记计 3 VP。

Detail Expander 智能体做的正是同样的事。在基础卡牌生成之后，它会审视这一批卡，并刻意打破套路：

- 至少一张把多个机制串联起来的组合技（combo）卡
- 至少一张在多数对局中平平无奇、但在合适情境下极具毁灭性的情境卡
- 至少一张奖励长期投入的高费后期卡

用 RFTG 的术语来说：基础批次可能产出一组平衡的 2–3 费世界，而细节扩展者随后会生成 Galactic Federation（6 费，每个 Development 标记计 2 VP）或 Pan-Galactic League（6 费，每个 Military 标记计 3 VP）。正是这些卡创造出了分化的策略。它们不是从平均中涌现的，而是从刻意制造的离群值中涌现的。

---

## 五卡生成规则

流水线完成首轮运行后，你会得到一个由五到八张彼此咬合、风格多样的卡牌组成的微型牌库——一个概念验证。但一款真正的游戏需要 30 张、50 张甚至 100 张卡。扩大规模会带来一个所有用过语言模型的人都眼熟的问题：上下文退化。

让 AI 做三件事，它表现出色；让它做 50 件，它会开局良好，但到第 14 项时，它已经忘了 12 项之前定下的约束。

系统强制执行一条硬性规则：单次请求生成的卡牌绝不超过五张。当要把牌库扩展到 50 张时，系统执行一个批量循环：顺序发起 10 次请求，每次生成五张。

不妨把它想成向朋友征求餐厅推荐。"给我推荐你最爱的三家"会换来三个精彩的精选；"报出 50 家餐厅"则会换来一阵慌乱，外加一份方圆十英里内所有连锁店的清单。语言模型的注意力跨度也是同样的道理。

![五卡批量循环]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/Five_Card_Batch_Loop.jpg)

*图 5. 批量循环每次生成五张卡。每个批次都会查看已有的完整卡牌列表，识别策略空白并加以填补。后续批次会自然演化出针对先前批次的反制。*

但五卡规则的作用不只是维持质量，它还创造了**渐进式改进**。

每个批次在生成之前都会分析已有的卡牌列表。如果前两个批次产出了廉价、激进的卡牌，第三个批次就会注意到这种失衡，生成高费防御卡来补偿。后续批次填补先前批次留下的策略空白。AI 甚至可能为它三分钟前生成的卡牌发明反制策略。

这恰恰是人类试玩测试中发生的事情：设计师打了几局，发现快攻策略独大，于是回到桌前设计一张能拖慢它的卡。批量循环把这一迭代周期从数周压缩到数分钟，但认知结构是一样的：观察生态系统，识别空白，设计反制。

---

## AI 能衡量什么

在谈论 AI 做不到什么之前，先让我具体说说它能做什么。

生成一副牌库后，系统会运行蒙特卡洛树搜索（Monte Carlo Tree Search, MCTS）模拟：让 AI 智能体用这副牌库自我对弈数百局。MCTS 不是语言模型，而是一种通过探索决策树来寻找制胜策略的规划算法。

在我为测试而设计的一款 30 张卡的引擎构筑（engine builder）原型上，MCTS 智能体学会了在早期购买生产卡、中期建设转化基础设施、后期购买计分卡。它对随机玩家赢得了 90% 的对局。AI 不只是平衡了这些卡牌，它还发现了设计师原本设想的策略。

90% 的胜率是一个有意义的信号。它告诉设计师："你的资源经济具有策略深度，存在一条可学习的曲线，生产链是行之有效的。"如果 MCTS 胜率是 50%，那就意味着这些卡牌在策略上可以互换，没有什么可学的，经济是扁平的。

这是卡牌设计的机械性一面，AI 处理得很好：成本曲线、资源平衡、优势策略检测、统计公平性。机器可以打上一千局游戏，然后告诉你 A 卡在 60% 的对局中获胜，而军事策略需要更强的回报。

---

## AI 做不到什么

![AI 看得到的与 AI 看不到的]({{ site.baseurl }}/images/how-ai-actually-designs-a-card/What_AI_Cannot_See.jpg)

*图 6. 系统能衡量的东西（结构维度、平衡指标、趣味评分）与它看不到的东西（四个朋友围着桌子开怀大笑）之间的鸿沟。架在两者之间的桥梁，是你。*

但它无法衡量乐趣。

我在[《生成式桌游设计理论》]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)一文中深入探讨过这个问题，此后这个结论只会愈加清晰：AI 无法*体验*乐趣。它从未感受过险胜时刻的兴奋、一记精妙组合技带来的满足，或是成功唬住对手时的社交电流。它没有品味，也没有情感。

但这里有个微妙之处：要识别产生乐趣的*设计模式*，并不需要亲身感受乐趣。隐藏信息制造紧张感，多个真实选项创造自主感，不断升级的赌注制造戏剧性。AI 可以从你的回合结构和机制选择中检测出这些结构性要素。桥梁工程师无须感受美，也能懂得让桥梁显得优雅的数学。

AI 做不到的是预测炼金术。同一个机制，能让一群人获得绝妙的紧张体验，换一群人却可能索然无味。算法可以保证数学上的公平，可以确保没有任何单一策略破坏生态系统，可以通过胜率检测出某张卡过强，或者发现生产链在游戏中后期停滞，又或者军事路线回报不足。

它无法告诉你打出那张卡时感觉是否令人满足；无法衡量一场胶着对局中的紧张感——两名玩家在同一回合都伸手去选同一个阶段；无法模拟生产引擎在第五回合突然齿轮咬合、开始运转的喜悦，或是眼睁睁看着对手在你差一点军力的情况下定居 Rebel Base 的痛苦。

当年我与 RFTG AI 对弈，败北之所以令人沮丧，并不是因为 AI 下得最优，而是因为*游戏本身*制造了那些最优决策会催生戏剧性结果的局面。AI 恰好在我的生产世界满载而对手的生产世界空空如也的那一刻选择了生产阶段；AI 在恰好把自己推过分数阈值的那一回合消耗货物换取 VP。那些时刻不是 AI 设计的，是 Tom Lehmann 设计的，AI 只是把它们打了出来。

数学确保游戏能运转，而人类设计师确保游戏值得玩。平衡的游戏与令人难忘的游戏之间的这种区别，正是人类角色得以长存之处。

在 2021 年的系列文章中，我绘制了 RFTG 架构的每一个组件：[游戏模型]({{ site.baseurl }}/game-architecture-card-ai-1)、[动作引擎]({{ site.baseurl }}/game-architecture-card-ai-2)、[神经网络 AI]({{ site.baseurl }}/game-architecture-card-ai-3)。除了胶着对局带来的那种感受，我把一切都形式化了。五年后，当我构建一个生成卡牌游戏的 AI 系统时，这道鸿沟并未弥合，只是被定义得更加精确。

有用的问题不是"AI 能理解乐趣吗？"，而是"AI 能识别那些往往能产生乐趣的设计模式，从而让你把精力集中在只有人类设计师才能提供的部分上吗？"第一个问题的答案是否定的，第二个问题的答案是肯定的。而这才是唯一重要的问题。

AI 负责管理地图，人类则在真实的地形中跋涉。地图能告诉你有哪些路、哪些路高效、哪些路通向死胡同，却无法告诉你哪条路上有让你驻足凝望的风景。这是设计师的工作，过去如此，将来也永远如此。

---

*系列下一篇：《当模式失效时》（When the Schema Breaks）——我们将用 13 款著名游戏对卡牌模式进行压力测试，看看哪些游戏会将其彻底击穿，并发现平衡的游戏与令人难忘的游戏之间的鸿沟，正是工具辅助设计中最重要的问题。*

---

## 参考文献

- Thomas Lehmann，[Race for the Galaxy（银河竞逐）](https://boardgamegeek.com/boardgame/28143/race-galaxy)，Rio Grande Games，2007。
- Keldon Jones，[RFTG AI 源代码](https://github.com/bnordli/rftg)（C，GPLv2），2009。
- Benny Cheung，[Game Architecture for Card Game Model（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)，bennycheung.github.io，2021。
- Benny Cheung，[Game Architecture for Card Game Action（第 2 部分）]({{ site.baseurl }}/game-architecture-card-ai-2)，bennycheung.github.io，2021。
- Benny Cheung，[Game Architecture for Card Game AI（第 3 部分）]({{ site.baseurl }}/game-architecture-card-ai-3)，bennycheung.github.io，2021。
- Benny Cheung，[Three Waves of Card Game Design Tools（卡牌游戏设计工具的三次浪潮）]({{ site.baseurl }}/three-waves-of-card-game-design-tools)，bennycheung.github.io，2026。
- Benny Cheung，[The Theory of Generative Board Game Design（生成式桌游设计理论）]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)，bennycheung.github.io，2026。
- Benny Cheung，[Generative Ontology: From Game Knowledge to Game Creation（生成式本体论：从游戏知识到游戏创造）]({{ site.baseurl }}/generative-ontology-from-game-knowledge-to-game-creation)，bennycheung.github.io，2026。
- Benny Cheung，[AI Playtesting: When Your Board Game Tests Itself（AI 试玩测试：当你的桌游开始测试自己）]({{ site.baseurl }}/ai-playtesting-when-your-game-tests-itself)，bennycheung.github.io，2026。
- [Cardiverse: LLM Card Game Prototyping（Cardiverse：LLM 卡牌游戏原型设计）](https://arxiv.org/abs/2502.07128)，EMNLP 2025。
- [Tabletop Simulator（桌面模拟器）](https://store.steampowered.com/app/286160/)，Berserk Games。
- [The Game Crafter](https://www.thegamecrafter.com/)，桌游按需印刷（print-on-demand）平台。