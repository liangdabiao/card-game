---
layout: post
title: 用 Prolog 求解逻辑谜题
date: 2016-04-21 12:00:00.000000000 -05:00
permalink: using-prolog-to-solve-logic-puzzles/
author: Benny Cheung
tags:
  - AI
  - Prolog
  - Logic
  - Puzzle
category: post
comments: true
image: images/using-prolog-to-solve-logic-puzzles/cover2x.jpg
images:
  cover: images/using-prolog-to-solve-logic-puzzles/cover.jpg
  header: images/using-prolog-to-solve-logic-puzzles/header.jpg
published: true
series: "Logic Puzzles"
series_order: 1
---

## 什么是逻辑谜题？
<!--excerpt.start-->
逻辑谜题是件非常有趣的事。我们都兴致勃勃地读这些谜题，并绞尽脑汁解上
一两个。我们以为，在证明自己足够有逻辑、能解开这些逻辑谜题之后，
脑力就会得到提升。当然，这个过程本身也很有趣。但它
真的改善了我们的大脑功能吗？还是这恰恰说明我们的大脑无力处理
大规模条件下的逻辑跳跃，比如 6 座房子、5 对夫妇和 7 种茶？显然，我们无法
在头脑中容纳这么大的逻辑空间——此例中共有 6 x 5 x 7 = 210
种可能的解。我们需要借助某种外部记录
装置来帮我们组织这"浩瀚"的信息。通过排除法和
演绎推理，我们才能得到一个可能的答案。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/using-prolog-to-solve-logic-puzzles/Podcast_Prolog_Logic_Puzzle.mp3
{% include open-embed.html %}
----

解开几个谜题之后，计算机科学家的自然反应是："我们为什么不
自动化地求解这些谜题？"当然，你作为计算机科学家的直觉永远是对的。经过几次粗略分析，我们可以看到
谜题是一个搜索问题。如果我们够笨、愿意去试，我们可以穷举
所有可能的答案，然后回到约束列表，逐一测试是否
有哪个答案满足全部约束。如果某个答案不产生任何矛盾，我们就知道
它就是正确答案。然而，我们的双手和大脑通常懒得做这种愚蠢的搜索。
谜题书的编写者帮了你一把：给你一张所有组合的表格，让你可以假装
自己没有在做愚蠢的搜索。你实际上是在写下一些东西来排除明显的
蠢选项。逻辑组合表格充当了一个"更聪明的笨搜索"装置。这让我们
感觉非常好。

下面是许多谜题爱好者使用的流行
[网格法（Grid Method）](http://www.logic-puzzles.org)，
相关杂志也专门刊登这类内容。

![用网格法求解逻辑谜题]({{ site.baseurl }}/images/using-prolog-to-solve-logic-puzzles/grid_method_sample.png){:width="670px" height="698px"}

打上那些对勾来表示某事可能或可以排除，这需要一些语言阅读技巧。如果我们得不出解，
我们甚至不确定自己是否正确读懂了所有英文句子。我们会回到原始陈述，
逐条核对我们的可能解中是否存在矛盾。


严肃一点说：计算机科学家面对逻辑问题时，会
使用一门编程逻辑语言，即 PRO-LOG（Prolog），来解决问题。这是个好主意——
直到你真正开始写 Prolog 代码。你也许会问，问题出在哪？我说，
你真的试过吗。问题在于：把逻辑谜题编码出来，比解开它更难。幸运的是，花无穷的时间编码的回报，
是期望更快地解出后续问题。按照摊销原则，我们最终是赚的。

世界上有一位教授，[Mihaela Malita](https://www.anselm.edu/faculty-directory/mihaela-malita)，她认真对待这件事，提供了一个极其有用的 [Prolog 谜题求解库](https://ypologist.com/mmalita17/HOMEPAGE/logic/index.html)。我真心感激她，因为她证明了自动谜题求解既有趣，也是一项值得掌握的编程技能。

## 斑马谜题
著名的 `斑马谜题`（Zebra Puzzle）包含 15 条事实和 2 个问题：
*谁养斑马？谁喝水？*

事实（即约束）列表：

1. 一排有 5 座不同颜色的房子，每座房有一位主人，每位主人有一只动物、一种爱抽的香烟、一种爱喝的饮料。
2. 英国人住在红房子里。
3. 西班牙人养狗。
4. 他们在绿房子里喝咖啡。
5. 乌克兰人喝茶。
6. 绿房子在白房子旁边。
7. 抽 Winston 烟的人养蛇。
8. 黄房子里的人抽 Kool。
9. 中间的房子里人们喝牛奶。
10. 挪威人住在从左数第一座房子里。
11. 抽 Chesterfield 烟的人住在养狐狸的人附近。
12. 在养马人房子附近的房子里，人们抽 Kool。
13. 抽 Lucky Strike 烟的人喝果汁。
14. 日本人抽 Kent。
15. 挪威人住在蓝房子旁边。

我们把房子表示为一个包含 5 个子列表的列表，从左到右：

{% highlight prolog %}
Sol = [[Man, Animal, Cigarette, Drink, Color], [..],[..],[..],[..] ]
{% endhighlight %}

我们使用 [SWI Prolog](http://www.swi-prolog.org)
来实现斑马谜题。借助谜题库 *bibmm.pl*（从 [Prolog 谜题求解库](http://www.anselm.edu/internet/compsci/faculty_staff/mmalita/HOMEPAGE/logic/index.html)下载），我们把约束翻译为如下 Prolog 代码，

{% highlight prolog %}
:-consult('bibmm.pl').

start(Sol):- length(Sol,5),                 % 1
    member([english,_,_,_,red],Sol),        % 2
    member([spanish,dog,_,_,_],Sol),        % 3
    member([_,_,_,coffee,green],Sol),       % 4
    member([ukrainian,_,_,tea,_],Sol),      % 5
    right([_,_,_,_,green],[_,_,_,_,white], Sol),    % 6
    member([_,snake,winston,_,_],Sol),      % 7
    member([_,_,kool,_,yellow],Sol),        % 8
    Sol= [_,_,[_,_,_,milk,_],_,_],          % 9
        Sol= [[norwegian,_,_,_,_],_,_,_,_],         % 10
    next([_,_,chesterfield,_,_],[_,fox,_,_,_],Sol), % 11
    next([_,_,kool,_,_],[_,horse,_,_,_], Sol),  % 12
    member([_,_,lucky,juice,_],Sol),        % 13
    member([japonese,_,kent,_,_],Sol),      % 14
    next([norwegian,_,_,_,_],[_,_,_,_,blue],Sol),   % 15
    member([_,_,_,water,_],Sol),            % someone drinks water
    member([_,zebra,_,_,_],Sol).            % someone has a zebra
{% endhighlight %}

然后，我们可以执行 Prolog 代码来求解斑马谜题。

{% highlight shell %}
% swipl
?- ['zebra'].
?- start(Sol), write_list(Sol), fail.
[norwegian,fox,kool,water,yellow]
[ukrainian,horse,chesterfield,tea,blue]
[english,snake,winston,milk,red]
[japonese,zebra,kent,coffee,green]
[spanish,dog,lucky,juice,white]
false.
?- halt.
{% endhighlight %}


## 编程求解逻辑谜题
我们将分析通过编写 Prolog 程序来求解逻辑谜题的思考过程。以下即是这些思考。

### 逻辑谜题中常见的情形是什么？
例如，某个东西挨着另一个东西，而那个东西是一条狗。我们来找几个这样的例子。

上文提到的斑马谜题更具挑战性：它需要额外的背景知识，
因为线索省略了一些细节，比如房子排成一排。
此外，由于谜题线索中没有提到斑马，必须推断出
有人养斑马。这类问题清楚地表明，我们需要一种表示方法，
它还能处理自然语言文本中隐含的假设。我们相信，
设计一个以精确理解自然语言为目标的计算系统，
必须认真对待语义与背景知识之间的这种互动。

### 描述约束的语言障碍是什么？
例如，"恰好"（exactly）这样的词似乎出现得很频繁。说了太多次
"某个东西挨着某个东西"之后，你的舌头会开始发酸。你把自己搞糊涂的速度，
比描述约束的速度还快。

### 状态的最优表示是什么？
这是表示问题。
通常，表示问题的方式数量本身就是个问题。
尤其是当你试图达到"最优"表示时。这是真的：
有了聪明的表示，解法一目了然；
而有了糟糕的表示，解法就模糊得多。

向量表示理论过于通用，以至于在弄清向量的模式（schema）如何陈述之前，
你无法知道它是什么意思。对每个字段，都有一个域和
相应的可能值集合。对于字段之间的关联，存在一种关系把
两个或更多字段串在一起以提供语义。就处理而言，使用向量表示非常便利。
对系统的解释器而言，这不过是一组符号。实际上，这不是向量表示的问题，
而是向量阅读者的问题。

用向量元素表示相邻关系非常方便。
它们的相对位置可以精确定义为它们的相邻关系，例如
[A,B]，这样的向量可以表示 A 在 B 左边、B 在 A 右边，或简单地说 A 挨着 B、
B 挨着 A。因此，它们的索引位置是一个逻辑空间，关系可以在其中建立。
这个逻辑空间不蕴含度量性或规则性。这是一个概念空间。

这样的概念空间不需要是紧凑的。它可以是对连通性的稀疏表示。
度量的"距离"函数没有定义。"距离"可以是
声明式的"距离"。例如 A->B->C，其中 distance(A,B) 为 1，distance(A,C) 为 2。

## 为什么这对软件设计者很有趣？
谈点正经的：业务规格说明就像逻辑谜题。何以见得？
在业务领域，约束通常可以直接感知，比如一个人必须
先有 1000 美元才能申请信用卡。然而，"一个人有 1000 美元"可能需要一些演绎推理才能得出。
他的账户里可能并不直接有 1000 美元。
如果他有多个账户呢？或者他有其他抵押物可以满足这 1000 美元的要求呢？
聪明的业务规格说明会把所有这些演绎推理列成一张检查清单，
或简单地列成逻辑，让软件开发者直接编码进程序。

想到这一点，我就非常有动力去理解逻辑谜题的机理，
希望它能用于业务规格说明。

## 搜索问题与智能
逻辑空间是一棵搜索树，其中所有连通性都表示为一条逻辑演绎。
例如，A->B、B->C，加上传递规则，得到 A->C。
业务规格说明很难完整。有许多事实被确认为真。
有些事实则要依赖逻辑演绎来确立。

在逻辑空间中有哪些逻辑动作？

- 搜索空间的表示
- 用于逻辑谜题的 Prolog 声明式编程
- 语言的局限
- 智能可以被自动化。你可以和业务用户一起回答问题。
- 一种解释推理过程、隐藏假设与需求的方法。


有人建议把逻辑谜题用作计算系统的测试套件，
以考察系统能否基于对自然语言输入的精确理解来回答问题。
用逻辑谜题评估自然语言理解系统有以下几个好处。
最重要的是，逻辑谜题的创建独立于
自然语言处理研究者议事日程上的特定目标。此外，
要解开这类谜题，计算系统必须能够处理广泛的语言现象，
并能够用自然语言输入进行推理。最后，逻辑谜题定义了精确而
结构良好的问题，为试图求解这类谜题的计算系统的性能
设定了清晰的评价标准。

我们相信，逻辑网格谜题可以为我们提供一种检验形式表示质量的有效测试。
要解开逻辑谜题，需要一种形式表示，以便
表示语义结构和背景知识。计算系统对文本的精确理解，
需要一个能把自然语言输入映射为语义表示、并能够结合
任何其他可用背景知识（与常识知识相关）对该输入进行推理的系统。

逻辑网格谜题之所以适合检验形式表示的质量，有三个主要原因。

1. 要解更复杂的谜题，计算系统必须能够处理量词
    与辖域、比较级、时态、代词与照应指称、关系从句、
    不同类型的副词、序数词等。
2. 为渐进式方法提供基础，目标是表示语言的逐步精化。也就是说，可以从一阶逻辑开始翻译
    非常简单的谜题，然后在后续步骤中加入处理照应
    或事件的附加特性。
3. 这些谜题结构非常良好，且总是有唯一解。
    逻辑网格谜题的解就是满足自然语言约束的（唯一）模型。

### 在真实世界中使用 Prolog
想进一步了解 Prolog 在现实问题中的应用，这里有更多值得探索的链接：

* http://stackoverflow.com/questions/130097/real-world-prolog-usage
* http://www.drdobbs.com/parallel/the-practical-application-of-prolog/184405220

## 参考文献

> **更新 2020/09/04**
> 我终于开窍了，发现了神奇的 [clp(fd) - 约束逻辑编程（有限域）](https://www.swi-prolog.org/man/clpfd.html)模块的用法。有兴趣的读者可以继续阅读
> "[Solving Puzzles using Constraint Logic Programming in Prolog]({{ site.baseurl }}/solving-puzzles-using-clp)"（用 Prolog 约束逻辑编程求解谜题），
> 它讲解了如何用 clp(fd) 求解三类谜题：(1) *密码算术谜题*、(2) *逻辑谜题*和 (3) *空间逻辑谜题*。
