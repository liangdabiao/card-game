---
layout: post
title: 地理空间粒度计算
date: 2018-12-20 00:00:00.000000000 -00:00
permalink: geospatial-granular-computing
author: Benny Cheung
artwork:
  author: Style of Byzantine mosaics transfer to a global projected map
tags:
  - Geospatial
  - Granular Computing
  - Spatial Granule
  - Geocoding
category: post
comments: true
image: images/geospatial-granular-computing/cover2x.jpg
images:
  cover: images/geospatial-granular-computing/cover.jpg
  header: images/geospatial-granular-computing/header.jpg
published: true
---

<!--excerpt.start-->
[粒度计算](https://en.wikipedia.org/wiki/Granular_computing)可以被构想为
一套理论、方法论、技术与工具的框架，
在问题求解过程中利用信息粒。尤其值得一提的是，
粒度计算已被隐式地用于地理空间表示与处理之中，
以征服大型地理数据集的复杂性与多样性。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/geospatial-granular-computing/Podcast_Geospatial_Granular_Computing.mp3
{% include open-embed.html %}
----

从"什么是粒度计算？"和"它对地理空间问题求解为什么重要？"这些问题出发，
我们将从具体问题求解的漫长历史中退后一步，重新构想一个理论、重新组织
空间构造物。借助粒度计算的思想，我们将重新发现一种地理空间问题的解法。
为了具体说明该理论，我们选择了一个重要的地理空间问题——地理编码（Geocoding）。

地理编码是接收一个地址或命名位置并返回经纬度的过程。有了地理配准的经纬度，
许多地理空间分析与处理成为可能，例如制图、路径规划、邻近搜索等。
由于我们聚焦于"地理配准"的空间部分，从现在起我们假设 *spatial* 即指 *geospatial*。
此外，本文构建的理论也可以扩展到其他地理空间分析问题。

![地理编码到地球上的经纬度]({{ site.baseurl }}/images/geospatial-granular-computing/Geocoding-Store-Locations-for-Maps.jpg)

*图：地理编码是把地址（如街道地址）转换为地理坐标（如经纬度）的过程，仿佛落下的红色图钉就是地理编码后的地址结果。*

在阐述*地理空间粒度计算*理论的过程中，我们将探索以下主题。

顺带一提，另一项统一化努力是基于地理空间语义网（Geospatial Semantic Web）的理论 [[ZhZhLi15]](#ZhZhLi15)。
地理空间语义网用 RDF 表示空间数据，用 GeoSPARQL 作为查询语言。
语义表示可以为知识引擎提供基础设施，支持从异构数据源进行
强大的推理与信息检索。
然而，用这些语言表示结构化地理空间数据的缺点
可能导致低效的数据访问，这正是我们避开这一方法的主要原因。

## 用粒度计算来思考？
从哲学意义上说，粒度计算一直是人类分析复杂世界的根本能力——
把世界抽象为相连的、层次化的粒度层级。
为了帮助围绕特定兴趣进行推理，我们会在不同粒度之间切换。
例如，规划一次长途旅行时，我们会先在*宏观*的国家层面规划行程。
然后在目的地国家的语境下，
我们切换到*微观*的城市层面，再到街道层面的粒度，最终到达某个兴趣点。
通过聚焦不同层级的粒度，相关层级的知识可以增强对
固有知识结构的理解。这种结构性系统思维因此是人类问题求解的关键技能，
也因此构成了智能的非常重要的侧面——在本文中即空间推理。

在《粒度计算统一框架》（[[PeSk08]](#PeSk08) 第 17 章）一书中，
Yiyu Yao 提出粒度计算为跨问题领域提供了三个统一的视角。

* **哲学视角**，提供一种通向结构化思维的新世界观。
* **方法论视角**，处理结构化的问题求解。
* **计算视角**，关注结构化的信息处理。

![粒度计算三角]({{ site.baseurl }}/images/geospatial-granular-computing/The-granular-computing-triangle.png)

*图：粒度计算的三个视角构成一个概念框架，统一跨问题领域的原则。（图片来源：Yiyu Yao）*

最重要的是，Yao 论证了把不同领域汇聚到粒度计算的共享概念框架中的重要优势：
可以提取不同领域的高层共性，并通过忽略特定的低层细节，把各领域的结果综合成一个整体。
随后，共享概念把隐藏在领域特定讨论中的想法显式化，
从而得出一套领域无关的原则。

把粒度计算的概念叠加到地理空间领域，
前述三个视角在系统思维上的整合，
产生了对地理空间处理的整体性理解，强调嵌入在粒之网中的结构。

## 空间粒
一个复杂的空间问题由相互连接、相互作用的多个空间部分组成。
每个空间部分又可以由其他部分组成。
每个空间部分或一组部分都可以被视为一个空间粒（spatial granule）。
这些空间粒将成为地理空间粒度计算的基本概念。
只有考虑特定的空间处理时，空间粒的物理含义才会清晰。
在为复杂的地理空间系统建模时，粒可以被结构化为不同层级的空间层。

在地理编码系统中，粒可以随不同层级的层而变化，
例如，国家是国家层中的一个粒，
省份是省份层中的一个粒。
在任何层中，粒都可以抽象地表示空间实体的各种划分。
考虑地理编码系统的应用，
空间粒可以被看作定位请求地址的某个阶段上的关注焦点。

对于空间粒的形式，我们将使用*矢量表示*来描述它们的几何。
地理编码系统不仅关心多边形，
例如国家和区域。当地理编码计算细化到街道级地址时，
它还包括点和折线。

### 地理空间粒结构
在地图中，地理数据通常组织成层。
每层表示关于地理要素的某个一致概念。
地理要素可以分类为三种关键几何数据类型，即点、折线和多边形。
相应地，这些几何可以表示位置要素、线状要素或面状要素。
许多数据类型——如标签、度量和质量——可以归于每个要素。
不失一般性，地图模型采用一种组合的层次结构。

![作为地图模型的地理空间粒]({{ site.baseurl }}/images/geospatial-granular-computing/Geospatial_Granule_Models.jpg)

*图：典型地图模型的地理空间粒结构。*

对每个地图模型，它收集许多层。对每层，它组合许多要素。对每个要素，它定义空间属性与字母数字属性。组合后的空间与字母数字属性选择图形表示及其含义。这一概念视图是地理空间粒结构的基础。

这只是描述具体地图模型的众多可能方法之一。
但以粒和多级粒度（由层表示）为要素的世界观，
构成相连且层次化的结构——这正是粒度计算的原则。

### 地理空间粒
要成为地理空间粒，该粒必须是地理配准的空间实体。
为了让空间实体可计算，我们需要把空间属性表示为几何结构。
我们为空间实体选择矢量表示：一个 Point 由其坐标元组表示。
一条 Polyline 或一个 Polygon 由 Point 表示之上的结构（列表、集合或数组）表示。
表示 Polyline 和 Polygon 向量的变体有很多。
关于所有流行矢量表示的全面综述，见 [[Westra15]](#Westra15) 第 2 章。

> 注意：我们有意避免使用开放地理空间联盟（OGC）的 WKT（well-known text）格式来表示简单要素；尽管 OGC 在业界极其重要。精确的几何表示在本文的讨论中价值不大。

为简单起见，使用以下矢量表示：

* 一个 Point 由 $$(x,y)$$ 坐标表示。
* 一条 Polyline 由点的列表 $$[p_1, …, p_n]$$ 表示，每个 pi 是一个顶点。每一对 $$(p_i, p_{i+1})$$（$$i < n$$）表示折线的一条边。
* 一个 Polygon 同样由点的列表表示。显著的差别是：该列表表示一条闭合折线，因此 $$(p_n, p_1)$$ 也是多边形的一条边。

在文本语境中，我们使用以下记号来指定空间值：

| 空间类型	| 文本记法 | 文本示例 |
|---------------|------------------|------------------|
| Point	        | (x,y)            | (4,7)
| Polyline	    | [{point}]        | [(4,4),(6,1),(3,0),(0,2)]
| Polygon	      | [{point}]        | [(4,4),(6,1),(3,0),(0,2),(4,4)]

.


### 粒关系
构建理论的重要一步，是建模信息处理起源处的*原子*粒。
粒关系（Granular Relation）是空间粒的原子表示。
由于这种关系表示是推导该理论的理论基础，
我们将以更好的数学精度与一般性来描述粒关系。

关系被建模为一个元组，由字母数字属性与空间属性共同组成。
我们采用以下约定：用 $$ST$$ 表示空间属性类型，
用 $$AT$$、$$AT_1, AT_2, …$$ 表示字母数字属性类型。
大写字母如 $$A$$、$$A_1, A_2, …$$ 用作字母数字属性名，
$$S$$、$$S_1, S_2, …$$ 用作空间属性名。值用小写字母表示。

一个 Layer 可以形式化地定义为一个粒关系，它收集一组粒元组（即原子粒）。
作为粒元组的示例，$$(A_1:a, A_2:b, S:r)$$ 指定一个要素，其空间属性 $$S$$ 的值为 $$r$$，
并带有两个字母数字属性 $$A_1$$ 与 $$A_2$$，值分别为 $$a$$ 与 $$b$$。

要成为空间粒，我们不要求字母数字属性必须存在
（例如 $$\{(S:r)\}$$ 表示包含单个要素的层）。
不过，一个层必须拥有*至少一个*空间属性。
为了更形式化地定义层，我们沿用 [[AbBe95]](#AbBe95) 的复杂对象方法
与 [[ScVo89]](#ScVo89) 的地图建模。层被定义为如下的带类型粒。

我们假设一个有限的属性名集合（无论是否为空间属性）、
一组给定的非空间域 $$\{D_1, …, D_n\}$$
以及一组给定的空间域 $$\{\triangle_1, …, \triangle_m\}$$，其中 $$\triangle_i \in \mathbb{R}^2$$。

类型由域、属性名以及集合 {} 和元组 () 构造器构建。
每层是一个类型的实例，类型定义如下：

1.	如果空间 $$\triangle$$ 是一个域，则 $$S:\triangle$$ 是一个空间类型。
2.	如果 $$AT_1, …, AT_n$$ 是类型且 $$A_1, …, A_n$$ 是未在其中使用的属性名，则 $$(A_1:AT_1, …, A_n:AT_n)$$ 是一个字母数字元组类型。
3.	如果 $$AT$$ 是一个类型且 $$A$$ 是未在其中使用的属性名，则 $$\{A:AT\}$$ 是一个集合类型。
4.	如果 $$AT$$ 是一个类型且 $$A$$ 是未在其中使用的名字，则 $$A:AT$$ 是一个命名类型。

我们用层及其关联类型的示例来说明这一定义：
* $$\{S:r, S:s\}$$，类型为 $$\{S:ST\}$$
* $$\{(A_1:a, A_2:b, S_1:r_1, S_2:r_2)\}$$，类型为 $$\{(A_1:string, A_2:string, S_1:\triangle, S_2:\triangle)\}$$
* $$\{(A_1:a, A_2:b, S:\{r_1,r_2,r_3\})\}$$，类型为 $$\{(A_1:string, A_2:string, S:{\triangle})\}$$

把前面描述的地理空间粒及其关系组合起来，
我们就能生成自己的地理空间粒结构。
我们定义层名 `Map` 为地理 `Layer` 的集合。
一个 `Layer` 是地理 `Feature` 的集合。在原子层面，我们定义 `Feature` 关系。

下面是地图模型的一般定义：

* Map = {Layer}
* Layer = {Feature}
* Feature = $$(\{A_i:AT_i\}, S:\triangle)$$

其中 $$\{A_i:AT_i\}$$ 表示命名类型（字母数字属性）的枚举。
$$AT_i$$ 对应地理空间系统支持的基本字母数字类型。
$$S$$ 表示一个空间属性的名字，其类型为 $$\triangle$$。
$$\triangle$$ 可以是地理空间系统支持的几何类型之一：Point、Polyline 或 Polygon。

作为图示，可以定义如下地理空间粒结构：
* 字母数字域集合为 {NUMBER,STRING,BOOL}
* 空间域集合为 {$$\triangle$$}
* 属性名集合为 {Name,Population,Area,Streetid,Class,Region,Shape}

粒关系为，
* City = $$(Name:STR, Population:NUM, Area:NUM, Region:\triangle)$$
* Street = $$(Streetid:NUM, Name:STR, Class:NUM, Shape:\triangle)$$

粒层为，
* City Layer = {City}
* Street Layer = {Street}

地图模型由以下构成，
* Map = {City Layer, Street Layer}

### 粒代数
继续计算视角，我们形式化地定义一个粒代数，通过一组良定义的算子实现粒处理。
粒代数的定义用于规定各域之间的操作语义。
我们参考文献中的大量成果来指导这种代数的开发。
地理空间查询语言必须遵循粒代数定义的语义，
以保持操作的适用性与封闭性。

> 注意：地理编码过程并不需要这里定义的全部粒代数。这些定义可以扩展用于执行更多的地理空间分析。

粒代数从把关系代数视为多类代数（many-sorted algebra）的视角出发。
这种代数在许多系统中已被充分发展
[[AbBe95]](#AbBe95) [[GuSc93a]](#GuSc93a) [[GuSc93b]](#GuSc93b) [[ScVo89]](#ScVo89)，
包括一些流行的商业空间数据库系统 [[RiMiVo02]](#RiMiVo02) [[ScVo92]](#ScVo92)。
代数的对象是原子值（如字符串、数字或布尔值）以及关系。
代数的算子包括数字上的算术算子（如加、减）以及关系算子（如选择、连接）。

沿循地理空间粒关系，空间属性是关系的一部分。空间类型与算子作为分类对象的一部分嵌入代数之中。空间类型封装了我们已在地理空间粒一节中定义的几何表示——点、折线和多边形。
此外，还引入了特定的空间算子用于访问和操作这些空间属性。
这里我们只窥探一下空间操作，这个话题的完整处理应当另文专门讨论。

我们的粒代数从以下排序集合开始：

| 域类型	| 描述                                                             |
|-------------|-------------------------------------------------------------------------|
| NUM	        | 数字，包括整数或实数
| STR	        | 字母数字字符串值
| BOOL        | 布尔值
| REL	        | 关系
| ST	        | 空间值

.

下面给出粒代数算子的示例清单。清单分为 3 类算子。第一类包含关系数据库系统的全部常规操作外加一些附加算子。第二类包含空间算子。其余一类包含组合算子。

#### 关系算子类表：

| 算子签名 | 算子符号 | 描述                                    |
|--------------------|-----------------|------------------------------------------------|
| $$NUM \times NUM \to NUM$$ | $$+, -, *, /$$ | 标准算术算子
| $$BOOL \times BOOL \to BOOL$$ |	and, or |	标准布尔算子
| $$BOOL \to BOOL$$	| not | 标准布尔算子
| $$REL \times REL \to REL$$ | union, intersect, difference, cross, extend, join | 二元关系算子
| $$REL \times AT_i \to REL$$ |	select, project	| 一元关系算子
| $$NUM \times NUM \to BOOL$$<br>$$STR \times STR \to BOOL$$<br>$$BOOL \times BOOL \to BOOL$$ | $$=, \lt, \gt, \le, \ge, \neq$$	| 比较算子
| $$STR \times STR \to BOOL$$ |	like, match | 字符串匹配算子
| $$REL \to NUM$$ |	count	| 统计算子
| $$NUM*$$ | sum, avg, min, max	| 统计算子

.

#### 空间算子类表：

| 算子签名 | 算子符号 | 描述                                    |
|--------------------|-----------------|------------------------------------------------|
| $$ST \to NUM$$ | diameter, length, area	| 空间度量计算算子
| $$ST \to ST$$ | bound, center, buffer	| 空间表示算子
| $$ST \times ST \to BOOL$$ |	$$=, \neq$$, inside, outside, intersect, near | 空间谓词算子
| $$ST \times AT_i \to ST$$	| select, project	| 空间对象选择算子
| $$ST \times ST \to ST$$ | union, window, clip, overlay | 二元空间算子
| $$ST \times ST \to NUM$$ | dist | 二元度量算子

.

#### 组合算子类表：

| 算子签名 | 算子符号 | 描述                                    |
|--------------------|-----------------|------------------------------------------------|
| $$REL* \to REL$$ | select:into:from,<br> select:into:from:where,<br> select:into:from:where:rankby | 扩展算子以支持类 SQL 操作

.

## 地理编码查询
在装备了地理空间粒关系与代数的全部形式化工具之后，
我们终于可以让地理编码过程活起来了。各层定义为：

```
Country = (Id:NUM, Name:STR, ShortName:STR, Shape:ST)
Region  = (Id:NUM, Name:STR, ShortName:STR, Shape:ST)
Postal  = (Id:NUM, Code:STR, Shape:ST)
City    = (Id:NUM, Name:STR, Shape:ST)
Street  = (Id:NUM, Name:STR, Shape:ST)
Place   = (Id:NUM, Name:STR, Shape:ST)
```

地理编码查询按照粒代数组合而成。
在这些示例组合中，表达式以函数式编程风格书写；
不过，确切的语言不在本文中定义。

| 查询 ID          | 层   | 粒代数                                                    |
|-------------------|---------|---------------------------------------------------------------------|
| [Geocode-Country] | Country | (project Country<br>(select (Country.Name match $country) or (Country.ShortName match $country)))
| [Geocode-Region]  | Region  | (project Region<br>(select (Region.Name match $region) or (Region.ShortName match $region) and (Region.Shape intersect Country.Shape)))
| [Geocode-Postal]  | Postal  | (project Postal<br>(select (Postal.Code match $postal) and (Postal.Shape intersect Country.Shape)))
| [Geocode-City]    | City    | (project City<br>(select (City.Name match $city) and (City.Shape intersect Region.Shape)))
| [Geocode-Street]  |	Street  | (project Street<br>(select (Street.Name match $street) and (Street.Shape intersect City.Shape)))
| [Geocode-Place]   | Place   | (project Place<br>(select (Place.Name match $place and (Place.Shape intersect City.Shape))))

.

地理编码过程可以很容易地被观察为一个过程：把一系列逐步缩小的空间粒精炼到一个原子的空间粒。
不过，粒度计算允许使用空间/关系算子集灵活地定义和处理层。
因此，粒度计算被证明是一个强大的概念框架，
足以表达地理空间问题并构建其解法。

## 地理空间分析预览
为了预览粒代数的全部效用，查询可以超越地理编码过程。
我们可以用粒结构与粒处理执行地理空间分析。

下面图示了几个地理空间分析查询，
可以用我们的地理空间粒度计算来表达。

* 找出"C"城中名为"X"的街道，且落在 UI 窗口之内。

```
(project Street
  select    (Street.Name match “X”)
        and (City.Name match “C”)
        and (Street.Shape intersect City.Shape)
        and (Street.Shape inside UIWindow.Shape))
```

* 找出靠近"Y"与"Z"交叉口附近的"X"街道。

```
(project Street
 select     (Street.Name match “X”)
        and (InterLayer.Name match “Y" and "Z”))
        and (Street.Shape near InterLayer.Shape))
```

* 找出"US"境内、靠近河流"R"的"X"街道。

```
(project Street
 select     (Street.Name match “X”)
        and (Country.Name = “US”)
        and (River.Name match “R”)
        and (Street.Shape inside Country.Shape)
        and (Street.Shape near River.Shape))
```

* 找出所有距我家 1 公里以内的街道。

```
(project Street
 select     (Place.Name match “my home”)
        and (Street.Shape intersect
             (bound (center Place.Shape) 1000)))
```

## 结语
我们展示了粒度计算应用于地理空间领域的核心概念。
虽然我们在文中避免使用任何现有的 GIS 技术与标准，但对那些
深谙最前沿 GIS 的读者来说，可以轻松地把这些概念与实际的地理空间实现对应起来。
通过消化这些抽象的讨论，
我们可以专注于粒度计算中结构性思维的高层共性；
随后，我们可以把类似的结构性思维综合到其他领域，去解决有趣的空间问题。

## 参考文献
* <a name="AbBe95">[AbBe95]</a> S. Abiteboul and C. Beeri, “On the power of languages for the manipulation of complex objects”, The VLDB Journal, vol.4, no.4, pp. 727-794, 1995.
* <a name="GuSc93a">[GuSc93a]</a> R.H. Guting and M. Schneider, “Realms: A Foundation for Spatial Data Types in Database Systems”, in Advance in Spatial Databases (SSD’93). Lecture Notes in Computer Science No. 692, 1993, Springer-Verlag, Berlin.
* <a name="GuSc93b">[GuSc93b]</a> R.H. Guting and M. Schneider, “Realm-Based Spatial Data Types: The ROSE Algebra”, Tech. Report 141, Fern University, Hagen, Germany, 1993.
* <a name="PeSk08">[PeSk08]</a> W. Pedrycz, A. Skowron, V. Kreinovich (Editors), "Handbook of Granular Computing", 2008, Wiley-Interscience, ISBN:9780470035542.
* <a name="RiScVo02">[RiScVo02]</a> P. Rigaux, M. Scholl and A. Voisard, “Spatial Databases: with Application to GIS”, Academic Press, Morgan Kaufmann Pub., 2002. ISBN:9781558605886
* <a name="ScVo89">[ScVo89]</a> M. Scholl and A. Voisard, “Thematic Map Modeling”, in Design and Implementation of Large Spatial Databases (SSD’89). Lecture Notes in Computer Science No. 409, 1989, pp. 167-192, Springer-Verlag, Berlin.
* <a name="Westra15">[Westra15]</a> E. Westra, “Python Geospatial Analysis Essentials”, 2015, Packt Publishing, UK. ISBN:9781782174516
* <a name="ZhZhLi15">[ZhZhLi15]</a> C. Zhang, T. Zhao and W. Li, "Geospatial Semantic Web", 2015, Springer, Switzerland. ISBN:9783319178011
