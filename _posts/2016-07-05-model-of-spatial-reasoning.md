---
layout: post
title: 空间构造模型
date: 2016-07-05 12:00:00.000000000 -05:00
permalink: model-of-spatial-construction/
author: Benny Cheung
tags:
  - Spatial Reasoning
  - Voronoi
  - Delaunay
  - Tessellation
category: post
comments: true
image: images/model-of-spatial-construction/cover2x.jpg
images:
  cover: images/model-of-spatial-construction/cover.jpg
  header: images/model-of-spatial-construction/header.jpg
published: true
series: "Spatial Reasoning"
series_order: 2
---

<!--excerpt.start-->
空间推理系统的核心是对空间知识的利用。
有了恰当表示的空间知识，空间推理任务就会变得直观、灵活而实用。
本文引入*空间构造*（Spatial Construction）的概念，
它是推导空间知识的理论。
该理论的长处在于其直观的基础，
使得空间知识的算法构造可以灵活而实用。
我们将集中论述如何搭建空间构造的基础。
至于空间推理的另一关切——如何有效且高效地利用空间知识——留待另一篇文章详述。
<!--excerpt.end-->

最后，我们将介绍
[PyDelaunay (https://github.com/bennycheung/PyDelaunay)](https://github.com/bennycheung/PyDelaunay)
作为一种实用的空间构造。它是一个高效的
Voronoi/Delaunay 镶嵌（Tessellation）Python 实现，
可以作为邻域概念的*基础地图*（Basic Map）。

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/model-of-spatial-construction/Podcast_Model_of_Spatial_Construction.mp3
{% include open-embed.html %}
----

## 空间推理模型
空间推理可以被描述为
众多空间问题的一种通用解法。
考虑到空间知识被表示为
一组实体和一组关系，
空间推理是一个知识增长过程，其设计目标是
利用空间原语（spatial primitives）不断推导出
任意两个实体之间越来越具体的关系。
例如，在一个环境 $$G_s$$ 中，
可能的解空间可以描述为
$$v_a$$ 与 $$v_b$$ 之间所有可能的连通性。

虽然这还不是 $$v_a$$ 与 $$v_b$$ 之间的一个具体关系，
空间推理过程倾向于产出 $$v_a$$ 与 $$v_b$$ 之间更具体的关系。
在这个例子里，
"$$v_a$$ 经由 $$v_c$$ 与 $$v_b$$ 相关联"
是一条新关系，它比完全图的连接更具体。

这条具体关系的推导可以是知识密集型过程，
也可以是纯算法过程。但无论哪种情况，
这条新推导出的关系都被定义为一个空间推理步骤。
在推导出这条新空间关系之后，它可以立即被记录
到知识库中。只要关系
"$$v_a$$ 经由 $$v_c$$ 与 $$v_b$$ 相关联"
仍然在知识库中成立，这块新知识就可能增强其他空间推理步骤。

空间推理可以进一步划分为两个子过程：
*空间构造*与*空间利用*。
空间构造是一切空间推理活动的基石，因为它提供了表示空间与空间语义的方法。
空间利用则是实际过程：
利用空间构造物与空间原语
来构造一些非常具体的关系。
空间构造可以看作面向任何应用的通用表示，而空间利用则更偏应用特定。
应用可能施加其领域知识，
从而影响空间利用。
虽然这个模型把这两个子过程分开了，
但它们应当被视为彼此互补的过程，如图所示。

![空间推理过程模型]({{ site.baseurl }}/images/model-of-spatial-construction/Spatial_Reasoning_Process_Model.jpg)

空间构造之后，基础知识可以立即
应用于利用步骤。利用步骤推导出
新的遍历信息之后，它可以重构部分基础知识。
要启动构造过程，需要一个外部过程。
这个外部过程可以是预定义的知识库，或某种知识
获取过程。

传统推理框架包含两个重要部分：
知识库与推理机。
把空间推理定义为过程，并没有偏离
这一基本构造，而是通过纳入特定过程对结构加以细化。
这些特定过程由密集的空间知识驱动。
因此，这个模型不是一种新型的推理机制，而是一种
新的过程控制，作为支撑机制来满足空间推理的需要。

## 空间构造
空间实体的自然分组是空间构造概念的基石。
把知识库与推理机分开的原始推理方案仍然适用。
不过，新提出的方案主张：空间知识库
应以其固有的空间知识来索引，
如图所示。

![空间构造模型]({{ site.baseurl }}/images/model-of-spatial-construction/Model_of_Spatial_Construction.jpg)

这个基础的被索引空间称为*基础地图*（Basic Map）。
基础地图不仅捕捉了空间的邻域性质，
还为几何信息提供了快速处理结构。

基础地图构造好之后，
其他多种地图（包括拓扑、结构等）
也可以从空间的多面性与层次性出发构造出来。
此外，根据应用的需求，
还可以派生出专门的地图。
"从一个基础地图出发，其他所有地图都由它派生"这一思想
得到了近年 AI 认知研究的支持。
关键在于：每张地图都派生自基础地图，从而维护成为可能。
而且，一旦基础地图发生改变，
通过派生机制，
其他地图可以被自动、系统地更新。
这是因为所有空间原语都可以基于这些地图高效实现。

这种做法并不是要打破传统推理方案，而是
为空间推理中识别出的特殊需要创建一种逃逸机制（escape mechanism）。
空间原语的定义被导出给推理系统。
每当系统遇到这些原语，
它就把任务交给专门的空间推理引擎
去执行这些原语。
这一逃逸机制是合理的，因为大多数图形系统已经证明了它，例如以硬件实现的 OpenGL，
它与基于三角形的几何处理密切相关。
构建在 OpenGL 之上的图形应用，可以用高级图形原语编程。
这种设计不仅提升了性能，
还简化了应用领域的建模。

空间推理的动态性可以通过
高层应用语义来展示。
智能很难在点、线、多边形的处理中直接看到，
而是体现在它们的组合结构与关系之中。
不过，通用性仍然保留在基础地图之内。

为了容纳更高层的空间推理任务，
在一组空间原语之上可能还会有更多层次。
这些层次很可能依应用而定。
基本构造在不同应用之间不会有差异，
因为它们直接处理空间本身，
与任何应用的空间属性无关。
一旦空间属性被施加在这些构造之上，
大量智能推理就能得以实现。

### 基本空间构造物
显然，我们需要一些技术，
能够利用全部可能空间属性的集合来求解空间问题。
相关的结构不应过于泛化，
以免丢失空间属性。
为了同时捕捉空间推理的内涵语义与外延语义，
有四种基本空间构造物：

* **空间单元（Spatial Cells）：**
连续空间必须由一组离散单元来表示，
称为空间单元。一块有限的区域由一个空间单元表示，
通常由唯一标识符引用。
如图所示，*空间单元*是覆盖所表示空间某一部分的区域。如果两个空间单元直接相连，
我们可以在它们之间放一条边以表示空间连通性。

![构造空间单元]({{ site.baseurl }}/images/model-of-spatial-construction/Construct_01_Basic_Map.jpg)

* **邻域（Neighbourhood）：**
确定对应于空间排序的邻域，
是把连续空间变成离散计算单元的前提。
通常，邻域由空间单元的相对位置确定。如图所示，$$V_0$$ 的*邻域*由
集合 $${V_1, V_2, ..., V_6}$$ 组成。

![构造邻域]({{ site.baseurl }}/images/model-of-spatial-construction/Construct_02_Neighborhood.jpg)

有时，如果两个空间单元共享公共边界，
这也可以称为邻域。
内涵语义就嵌入在邻域的使用之中，因为一个对象可以由它
相对于邻居的位置唯一描述。

* **分类空间（Categorized Space）：**
一组空间单元可以聚合在一起，并由一个
唯一标识符引用。
这些聚合区域被称为分类空间，
因为在这一层级上，一组空间单元被当作一个单元来操作。
如图所示，*分类空间*把空间单元划分为障碍物（棕色）与自由空间（白色）。

![构造分类空间]({{ site.baseurl }}/images/model-of-spatial-construction/Construct_03_Categorized_Space.jpg)

* **地图（Maps）：**
地图可以被可视化为完整空间的表示，
它可以是精确形式，也可以是抽象形式。
地图提供整个问题空间的全局视图。
如图所示，*自由空间地图*由关键决策点组成。
$$S$$ 是源点，$$D$$ 是决策点。我们可能从 $$D$$ 出发前往 $$A$$ 或 $$B$$ 点。

![构造自由空间]({{ site.baseurl }}/images/model-of-spatial-construction/Construct_04_Free_Space.jpg)

因此，外延语义嵌入在地图的使用之中，
因为地图的使用可以剪除许多不合理的空间选择。

空间构造物融入了前述空间推理的概念——
一个推导越来越具体关系的过程。
邻域信息是形成有意义知识的最基本关系。
构造邻域类似于
为离散空间实体构造可能的计算关系。
它表示空间推理中可以纳入的局部决策。
分类空间是把
这些关系组合成更具体关系的遍历信息。
全局知识在地图的使用中得到增强。
此外，地图的可用性实际上也能增强局部决策。

### 邻域作为基础地图
邻域结构
应当是空间推理系统的基础地图。
对象邻域的健全概念，对于分析并
确定局部最优决策至关重要。
这种局部性之所以可取，
是因为它可以通过限制选择的数量来增强计算能力。
对空间实体的任何刻画或比较，都必须基于
相对空间排布。
然而，确定一组邻域的固有问题
以两种方式之一出现。
首先，只有当存在一个类型已知的已知过程时，才能对空间实体之间的关系进行推理。
然而一般而言，这类先验知识或类型化过程模型
通常是不可得的。
其次，空间实体的全局特征对其关系的感知非常重要。
然而，考虑全局特征意味着自顶向下的方法，
因而依赖于模型的可得性。
在没有模型、也没有对象集合的任何先验知识时，
对结构与关系的任何推理都必须是数据驱动的。
结构化描述（例如邻域）必须使用空间实体的
相对位置来构建。
因此，一个健全的邻域概念是必要的。

以往定义邻域的所有尝试都苦于同一个问题：
无法处理空间集合中密度不均的情况。
这些方法的性能取决于参数取值。
因此，针对给定密度调参以获得最佳性能，可能永远是必需的。
然而，应当注意到：人类可以轻松地在各种模式中——包括密度不均的模式——
识别邻域和空间实体的邻居。
这暗示存在一个通用的、无参数的邻域概念。
在计算几何领域，Voronoi 镶嵌被用来
构造这样一种基础邻域结构，
它有望克服前述的许多问题。
因此，新的空间推理系统的基础地图，应当具备
与 Voronoi 镶嵌相同的能力。

## Voronoi/Delaunay 镶嵌作为基础地图
点集中所有离给定点比离集合中其他所有点都更近的点的集合，
构成一个有趣的空间结构，称为该点的
*Voronoi 多边形*。一个点集的所有 Voronoi 多边形的并集
称为 *Voronoi 镶嵌*。

基于该镶嵌提供的邻域信息，人们已经发现了许多应用。
Voronoi 镶嵌的对偶是
*Delaunay 镶嵌*，也称为 Delaunay 三角剖分或
不规则三角网（TIN），
它在 Voronoi 多边形共享公共边的点对之间连线。

Delaunay 镶嵌是最基本的邻域结构，
因为许多其他重要的邻域结构——例如
*Gabriel 图*、*相对邻域图*
和*最小生成树*——
都可以从它派生。

[PyDelaunay](https://github.com/bennycheung/PyDelaunay) 是增量式 Delaunay 镶嵌构造算法的一个 Python 实现。该算法
基于 [Guibas & Stolfi 85] 的工作。
主算法在 `TinAlgo.py` 中给出。
我们还提供了一个便捷的 `TinBuild.py`，
可以通过命令行输入轻松运行该算法。

`python TinBuild.py --help` 会显示用法。

```
TinBuild - compute Voronoi diagram or Delaunay triangulation

TinBuild [-t -v -p -d]  [filename]

TinBuild reads from filename (or standard input if no filename given) for a set
of points in the plane and writes either the Voronoi diagram or the Delaunay
triangulation to the standard output.  Each input line should consist of two
real numbers, separated by white space.

If option -t is present, the Delaunay triangulation is produced.
Each output line is a triple i j k, which are the indices of the three points
in a Delaunay triangle. Points are numbered starting at 0.

If option -v is present, the Voronoi diagram is produced.

Other options include:

d    Output textually
p    Plot graphically
```

你可以执行 `TinBuild.py`，用以下命令行和选项交互式地绘制 Voronoi 与 Delaunay 镶嵌。

```
python TinBuild.py -v -t -p test.pts
```

![Voronoi/Delaunay 镶嵌绘图输出]({{ site.baseurl }}/images/model-of-spatial-construction/PyDelaunay_TinBuild_Plot.jpg)

另一个测试程序是 `TinAlgoTest.py`。
它生成 100 个随机点来构造 Voronoi 与 Delaunay 镶嵌，然后把输出绘制到 matplotlib 窗口。

* 注意：`TinAlgo.py` 只需要标准 Python 库。不过，如果你想使用 `TinAlgoTest.py` 或 `TinBuild.py` 提供的绘图功能，则需要 NumPy 和 mathplotlib 模块。

## 参考文献
* [Guibas & Stolfi 85] Leonidas Guibas and Jorge Stolfi, *Primitives for the Manipulation of General Subdivisions and the Computation of Voronoi Diagrams*, ACM Trans. on Graphics, Vol 4. no.2, April 1985, pp 75-123.
