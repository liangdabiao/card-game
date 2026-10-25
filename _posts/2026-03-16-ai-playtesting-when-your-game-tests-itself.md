---
layout: post
title: "AI 游戏测试——当你的桌游自己测试自己"
date: 2026-03-16 12:00:00.000000000 -00:00
permalink: ai-playtesting-when-your-game-tests-itself/
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- Automated Testing
- Game Design
- Tabletop Games
- Playtesting
- Monte Carlo Tree Search
- Game Architecture
- Board Games
category: post
comments: true
image: images/ai-playtesting-when-your-game-tests-itself/post_cover.jpg
images:
  cover: images/ai-playtesting-when-your-game-tests-itself/cover2x.jpg
  header: images/ai-playtesting-when-your-game-tests-itself/header.jpg
  animated-cover: images/ai-playtesting-when-your-game-tests-itself/animated-cover.webp
published: true
series: "Game Architecture"
series_order: 9
---

<!--excerpt.start-->
设计师在 Nova 里输入"测试我的游戏是否有平衡问题"。片刻之后，他们收到一份结构化的批评：哪个玩家座次拥有不公平的优势、游戏是否奖励策略性玩法，以及三个干预选项。不用做原型，不用招募测试者，不用电子表格。只是一场对话，外加一个每当你改动一个数字就会运行的反馈回路。这就是我们教一个系统玩桌游的故事——什么失败了，败得多惨，以及那次失败又意外地发明了什么。
<!--excerpt.end-->

![AI 游戏测试：当你的游戏自己测试自己]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Overview.png)

_图：自动化游戏测试流水线把一份结构化的游戏本体转化为自动化平衡分析、技能差距测量与规则清晰度评分——全程通过与 Nova 的对话完成。_

这是游戏架构系列的第 9 部分。在[第 5 部分]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)，我们演示了结构化的游戏生成。在[第 6 部分]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)，我们探索了生成式本体背后的理论。在[第 7 部分]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)，我们介绍了对话式 AI 联合设计师 Nova。在[第 8 部分]({{ site.baseurl }}/generative-ontology-from-game-knowledge-to-game-creation)，我们展示了从知识到创造的完整流水线。

但存在一个缺口。GameGrammar 能在几分钟内生成一款结构有效的游戏。Nova 能帮你在多个会话中打磨它。然而，在"设计已存在于纸上"与"我们知道它在桌上是否行得通"之间，横着每个设计师都要面对的那堵墙：做原型、拉朋友、约时间、手工记录结果，并在每次改动之后把整个流程重来一遍。

本文讲的就是我们如何拆掉那堵墙。

---

## 那堵墙：设计死去的地方

![第 2 阶段之墙]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Stage2_Wall.png)

_图：桌游设计流水线有九个阶段。第 2 阶段（迭代式游戏测试）是大多数业余设计搁浅之处。_

每个游戏设计师都知道那种感觉。你花了一个周末打造一款带 push-your-luck 机制的牌库构筑游戏。卡牌类型感觉对了。经济看似平衡。主题动人。然后现实袭来：你需要打印卡牌、招募四位同晚有空的朋友、讲解规则、玩三个 session、做笔记、改数字，然后全部重来。到第三次迭代，你的朋友们开始礼貌地"没空"，游戏躺进了抽屉。

桌游设计流水线有一个众所周知的瓶颈，而它不是创造力。生成想法、勾勒机制、甚至产出完整游戏本体的工具已经大幅加速。但判断一个设计是否平衡、是否具有策略趣味，仍然需要实体原型、玩家招募、观察与会后分析。这个过程持续数周到数月。它是大多数业余设计搁浅的地方，即便专业工作室也把大部分开发时间花在这里 [1]。

GameGrammar 的本体流水线已经自动化了概念生成、结构分析以及经 Nova 的对话式协同设计。而本体输出其实包含模拟器所需的一切。组件规格定义游戏对象。机制细节定义合法动作。计分公式定义如何获胜。平衡参数定义约束。游戏弧线定义回合结构。

数据就在那里。问题在于它能否被变得可执行。事实证明，可以。在解释如何做到之前，让我们先给你看它在实践中的样子。

---

## 设计师如何使用：与 Nova 的一场对话

整个游戏测试流水线经由 Nova——我们在[第 7 部分]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)介绍的对话式联合设计师——呈现。设计师从不直接看到解析器、代理或指标。他们看到的是一场对话。

{% include youtubePlayer.html id="8KCOMVEytK0" %}

_视频：GameGrammar AI 游戏测试：Nova 从一条自然语言请求编排整个测试流水线。_

### 设计循环

1. 设计师说："为这款游戏跑一次平衡性测试"
2. Nova 解析游戏规则，用随机代理模拟 50 局，并分析结果
3. Nova 呈现带推理链的结构化批评：结论（"Love Letter 显示显著的先手优势"）、观察、数据、机制解释与竞争性影响
4. 出现决策层级：**结构层（重构）**建议如轮换先手、**调参层**建议如调整卡牌数值，以及**分叉**以探索备选设计
5. 设计师选择一项干预，Nova 提出本体变更，并重跑测试以验证修复

![Nova 测试会话]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Nova_Session.png)

_图：Nova 在 GameGrammar 内呈现游戏测试结果。批评推理链通过自然对话浮出平衡发现、技能差距测量与干预选项。_

对比传统工作流：改一个数字、重印卡牌、招募玩家、约一个晚上、玩一遍、做笔记、汇总结果。过去数周的迭代循环，如今变成一场对话之内的连续反馈回路。

### 测试历史

![测试历史标签页]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_History.png)

_图：测试标签页显示运行历史，游戏日志可展开。设计师可以追踪平衡指标如何随设计迭代演化。_

每次测试运行都会连同其指标一起保存，设计师可以追踪平衡数字如何随设计迭代变化。那次计分微调修复了先手优势吗？给起始手牌加一张牌减少了僵局吗？历史为设计决策及其测得的影响提供了量化记录。

你已经看到了体验的样子，接下来让我们看看引擎盖下如何运作。

---

## 方法：LLM 翻译，算法对弈

在抵达生产系统之前，我们探索过三种方法。先快速总结落选的两种，以及它们为什么仍然重要。

**直接模拟器**把本体解析为确定性游戏引擎。它对卡牌游戏效果很好，第一天就发现了真实的平衡信号，但模式匹配解析器在任何超出简单机制的东西上都会崩溃。瓶颈是解析器，而不是游戏。

**让 LLM 玩游戏**听起来很有前景：每回合把本体与游戏状态喂给 LLM，让它选一个动作，无需形式化规则。我们构建了七种玩家原型（激进型、谨慎型、引擎构筑型、新手型等）。但在每一次延长实验中，**LLM 代理的表现都差于随机**（100 局后技能差距 -39%）。这与 GameBench [2] 和 GTBench [3] 的发现相互印证。LLM 无法在多个回合间保持一致的战略推理 [4]。不过，这次失败变成了一项创新：虽然 LLM 无法策略性地玩游戏，但它们的困惑模式惊人地一致。当 LLM 系统性地回避某个机制时，该机制的描述多半是含糊的。我们意外地造出了一台**规则清晰度分析器**——任何现有的游戏设计工具包都不提供的东西。

### 胜出的混合方案

![混合架构]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Hybrid_Architecture.png)

_图：胜出的方法：用 LLM 做它擅长的事（把自然语言翻译为形式规则），用传统游戏 AI 做它擅长的事（通过搜索找到最优玩法）。_

生产系统让每种 AI 各尽其长。LLM 读取你游戏的机制描述并把它们翻译为形式化的游戏动作，对 Love Letter 的机制达到约 90% 的覆盖率 [5]。随后一台确定性引擎以完美的规则执行来运行那些动作。

策略性玩法方面，我们使用 MCTS（蒙特卡洛树搜索）——一种成熟的游戏 AI 技术 [6]——它通过采样对手可能持有的手牌、跨这些可能性搜索最佳走法来处理隐藏信息。在 Love Letter 上，MCTS 对随机玩法胜率 81%，技能差距 +62.4%，且在重复运行中保持一致。它完全运行在本地计算上、零 API 调用，设计师可以在每次改动之后重跑分析。

关键洞见是每种代理类型服务于不同目的：

| 代理类型 | 技能差距 | 它测量什么 |
|-----------|-----------|-----------------|
| **MCTS** | +62.4% | 你的游戏是否奖励策略性玩法 |
| **LLM** | -39% | 哪些规则令人困惑（清晰度信号） |
| **随机** | 0%（基线） | 平衡与统计公平性 |

负的技能差距意味着 LLM 输给随机玩法的次数多于获胜。换句话说，尝试对游戏进行推理反而损害表现。LLM 的失败不是因为它不聪明。它失败是因为无法跨回合保持一致的游戏状态：它忘记哪些牌已经打出、错误应用一回合前还理解的规则、对自己正确的策略反复怀疑。一个完全不需要记忆的随机代理，仅仅通过避免这些复利式错误就胜过了它。

MCTS 证明游戏是否奖励技能。LLM 揭示哪些规则令人困惑。每种代理类型在对方的工作上都一无是处。

---

## 系统能检测什么

游戏测试流水线产出四类分析。前三个在初始 LLM 解析（按设计版本缓存）之后完全运行于本地计算。第四类使用 LLM 代理做规则清晰度评分。

### 平衡指标（随机代理）

来自 100+ 局随机代理自对弈的六项统计指标：

| 指标 | 能抓到什么 |
|--------|----------------|
| 座次优势 | 先手/末手玩家赢得太频繁 |
| 策略多样性 | 某个动作支配所有其他动作 |
| 死动作 | 没人使用的游戏元素 |
| 游戏时长 | 太短、太长或方差太大 |
| 出局时机 | 玩家过早被淘汰 |
| 僵局率 | 永远打不完的游戏 |

### 技能差距（MCTS 代理）

MCTS 代理坐一半座位，随机代理坐另一半。胜率差衡量游戏是否奖励策略性玩法。高于 +50% 是强劲的战略深度。低于 +20% 可能显得随机。负值意味着策略性玩法适得其反——某种东西坏了的信号。至关重要的是，这一分析运行得足够快：设计师在本体里改一个数字，就能在同一场对话内复查。

### 拓扑平衡（空间类游戏）

![拓扑驱动的平衡]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Topology_Balance.png)

_图：对 Catan Simple 的三个受控实验把座次优势分解为先手成分与拓扑成分，揭示棋盘连通性比行动顺序更能决定结果。_

这是我们最兴奋的发现。对于具有结构化棋盘拓扑的游戏，模拟器能检测从棋盘形状本身涌现的平衡性质。我们在一个简化的 Catan 上（7 个六边形区域、2 名玩家、先控制 4 个区域者胜）跑了三个实验：

**表 II：拓扑驱动的平衡（Catan Simple，各 100 局）**

| 实验 | P0 起点（邻居数） | P1 起点（邻居数） | P0 胜率 | P1 胜率 |
|-----------|---------------------|---------------------|---------|---------|
| 基线 | Forest (3) | Quarry (2) | 76% | 24% |
| 交换起点 | Quarry (2) | Forest (3) | 45% | 55% |
| 对称起点 | Forest (3) | Mountain (3) | 49% | 51% |

关键发现：**76% 的优势是拓扑性的，而非时序性的。**当起点被交换时，优势反转。当两名玩家都从等连通性的区域出发时，游戏近乎平衡（49/51）。棋盘的图结构对结果的决定作用远大于行动顺序。

这是盯着地图看的人类设计师很可能会漏掉的东西。看六边形网格时，你不会自然而然地去数邻接度。模拟器会。而且设计干预从根本上是不同的：不是调整行动顺序或行动成本，而是修复棋盘的连通性。

### 规则清晰度（LLM 代理）

LLM 代理以不同原型玩游戏（包括一个故意误读规则的"新手"），其困惑模式产出按机制的清晰度评分：

**表 III：按机制的清晰度（Love Letter）**

| 机制 | 评分 | 信号 |
|-----------|-------|--------|
| 回合开始抽一张牌 | 10.0 | 强制性，总是清晰 |
| 神父：看对手手牌 | 9.9 | 清晰，常被选择 |
| 打出一张牌 | 9.6 | 不确定性低 |
| 卫兵：猜对手的牌 | 9.0 | 很少被选 |
| 王子：强制弃牌再抽 | 8.6 | 接近零使用 |
| **男爵：比较手牌** | **8.5** | **在 135 次机会中从未被选择** |

男爵的比较机制每次都得分最低。它需要对卡牌相对数值进行推理，这是 Love Letter 中最复杂的规则。LLM 系统性地回避它。对设计师而言，这是具体、可行动的反馈：男爵的规则是你规则书中最需要花力气澄清的地方。

---

## 现在能用的：游戏层级覆盖

![游戏层级覆盖]({{ site.baseurl }}/images/ai-playtesting-when-your-game-tests-itself/AI_Playtesting_Tier_Coverage.png)

_图：模拟器覆盖第 1 至第 5 层级，约为现代桌游设计的 80-85%。每个层级新增机制类别。_

模拟器支持第 1 至第 5 层级的游戏，覆盖约 80-85% 的现代桌游设计。

**表 IV：机制层级覆盖**

| 层级 | 复杂度 | 例子 | 关键机制 |
|------|-----------|----------|---------------|
| 1 | 轻量卡牌游戏 | Love Letter, Coup | 抽、出、比较、淘汰 |
| 2 | 资源/市场游戏 | Splendor, Star Realms | 资源池、市场、桌面阵列 |
| 2.5 | 骰子/牌库构筑 | Farkle, Dominion | 骰子池、push-your-luck、牌库循环 |
| 3 | 工人放置、合作 | Lords of Waterdeep, Pandemic | 行动格、封锁、共享资源、团队胜负 |
| 4 | 同时行动、卡牌轮抽 | Sushi Go | 分阶段选择、轮抽传递、集合收集计分 |
| 5 | 空间、区域控制 | Catan | 棋盘图、区域所有权、参数化动作 |

### 尚不支持的

我们信奉对局限保持诚实。以下类别仍不支持，Nova 的覆盖门控会在设计师尝试之前告知这一点：

| 类别 | 例子 | 缺什么 |
|----------|----------|----------------|
| 路线修建 | Ticket to Ride | 寻路、最长路径计分 |
| 拍卖/竞标 | Power Grid, For Sale | 出价状态、拍卖结算 |
| 轮取牌 | The Crew, Hearts | 轮次结构、跟牌、王牌 |
| 复杂触发 | Wingspan, Terraforming Mars | 级联效应、条件激活 |
| 非对称能力 | Root, Vast | 每位玩家独有的动作与胜利条件 |

这些缺口代表真正的架构性断裂：复杂触发需要事件系统、非对称能力需要按玩家的动作集、路线修建需要寻路算法。每一项都是未来的工程史诗，而非一次配置更改。目标不是模拟有史以来的每款游戏。而是把 GameGrammar 能良好生成的那些游戏覆盖得足够好，让游戏测试变成一场对话，而不是一个项目。

---

## 诚实的挑战

没有系统没有局限，而我们相信对自己的局限保持透明，只会强化而非削弱自动化游戏测试的正当性。

**LLM 解析是非确定性的。**对同一游戏的两次独立解析会产出略有不同的规则解读。修正是务实的：解析一次、缓存结果，并把缓存用于所有后续模拟。相同缓存加相同种子等于相同结果。但设计师应该知道：初始解析定义了模拟器所玩的那款游戏，它可能并不完美匹配你的意图。

**评估使用的是简化版游戏。**我们的测试夹具是已出版游戏的简化版：8 种卡牌类型的 Love Letter、7 个区域的 Catan、有限卡池的 Dominion。全复杂度的游戏（200+ 项目卡的 Terraforming Mars、非对称角色牌库的 Gloomhaven）会以我们尚未测试过的方式给系统施压。

**规则清晰度尚未对照人类验证。**男爵的比较机制始终得分最低——这符合直觉，因为它客观上就是 Love Letter 中最复杂的规则。但我们尚未把自动化清晰度评分与真实人类的困惑评分做比较。这个指标看起来合理且有用，但尚未正式验证。

**模拟器玩的是"解析后的游戏"，不是"意图中的游戏"。**如果解析器误读了一个机制，平衡发现就是对错误游戏的真实发现。模式解析器在 Love Letter 中人为制造的 P3 优势正是这类错误。LLM 解析器 90% 的准确率很高，但剩下的 10% 可能产生微妙的扭曲。

这些是已知的局限，不是被隐藏的局限。Nova 的覆盖门控与模拟器的确定性重放，使它们在实践中可控。

---

## 结语

这项研究开始时，问题很简单：AI 能否通过阅读一份结构化的设计描述来学会玩一款桌游？答案比"能"或"不能"更微妙。

**MCTS 能玩，而且玩得好。**Love Letter 上 +62.4% 的技能差距，纯本地计算，零 API 调用。传统游戏 AI 算法，一旦被喂入从自然语言解析出的结构化游戏定义，就能产出可靠的战略玩法。游戏奖励技能，而 MCTS 找到了技能。

**LLM 不能玩，而这正是意义所在。**100 局后技能差距 -39%。但它们的困惑模式度量着其他任何工具都无法度量的东西：规则清晰度。一个系统性回避某机制的 LLM，是在告诉你那个机制难以理解。这颠倒了标准叙事——在那里，LLM 玩游戏的失败是一个待克服的缺陷。而在我们的系统里，失败即特性。

**结构揭示直觉错过的东西。**3 邻居区域对 2 邻居区域的 76% 胜率。没有任何人类设计师盯着地图能看出来。模拟器能。而设计干预（修复棋盘连通性）与设计师原本会尝试的（调卡牌成本）在根本上是不同的。

目标从来不是取代人类测试者。人类游戏测试揭示社交动态、情绪弧线与任何模拟器都无法捕捉的"手感"。目标是压缩迭代循环。设计变更与反馈之间不再是数周，这个循环如今可以放进一场与 Nova 的单一对话。改一个参数，重跑分析，看影响，再迭代。对第 1-5 层级的游戏而言，这个连续反馈回路把游戏测试从一个项目变成一场对话。

如果你想在自己的设计上尝试自动化游戏测试，[GameGrammar](https://gamegrammar.dynamindresearch.com) 正在公测，免费层包含经 Nova 的平衡分析。已经有游戏了？你不必从零生成——把你现有的设计描述给 Nova，就能在你构建的游戏上运行平衡分析。

---

## 系列导航

* [解锁桌游本体论的奥秘（第 4 部分）]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology)
* [GameGrammar 亮相：AI 驱动的桌游设计（第 5 部分）]({{ site.baseurl }}/introducing-gamegrammar-ai-powered-board-game-design)
* [GameGrammar：生成式桌游设计理论（第 6 部分）]({{ site.baseurl }}/gamegrammar-the-theory-of-generative-board-game-design)
* [Nova：学习你品味的 AI 联合设计师（第 7 部分）]({{ site.baseurl }}/nova-the-ai-co-designer-that-learns-your-taste)
* [生成式本体：从游戏知识到游戏创造（第 8 部分）]({{ site.baseurl }}/generative-ontology-from-game-knowledge-to-game-creation)
* **>>** [AI 游戏测试：当你的游戏自己测试自己（第 9 部分）]({{ site.baseurl }}/ai-playtesting-when-your-game-tests-itself)

---

## 参考文献

[1] Geoffrey Engelstein and Isaac Shalev. [*Building Blocks of Tabletop Game Design: An Encyclopedia of Mechanisms*](https://www.routledge.com/Building-Blocks-of-Tabletop-Game-Design/Engelstein-Shalev/p/book/9781138365490). CRC Press, 2020.
  - 综合的机制分类学与迭代式游戏测试的挑战

[2] D. Costarelli et al. [*GameBench: Evaluating Strategic Reasoning Abilities of LLM Agents*](https://arxiv.org/abs/2406.06613). arXiv:2406.06613, 2024.
  - 记录 LLM 的系统性失败，包括状态追踪丢失与规则幻觉

[3] Z. Duan et al. [*GTBench: Uncovering the Strategic Reasoning Limitations of LLMs via Game-Theoretic Evaluations*](https://arxiv.org/abs/2402.12348). NeurIPS 2024.
  - 发现带思维链的 LLM 在对 MCTS 时普遍失败

[4] Y. Hu et al. [*Large Language Models and Games: A Survey and Roadmap*](https://arxiv.org/abs/2402.18659). arXiv:2402.18659, 2024.
  - 综合综述，包括面向 LLM 游戏交互的后状态文本协议（After-State Text Protocol）模式

[5] C. Becker et al. [*Boardwalk: Towards a Framework for Creating Board Games with LLMs*](https://arxiv.org/abs/2508.16447). SBGames 2025.
  - LLM 生成桌游代码，达到 55.6% 的无错率

[6] R. Coulom. [*Efficient Selectivity and Backup Operators in Monte-Carlo Tree Search*](https://link.springer.com/chapter/10.1007/978-3-540-75538-8_7). CG 2006, LNCS 4630, Springer, 2007.
  - 提出 UCT 选择——我们 MCTS 代理的基础

[7] E. Piette et al. [*Ludii: The Ludemic General Game System*](https://arxiv.org/abs/1905.05013). arXiv:1905.05013, 2020.
  - 需要形式化游戏描述的通用游戏系统（而我们的系统接受自然语言）

[8] Benny Cheung. [*Generative Ontology: When Structured Knowledge Learns to Create*](https://arxiv.org/abs/2602.05636). arXiv:2602.05636, Feb 2026.
  - 关于生成式本体框架及消融研究的正式论文

[9] [*GameGrammar*](https://gamegrammar.dynamindresearch.com). Dynamind Research, 2026.
  - 集成自动化游戏测试的桌游设计平台
