---
layout: post
title: "卡牌语法——教机器理解复杂卡牌游戏的规则"
date: 2026-03-30 00:00:00.000000000 -00:00
permalink: card-grammar-for-complex-card-games/
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
- Philosophy
- Nelson Goodman
category: post
comments: true
image: images/card-grammar-for-complex-card-games/post_cover.jpg
images:
  cover: images/card-grammar-for-complex-card-games/cover2x.jpg
  header: images/card-grammar-for-complex-card-games/header.jpg
  animated-cover: images/card-grammar-for-complex-card-games/animated-cover.webp
published: true
series: "Card Architecture"
series_order: 3
featured: true
featured_order: 3
---

<!--excerpt.start-->
我们构建了一条流水线：生成机制连贯的卡牌、以每批五张的方式扩展、直接导出到 Tabletop Simulator，并用锦标赛算法对平衡做压力测试。这听起来像卡牌游戏设计的未来。但当我们拿出 13 款最具影响力的已出版卡牌游戏，试图把它们的机制塞进流水线的五字段模式时，结果令人清醒。《Dominion》完美映射。《Sushi Go》轻松搞定。接着《展翅翱翔》（Wingspan）撑破了盒子，《殖民火星》把它压垮，《KeyForge》则彻底把它弄坏。这就是自动化卡牌设计撞上边界的故事、这些边界揭示了什么关于游戏复杂性的本质，以及解法为何不在于更好的算法，而在于对"一张卡究竟是什么"的完全不同的思考方式。
<!--excerpt.end-->

![卡牌语法——教机器理解复杂卡牌游戏的规则]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/post_cover.jpg)

这是[卡牌架构系列]({{ site.baseurl }}/three-waves-of-card-game-design-tools)的第 3 部分。在[第 1 部分]({{ site.baseurl }}/three-waves-of-card-game-design-tools)，我追溯了卡牌游戏工具从脚本到设计平台的演化。在[第 2 部分]({{ site.baseurl }}/how-ai-actually-designs-a-card)，我走进流水线内部，考察了卡牌设计中哪些部分是机械的、哪些不是。本文提出一个更难的问题：当流水线遇上真实游戏时会发生什么？

---

## 压力测试

本系列此前的文章描述了一条强大的卡牌生成流水线：一个读取游戏本体、生成具有真实机制深度的卡牌、通过批量循环扩展它们、并导出可玩原型的系统。它确实是令人印象深刻的技术。

但令人印象深刻的技术值得诚实的测试。为了理解这一方法的真实边界，我们拿出有史以来最具影响力的 13 款卡牌游戏——横跨七种不同的原型——并激进地尝试把它们的卡牌映射进流水线使用的基础五字段模式。

温习一下，那个模式是一个刚性的卡牌模板，有五个字段：卡牌名称、卡牌类型、效果文本、费用与战略角色。每张生成的卡都必须装进这个模板。如果你用索引卡做过原型，你懂那种感觉：卡上五行，写下 "Village / Action / 3 coins / Draw 1 card, +2 Actions"。干净、易读、完整。

问题是：当一款游戏的卡牌需要多于五行时，会发生什么？

结果自然分成了四个不同的覆盖层级，从完美适配到完全的结构错配。

![当前引擎的边界]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Schema_Coverage_Tiers.jpg)

_图：从 A 级到 D 级的覆盖悬崖——市场机会就住在那里。_

## A 级（完全）：五行就够了

| 层级 | 模式适配 | 会发生什么 | 游戏 |
|------|-----------|-------------|-------|
| **完全（A 级）** | 近乎完美 | 卡牌完美映射。每个机制细节在压缩中幸存。平衡测试反映真实游戏。 | Dominion, Star Realms, Sushi Go! |

![A 级游戏]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/CardGame_Tier_A_Games.jpg)

牌库构筑与简单的轮抽游戏是模式的舒适区。一张 Dominion 卡有名字（Village）、类型（Action）、费用（3 coins）与效果（"Draw 1 card, +2 Actions"）。索引卡上五行，无一遗漏。Star Realms、Sushi Go、Ascension：都是近乎完美的适配。

但这些游戏代表的是复杂性泳池的浅水区。

## B 级（部分）：眯着眼看规则

| 层级 | 模式适配 | 会发生什么 | 游戏 |
|------|-----------|-------------|-------|
| **部分（B 级）** | 方向上正确 | 核心机制可用，但次要系统丢失。平衡测试错过跨系统交互。 | 7 Wonders, Blood Rage, Res Arcana, Everdell |

![B 级游戏]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/CardGame_Tier_B_Games.jpg)

像 7 Wonders 和 Blood Rage 这样的游戏引入了模式无法干净表达的机制：基于纪元的卡牌分期、跨纪元的前置链条、绑定到特定棋盘位置的条件计分触发。你可以把这些信息硬塞进效果文本字符串，但模拟器最终只能眯着眼理解规则。模式不会崩溃。它优雅地退化，对它看不见的游戏部分视而不见。

## C 级（不足）：模板溢出

| 层级 | 模式适配 | 会发生什么 | 游戏 |
|------|-----------|-------------|-------|
| **不足（C 级）** | 约 60-70% 数据丢失 | 模式只抓到卡名和一个被压平的费用。经济引擎、标签系统与触发时机全部蒸发。 | Wingspan, Terraforming Mars, Race for the Galaxy |

![C 级游戏]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/CardGame_Tier_C_Games.jpg)

引擎构筑游戏是模式真正断裂的地方。索引卡上的五行远远不够。

![模式压缩危机]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Schema_Compression_Crisis.jpg)

试着在那张索引卡上写一张展翅翱翔的鸟卡。你需要食物费用（1 无脊椎 + 1 种子，或 2 任意）、栖息地限制（仅湿地）、蛋容量（2）、能力触发时机（激活时，而非打出时）、能力文本、巢型、翼展测量，以及用于轮末计分的奖励特征。那是至少八个结构化字段。你开始写得更小，把文字塞进页边，缩写直到卡牌无法阅读。模拟器面对同样的问题：一张鸟卡至少携带八个结构化数据字段，无法在不丢失约 60% 卡牌实际机制数据的情况下折叠进效果文本字符串。

殖民火星更糟。它的 208 张项目卡层层叠叠压着四个问题：一个标签系统，卡牌在其中触发其他卡牌的效果、横跨每位玩家的桌面阵列；三种卡牌颜色，其生命周期行为根本不同（一次性触发、反复触发、或触发后自毁）；为出牌设门的游戏状态前置条件（"需要 5% 氧气"）；以及一个双轨经济——六种资源各自同时拥有永久生产速率与可花费的库存。基础模式错过了游戏的一半以上。

在重引擎构筑游戏中，卡牌是社交的。它们彼此交谈。基础模式把每张卡当作孤岛上的孤立个体。

## D 级（崩坏）：结构错配

| 层级 | 模式适配 | 会发生什么 | 游戏 |
|------|-----------|-------------|-------|
| **崩坏（D 级）** | 约 75-85% 数据丢失 | 模式与游戏的卡牌模型结构性不兼容。不是缺字段的问题，而是根本的架构错配。 | KeyForge, Spirit Island, Gloomhaven |

![D 级游戏]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/CardGame_Tier_D_Games.jpg)

在最底层，模式不只是缺字段。它与游戏的卡牌模型结构性不兼容。

![D 级结构错配]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Tier_D_Structural_Mismatch.jpg)

Spirit Island 在一条完全不同的轴上崩坏：跨卡累积。每张能力卡携带元素符号（火、气、水），它们在一回合内打出的所有卡之间累积，解锁精魂面板上由阈值门控的天赋能力。你打出一张卡不只是为了它的印刷效果。你部分是为了它的元素图标打出它——那些图标可能在别处解锁一个完全不同的、更强的能力。模式没有这种每回合重置的元素经济的概念。

KeyForge 和 Gloomhaven 在另一条轴上崩坏模式：时间。一张 KeyForge 生物卡塞着四种不同时机触发的能力（打出时、收割时、战斗时、被摧毁时）。如果模拟器把卡牌文本当作单个脚本读取并同时触发一切，它就从根本上打破了游戏的物理现实。这是把一部四幕剧当成一个场景来演。Gloomhaven 推得更远：每张行动卡有两个独立的半张，选择上半张意味着下半张在该回合*不复存在*。当文本实际上是一个多层条件时序谜题时，标准的自然语言处理彻底失败。

问题不是我们需要一个更聪明的文本阅读器。问题是：读文本从来就不是正确的方法。

---

## 解法：卡牌语法

那么，技术如何演化以处理这些断裂？

显而易见的方法——给模式加 50 个新字段并祈祷最好结果——会让语言模型在提示词重量下崩溃，幻觉出垃圾。聪明的方法——构建更聪明的文本阅读器——因为我们刚描述的不可见时间维度而失败。而每款游戏都从零硬编码，在经济上不可能规模化。

解法要求系统对卡牌的思考方式来一次范式转变。

### 卡牌即游戏

旧的失败架构把卡牌当作在游戏规则内部弹来弹去的通用对象。游戏是一个盒子；卡是里面的一个部件。但对展翅翱翔、对殖民火星、对任何认真的引擎构筑游戏而言，卡牌*就是*游戏。卡牌模式不是待在本体内部。它几乎*就是*本体。

这一洞见翻转了整个架构。系统不再试图把复杂卡牌塞进通用模板，而是每款游戏声明自己的**卡牌语法**：一个结构化定义，说明在这款特定游戏中卡牌上存在哪些字段、这些字段何时触发、以及卡牌之间被允许如何通信。

### 卡牌智能的三层

一个卡牌语法有三层，每层解决压力测试中的一种特定失败模式：

![卡牌语法架构]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Card_Grammar_Three_Layers.jpg)

_图：一个三层可选扩展。基础游戏完全跳过它，零回归。_

**解剖**定义一款给定游戏中允许存在的严格物理字段。殖民火星的卡牌语法声明标签、卡牌颜色、需求与生产效果的字段。展翅翱翔的语法声明食物费用、栖息地、蛋容量与能力触发类型的字段。这些字段是数学上带类型的。标签就只是标签。费用就只是费用。系统永远不需要从上下文猜一段数据是什么意思。

**生命周期**定义允许事物触发的刚性时机窗口。这是对 KeyForge 与 Gloomhaven 时机问题的直接回答。语法不再把所有效果倒进一个文本块，而是声明不同的阶段：打出时触发的效果、激活时触发的效果、回合间触发的效果、游戏结束时触发的效果。模拟器检查触发类型，只在相应的游戏阶段触发匹配的能力。

**协同**定义卡牌之间被允许通信的严格规则。这正是殖民火星标签系统得以工作的原因：当一张科学标签卡被打出时，引擎检查桌面阵列中的所有卡牌以寻找匹配的触发。语法预先声明交互规则，模拟器就能在无需猜测的情况下监控跨卡效果。

### 乐谱，而非现场音乐会

哲学家 Nelson Goodman 在 1968 年的写作中形式化了一个区分，它在这里直接有用。Goodman 描述了**乐谱**与**演奏**之间的差别——谱面音乐与现场音乐会。乐谱是严格、无歧义的记谱。音乐会是丰富的、语境化的、活生生的执行。

![Goodman 连接]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Goodman_Paradigm_Shift.jpg)

_图：乐谱（左）与演奏（右）。语法是严格的记谱。生成的卡牌是活生生的执行。_

卡牌语法定义（解剖、生命周期、协同规则）是乐谱。任何符合该语法生成的卡牌都是一次合规的演奏。不同游戏的乐谱完全不同。展翅翱翔的乐谱与殖民火星的乐谱毫无相似之处。但底层系统——那位音乐家——能读懂所有乐谱。只要你提供逻辑健全的乐谱，引擎就能演奏任何游戏。

Goodman 称之为**有限区分**（finite differentiation）：记谱中的每个元素都截然分离、数学上定义、不可能混淆。旧的失败模式恰恰相反，饱受 Goodman 所谓**语义致密**（semantic density）之苦：标签、费用与需求之间的边界全被糊成一团密集的散文段落，而机器不具备解开这种致密性所需的活生生的人类经验。卡牌语法强制出形式系统进行计算所需的干净边界。

### 这对设计师意味着什么

当设计师说"我要做一款关于繁育恐龙的引擎构筑游戏"，系统不只是生成一段霸王龙咆哮的风味文本。它为这款新游戏提出一份具体的卡牌语法：一个解剖层，带食肉动物与食草动物的标签；一个生命周期层，轮末事件引发灭绝触发；以及一个协同层，处理食物链生产经济。生成的原型卡牌把严格带类型的效果内置其中。

而至关重要的是，因为系统理解底层语法，平衡测试引擎可以立即模拟这些机制。它会运行数百局自动化游戏并报告："你的火山爆发卡超模了。由于特定的协同语法，它的火焰标签在标准牌组构成下与其他四张食草动物卡意外触发了无限资源循环。"

市面上没有其他工具能在那种机制复杂度上完成生成、模拟、平衡测试与导出。结构化的卡牌模式就是护城河。


### 我们实际上如何学会游戏

让这一架构令人信服的是：它映照着人类大脑实际处理复杂桌游的方式。当你坐下来学殖民火星时，你不会在玩之前背下全部 208 张卡的文本。相反，你花最初 20 分钟学习那个游戏宇宙的特定*语法*：这些图标代表生产、那些边框代表一次性事件、这个阶段发生在那个阶段之前。

![在桌上映照人类认知]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/Human_Cognition.jpg)

一旦你的大脑内化了这套语法，有人递给你一张从未见过的卡，你会立刻知道如何处理它。你正在运行一个心智卡牌语法模拟器。平台把同样的认知过程形式化了。

---

## 自动化游戏测试实际揭示了什么

有了卡牌语法解决数据结构问题，平衡测试引擎终于可以在复杂游戏上做有意义的工作。用蒙特卡洛树搜索（MCTS）——AlphaGo 背后的同一算法家族——运行数百局模拟游戏，产出的结果足以让一个人类测试小组花数月才发现。

![自动化 MCTS 测试的力量]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/MCTS_Playtesting.jpg)

一个 30 卡的引擎构筑原型用 MCTS 测试，显示出 90% 的技能差距：策略型代理十次里赢随机代理九次。这个数字是经济深度的信号。它意味着生产链、资源转换与计分路径创造的是真正可学习的策略，而不只是运气抽牌。一个糟糕的原型显示策略与随机玩法五五开：游戏没有有意义的决策。50% 与 90% 之间的差距，就是"感觉靠运气"与"奖励精通"之间的差别。

但这里是诚实的边界。平衡引擎能识别某个策略在所有对局中获胜 60%。它能保证数学上的公平。它能精确定位破坏 meta 的那张卡并解释*为什么*：哪个标签触发哪条级联、哪条生产链占支配。

它无法度量乐趣。

它无法告诉你打出某张卡是否感觉满足。它无法模拟险胜的紧张。它无法量化把朋友骗进一笔糟糕交易时的社交体验。Goodman 会说：任何形式系统都必须拿**丰满性**（repleteness，活生生经验的完整致密丰富）换**可表述性**（articulateness，计算所需的锋利边界）。你无法兼得。卡牌数据库有锋利的边界：这张卡费用 3、这个策略赢 60%。玩游戏的体验——笑声、失误的懊恼——是致密的、语境化的、不可还原为人类的。

平台管理可表述的地图。设计师导航丰满的疆域。平台读乐谱。设计师感受管弦乐团。

---

## 结语

卡牌语法解决了结构问题。自动化游戏测试解决了迭代速度问题。但一位已经勾画 50 张卡、测试过两轮、打磨过核心循环的设计师，并不想让平台*生成*游戏。他们想让平台*分析*游戏。导入规则，跑 200 局模拟，告诉他们哪 12 张卡从未被使用。那才是有价值的工作。

![新的共生]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/New_Symbiosis.jpg)

平台承担平衡测试、策略验证与规则清晰度分析的计算劳作。设计师提供愿景、品味，以及对"什么让一款游戏值得玩"的判断。

电子表格时代结束了。在结构上理解、模拟并平衡复杂卡牌游戏的技术已经到来。而茁壮成长的设计师，将是那些理解"平衡的游戏"与"活着的游戏"之间差别的人。

---

*这是[卡牌架构系列]({{ site.baseurl }}/three-waves-of-card-game-design-tools)的第 3 部分，也是最终篇。关于这一分析的哲学基础，见[第 1 部分：卡牌游戏设计工具的三个浪潮]({{ site.baseurl }}/three-waves-of-card-game-design-tools)与[第 2 部分：AI 实际上如何设计一张卡]({{ site.baseurl }}/how-ai-actually-designs-a-card)。*

---

## 附录：压力测试中的 12 款游戏

这些是我们用五字段模式测试的卡牌游戏，按它们落入的覆盖层级分组。如果你对其中任何一款不熟悉，链接指向它们的 BoardGameGeek 页面——桌游的权威社区资源。

![卡牌游戏全层级示例]({{ site.baseurl }}/images/card-grammar-for-complex-card-games/CardGame_Collage_4_Tiers.jpg)

### A 级（完全）

**[Dominion](https://boardgamegeek.com/boardgame/36218/dominion)**（Donald X. Vaccarino，2008）。发明牌库构筑类型的游戏。玩家从相同的 10 张弱卡牌库起步，从共享市场购买越来越强的卡，把购入的卡洗进不断增长的牌库。每张卡都是一个简单的名字-类型-费用-效果元组。模式的完美匹配。

**[Star Realms](https://boardgamegeek.com/boardgame/147020/star-realms)**（Robert Dougherty & Darwin Kastle，2014）。一款带派系协同的双人牌库构筑游戏。卡牌与同派系的其他卡一起打出时获得附加能力，这稍稍拉伸了模式但没有弄坏它。条件逻辑留在效果文本之内。

**[Sushi Go!](https://boardgamegeek.com/boardgame/133473/sushi-go)**（Phil Walker-Harding，2013）。轻量卡牌轮抽游戏，玩家同时从传递的手牌中选卡。完全没有资源费用。费用字段为 null 而游戏照常运转。纯集合收集计分，模式轻松处理。

### B 级（部分）

**[7 Wonders](https://boardgamegeek.com/boardgame/68448/7-wonders)**（Antoine Bauza，2010）。跨越三个纪元的卡牌轮抽，带链式机制：在较早纪元建造某些卡，让你在较晚纪元免费建造特定卡。模式没有纪元分期或前置链条的字段，因此平衡测试器错过这些跨纪元交互。

**[Blood Rage](https://boardgamegeek.com/boardgame/170216/blood-rage)**（Eric M. Lang，2015）。维京主题的卡牌轮抽区域控制游戏。卡牌携带可变的战斗强度值与纪元特定的任务条件，编码着空间与时间计分触发——单个效果文本字符串无法干净地表示。

**[Res Arcana](https://boardgamegeek.com/boardgame/262712/res-arcana)**（Thomas Lehmann，2019）。每名玩家只有 8 张卡的紧凑引擎构筑游戏。每张卡把特定精髓类型转换为其他精髓或胜利点。多资源转换经济超出了扁平费用字段的表达能力。

**[Everdell](https://boardgamegeek.com/boardgame/199792/everdell)**（James A. Wilson，2018）。桌面阵列构筑游戏，玩家把小动物与建筑放入个人城市。卡牌有占用上限、季节可用性以及基础模式会丢失的跨卡配对奖励。

### C 级（不足）

**[Wingspan](https://boardgamegeek.com/boardgame/266192/wingspan)**（Elizabeth Hargrave，2019）。170 张独特的鸟卡，每张携带 8 个以上的结构化数据字段：多类型食物费用、栖息地放置限制、蛋容量、四种不同的能力触发时机，以及用于轮末计分的奖励特征标签。模式丢失每张卡约 60% 的机制数据。

**[Terraforming Mars](https://boardgamegeek.com/boardgame/167791/terraforming-mars)**（Jacob Fryxelius，2016）。208 张项目卡编码了一整个经济子系统：标签驱动的跨卡协同、为出牌设门的游戏状态前置条件、横跨六种资源类型的双轨生产/库存经济，以及三种生命周期行为根本不同的卡牌颜色。约 70% 数据丢失。

**[Race for the Galaxy](https://boardgamegeek.com/boardgame/28143/race-for-the-galaxy)**（Thomas Lehmann，2007）。每张卡承担三重职责：货币（弃掉以支付）、桌面阵列引擎（持续的生产与消费能力）、以及胜利点（条件性的终局计分公式）。弃一张卡去支付另一张卡*就是*资源系统——这种统一的卡牌经济在基础模式中没有任何表示。

### D 级（崩坏）

**[KeyForge](https://boardgamegeek.com/boardgame/257501/keyforge-call-of-the-archons)**（Richard Garfield，2018）。每张生物卡在不同时机触发器上（打出、收割、战斗、被摧毁）最多有四种不同的能力，而且选择家族这一元机制取代了资源成本的全部概念。模式与游戏的设计结构性不兼容。

**[Spirit Island](https://boardgamegeek.com/boardgame/162886/spirit-island)**（R. Eric Reuss，2017）。能力卡携带元素符号，它们在一回合内打出的所有卡之间累积，解锁精魂面板上由阈值门控的天赋能力。这一跨卡元素累积系统（每回合重置，不同于殖民火星的标签）没有任何模式表示。

**[Gloomhaven](https://boardgamegeek.com/boardgame/174430/gloomhaven)**（Isaac Childres，2017）。每张行动卡有两个独立的半张（上半与下半），由一个先攻数值分隔。玩家每回合选两张卡，使用一张的上半与另一张的下半。组合式的双半选择、基于先攻的回合顺序、以及作为耐力时钟的永久卡牌损失，产生了 85% 的压缩损失——所有被测游戏中最高的。

---

## 参考文献

- Nelson Goodman, *Languages of Art: An Approach to a Theory of Symbols*, Hackett Publishing, 1968.
- Jesse Schell, *The Art of Game Design: A Book of Lenses*, CRC Press, 3rd Edition, 2019.
- Geoffrey Engelstein and Isaac Shalev, *Building Blocks of Tabletop Game Design*, CRC Press, 2019.
- [LUDUS: Auto Battler Card Balancing](https://cdn.aaai.org/ojs/21550/21550-13-25563-1-2-20220628.pdf), AAAI 2022
- [Cardiverse: LLM Card Game Prototyping](https://arxiv.org/abs/2502.07128), EMNLP 2025
- Alexandre Verlaine, "Introducing Card Games in Ludii," UCLouvain Master's Thesis, 2025.
- Benny Cheung, [生成式本体：从游戏知识到游戏创造]({{ site.baseurl }}/generative-ontology-from-game-knowledge-to-game-creation), bennycheung.github.io, 2026.
- Benny Cheung, [AI 游戏测试：当你的桌游自己测试自己]({{ site.baseurl }}/ai-playtesting-when-your-game-tests-itself), bennycheung.github.io, 2026.
