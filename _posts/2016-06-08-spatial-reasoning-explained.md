---
layout: post
title: 空间推理详解
date: 2016-06-08 12:00:00.000000000 -05:00
permalink: spatial-reasoning-explained/
author: Benny Cheung
tags:
  - Spatial Reasoning
  - AI
  - Prolog
  - Logic
category: post
comments: true
image: images/spatial-reasoning-explained/cover2x.jpg
images:
  cover: images/spatial-reasoning-explained/cover.jpg
  header: images/spatial-reasoning-explained/header.jpg
published: true
series: "Spatial Reasoning"
series_order: 1
---

<!--excerpt.start-->
空间推理（Spatial Reasoning）是一种逻辑推理系统，它假设实体位于空间之中并具有空间结构。
制造能够感知和理解空间的机器，一直是
研究者的梦想。它们 assistance 可以改善我们的生活；
凭借机器的空间智能，我们将拥有通过空间朝向来
规划与导航的新方法。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/spatial-reasoning-explained/Podcast_Spatial_Reasoning_Explained.mp3
{% include open-embed.html %}
----

## 空间推理的议题
计算机科学中有一系列需要空间推理能力的有趣议题：

* **人工智能** ——
我们的日常概念中充满了定性的空间描述。某物坐在另一物旁边。
这些描述通常模糊而不精确，但我们依然能推断出一些非平凡的结论。
例如，一列火车正向西行驶。如果车站出口更靠近南侧，
我们该坐在车厢的什么位置才能更快出站？
这涉及火车的空间结构，以及对火车行进方向的定性理解。

* **空间数据库** ——
除了广为人知的数值或字符对象之外，"空间对象"是"数据库对象"集合的一个流行扩展。
当数据带有空间属性时，例如街道地址或电话号码，
我们可以利用空间邻近性来计算和推断更多信息。
我们是在为这些空间实体指派有限的几何表示与运算。

* **图像识别** ——
当一个物体出现在视野中时，图像识别应用对它兴趣盎然。
物体通常出现在空间上下文中，例如，
放在桌面上的零食。为了精确定位零食的位置和空间占位，需要对它们进行几何与空间描述。

* **语言处理** ——
在交流中，信息通常伴随空间上下文。
例如，战场后勤依赖空间优化来找到
最有战略优势的位置。这一方法既以语言描述，也在地图上以视觉方式呈现。
再举一例，当一个人迷路时，
他们通常能以片段化的形式描述自己的空间处境。
听完描述之后，我们就能重建当时的场景，在空间上定位迷路者所处的位置。

## 空间逻辑
考察一个逻辑系统时，我们可以分别求取实体 $$A$$ 和 $$B$$ 的真假来评估
公式 $$A \land B$$。$$\land$$ 作为逻辑系统内定义的
`AND` 逻辑算子，规定 $$A$$ 与 $$B$$ 必须都为真，
$$A \land B$$ 才为真。如果 $$A$$ 与 $$B$$ 都是空间实体，
我们就为公式扩展了一个空间结构，用以判定 $$A \land B$$ 在 $$S$$ 中是否可能。
若可能，则求取 $$A \land B$$ 在空间 $$S$$ 内为真的情况。

假设有一组空间实体 $$A=\{A_1, A_2, ..., A_n\}$$，
我们可以断言空间结构
$$S$$ 对集合 $$A$$ 成立。我们称集合 $$A$$ 为 $$S$$ 的可能空间构型。
举一个具体例子：如果我们需要从陆地上的 X 地导航到 Y 地，
我们会假设存在一个道路网络结构
连接这两个位置。
我们可以看到集合 $$A=\{X, ..., Y\}$$，
而连接 $$A$$ 中所有元素的道路网络就是 $$S$$。

有时，空间上下文并不需要一个占据某位置的空间实体。
例如，我们最熟悉的 2D 笛卡尔坐标系就是一个空间参考系统。
选定原点（如 2D 中的 (0,0)）后，所有可能的位置都显式排布出来。如果有一个空间实体
位于这个 2D 笛卡尔坐标系中，我们通过相对所选原点的度量来指称该实体的
位置。我们将在后续讨论中回到*隐式*空间位置（拓扑关系）
与*显式*空间位置（几何关系）的区别上来。
本文假设我们讨论的是隐式空间位置（拓扑关系）。

## 需要空间逻辑的谜题
"一个具备空间推理能力的系统是什么样子？"
用一个例子来回答最好不过。
通过编写一个 Prolog 程序，我们将展示
自动求解空间逻辑谜题的可能性。
下面是一个谜题书中常见的典型空间逻辑谜题。

### 求解空间逻辑谜题
五对夫妇住在下图地图所示的房屋里。

![空间推理房屋谜题]({{ site.baseurl }}/images/spatial-reasoning-explained/house-spatial-puzzle-400.jpg)

1. Claire 住在 Walter 以东，而 Walter 没有和 Mary 结婚。
2. Walter 住在 Debbie 以西，而 Debbie 住在 Bill 以南。
3. Sandra 住在 Katie 以北，而 Katie 住在 Adam 以东。
4. Lenny 住在 Martin 以北。

多亏了 [Mihaela Malita](https://www.anselm.edu/faculty-directory/mihaela-malita)，
我们有了极其有用的
[Prolog 谜题求解库](https://ypologist.com/mmalita17/HOMEPAGE/logic/index.html)。
我们将在 `bibmm.pl` 之上用 Prolog 构建空间谜题求解器。下载该库之后，程序可以这样导入（使用 [SWI Prolog](http://www.swi-prolog.org)）：

```prolog
:-consult('bibmm.pl').
```

仔细分析空间配置之后，我们可以编写如下空间结构。
（注意：为节省篇幅，我们把多条 Prolog 语句放进同一行）

```prolog
eastof(2,1). eastof(2,3). eastof(2,7). eastof(4,1).
eastof(4,3). eastof(4,7). eastof(7,1). eastof(7,3).

westof(1,2). westof(1,4). westof(1,7). westof(3,2).
westof(3,4). westof(3,7). westof(7,2). westof(7,4).

northof(7,1). northof(7,2). northof(7,3). northof(7,4).
northof(1,3). northof(1,4). northof(2,3). northof(2,4).

southof(1,7). southof(2,7). southof(3,7). southof(4,7).
southof(3,1). southof(4,1). southof(3,2). southof(4,2).
```

这是有意为之：穷举该谜题空间结构中所有可能的拓扑关系。稍后我们会用空间逻辑系统来改进它。

有了空间结构的定义，我们就可以继续编写 Prolog
程序来编码全部谜题约束。例如在我们的例子中，有 4 条约束语句。
仔细拆解每条约束语句，我们将得到
基本逻辑语句——空间约束或非空间约束。

```prolog
% include the facts of spatial structure

house([1,2,3,4,7]).
husband([adam,bill,lenny,martin,walter]).
wife([claire,debbie,katie,mary,sandra]).

start(Sol):- husband(M),wife(F),house(H),
    Sol=[[M1,W1,H1],[M2,W2,H2],[M3,W3,H3],[M4,W4,H4],[M5,W5,H5]],
    set_equal([H1,H2,H3,H4,H5],H),
    % 1. Claire lives further east than Walter, who isn't married to Mary.
    eastof(X1,X2),
    member([_,claire,X1],Sol),
    member([walter,_,X2],Sol),
    member([_,mary,X3],Sol),
    X2 \= X3,
    % 2. Walter lives further west than Debbie, who lives further south than Bill.
    member([_,debbie,X4],Sol),
    westof(X2,X4),
    member([bill,_,X5],Sol),
    southof(X4,X5),
    % 3. Sandra lives further north than Katie, who lives further east than Adam.
    member([_,sandra,X6],Sol),
    member([_,katie,X7],Sol),
    northof(X6,X7),
    member([adam,_,X8],Sol),
    eastof(X7,X8),
    % 4. Lenny lives further north than Martin.
    member([lenny,_,X9],Sol),
    member([martin,_,X10],Sol),
    northof(X9,X10),
    set_equal([M1,M2,M3,M4,M5],M),
    set_equal([W1,W2,W3,W4,W5],W),
    !.
```

如何用 `swipl` 运行：

```
?- ['house'].
?- start(Sol), write_list(Sol), fail.
```

执行 Prolog 程序后，谜题的解答将呈现为，

```
[walter,sandra,1]
[lenny,debbie,2]
[adam,mary,3]
[martin,katie,4]
[bill,claire,7]
```

以防你不知道如何退出 `swipl`：

```
?- halt.
```

### 改进的空间关系编码
在示例中，我们通过枚举所有可能的两两关系，用拓扑关系编码了空间结构。
这并不是一个聪明的编码方案，而且完全是穷举式的。
聪明的读者会立即注意到：凭借我们的空间智能，
我们有一套可以应用的空间逻辑。

例如，

```prolog
eastof(2,1).
westof(1,2).
```

可以看到关系 `eastof` 与 `westof` 是对称关系：
由空间对称性，`eastof(X,Y)` 立即蕴含 `westof(Y,X)`。

因此，无需编码所有对称的空间关系，我们可以输入如下逻辑规则，

```prolog
westof(X,Y) :- eastof(Y,X).
```

这样一来，我们不需要编码任何 `westof` 关系，因为 `westof`
可以从对称的 `eastof` 关系推导出来。

类似地，我们可以写下，

```prolog
southof(X,Y) :- northof(Y,X).
```

这样我们也不需要编码任何 `southof` 关系，因为 `southhof`
可以从对称的 `northof` 关系推导出来。

于是，空间关系集合缩减为只需编码 `eastof` 与 `northof` 关系。

```prolog
eastof(2,1). eastof(2,3). eastof(2,7). eastof(4,1).
eastof(4,3). eastof(4,7). eastof(7,1). eastof(7,3).

northof(7,1). northof(7,2). northof(7,3). northof(7,4).
northof(1,3). northof(1,4). northof(2,3). northof(2,4).
```

那么传递关系呢？我们可以只编码直接的 `northof` 实体来缩减 `northof` 关系，
让传递逻辑规则负责隐含的空间关系。

```prolog
northof(X,Y) :-
  X \= Y,
  northof(X,Z),
  northof(Z,Y).
```

最后，我们可以把空间关系精炼为这个缩减后的集合：
去掉 `northof(7,3)` 和 `northof(7,4)` 这两条间接的 `northof` 关系。

```prolog
eastof(2,1). eastof(2,3). eastof(2,7). eastof(4,1).
eastof(4,3). eastof(4,7). eastof(7,1). eastof(7,3).

northof(7,1). northof(7,2).
northof(1,3). northof(1,4). northof(2,3). northof(2,4).
```

## 空间推理是一个逻辑系统
我们用 Prolog 编写的拓扑语言展示了空间推理。
空间实体被隐式地表示为一个符号——在本例中是房屋编号。
或者，我们也可以显式地按几何属性编码空间实体，
然后依据罗盘基本方位的几何规则计算它们的空间关系。
通过这种方法，拓扑关系集合可以被生成出来，而不必完全依赖手工编码。

空间推理是一种在逻辑系统中推导空间真理的策略。
我们正在进入经典逻辑推理的领域，其中对象被限定为空间实体。推理规则由空间逻辑支配；因此，这一领域有大量文献可以参考研究。一个很好的起点是 Marco Aiello 等人的《The Handbook of Spatial Logics》。

空间逻辑在可能性结构之上叠加约束，许多非逻辑的空间情形可以被快速排除。这一结构就是空间有效性的工作前提。
我们需要遵循空间公式的严格逻辑规则，包括本文展示的对称性与传递性空间规则。
间接的空间关系可以被逻辑地推导出来。
空间关系不一定只是描述方向的简单"罗盘"规则。
空间关系可以由任何计算几何生成，例如凸包或交集，从而支持更广泛的空间推理能力。
理解了这一基础，你可能会发现空间推理适用于生活的许多领域，并对这个主题产生新的兴趣。

## 参考文献

> **更新 2020/09/04**
> 我终于开窍了，发现了神奇的 [clp(fd) - 约束逻辑编程（有限域）](https://www.swi-prolog.org/man/clpfd.html)模块的用法。有兴趣的读者可以继续阅读
> "[Solving Puzzles using Constraint Logic Programming in Prolog]({{ site.baseurl }}/solving-puzzles-using-clp)"（《用 Prolog 约束逻辑编程求解谜题》），
> 它讲解了如何用 clp(fd) 求解三类谜题：(1) *密码算术谜题*、(2) *逻辑谜题*和 (3) *空间逻辑谜题*。
