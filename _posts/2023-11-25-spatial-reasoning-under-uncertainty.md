---
layout: post
title: 不确定性下的空间推理
date: 2023-11-25 12:00:00.000000000 -00:00
permalink: spatial-reasoning-under-uncertainty
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- Spatial Reasoning
- Philosophy
- Mereology
- Mereotopology
- AI
category: post
comments: true
image: images/spatial-reasoning-under-uncertainty/cover2x.jpg
images:
  cover: images/spatial-reasoning-under-uncertainty/cover.jpg
  header: images/spatial-reasoning-under-uncertainty/header.jpg
published: true
series: "Spatial Reasoning"
series_order: 4
---

<!--excerpt.start-->
空间推理是许多现实世界应用的关键环节，包括城市规划、环境监测和交通运输物流。它涉及对空间数据的处理与解读，以理解和分析给定空间中各对象之间的关系。传统 GIS 系统和几何处理算法通常假设空间数据是精确而准确的。然而，现实世界中的空间数据往往并不精确，将这种不确定性纳入空间推理，可以带来更稳健、更可靠的结果。像人类认知能力那样处理不精确空间数据的能力，是空间推理与分析领域中一项宝贵的资产。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以将本文当作一期**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主播既风趣又富有洞见。

{{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/Podcast_Spatial_Reansoning_under_Uncertainty.mp3
{% include open-embed.html %}
----

![不确定性下的空间推理抽象图]({{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/Spatial_Reasoning_under_Uncertainty_Abstract.jpg)

_图：展示不确定性下空间推理的复杂性。（图片来源：Stable Diffusion AI）_

在整篇文章中，我们将以虚构小镇 "Sensorville" 为例，说明在空间决策过程中考虑不确定性的重要性。读完本文，你将对用于驾驭不确定性下空间推理的方法与技术，以及它们在各个领域的潜在应用，获得有价值的洞见。

> 在我们此前题为 [AGI 中的空间推理——来自哲学视角的洞见]({{ site.baseurl }}/spatial-reasoning-in-agi) 的讨论中，我们强调了空间推理能力对于实现通用人工智能（AGI）的关键作用。显然，当前的大语言模型（LLM）在空间理解方面存在不足，而空间理解对于解决众多实际挑战至关重要。目前的趋势正转向引入知识图谱（Knowledge Graph）技术，以增强 LLM 与真实世界之间更具体的联系。对于密切关注这一进展的读者，那篇文章就如何构建在不确定性中纳入空间关系的知识图谱提供了宝贵的见解。


### 空间数据不确定性

在处理城市规划、土地利用决策和基础设施需求等现实世界应用时，空间数据不确定性是需要考虑的关键方面。与教科书式的 GIS 和计算几何不同，现实世界的数据往往包含不确定性，可能显著影响分析与决策过程。通过理解不确定性的来源，我们可以决定是预先消除它们，还是引入空间推理技术在处理过程中加以解决。

不确定性可能来自许多源头，包括测量误差、属性不一致和几何不确定性。数据收集过程很容易混入不完整或缺失的数据，数据质量问题与数据集成挑战也都会给空间处理任务带来不确定性。

当空间数据无法通过消除不确定性成因来预先解决时，我们就必须制定管理它的策略。为了更直观地理解这个问题，让我们讲讲 Sensorville 的故事。

### 想象小镇的故事——Sensorville

从前，在一座名为 Sensorville 的繁华城市里，一群城市规划者面临着一顶挑战：安装一套新的应急传感器网络，以保护居民免受地震、洪水和火灾等各种潜在灾害的侵害。城市被划分为许多不同的区域，每个区域都有其独特的特征、风险等级和空间布局。

![Sensorville 市的城市规划者监测]({{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/SP_Mereology_City_Planners_Monitoring.jpg)

_图：描绘 Sensorville 市中的一群城市规划者。（图片来源：Stable Diffusion AI）_

规划团队负责人 Mereo 博士是空间推理与分体论（mereology）领域的专家。他知道，只要理解区域与其部分之间的关系，并对环境的空间属性进行推理，就能在传感器的布设位置上做出明智的决策。

我们将在后文中看看故事如何展开。

## 基于分体论（Mereology）与分体拓扑学（Mereotopology）的空间推理

空间推理是一个更宽泛的概念，涵盖理解、操作对象及其空间关系并从中得出结论的能力。它本身并没有内置的理论形式化体系。回顾该领域的历史发展，我们发现[分体论（Mereology）](https://plato.stanford.edu/entries/mereology/)与[分体拓扑学（Mereotopology）](https://plato.stanford.edu/entries/location-mereology/)提供了必要的理论框架，可用于表示和推理空间关系，并处理空间数据中的不确定性。在本节中，我们将以最少的符号记法介绍这些概念，并讨论它们在处理空间不确定性方面的潜在应用。

### 分体论：部分与整体关系的研究

分体论（Mereology）是形式本体论的一个分支，研究部分与整体之间的关系。它提供了一组公理和原则，用以描述和推理空间对象的组合与分解。分体论中的一些关键原则包括自反性、反对称性、传递性和补充性。

取谓词常元 $P$，将其解释为部分关系（parthood relation）。

(P.1)  自反性（Reflexivity）：每个对象都是它自身的一部分。$A \le A$，用谓词 $P(x,x)$ 表示
  - 例子：你的手指是你手指的一部分。

(P.2)  反对称性（Antisymmetry）：如果对象 A 是对象 B 的一部分，且对象 B 是对象 A 的一部分，则对象 A 与对象 B 是同一个对象。$(A \le B \land B \le A) \rightarrow A = B$，用谓词 $((P(x, y) \land P(y, x)) \rightarrow x = y$ 表示
  - 例子：你的手指是你手的一部分，但你的手不是你手指的一部分。

(P.3)  传递性（Transitivity）：如果对象 A 是对象 B 的一部分，且对象 B 是对象 C 的一部分，则对象 A 是对象 C 的一部分。$(A \le B \land B \le C) \rightarrow A \le C$，用谓词 $((P(x, y) \land P(y, z)) \rightarrow P(x,z)$ 表示
  - 例子：你的手指是你手的一部分，你的手是你身体的一部分，因此你的手指是你身体的一部分。

给定基本分体论公理 (P.1) - (P.3)，我们可以通过定义引入更多分体论谓词。例如，

- 相等（Equality）：两个实体 $x$ 与 $y$ 之间的相等（$EQ$）定义为：$x$ 是 $y$ 的一部分（$P(x, y)$），且 $y$ 是 $x$ 的一部分（$P(y, x)$）。
- 真部分（Proper Parthood）：两个实体 $x$ 与 $y$ 之间的真部分（$PP$）定义为：$x$ 是 $y$ 的一部分（$P(x, y)$），且 $x$ 不等于 $y$（$\lnot (x = y)$）。
- 真扩展（Proper Extension）：两个实体 $x$ 与 $y$ 之间的真扩展（$PE$）定义为：$y$ 是 $x$ 的一部分（$P(y, x)$），且 $x$ 不等于 $y$（$\lnot (x = y)$）。
- 重叠（Overlap）：两个实体 $x$ 与 $y$ 之间的重叠（$O$）定义为：存在一个实体 $z$，使得 $z$ 既是 $x$ 的一部分也是 $y$ 的一部分（$P(z, x) \land P(z, y)$）。
- 下覆盖（Underlap）：两个实体 $x$ 与 $y$ 之间的下覆盖（$U$）定义为：存在一个实体 $z$，使得 $x$ 与 $y$ 都是 $z$ 的一部分（$P(x, z) \land P(y, z)$）。

这些附加谓词有助于描述分体论系统中实体之间更复杂的关系。

![分体论基本原理]({{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/Merelogy_Basic_Principles.jpg)

_图：这些关系的一个直观模型，其中部分关系 $P$ 被解释为空间包含。阅读图中的表时，假设 $y$ 为 $x_4$，那么 $x_n$ 与 $y$ 之间的分体关系将以 $+$（真）或 $-$（假）显示。（图片来源：Mereology, Stanford Encyclopedia of Philosophy）_

分体论原则为理解和形式化空间领域中的部分-整体关系提供了坚实的基础。通过应用这些原则，我们可以发展出一种严谨的空间推理方法，以应对现实世界空间数据的复杂性。

例如，自反性原则 (P.1) 指出，每个空间对象都是它自身的一部分。这一原则可以用来理解：城市中的任何区域（例如一个街区）也是城市整体的一部分。类似地，传递性原则 (P.2) 使我们能够跨多个粒度层次推理空间关系。如果一栋建筑是某个街区的一部分，而该街区是某个城区的一部分，那么我们可以推出：这栋建筑也是该城区的一部分。

反对称性原则 (P.3) 确保只有当两个空间对象互为对方的一部分时，才能将它们视为相等。这一原则在空间数据管理与分析中很有用，因为它通过防止空间对象的重复，帮助维护空间数据的一致性和完整性。

通过将分体论原则与其他空间推理技术（如 _拓扑关系_）结合使用，我们可以构建一个强大的框架来分析复杂的空间场景。

### 分体拓扑学：分体论与拓扑学的结合

作为一种形式化框架，分体拓扑学（Mereotopology）通过将分体论（部分-整体关系）的优势与拓扑学（研究在连续变换下保持不变的空间性质）相结合，为空间推理提供了一种全面的方法。这种整合使得对象之间空间关系的表示更加细腻、更具表达力，从而实现更精确的空间分析与决策。

分体拓扑学的关注点在于空间区域、其组成部分以及它们之间拓扑关系的表示与推理。区域是分体拓扑学中的基本构建块，它们之间的关系通过考察其边界与内部的性质来确定。

通过将分体论概念（如部分关系与重叠）与拓扑概念（如连接与分离）相结合，分体拓扑学为刻画复杂的空间关系提供了强有力的手段。

以下是几种关键的分体拓扑关系（实际上还有更多）。例如：

1.  部分关系（Parthood，$\leq$）：区域 $A$ 是区域 $B$ 的一部分（$A \leq B$），当且仅当 $A$ 是 $B$ 的子区域。该关系是自反的（$A \leq A$）、反对称的（若 $A \leq B$ 且 $B \leq A$，则 $A = B$），并且是传递的（若 $A \leq B$ 且 $B \leq C$，则 $A \leq C$）。

2.  真部分关系（Proper parthood，$<$）：如果 $A$ 是 $B$ 的一部分，且 $A$ 不等于 $B$，则区域 $A$ 是区域 $B$ 的真部分（$A < B$）。

3.  重叠（Overlap，$O$）：两个区域 $A$ 与 $B$ 重叠（$A O B$），当且仅当它们拥有共同的子区域。如果两个区域共享一个公共部分，就称它们重叠。

4.  连接（Connection，$C$）：两个区域 $A$ 与 $B$ 相连接（$A C B$），当且仅当它们的边界或内部相交。连接是分体拓扑学中最基本的拓扑关系，可用于定义其他拓扑关系，如分离、切接真部分和外部连接。

5.  分离（Disjointness，$D$）：两个区域 $A$ 与 $B$ 分离（$A D B$），当且仅当它们不相连。

在推理对象之间空间关系至关重要的场合，可以应用分体拓扑学。它提供了一种定性而抽象的方式来表示和推理空间信息，在精确数值坐标不可得或不必要的情况下颇具优势。

#### 计算分体拓扑学：区域连接演算

分体拓扑学为区域间的连接提供了一个更一般化的框架。分体拓扑学是灵活的，可以根据应用需要扩展出更多关系。

RCC-8 是[区域连接演算（Region Connection Calculus）](https://en.wikipedia.org/wiki/Region_connection_calculus)的一个具体实例，它对空间关系提供了更细粒度的描述。RCC-8 定义了区域之间的 8 种基本拓扑关系：分离（disconnected，DC）、外部连接（externally connected，EC）、相等（equal，EQ）、部分重叠（partially overlapping，PO）、切接真部分（tangential proper part，TPP）、切接真部分逆（tangential proper part inverse，TPPi）、非切接真部分（non-tangential proper part，NTPP）以及非切接真部分逆（non-tangential proper part inverse，NTPPi）。

RCC-8 更适合需要精确计算和详细拓扑关系的应用，而分体拓扑学在处理不确定性和不精确几何方面更为先进。

![使用 RCC8 计算模型的分体拓扑学]({{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/Mereotopology_using_RCC8_Computational_Model.jpg)

_图：RCC 中区域之间八种可能的关系，其中一个区域垂直着色，另一个水平着色。（图片来源：John G. Stell, Mereotopology and Computational Representations of the Body）_

#### 用 Python 实现分体拓扑学

并没有专门面向分体拓扑学的 Python 模块。不过，我们可以使用现有的空间库（如 `Shapely`，它提供几何运算和空间关系），并在代码中应用分体拓扑概念。

Shapely 能够处理各种空间关系与运算，例如交集、并集和差集，这在运用分体拓扑概念时会很有用。

下面是一个使用 Shapely 判断若干分体拓扑关系的例子：

```python
from shapely.geometry import Polygon

# 定义两个多边形区域
region_A = Polygon([(0, 0), (3, 0), (3, 3), (0, 3)])
region_B = Polygon([(2, 2), (5, 2), (5, 5), (2, 5)])

# 部分关系（A 是 B 的一部分）
parthood = region_A.within(region_B)
print(f"Parthood: {parthood}")

# 真部分关系（A 是 B 的真部分）
proper_parthood = region_A.within(region_B) and not region_A.equals(region_B)
print(f"Proper parthood: {proper_parthood}")

# 重叠（A 与 B 拥有共同的子区域）
overlap = region_A.intersects(region_B)
print(f"Overlap: {overlap}")

# 连接（A 与 B 相连接）
connection = region_A.touches(region_B) or overlap
print(f"Connection: {connection}")

# 分离（A 与 B 相互分离）
disjointness = region_A.disjoint(region_B)
print(f"Disjointness: {disjointness}")
```

虽然 Shapely 并不直接支持分体拓扑关系，但你可以通过应用相应的几何与拓扑运算，利用它的功能来判断这些关系。

#### 分体拓扑学定理

有了坚实的理论基础，我们便有能力推导出支持更复杂推理任务的定理。分体拓扑学提供了大量有价值的定理，有助于更深入地理解空间关系，并更有效地对空间关系进行推理。其中一些值得注意的定理包括：

1.  **边界连接定理（Boundary Connection Theorem）**：如果两个区域 A 和 B 相连接，那么它们的边界要么重合，要么相交。该定理建立了区域的连接与其边界之间的关系。
  - 例子：地块 A 和 B 共享一条公共边界（例如一条道路或一道栅栏）。我们知道存在一个区域 D，它既是 A 的边界的一部分，也是 B 的边界的一部分。这一信息可用于规划目的，例如确定管线或人行道的布设位置。

2.  **怀特海原理（Whitehead's Principle）**：如果区域 A 和 B 相连接，且它们的内部不重叠，那么其中一个是另一个边界的一部分。该原理将区域的连接与其内部和边界之间的关系联系起来。
  - 例子：地块 A 和 B 相连接，且它们的内部不重叠（即它们相邻但不重叠）。我们可以推断：其中一个地块（设为 A）是另一个地块 B 的边界的一部分。这一信息对于理解两个地块之间的相对位置关系至关重要，并可为分区或共享基础设施的布设决策提供指导。

3.  **融合定理（Fusion Theorem）**：如果两个区域 A 和 B 拥有公共部分 C，则存在一个区域 D，它是 A 与 B 的和（融合）。该定理使我们能够将重叠的区域合并为一个单一区域。
  - 例子：该市计划建造一个公园，它将包含地块 A、B 和 C 的部分区域。但公园必须是单一连通区域。我们可以确定：存在一个区域 P（即公园），它是来自地块 A、B 和 C 的各部分之和。这保证了公园将是一个单一连通的区域，符合城市规划者的要求。

4.  **传递连接定理（Transitive Connection Theorem）**：如果区域 A、B 和 C 相连接，且区域 A 与区域 C 相连接，那么区域 A 与区域 B 也必然相连接。该定理可用于推理多个区域之间的连接关系。
  - 例子：地块 A 和 C 互不相连，而地块 B 与 A、C 都相连。我们知道，如果存在另一个区域（设为 D）同时连接 A 和 C，那么 A 和 C 也必然相连。由于 A 和 C 不相连，我们可以推断：这样的区域 D 并不存在。这一信息可以帮助规划者理解这些地块之间的空间关系，对于确定城市的通行路线、交通或不同区域之间的连通性可能至关重要。

这些定理，连同分体拓扑学中的其他概念与原则，为推理部分关系、连接和重叠等空间关系奠定了基础。


### 回到 Sensorville 的故事

Mereo 博士和他的团队首先运用空间推理来分析每个区域的空间属性。他们考虑了建筑密度、建筑类型以及建筑之间的互连方式等因素。他们还考虑了城市的基础设施（如道路、桥梁和隧道）及其对应急车辆通行的影响。

随后，团队应用分体拓扑学来识别区域及其部分之间的连接与关系。这使他们能够确定每个区域中最关键的地段，并弄清这些地段彼此之间的关联，以应对不确定性下的空间推理：

-   定性空间推理：通过聚焦于空间对象之间的定性关系，分体拓扑学在存在不确定或不精确数据的情况下，依然能提供稳健可靠的推理。

-   一致性检查：分体拓扑推理有助于发现空间数据中的不一致或矛盾，从而检测并纠正错误或不确定性。

-   空间决策：分体拓扑学的形式化框架通过对空间关系提供结构化表示，并支持对备选场景或方案的探索，来支持不确定性下的决策过程。

在他们着手解决这个问题时，Dempster-Shafer 理论的著名专家 Shafer 博士加入了团队。他建议利用他的理论，根据现有的传感器数据和其他空间因素，计算某个区域处于紧急状态的确信度。Shafer 博士解释了信任函数（belief function）的概念，以及如何利用它们来融合来自不同来源的信息，例如传感器提供的部分覆盖信息和区域的空间属性。


## 用于不确定性空间推理的 Dempster-Shafer 理论

[Dempster-Shafer 理论](https://encyclopediaofmath.org/wiki/Dempster-Shafer_theory)（又称证据理论）是一个强大的数学框架，用于管理不确定性，并在信息不完整或相互冲突的情况下进行推理。在本节中，我们将介绍 Dempster-Shafer 理论，并讨论如何将其应用于不确定性空间推理。

### Dempster-Shafer 理论的基本概念

Dempster-Shafer 理论使用信任函数（belief function）来表示不确定信息，信任函数是概率函数的推广。它使用质量函数（mass function）的概念，为不同的可能结果集合分配一个信度度量。Dempster-Shafer 理论的关键组成部分包括：

> 我们在此前的文章 [用 Python 进行分类的 Dempster-Shafer 理论]({{ site.baseurl }}/dempster-shafer-theory-for-classification) 中更详细地介绍过 Dempster-Shafer 理论。

-   识别框架（frame of discernment）：一组有限的、互斥且穷举的假设，描述系统可能的状态。
-   质量函数（mass function）：为识别框架的每个子集分配一个信度质量的函数，表示对该特定子集的信任程度。
-   信任函数与似真函数（belief and plausibility functions）：由质量函数导出，分别量化识别框架中每个假设的信任程度与似真程度。

Dempster-Shafer 理论的一大优势在于它能够组合多份证据，即使这些证据相互冲突或不确定。Dempster 组合规则（Dempster's rule of combination）提供了一种通过合并多个质量函数来更新信念的机制，最终得到一个代表组合证据的新质量函数。

### Sensorville 故事的延续

团队协同合作，制定了一项策略。他们使用空间推理和分体论来确定每个区域中最关键的地段以及它们彼此之间的关系。然后，他们应用 Dempster-Shafer 理论，基于部分覆盖区域的传感器数据和区域的空间属性，计算每个区域处于紧急状态的确信度。

![运用空间推理进行监测的城市规划者]({{ site.baseurl }}/images/spatial-reasoning-under-uncertainty/SP_Mereology_City_Planners.jpg)

_图：想象一群城市规划者借助各种先进工具进行带有空间推理的监测。（图片来源：Stable Diffusion AI）_

有了 Dempster-Shafer 理论的加持，便可将其用于处理空间数据和推理任务中的不确定性，例如：

-   数据融合：组合来自多个来源、可能不确定或相互冲突的空间数据，以生成更准确、更可靠的空间关系表示。

-   不确定性传播：对不确定性在空间过程中的传播建模，例如空间变换、测量或分析运算。

-   决策制定：通过提供一个以不确定空间信息进行推理的形式化框架，支持不确定性下的决策过程。

随着传感器安装到位，团队持续分析传入的数据，并更新每个区域的信任函数。他们还利用空间推理实时调整决策，以应对环境的变化，例如施工或道路封闭。通过这些工作，他们能够在紧急情况下做出明智决策并高效分配资源，保障 Sensorville 居民的安全。


## 结语

不确定性下的空间推理是许多现实世界应用和决策过程的关键方面。正如我们在本文中所看到的，分体论、分体拓扑学和 Dempster-Shafer 理论等多种技术，可以帮助应对不确定空间数据和空间关系带来的挑战。通过理解并应用这些方法，从业者即使面对不完整或不精确的信息，也能有效地分析和推理复杂的空间问题。

在城市规划和环境监测等实际应用中，运用这些技术可以带来更明智的决策和更好的资源管理。此外，NLP 与 AI 技术的进步可以进一步支持将这些方法整合到现实世界的工作流程中，使从业者能够高效处理海量不确定空间数据，并从中获取有价值的洞见。

总而言之，不确定性下的空间推理是一个重要且不断发展的领域，在解决各领域的复杂问题方面前景可期。通过持续探索和开发新的技术与方法，研究者和从业者可以加深我们对空间关系的理解，并在面对不确定性时推动更有效的决策。


## 参考文献

这些参考文献为读者提供了坚实的基础。通过研读这些资料，读者可以更深入地理解不确定性下的空间推理，并探索它在各个领域的应用。

### 分体论（Mereology）

- [Mereology](https://plato.stanford.edu/entries/mereology/)（分体论），Stanford Encyclopedia of Philosophy, Feb 12, 2016.
  - 该条目全面概述了分体论的历史与当前发展。

- Carneades.org, [Mereology (Basic)](https://www.youtube.com/watch?v=GOjTzQlzLZ4)（分体论（基础）），视频，Mar 23, 2014.
    - 该视频以一种极佳且易懂的方式讲解了分体论。

- Roberto Casati & Achille Varzi, [Parts and Places](https://www.amazon.ca/Parts-Places-Structures-Spatial-Representation/dp/0262517078)（《部分与位置》），Bradford Books, Jan 2003. ISBN: 978-0253345707

### 分体拓扑学（Mereotopology）

- [Location and Mereology](https://plato.stanford.edu/entries/location-mereology/)（《位置与分体论》），Stanford Encyclopedia of Philosophy, Mar 12, 2018.
  - 该条目全面概述了分体拓扑学的历史与当前发展。

- John G. Stell, [Mereotopology and Computational Representations of the Body](http://computationalculture.net/mereotopology-and-computational-representations-of-the-body/)（《分体拓扑学与身体的计算表示》），Computational Culture, Nov 28, 2017.

- Randell, D. A., Cui, Z., & Cohn, A. G. (1992). [A Spatial Logic Based on Regions and Connection](https://www.researchgate.net/publication/221393453_A_Spatial_Logic_based_on_Regions_and_Connection)（《一种基于区域与连接的空间逻辑》）. In KR (Vol. 92, pp. 165-176).

- Grigni, M., Papadias, D., & Papadimitriou, C. H. (1995). [Topological Inference](https://www.researchgate.net/publication/220816401_Topological_Inference)（《拓扑推断》）. In IJCAI (Vol. 95, pp. 901-906).

### Dempster-Shafer 理论

- Shafer, G. (1976). [A mathematical theory of evidence (Vol. 1)](https://www.amazon.ca/Mathematical-Theory-Evidence-Glenn-Shafer/dp/069110042X)（《证据的数学理论（第 1 卷）》）. Princeton University Press.

- Karl Sentz and Scott Ferson, [Combination of Evidence in Dempster-Shafer Theory](https://prod.sandia.gov/techlib-noauth/access-control.cgi/2002/020835.pdf)（《Dempster-Shafer 理论中的证据组合》），Sandia National Laboratories Report, April 2002

- Richard Bowles, *Dempster-Shafer Theory*（*Dempster-Shafer 理论*），视频 <https://www.youtube.com/watch?v=51ssBAp_i5Y>
    * 这是互联网上对 D-S 理论最好的讲解，配有说明性示例和足以理解的数学。
    * 幻灯片下载地址 <http://www.richardbowles.co.uk/ai_with_js/code11/#slides>

### 地理信息系统（Geographic Information System）

- Freksa, C., & Barkowsky, T. (1996). [On the relation between spatial concepts and geographic objects](https://cindy.informatik.uni-bremen.de/cosy/staff/freksa/publications/GISDATA96FrBa.pdf)（《论空间概念与地理对象之间的关系》）. In International Conference on Geographic Information Science (pp. 115-131). Springer, Berlin, Heidelberg.

- Hunter, G. J., & Goodchild, M. F. (1997). [Modeling the uncertainty of slope and aspect estimates derived from spatial databases](https://onlinelibrary.wiley.com/doi/epdf/10.1111/j.1538-4632.1997.tb00944.x)（《对由空间数据库导出的坡度与坡向估计的不确定性建模》）. Geographical Analysis, 29(1), 35-49.

- Longley, P. A., Goodchild, M. F., Maguire, D. J., & Rhind, D. W. (2011). [Geographic Information Systems and Science](https://www.amazon.ca/Geographic-Information-Science-Systems-Longley/dp/1118676955)（《地理信息系统与科学》）. John Wiley & Sons.

- Smith, B., & Mark, D. M. (1998). [Ontology and geographic objects: An empirical study of cognitive categorization](http://ontology.buffalo.edu/smith/articles/COSIT99MST.pdf)（《本体论与地理对象：认知分类的实证研究》）. In International Conference on Spatial Information Theory (pp. 283-298). Springer, Berlin, Heidelberg.

- Worboys, M., & Duckham, M. (2004). [GIS: A Computing Perspective](https://www.amazon.ca/GIS-Computing-Perspective-Michael-Worboys/dp/0415283752)（《GIS：计算视角》）. CRC Press.

### 相关文章

- Benny Cheung, [Spatial Reasoning Explained]({{ site.baseurl }}/spatial-reasoning-explained), Benny's Mind Hack, Dec 2016.

- Benny Cheung, [Geospatial Granular Computing]({{ site.baseurl }}/geospatial-granular-computing), Benny's Mind Hack, Dec 2018.

- Benny Cheung, [Dempster-Shafer Theory for Classification using Python]({{ site.baseurl }}/dempster-shafer-theory-for-classification), Benny's Mind Hack, Aug 2020.

- Benny Cheung, [Spatial Reasoning in AGI - Insights from Philosophical Perspectives]({{ site.baseurl }}/spatial-reasoning-in-agi), Benny's Mind Hack, Mar 2023.
