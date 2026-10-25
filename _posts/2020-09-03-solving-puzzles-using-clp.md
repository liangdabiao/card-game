---
layout: post
title: 用 Prolog 约束逻辑编程求解谜题
date: 2020-09-03 12:00:00.000000000 -00:00
permalink: solving-puzzles-using-clp/
author: Benny Cheung
artwork:
  author: Gijs Van Vaerenbergh
tags:
- AI
- Prolog
- Logic
- Puzzle
category: post
comments: true
image: images/solving-puzzles-using-clp/cover2x.jpg
images:
  cover: images/solving-puzzles-using-clp/cover.jpg
  header: images/solving-puzzles-using-clp/header.jpg
published: true
series: "Logic Puzzles"
series_order: 2
---

<!--excerpt.start-->
自上一篇"[Using Prolog to Solve Logic Puzzles]({{ site.baseurl }}/using-prolog-to-solve-logic-puzzles)"（用 Prolog 求解逻辑谜题）四年后，
我终于开窍了，发现了神奇的 [clp(fd) - 约束逻辑编程（有限域）](https://www.swi-prolog.org/man/clpfd.html)模块的用法。
clp(fd) 在不同 Prolog 方言中有多种实现，但概念本质上是共享的。
为了说明 clp(fd) 与许多组合问题的完美契合，我们将使用 [SWI Prolog](https://www.swi-prolog.org/) 的 clp(fd) 实现来求解几类逻辑谜题。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/solving-puzzles-using-clp/Podcast_Solving_Puzzle_Constraint_Logic.mp3
{% include open-embed.html %}
----

约束逻辑编程天然契合 Prolog 这类逻辑语言的范式：
"变量之间的关系以约束的形式陈述"。例如以下表达式：

* X + Y（X 和 Y 不受约束）
* X + Y > 0（X 和 Y 被约束为公式必须大于 0，求解 X 和 Y 时必须满足该条件）

"有限域"（Finite Domain）这个术语，说白了就是所有数 $$\in \mathbb{Z}$$。
如果我们能把谜题转化为整数，clp(fd) 就能派上用场，把谜题转化为一个高效的组合搜索问题。

由于 SWI Prolog 启动时不会加载 clp(fd)，你只需这样导入模块，

```
:- use_module(library(clpfd).
```

作为学习练习，我们转换了三类常见的逻辑谜题：密码算术谜题、逻辑谜题和空间逻辑谜题。它们都可以用 Prolog 加约束优雅而高效地求解。前两类谜题可以非常直接地建模为组合任务来求解；第三类则需要更多功夫来找到合适的表述形式。把这些谜题都编码为整数后，Prolog 约束就能在不同状态之间高效搜索。

## 密码算术谜题——求和问题
经典的 `SEND + MORE = MONEY` 密码算术谜题，约束的是字母与 0 到 9 之间的数字的指派。它们拼出 "SEND MORE MONEY"，按十进制读数时构成一个真实的数学等式。附加约束是：首字母不允许为零。

这里我们求解另一个类似的密码算术谜题：`FORTY + TEN + TEN = SIXTY`。

![密码算术谜题]({{ site.baseurl }}/images/solving-puzzles-using-clp/Puzzle_cryptoarithemetic.jpg)

使用 clp(fd)，解法可以自然地表达为指定的约束 [[CLPFD-TUTOR]](#CLPFD-TUTOR)，

```prolog
:- use_module(library(clpfd)).

puzzle_sixty([F,O,R,T,Y] + [T,E,N] + [T,E,N] = [S,I,X,T,Y]) :-
        Vars = [F,O,R,T,Y,E,N,S,I,X],
        Vars ins 0..9,
        all_different(Vars),
        F*10000 + O*1000 + R*100 + T*10 + Y +
        2*(T*100 + E*10 + N) #=
        S*10000 + I*1000 + X*100 + T*10 + Y,
        F #\= 0, T #\= 0, S #\= 0,
        label(Vars).
```

启动 SWI Prolog 并导入 `puzzles.pl` 文件，

```prolog
$ swipl

?- [puzzles].
true.

?- puzzle_sixty(X).
X =  ([2, 9, 7, 8, 6]+[8, 5, 0]+[8, 5, 0]=[3, 1, 4, 8, 6]) ;
false.
```

这里仍有改进空间。例如，字母位置决定了 10 的倍率，
比如 "FORTY" 被表示为，

```
F*10000 + O*1000 + R*100 + T*10 + Y
```

这样写并不理想，而且容易出错。
我们可以加以泛化：定义一列数字与所表示数值之间的关系：

```prolog
digits_number(Ds, N) :-
        length(Ds, _),
        Ds ins 0..9,
        reverse(Ds, RDs),
        foldl(pow, RDs, 0-0, N-_).

pow(D, N0-I0, N-I) :-
        N #= N0 + D*10^I0,
        I #= I0 + 1.
```

然后，我们可以改用 `digits_number/2` 关系，让程序读起来更优雅，

```prolog
:- use_module(library(clpfd)).

puzzle_sixty_new([F,O,R,T,Y] + [T,E,N] + [T,E,N] = [S,I,X,T,Y]) :-
        Vars = [F,O,R,T,Y,E,N,S,I,X],
        Vars ins 0..9,
        all_different(Vars),
        digits_number([F,O,R,T,Y], FORTY),
        digits_number([T,E,N], TEN),
        digits_number([S,I,X,T,Y], SIXTY),
        FORTY + 2 * TEN #= SIXTY,
        F #\= 0, T #\= 0, S #\= 0,
        label(Vars).
```

如预期，解应与之前相同。

```prolog
?- puzzle_sixty_new(X).
X =  ([2, 9, 7, 8, 6]+[8, 5, 0]+[8, 5, 0]=[3, 1, 4, 8, 6]) .
```

## 逻辑谜题——重访斑马谜题

我们还可以用约束编程的构造重访[斑马谜题](https://en.wikipedia.org/wiki/Zebra_Puzzle)。
为方便起见，我们把谜题重述一遍，然后用 clp(fd) 给出解法 [[CLPFD-PUZZLE]](#CLPFD-PUZZLE)。

`斑马谜题` 包含 15 条事实和 2 个问题：
*谁养斑马？谁喝水？*

![斑马谜题]({{ site.baseurl }}/images/solving-puzzles-using-clp/Puzzle_zebra.jpg)

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

clp(fd) 表述如下 [[CLPFD-PUZZLE]](#CLPFD-PUZZLE)，

```prolog
zebra_puzzle(Pairs, Water, Zebra, Vs) :-
    Table   = [Houses,Nations,Drinks,Smokes,Animals],
    Houses  = [Red,Green,Yellow,Blue,Ivory],
    Nations = [England,Spain,Ukraine,Norway,Japan],
    Names   = [england,spain,ukraine,norway,japan],
    Drinks  = [Coffee,Milk,OrangeJuice,Tea,Water],
    Smokes  = [OldGold,Kools,Chesterfield,LuckyStrike,Parliaments],
    Animals = [Dog,Snails,Horse,Fox,Zebra],
    pairs_keys_values(Pairs, Nations, Names),
    maplist(all_distinct, Table),
    append(Table, Vs),
    Vs ins 1..5,
    England #= Red,               % hint 2
    Spain #= Dog,                 % hint 3
    Coffee #= Green,              % hint 4
    Ukraine #= Tea,               % hint 5
    Green #= Ivory + 1,           % hint 6
    OldGold #= Snails,            % hint 7
    Kools #= Yellow,              % hint 8
    Milk #= 3,                    % hint 9
    Norway #= 1,                  % hint 10
    next_to(Chesterfield, Fox),   % hint 11
    next_to(Kools, Horse),        % hint 12
    LuckyStrike #= OrangeJuice,   % hint 13
    Japan #= Parliaments,         % hint 14
    next_to(Norway, Blue).        % hint 15

next_to(H, N) :- abs(H-N) #= 1.
```

研究了 Markus 用 clp(fd) 给出的解法之后，我立刻爱上了这种把谜题提示直接翻译为约束关系的直白。
虽然 clp(fd) 允许把域直接表达为房屋标签，但把房屋标签映射为整数能让 clp 更高效地优化搜索。每座房子被映射为 {1,2,3,4,5} 中的一个整数。另外我想指出，用约束 `abs(H-N) #= 1` 表示"H 与相邻房屋 N"的关系非常优雅：绝对位置差必须为 1。

```prolog
abs(H-N) #= 1
```
    
这个关系在所有方向上都正确工作，无论两个变量中哪一个（如果有的话）已被实例化。因此，把这类谜题翻译为整数通常会让期望关系的表达更方便，而且由于自动应用的剪枝技术，往往还能提升性能。此外，保持程序纯粹还能让你受益于声明式调试等强大的附加技术。


使用 labeling，我们得到谜题的唯一解：

```prolog
?- zebra_puzzle(Pairs, Water, Zebra, Vs), label(Vs).
Pairs = [3-england, 4-spain, 2-ukraine, 1-norway, 5-japan],
Water = 1,
Zebra = 5,
Vs = [3, 5, 1, 2, 4, 3, 4, 2, 1|...] ;
false.
```

## 空间逻辑——重访房屋谜题
上一篇"[Spatial Reasoning Explained]({{ site.baseurl }}/spatial-reasoning-explained)"（空间推理详解）展示了一个谜题书中常见的典型空间逻辑谜题。我们用房屋的空间拓扑网络来支持它们的方向关系（东、西、南、北）并给出了一种解法。
现在，我们用 clp(fd) 重新求解这道房屋谜题。

谜题说：五对夫妇住在下图地图所示的房屋里。

1. Claire 住在 Walter 以东，而 Walter 没有和 Mary 结婚。
2. Walter 住在 Debbie 以西，而 Debbie 住在 Bill 以南。
3. Sandra 住在 Katie 以北，而 Katie 住在 Adam 以东。
4. Lenny 住在 Martin 以北

![房屋谜题]({{ site.baseurl }}/images/solving-puzzles-using-clp/Puzzle_house_0.jpg)

为了使用 clp(fd)，我们需要再多走一步：把房屋的拓扑关系（东、西、南、北）翻译到整数域。在地图上叠加一个网格后，我们可以把房屋位置描述为 `(X,Y)` 坐标；随后翻译为 `X-Y` 整数对作为约束。

![带坐标的房屋谜题]({{ site.baseurl }}/images/solving-puzzles-using-clp/Puzzle_house.jpg)

下面是 clp(fd) 解法，最值得注意的是房屋编号与位置对之间关系的编码。
`Houses` 编号是固定的，而 `Husbands` 与 `Wifes` 是受约束的变量，被限定在 1..5 之间且互不相同，将依据它们的空间关系与 `Houses` 编号合一。

```prolog
:- use_module(library(clpfd)).

house_puzzle(Sol) :-
    Houses       = [1,2,3,4,5],
    HousePos     = [(1-1),(3-1),(1-2),(3-2),(2-3)],
    Husbands     = [Adam,Bill,Lenny,Martin,Walter],
    HusbandNames = [adam,bill,lenny,martin,walter],
    Wifes        = [Claire,Debbie,Katie,Mary,Sandra],
    WifeNames    = [claire,debbie,katie,mary,sandra],
    pairs_keys_values(HusbandPairs, Husbands, HusbandNames),
    pairs_keys_values(WifePairs, Wifes, WifeNames), pairs_keys_values(HousePairs, Houses, HousePos),
    maplist(all_distinct,[Husbands,Wifes]),
    Husbands ins 1..5, Wifes ins 1..5,
    % 1. Claire lives further east than Walter, who isn't married to Mary.
    Claire #\= Walter, Mary #\= Walter,
    member(Claire-X1, HousePairs),
    member(Walter-X2, HousePairs),
    dif(X1,X2), eastof(X1,X2),
    % 2. Walter lives further west than Debbie, who lives further south than Bill.
    Debbie #\= Walter, Walter #\= Bill,
    member(Walter-X2, HousePairs),
    member(Debbie-X3, HousePairs),
    member(Bill-X4, HousePairs),
    dif(X3,X2), eastof(X3,X2),
    dif(X4,X3), northof(X4,X3),
    % 3. Sandra lives further north than Katie, who lives further east than Adam.
    Sandra #\= Adam,
    member(Sandra-X5, HousePairs),
    member(Adam-X6, HousePairs),
    member(Katie-X7, HousePairs),
    dif(X5,X7), northof(X5,X7),
    dif(X7,X6), eastof(X7,X6),
    % 4. Lenny lives further north than Martin.
    member(Lenny-X8, HousePairs),
    member(Martin-X9, HousePairs),
    dif(X8,X9), northof(X8,X9),
    % solution as a sorted [[husband,wife,house]...] list
    house_solution(HusbandPairs,WifePairs,HousePairs,Sol).
```

房屋拓扑关系可以用整数约束配合相应的 X-Y 坐标对来表达。如果 `X1` 坐标大于 `X2`，那么房屋位置 `X1-_` 就在房屋位置 `X2-_` 以东。类似地，Y 坐标用来确定房屋的南北关系。

```prolog
eastof((X1-_),(X2-_)) :-
    X1 #> X2.

northof((_-Y1),(_-Y2)) :-
    Y1 #> Y2.
```

最后，`house_solution/4` 是 HusbandPairs、WifePairs 和 HousePairs 值按整数键排序的结果。然后调用 `write_list/1` 把解漂亮地显示为 `[husband, wife, house]` 元组。

```prolog
house_solution(HusbandPairs,WifePairs,HousePairs,Sol) :-
    maplist(keysort,[HusbandPairs,WifePairs,HousePairs],[HSort,WSort,XSort]),
    house_solution_(HSort,WSort,XSort,Sol).

house_solution_([],[],[],[]).
house_solution_([_-H|HTail],[_-W|WTail],[XK-_|XTail],[[H,W,XK]|NextSol]) :-
    house_solution(HTail,WTail,XTail,NextSol).

write_list([First|Rest]) :-
    writeln(First),
    write_list(Rest).
```

房屋谜题的解可以求出（最后的 `false` 表示没有更多解），

```prolog
?- house_puzzle(Sol),write_list(Sol).
[adam,mary,1]
[martin,katie,2]
[walter,sandra,3]
[lenny,debbie,4]
[bill,claire,5]
false.
```

## 更多逻辑谜题
你可以从 [[MP18]](#MP18) [[CLPFD-TUTOR]](#CLPFD-TUTOR) 进一步了解 clp(fd)。
尤其是直接向 clp(fd) 作者学习谜题求解 [[POWER-PROLOG]](#POWER-PROLOG) 最好不过。我发现以下教程视频非常赏心悦目！
* [N-Queens Problem](https://www.youtube.com/watch?v=l_tbL9RjFdo)
* [Sudoku](https://www.youtube.com/watch?v=5KUdEZTu06o)

总之，我彻底爱上了 Prolog 的约束逻辑编程！当心，这是一条不归路。:)

## 参考文献
* <a name="SWIPROLOG-CLP">[[SWIPROLOG-CLP]](https://www.swi-prolog.org/man/clpfd.html) SWI-Prolog Documentation - library(clpfd)
  * clp(fd) 的 SWI-Prolog 实现，作者 Markus Triska。Markus Triska 还有一个 YouTube 频道，对 Prolog 编程有出色的教学。

* <a name="CLPFD-TUTOR">[[CLPFD-TUTOR]](http://www.pathwayslms.com/swipltuts/clpfd/clpfd.html) Anne Ogborn, CLP(FD) Constraint Logic Programming over Finite Domains
  * 本教程面向希望使用 clp(fd)、有一定经验的 SWI-Prolog 程序员。

* <a name="CLPFD-PUZZLE">[[CLPFD-PUZZLE]](https://www.metalevel.at/prolog/puzzles) Markus Triska, Logic Puzzles with Prolog
  * 该文用 clp(fd) 求解了若干著名的逻辑谜题。

* <a name="POWER-PROLOG">[[POWER-PROLOG]](https://www.youtube.com/channel/UCFFeNyzCEQDS4KCecugmotg) Markus Triska, The Power of Prolog, YouTube channel.
  * 一个出色的 Prolog 编程频道，可以向最好的老师 Markus Triska 学习。

* <a name="MP18">[[MP18]](https://www.youtube.com/watch?v=N5NSBSkWzco) Matt Parker, Constraint Logic Programming Over Finite Domains in Project, May 2018, Youtube video.
  * 该视频提供了 clp(fd) 的简单入门。
