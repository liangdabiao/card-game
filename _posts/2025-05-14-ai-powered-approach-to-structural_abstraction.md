---
layout: post
title: "AI 驱动的结构抽象方法——逆向工程中的 KRA 模型"
date: 2025-05-14 12:00:00.000000000 -00:00
permalink: ai-powered-approach-to-structural_abstraction/
author: Benny Cheung
artwork:
  author: GPT-4o
tags:
- AI
- Abstraction
- Reverse Engineering
- Philosophy
category: post
comments: true
image: images/ai-powered-approach-to-structural_abstraction/post_cover.jpg
images:
  cover: images/ai-powered-approach-to-structural_abstraction/cover2x.jpg
  header: images/ai-powered-approach-to-structural_abstraction/header.jpg
published: true
series: "Philosophy of AI"
series_order: 4
---

<!--excerpt.start-->
你是否曾感到完全被淹没？面对一个复杂系统、一座数据大山或一篇晦涩难懂的技术文档，就像试图从消防水管里喝水。未经筛选的海量原始信息实在令人应接不暇。穿透这些噪声、找到核心本质、真正领会复杂思想的精髓，是一项根本性的挑战。
<!--excerpt.end-->



这正是抽象（abstraction）发挥作用的地方。抽象绝非一个陈腐的学术概念，而是一种至关重要、无处不在的工具，它塑造着我们与世界互动的方式、我们的思维方式，以及最关键的——我们构建和理解智能系统的方式。透过 **KRA（Knowledge Representation & Abstraction，知识表示与抽象）模型** [1] 的视角，并考虑如何利用 AI，我们可以看到抽象不仅是一个哲学概念，更是驾驭复杂性的一种操作性必需品。

![复杂系统信息抽象化]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/Complex_System_Information_Abstracted.jpg)

*图：被复杂性淹没——在一个信息未经筛选、系统错综复杂的世界里，抽象是获得清晰与理解的关键。*

抽象最引人注目的是它的普遍性。它的词源本义是"抽取"，但其含义早已演化。它不仅仅是把东西拿走，更是有目的的简化。正如 Goldstone 和 Barsalou [4] 所说："抽象就是从表层的包装中提炼出本质。"这一过程无处不在——在哲学中，它规范化了我们如何形成抽象观念；在数学中，它在具体数字之外寻找一般模式；它还存在于自然语言、艺术以及我们自身的认知过程之中。在计算机科学中，它更是绝对不可或缺：没有抽象，我们根本无法应对现代软件和数据结构的庞大规模与错综复杂，只会深陷于细枝末节而无法自拔。

本文与 Saitta 和 Zucker 在《Abstraction In Artificial Intelligence And Complex Systems》[1] 中提出的观点高度一致，同时加入我们自己的视角，探讨如何在逆向工程 [2,3] 的语境下实际应用抽象。

## AI 中的抽象

尽管抽象的身影遍布许多领域，但当我们谈到人工智能时，它变得绝对不可或缺。要让 AI 攻克真正艰难的问题——规划复杂的动作序列、求解调度这类大规模约束满足问题、表示海量知识，或者让智能体（如机器人）在现实世界中运作——它不能只在最底层的细节上运转，而是需要抽象视图。纵观近期的研究可以发现，抽象对先进的 AI 而言绝不仅仅是锦上添花，而是基石。如果没有对抽象的扎实掌握和运用，我们所设想的许多精巧 AI 根本不可能实现。

![复杂系统信息抽象]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/Complex_System_Information_Abstraction.jpg)

*图：跨学科的抽象——突出其在哲学、数学、艺术/自然语言和计算机科学中的基础性作用。*

鉴于其重要性，如何精确定义抽象本身就颇为棘手——这或许再恰当不过了。目前并不存在一个得到普遍认同的唯一定义。不同的理论关注不同的方面：句法方法着眼于对信息结构的操作（比如通过缩短句子来概述文本），而语义方法则关注意义的保留（确保摘要抓住核心信息）。粒度（granularity，细节层级，就像看城市地图与街道地图的区别）和重构（reformulation，改变表示方式，比如讨论消费趋势而不是罗列单笔购买记录）等相关概念都与抽象相互作用，但抽象的核心特征使它与众不同。

这些特征中最关键的是**信息缩减**（information reduction）——刻意去除与当前目标无关的细节。它通常还是一种内涵属性，关乎概念和意义，而非具体实例。至关重要的是，抽象是**相对的**：什么是抽象，完全取决于情境和目的。它本质上是一个**过程**，一种主动的变换，而非静态的状态。它还关键地涉及**信息隐藏**（information hiding），即有意隐藏细节以简化关注焦点。虽然泛化（找出共同特征以便归类）或逼近（用不那么精确的信息替换精确信息）与之相关，但抽象的独特之处在于，它把信息缩减与针对特定任务对本质内容的目的性聚焦结合在了一起。

## KRA 模型：可操作抽象的框架

好了，我们已经理解了这个概念，但该如何*动手*实现它，尤其是以结构化的方式为 AI 服务？这正是 KRA 模型 [1] 的用武之地。作为一个核心框架，KRA 提供了一种系统化的方式来理解和执行抽象，使其具有可操作性。它对抽象的定义并非凭空进行，而是与以下要素绑定在一起：

*   **查询环境（Q，Query Environment）：**驱动抽象需求的具体任务或问题。抽象是*相对于* Q 执行的——什么内容是相关的，取决于你想要弄清楚或达成什么。
*   **描述框架（Γ，Description Frame）：**这是模式或词汇表，定义了在被建模系统中*可以被观察和推理*的元素类型（对象、属性、函数、关系）。它是潜在的结构。
*   **配置空间（Ψ，Configuration Space）：**这是基于描述框架（Γ）可以构建的所有可能系统状态或描述的全体。
*   **数据生成（Data Generation）：**我们从系统获取实际信息的过程——即填充模型的原始观察数据。

它与描述框架（Γ）不同——后者是*潜在*可观察元素的模式；也与配置空间（Ψ）不同——后者是基于 Γ 的*所有可能*描述构成的空间。P-Set 则是在特定时刻填充这一框架的具体观测数据。

### P-Set：基准真值数据

由此我们来到 KRA 中观测数据的核心：**P-Set**，即*感知集*（perception set）。它是我们的"传感器"所感知到的系统快照，具体而入微。如果说描述框架（Γ）是蓝图，那么 P-Set 就是某一给定时刻实际填写好的表单。

![P-Set 定义]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/PSet_Definition.jpg)

P-Set 被形式化地定义为一个四元组：`P = (O, A, F, R)`，其中：

1.  **O（对象，Objects）：**这是在系统 S 中观察到的（带类型的）对象标识符的实际集合（例如，`'server1'` 的类型是 `'server'`）。它们是 Γ 中所定义类型的实例。
2.  **A（属性，Attributes）：**这表示在 O 中的实际对象上观察到的属性（在 Γ 中定义）的具体实例化。它是针对检测到的具体对象测量得到的"属性-值"对的集合（例如，`('server1', 'status') = 'online'`）。
3.  **F（函数，Functions）：**这包含在 O 中的实际对象上观察到的函数（在 Γ 中定义）的具体实例化（即满足该函数的覆盖集或元组集合）。它表示由函数捕捉到的、针对检测到的对象的观测关系（例如，`calculate_distance('point A', 'point B') = 5.3`）。函数把对象元组映射到值。
4.  **R（关系，Relations）：**这包含在 O 中的实际对象上观察到的关系（在 Γ 中定义）的具体实例化（即满足该关系的覆盖集或元组集合）。它表示由关系捕捉到的、针对检测到的对象的观测关系（例如，`'server1' connects_to 'database alpha'`）。关系描述对象元组之间的关系（隐含布尔真值）。

从本质上说，P-Set 是：

*   **观测到的数据：**它是你从系统 S"感知"到的东西。
*   **描述框架（Γ）的一个实例：**Γ 定义的是*可能*被观察到的类型、属性、函数和关系，而 P-Set 包含的是特定情境下*实际观察到的*实例和值。
*   **与配置空间（Ψ）相关联：**一个 P-Set 与 Ψ 中某个配置子集相容。如果所有观测都被完整指定（没有 "UN" 值），P-Set 就对应唯一的配置；如果存在 "UN" 值，它就对应一组相容的配置 [1, p. 156-157]。
*   **KRA 中抽象的起点：**在 KRA 模型自底向上的视角中，P-Set（$P_g$）代表"基准真值"（ground truth），是与抽象算子相关联的方法（如 `meth[Pg, ω]`）的输入，用于生成更抽象的 P-Set（$P_a$）[1, p. 175, 204]。

这个 P-Set 就是 KRA 抽象算子所作用的细粒度基准真值。

#### 示例：电子商务系统（微服务架构）

假设我们正在观察一个运行中的电子商务系统，由传感器（Σ）收集运行时的架构数据。描述框架（Γ）定义了该系统中*可能*存在哪些类型的组件、连接和属性。

我们可以在 Γ 中定义如下元素：

*   **类型：**`Service`、`Database`、`API Gateway`
*   **属性：**`status(Service) -> {Running, Stopped}`、`latency(Service) -> Float`、`replicas(Service) -> Int`
*   **函数：**`connects_to(Service, Service) -> Boolean`、`persists_to(Service, Database) -> Boolean`
*   **关系：**`co_located(Service, Service)`——用于部署在同一区域的服务。

现在假设你的监控工具观察该系统，并在给定时间点生成一个**具体 P-Set（观测到的系统状态）**：

![微服务系统示例的抽象 P-Set]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/Abstraction_PSet_Microservice_System_Example.jpg)

*图：用一个具体的 P-Set 示例来说明观测到的系统状态。*

这个 P-Set 结构给出了电子商务系统在运行时架构的一个精确的、*被观测到的*实例。它展示了：

*   存在哪些服务和数据库（**O**）
*   它们当前的状态和性能指标（**A**）
*   服务之间如何连接或者如何存储数据（**F**）
*   服务之间在物理或逻辑上如何关联（**R**）

具体而言，根据图示和说明，P-Set 可以表示为：

##### 1. O——对象

```
O = {
    s1:Service ('AuthService'),
    s2:Service ('OrderService'),
    s3:Service ('InventoryService'),
    db1:Database ('UserDB'),
    db2:Database ('OrderDB'),
    gw1:API Gateway ('MainGateway')
}
```
*（注：在典型的 P-Set 中，对象很可能只是 's1'、's2' 之类的 ID，并显式列出其类型。这里的描述性名称 'AuthService' 只是为了清晰起见。）*

##### 2. A——属性

```
A = {
    ('s1', 'status') = 'Running',
    ('s2', 'status') = 'Running',
    ('s3', 'status') = 'Stopped',
    ('s1', 'latency') = 12.5,
    ('s2', 'latency') = 35.1,
    ('s3', 'latency') = 'UN',       # 因服务宕机而未知
    ('s1', 'replicas') = 3,
    ('s2', 'replicas') = 2,
    ('s3', 'replicas') = 1
}
```

##### 3. F——函数

```
F = {
    'connects_to': [('s1', 's2'), ('s2', 's3')], # AuthService -> OrderService, OrderService -> InventoryService
    'persists_to': [('s1', 'db1'), ('s2', 'db2')] # AuthService -> UserDB, OrderService -> OrderDB
}
```
*（注：函数把元组映射到值。如果 `connects_to` 是布尔型的，元组的存在即蕴含 `True`。图示明确画出了这些连接。）*

##### 4. R——关系

```
R = {
    'co_located': [('s1', 's2')] # Auth 和 Order 位于同一部署区域
}
```
*（注：与函数类似，元组的存在即蕴含该关系成立。）*

### KRA 抽象算子

KRA 抽象算子 [1, p. 203+] 是执行这种从细粒度 P-Set（$P_g$）到更抽象 P-Set（$P_a$）变换的工具。它们在概念上定义于描述框架（Γ）的层面，并有对应的*方法*（`meth`）把变换应用到 P-Set 中的实际数据上。这些方法被组织为修改 P-Set 的 O、A、F、R 组件，执行"提炼"工作。

这些算子按其机制分类 [1, p. 204-209]：

![KRA 抽象算子图解]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/KRA_Abstraction_Operators_Illustrated.jpg)

*图：KRA 抽象算子通过方法 ω 将 P-Set P(O,A,F,R) 变换为抽象 P-Set P'(O',A',F',R')。*

*   **隐藏算子（`ωh...`）：**有选择地从 P-Set 视图中移除元素、属性、属性值、函数定义、关系定义或参数。它们使信息不可用。
*   **等价算子（`ωeq...`）：**把相似的元素或值归入等价类。它们用单个抽象代表替换多个具体实例。
*   **层次生成算子（`ωhier...`）：**构建组合层次或类型层次。它们从低层元素派生出新的、更高层的元素或属性。
*   **组合算子（`ωcoll`、`ωaggr`、`ωgroup`、`ωconstr`）：**把多个已有事物归并为单一实体，或基于已有元素构造新元素。它们聚合信息。
*   **逼近算子（`pid...`、`prepl`）：**用不那么精确但有代表性的值替换精确信息，往往损失一些保真度却换来简洁性。它们改变信息的细节层级。

下面这张汇总表根据 KRA 模型文献 [1, Appendix E Table E.6, Chapter 7] 整理而成，展示了各个类别及具体算子：

---

**抽象与逼近算子汇总表（根据 KRA 模型 [1] 整理）**

| 类别        | 算子       | 类型         | 作用于 Γ（算子定义中的参数） | 方法作用于 P | 描述/目的                                                                                                          |
| :-------------- | :------------- | :----------- | :------------------------------------------- | :--------------- | :----------------------------------------------------------------------------------------------------------------- |
| **隐藏**      | `ωhobj`        | 抽象  | o（对象标识符）                        | P-Set P          | 在视图中隐藏特定对象。                                                                                  |
|                 | `ωhtype`       | 抽象  | t（类型标识符）                          | P-Set P          | 在视图中隐藏特定类型。                                                                                    |
|                 | `ωhattr`       | 抽象  | (Am, Am)（属性及其域）          | P-Set P          | 隐藏特定的属性定义/视图。                                                                        |
|                 | `ωhfun`        | 抽象  | fh（函数标识符）                     | P-Set P          | 隐藏特定的函数定义/视图。                                                                         |
|                 | `ωhrel`        | 抽象  | Rk（关系标识符）                     | P-Set P          | 隐藏特定的关系定义/视图。                                                                         |
|                 | `ωhfunarg`     | 抽象  | fh, xj（函数标识符，参数索引） | P-Set P          | 隐藏函数定义中的特定参数。                                                                 |
|                 | `ωhrelarg`     | 抽象  | Rk, xj（关系标识符，参数索引） | P-Set P          | 隐藏关系定义中的特定参数。                                                                 |
|                 | `ωhattrval`    | 抽象  | (Am, Am), vi（属性，域，值）      | P-Set P          | 隐藏属性域中的特定值（可用于离散化）。                                   |
|                 | `ωhfunargval`  | 抽象  | fh, xj, o（函数标识符，参数索引，参数值）    | P-Set P          | 隐藏函数参数域中的特定值。                                                               |
|                 | `ωhrelargval`  | 抽象  | Rk, xj, o（关系标识符，参数索引，参数值）     | P-Set P          | 隐藏关系参数域中的特定值。                                                               |
|                 | `ωfuncodom`    | 抽象  | fh, CD(fh), v（函数标识符，余域，值）     | P-Set P          | 隐藏函数余域中的特定值。                                                                    |
| **等价**    | `ωeqobj`       | 抽象  | φeq, o(a)（条件，抽象对象名）  | P-Set P          | 基于条件为对象构建等价类。                                                       |
|                 | `ωeqattr`      | 抽象  | φeq, (A(a), ΛΑ(a))（条件，抽象属性） | P-Set P          | 基于条件为属性构建等价类。                                                      |
|                 | `ωeqfun`       | 抽象  | φeq, f(a)（条件，抽象函数）     | P-Set P          | 基于条件为函数构建等价类。                                                       |
|                 | `ωeqrel`       | 抽象  | φeq, R(a)（条件，抽象关系）     | P-Set P          | 基于条件为关系构建等价类。                                                       |
|                 | `ωeqtype`      | 抽象  | φeq(t1,..., tk), t(a)（条件，抽象类型） | P-Set P          | 基于条件为类型构建等价类。                                                         |
|                 | `ωeqattrval`   | 抽象  | (Α, ΛΑ), ΛΑ,eq, v(a)（属性，域，值集合，抽象值） | P-Set P          | 为属性值构建等价类（可用于离散化）。                                |
|                 | `ωeqfunargval` | 抽象  | fh, xj, ГО,eq, o(a)（函数标识符，参数索引，等价集合，抽象值） | P-Set P          | 为函数参数值构建等价类。                                                           |
|                 | `ωeqrelargval` | 抽象  | Rk, xj, ГО,eq, o(a)（关系标识符，参数索引，等价集合，抽象值） | P-Set P          | 为关系参数值构建等价类。                                                           |
|                 | `ωeqfuncodom`  | 抽象  | fh, CD(fh), Veq, v(a)（函数标识符，余域，等价集合，抽象值） | P-Set P          | 为函数余域值构建等价类。                                                           |
|                 | `ωeqfunarg`    | 抽象  | fh, Zeq, z(a)（函数标识符，等价集合，抽象值） | P-Set P          | 为函数参数构建等价类（基于其取值集合）。                          |
|                 | `ωeqrelarg`    | 抽象  | Rk, Zeq, z(a)（关系标识符，等价集合，抽象值） | P-Set P          | 为关系参数构建等价类（基于其取值集合）。                          |
| **层次**   | `ωhierattr`    | 抽象  | ΓΑ,child, (A(a), Λ(a))（子属性，抽象属性） | P-Set P          | 构建属性层次，从子属性派生出抽象属性。                                  |
|                 | `ωhierfun`     | 抽象  | ΓF,child, f(a)（子函数，抽象函数）  | P-Set P          | 构建函数层次，从子函数派生出抽象函数。                                   |
|                 | `ωhierrel`     | 抽象  | ΓR,child, R(a)（子关系，抽象关系）    | P-Set P          | 构建关系层次，从子关系派生出抽象关系。                                   |
|                 | `ωhiertype`    | 抽象  | ΓΤΥΡΕ, child, t(a)（子类型，抽象类型） | P-Set P          | 构建类型层次，从子类型派生出抽象类型。                                                 |
|                 | `ωhierfuncodom`| 抽象  | fh, CD(fh)child, v(a)（函数标识符，子余域值，抽象值） | P-Set P          | 为函数余域值构建层次。                                                                   |
| **组合**   | `ωcoll`        | 抽象  | t, t(a)（原类型，新类型）            | P-Set P          | 通过把多个同类型对象归为一组来构建聚合对象。                                           |
|                 | `ωaggr`        | 抽象  | (t1,..., ts), t(a)（原类型，新类型） | P-Set P          | 把不同类型的对象聚合为一个复合对象。                                                     |
|                 | `ωgroup`       | 抽象  | φgroup, G(a)（条件，抽象组名） | P-Set P          | 基于条件构建组对象（通常是 `ωeqobj` 的扩展，为等价类创建一个新对象）。 |
|                 | `ωconstr`      | 抽象  | Constr, y（构造规则，新元素名） | P-Set P          | 使用规则从已有元素构造新的描述元素（属性、函数、关系）。            |
| **逼近**| `prepl`        | 逼近| X(g), y, y(ap)（所涉集合，元素，逼近） | P-Set P          | 用一个逼近元素替换一组描述元素。                                                  |
|                 | `pidobj`       | 逼近| φid, o(p)（条件，代表对象名） | P-Set P          | 通过选择一个代表使一组对象变得不那么精确。                                               |
|                 | `pidtype`      | 逼近| φid（类型）                                  | P-Set P          | 使一组类型变得不那么精确。                                                                               |
|                 | `pidattr`      | 逼近| φid（属性）                             | P-Set P          | 使一组属性定义变得不那么精确。                                                                 |
|                 | `pidfun`       | 逼近| φid（函数）                              | P-Set P          | 使一组函数定义变得不那么精确。                                                                  |
|                 | `pidrel`       | 逼近| φid（关系）                              | P-Set P          | 使一组关系定义变得不那么精确。                                                                  |
|                 | `pidattrval`   | 逼近| (Α, ΛΑ), ΛΑ,id, v(p)（属性，域，值集合，代表值） | P-Set P          | 逼近属性值（用一个代表值替换一组值）。                                       |
|                 | `pidfunargval` | 逼近| fh, xj, ГО,eq, o(a)（函数标识符，参数索引，等价集合，代表值） | P-Set P          | 逼近函数参数值（用一个代表值替换一组值）。                               |
|                 | `pidrelargval` | 逼近| Rk, xj, ГО,eq, o(a)（关系标识符，参数索引，等价集合，代表值） | P-Set P          | 逼近关系参数值（用一个代表值替换一组值）。                               |
|                 | `pidfuncodom`  | 逼近| fh, CD(fh), Veq, v(p)（函数标识符，余域，等价集合，代表值） | P-Set P          | 逼近函数余域值（用一个代表值替换一组值）。                               |
|                 | `pidfunarg`    | 逼近| fh, Zid, z(a)（函数标识符，等价集合，代表值） | P-Set P          | 逼近函数参数（基于其取值集合）。                                            |
|                 | `pidrelarg`    | 逼近| Rk, Zid, z(a)（关系标识符，等价集合，代表值） | P-Set P          | 逼近关系参数（基于其取值集合）。                                            |

----

*这些算子根据其所采用的机制以及所作用的描述框架元素或值进行分类。文中提到的主要抽象机制类别包括：隐藏信息、通过构建等价类使信息变得不那么细化、生成层次结构，以及组合元素。汇总表中还包含逼近算子，其定义为：识别出一组元素并用其中某个成员替换它们。*

定义在描述框架（Γ）上的每个算子概念（`ω`）都有一个对应的方法（`meth`），实现作用于 P-Set 数据的具体变换逻辑。

### 示例：实现 `meth_hidetype`

我们从隐藏类别中选取 `ωhtype` 算子来考察，它会隐藏与特定类型的对象相关的所有信息。我们想把它的方法 `meth_hidetype` 实现为一个作用于 P-Set 的 Python 函数。

这个 Python 实现把 `ωhtype` 算子经由其 `meth_hidetype` 方法作用于 P-Set 结构的效果转化为代码，演示了这一特定抽象算子在 KRA 模型的数据层面是如何工作的。

```python
import copy

def meth_hidetype(pg_pset, target_type):
    """
    whtype 抽象算子的 meth[Pg, whtype(target_type)] 方法实现，
    基于 KRA 模型结构 [1, Appendix E, Chapter 7]。

    该方法将"隐藏类型"抽象应用到一个特定的 P-Set（pg_pset）上。

    它把 target_type 类型的所有对象的类型替换为通用的 'obj' 类型
    （或者移除它们，取决于具体的抽象目标——这里我们采用重命名）。
    同时，它把这些对象的特定属性值设为 'UN'（未定义/未知），
    并从函数和关系覆盖中移除包含原类型对象的元组。

    抽象 P-Set（pa_pset）中会添加/更新一个 MEMORY 字段，
    用于存储潜在具体化（下钻）所需的信息。

    参数：
        pg_pset (dict)：基准 P-Set，其结构为：
                        {'O': [(obj_id, type), ...],
                         'A': [(obj_id, attr_name, value), ...],
                         'F': {func_name: [tuple, ...]},
                         'R': {rel_name: [tuple, ...]},
                         'MEMORY': {}}  # 初始或已有的记忆字段
        target_type (str)：要隐藏的类型标识符。

    返回：
        dict：应用 meth_hidetype 方法后得到的抽象 P-Set（Pa）。
              （注：此处并未完整实现用于复杂具体化的完整记忆结构，
              但已勾画出基本结构和数据存储方式。）
    """
    # 从深拷贝开始，避免修改原始 P-Set
    pa_pset = copy.deepcopy(pg_pset)

    # --- 找出目标类型的对象 ---
    # 我们需要目标类型对象的 ID，以便知道稍后要修改哪些属性/元组。
    # 同时为记忆/具体化保存原始类型。
    objects_of_target_type = [(obj_id, obj_type) for obj_id, obj_type
                             in pg_pset['O'] if obj_type == target_type]
    objects_of_target_type_ids = [obj_id for obj_id, _ in objects_of_target_type]

    # --- 为具体化信息准备记忆结构 ---
    # 这是一个占位结构，用于存储逆转该过程或提供下钻细节
    # 可能需要的信息。
    # 在完整的 KRA 模型中，这个结构会更详细，
    # 并且可能处理对连续操作记忆的合并。
    # 对于这个简单算子，我们只存储它所做的更改。
    memory_for_this_op = {
        'type_changes': [],         # 记录类型被更改对象的 (obj_id, original_type)
        'attribute_value_changes': [], # 记录被设为 'UN' 的值的 (obj_id, attr_name, original_value)
        'removed_f_tuples': {},     # 存储 {func_name: [removed_tuple, ...]}
        'removed_r_tuples': {}      # 存储 {rel_name: [removed_tuple, ...]}
    }

    # --- 对 P-Set 组件应用变换 ---

    # 1. 修改对象（O）：把 target_type 对象的类型改为 'obj'
    # 我们为抽象 P-Set 中的 O 构建一个新列表。
    pa_o = []
    for obj_id, obj_type in pg_pset['O']: # 遍历原始 O
        if obj_type == target_type:
            # 把类型改为通用的 'obj'
            pa_o.append((obj_id, 'obj'))
            # 把原始类型存入记忆
            memory_for_this_op['type_changes'].append((obj_id, obj_type))
        else:
            # 其他类型的对象保持原样
            pa_o.append((obj_id, obj_type))
    pa_pset['O'] = pa_o # 更新抽象 P-Set 中的 O 组件

    # 2. 修改属性（A）：把原类型对象的属性值设为 'UN'
    # 我们遍历原始 A（pg_pset['A']）以捕获原始值供记忆使用，
    # 并构建新的 A（pa_a）。
    pa_a = []
    for obj_id, attr_name, value in pg_pset['A']:
        if obj_id in objects_of_target_type_ids:
            # 把被隐藏类型对象的值设为 'UN'
            # 注意：假定 'UN' 是未定义/隐藏值的有效占位符
            # 在真实系统中，可能需要基于属性域进行特定处理
            pa_a.append((obj_id, attr_name, 'UN'))
            # 把原始属性值存入记忆
            memory_for_this_op['attribute_value_changes'].append((obj_id, attr_name, value))
        else:
            # 其他对象的属性保持原样
            pa_a.append((obj_id, attr_name, value))
    pa_pset['A'] = pa_a # 更新抽象 P-Set 中的 A 组件


    # 3. 修改函数（F）：移除包含原类型对象的元组
    # 我们遍历原始 F 覆盖（pg_pset['F']）以便干净地进行元组检查。
    pa_f = {}
    # 辅助函数：检查元组中是否有元素是目标对象 ID 之一
    def tuple_contains_target_object(tup, target_ids_list):
        # 简单检查：该元素是否可能是对象 ID（即为字符串/ID 类型），
        # 并且是否在目标 ID 列表中？
        # 假定 P-Set 元组中的对象 ID 的表示方式可与 target_ids_list 的元素比较。
        for element in tup:
            if isinstance(element, str) and element in target_ids_list:
                 # 注意：这只是简单检查。根据元组结构以及哪些元素
                 # 可以是对象 ID，可能需要更稳健的检查。
                return True
        return False

    for func_name, cover_tuples in pg_pset['F'].items():
        pa_f[func_name] = [] # 在 pa_f 中为该函数初始化元组列表
        memory_for_this_op['removed_f_tuples'][func_name] = [] # 为被移除的元组初始化记忆

        for tup in cover_tuples:
            if not tuple_contains_target_object(tup, objects_of_target_type_ids):
                # 保留不涉及被隐藏类型对象的元组
                pa_f[func_name].append(tup)
            else:
                # 移除涉及被隐藏类型对象的元组
                # 把被移除的元组存入记忆
                memory_for_this_op['removed_f_tuples'][func_name].append(tup)

    pa_pset['F'] = pa_f # 更新抽象 P-Set 中的 F 组件


    # 4. 修改关系（R）：移除包含原类型对象的元组
    # 我们遍历原始 R 覆盖（pg_pset['R']）以便干净地进行元组检查。
    pa_r = {}
    for rel_name, cover_tuples in pg_pset['R'].items():
        pa_r[rel_name] = [] # 在 pa_r 中为该关系初始化元组列表
        memory_for_this_op['removed_r_tuples'][rel_name] = [] # 为被移除的元组初始化记忆

        for tup in cover_tuples:
            if not tuple_contains_target_object(tup, objects_of_target_type_ids):
                # 保留不涉及被隐藏类型对象的元组
                pa_r[rel_name].append(tup)
            else:
                # 移除涉及被隐藏类型对象的元组
                # 把被移除的元组存入记忆
                memory_for_this_op['removed_r_tuples'][rel_name].append(tup)

    pa_pset['R'] = pa_r # 更新抽象 P-Set 中的 R 组件


    # --- 更新 P-Set 记忆 ---
    # 'MEMORY' 键存储抽象历史。每次操作都可以
    # 以方法名或算子参数为键，添加自己的记忆结构。
    # 在本例中，我们添加/替换与这一特定操作相关的记忆。
    if 'MEMORY' not in pa_pset:
        pa_pset['MEMORY'] = {}
    pa_pset['MEMORY']['meth_hidetype'] = memory_for_this_op

    return pa_pset
```

#### 测试 `meth_hidetype`

为了测试这一实现，我们来构造一个带有多种属性的图形的合成示例，类似 KRA 文献 [1] 中的例子。

![抽象 P-Set 示例]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/Abstraction_PSet_Example.jpg)

*图：合成示例——图形 'a'、'b'、'c'、'd' 以及一条线段 'AB'。*

我们基于 `{'O': [...], 'A': [...], 'F': {...}, 'R': {...}, 'MEMORY': {}}` 这一结构定义一个简单的基准 P-Set（`ground_pset`）：

```python
# 使用示例

# 基于 KRA 示例思想 [1, 如 6.3/6.5] 定义一个简单的基准 P-Set
# 假定对象 ID 是 'a'、'b' 之类的字符串，类型是字符串，属性是字符串/数字。
ground_pset = {
    'O': [('a', 'figure'), ('b', 'figure'), ('c', 'figure'), ('d', 'figure'),
          ('A', 'point'), ('B', 'point'), ('AB', 'segment')], # 加入 AB 线段
    'A': [('a', 'Color', 'green'), ('a', 'Shape', 'triangle'),
          ('b', 'Color', 'blue'), ('b', 'Shape', 'square'),
          ('c', 'Color', 'red'), ('c', 'Shape', 'circle'),
          ('d', 'Color', 'green'), ('d', 'Shape', 'rectangle'),
          ('AB', 'Color', 'black'), ('AB', 'Length', 10)], # AB 线段的属性
          # 在这个简化示例中，点 'A' 和 'B' 可能没有属性
    'F': {'Center': [('c', 'A')], # 示例函数覆盖：'c' 的 Center 是 'A'。注意：为简单起见，与图形 'A'/'B' 点并不一致。
          'Radius': [('c', 5)]},  # 示例函数覆盖：'c' 的 Radius 是 5。
    'R': {'Rontop': [('a', 'b'), ('c', 'd')], # 示例关系覆盖：'a' 在 'b' 之上，'c' 在 'd' 之上
          'Rleftof': [('a', 'c')],         # 'a' 在 'c' 左边
          'RisPortof': [('A', 'a'), ('B', 'b'), ('AB', 'AB')]}, # 'A' 是 'a' 的一部分，'B' 是 'b' 的一部分，'AB' 是 'AB' 线段对象自身的一部分。
    'MEMORY': {} # 初始空记忆
}

print("--- Original P-Set ---")
print(f"O: {ground_pset['O']}")
print(f"A: {ground_pset['A']}")
print(f"F covers: {ground_pset['F']}")
print(f"R covers: {ground_pset['R']}")
print(f"MEMORY: {ground_pset['MEMORY']}")
print("-" * 30)

# 测试 1：要隐藏的目标类型（如 'point'）
target_type_to_hide_1 = 'point'
abstract_pset_1 = meth_hidetype(ground_pset, target_type_to_hide_1)

print(f"--- Abstract P-Set (after hiding '{target_type_to_hide_1}') ---")
print(f"O: {abstract_pset_1['O']}")
print(f"A: {abstract_pset_1['A']}")
print(f"F covers: {abstract_pset_1['F']}")
print(f"R covers: {abstract_pset_1['R']}")
print(f"MEMORY (meth_hidetype): {abstract_pset_1['MEMORY'].get('meth_hidetype', 'Not Stored')}")
print("-" * 30)


# 测试 2：另一种类型的示例（如 'figure'）
# 为了得到干净的对比，再次对*原始* ground_pset 应用抽象
target_type_to_hide_2 = 'figure'
abstract_pset_2 = meth_hidetype(ground_pset, target_type_to_hide_2)

print(f"--- Abstract P-Set (after hiding '{target_type_to_hide_2}') ---")
print(f"O: {abstract_pset_2['O']}")
print(f"A: {abstract_pset_2['A']}")
print(f"F covers: {abstract_pset_2['F']}") # 注意：如果图形出现在函数元组中，F 覆盖可能会改变
print(f"R covers: {abstract_pset_2['R']}") # 注意：由于图形（'a'、'b'、'c'、'd'）已从 R 元组中移除，R 覆盖现在为空
print(f"MEMORY (meth_hidetype): {abstract_pset_2['MEMORY'].get('meth_hidetype', 'Not Stored')}")
print("-" * 30)


# 测试 3：使用一个不存在的类型的示例（应照常运行，除添加记忆外不做任何更改）
target_type_to_hide_3 = 'car'
abstract_pset_3 = meth_hidetype(ground_pset, target_type_to_hide_3)

print(f"--- Abstract P-Set (after hiding '{target_type_to_hide_3}') ---")
# 我们预计除为本操作添加 'MEMORY' 字段之外没有任何更改
print("O: (No change)", abstract_pset_3['O'] == ground_pset['O'])
print("A: (No change)", abstract_pset_3['A'] == ground_pset['A'])
print("F covers: (No change)", abstract_pset_3['F'] == ground_pset['F'])
print("R covers: (No change)", abstract_pset_3['R'] == ground_pset['R'])
print(f"MEMORY (meth_hidetype): {abstract_pset_3['MEMORY'].get('meth_hidetype', 'Not Stored')}")
print("-" * 30)
```

运行测试后，输出如下：

```text
--- Original P-Set ---
O: [('a', 'figure'), ('b', 'figure'), ('c', 'figure'), ('d', 'figure'), ('A', 'point'), ('B', 'point'), ('AB', 'segment')]
A: [('a', 'Color', 'green'), ('a', 'Shape', 'triangle'), ('b', 'Color', 'blue'), ('b', 'Shape', 'square'), ('c', 'Color', 'red'), ('c', 'Shape', 'circle'), ('d', 'Color', 'green'), ('d', 'Shape', 'rectangle'), ('AB', 'Color', 'black'), ('AB', 'Length', 10)]
F covers: {'Center': [('c', 'A')], 'Radius': [('c', 5)]}
R covers: {'Rontop': [('a', 'b'), ('c', 'd')], 'Rleftof': [('a', 'c')], 'RisPortof': [('A', 'a'), ('B', 'b'), ('AB', 'AB')]}
MEMORY: {}
------------------------------
--- Abstract P-Set (after hiding 'point') ---
O: [('a', 'figure'), ('b', 'figure'), ('c', 'figure'), ('d', 'figure'), ('A', 'obj'), ('B', 'obj'), ('AB', 'segment')]
A: [('a', 'Color', 'green'), ('a', 'Shape', 'triangle'), ('b', 'Color', 'blue'), ('b', 'Shape', 'square'), ('c', 'Color', 'red'), ('c', 'Shape', 'circle'), ('d', 'Color', 'green'), ('d', 'Shape', 'rectangle'), ('AB', 'Color', 'black'), ('AB', 'Length', 10)]
F covers: {'Center': [('c', 'A')], 'Radius': [('c', 5)]} # 'A' 是被隐藏的类型，但它是 F 中的值，而不是元组中的对象
R covers: {'Rontop': [('a', 'b'), ('c', 'd')], 'Rleftof': [('a', 'c')], 'RisPortof': [('AB', 'AB')]} # ('A','a') 和 ('B','b') 因包含 'point' 类型的对象 'A'、'B' 而被移除
MEMORY (meth_hidetype): {'type_changes': [('A', 'point'), ('B', 'point')], 'attribute_value_changes': [], 'removed_f_tuples': {'Center': [], 'Radius': []}, 'removed_r_tuples': {'Rontop': [], 'Rleftof': [], 'RisPortof': [('A', 'a'), ('B', 'b')]}}
------------------------------
--- Abstract P-Set (after hiding 'figure') ---
O: [('a', 'obj'), ('b', 'obj'), ('c', 'obj'), ('d', 'obj'), ('A', 'point'), ('B', 'point'), ('AB', 'segment')]
A: [('a', 'Color', 'UN'), ('a', 'Shape', 'UN'), ('b', 'Color', 'UN'), ('b', 'Shape', 'UN'), ('c', 'Color', 'UN'), ('c', 'Shape', 'UN'), ('d', 'Color', 'UN'), ('d', 'Shape', 'UN'), ('AB', 'Color', 'black'), ('AB', 'Length', 10)] # 图形对象的属性为 UN
F covers: {'Center': [], 'Radius': []} # 包含 'c'（一个图形）的元组被移除
R covers: {'Rontop': [], 'Rleftof': [], 'RisPortof': [('A', 'a'), ('B', 'b'), ('AB', 'AB')]} # 包含 'a'、'b'、'c'、'd'（图形）的元组被移除
MEMORY (meth_hidetype): {'type_changes': [('a', 'figure'), ('b', 'figure'), ('c', 'figure'), ('d', 'figure')], 'attribute_value_changes': [('a', 'Color', 'green'), ('a', 'Shape', 'triangle'), ('b', 'Color', 'blue'), ('b', 'Shape', 'square'), ('c', 'Color', 'red'), ('c', 'Shape', 'circle'), ('d', 'Color', 'green'), ('d', 'Shape', 'rectangle')], 'removed_f_tuples': {'Center': [('c', 'A')], 'Radius': [('c', 5)]}, 'removed_r_tuples': {'Rontop': [('a', 'b'), ('c', 'd')], 'Rleftof': [('a', 'c')]}}
------------------------------
--- Abstract P-Set (after hiding 'car') ---
O: (No change) True
A: (No change) True
F covers: (No change) True
R covers: (No change) True
MEMORY (meth_hidetype): {'type_changes': [], 'attribute_value_changes': [], 'removed_f_tuples': {'Center': [], 'Radius': []}, 'removed_r_tuples': {'Rontop': [], 'Rleftof': [], 'RisPortof': []}} # 添加了记忆，但由于没有找到 'car' 类型的对象，记忆为空
------------------------------
```

看到一个方法的具体实现代码和测试，会让它的可操作性变得真切。例如，`meth_hidetype` 不只是在概念上隐藏一个类型，它会实际修改 P-Set：把对象类型重新标注为通用的 `'obj'`，把该类型特有的属性值设为 `'UN'`，并从函数/关系覆盖中移除涉及这些对象的元组。

### 具体化与记忆

至关重要的一点是，KRA 的 P-Set 结构包含一个 `MEMORY` 组件。当方法（`meth`）运行时，它会记录所执行变换的细节。这一记忆对**具体化**（concretion）至关重要——也就是从抽象视图"下钻"回底层细节的能力，让用户能够探索催生该抽象表示的基准真值。

例如，利用 `meth_hidetype` 存储的 `memory_for_this_op`：

*   如果用户正在查看抽象 P-Set（测试 2，隐藏 'figure'），看到对象 `'a'` 的类型是 `'obj'` 且属性 `('a', 'Color') = 'UN'`，就可以查阅 `'meth_hidetype'` 下的记忆：
    *   在 `'type_changes'` 中查找 `'a'`，可以发现其原始类型是 `'figure'`。
    *   在 `'attribute_value_changes'` 中查找 `('a', 'Color')`，可以发现其原始值是 `'green'`。
*   如果他们想知道抽象之前 `'a'` 参与过哪些关系，可以在 `'removed_r_tuples'` 中检查 `'a'` 出现过的关系，在原始的 `'Rontop'` 关系覆盖中找到 `('a', 'b')` 这样的元组。

这种在抽象视图与具体视图之间往返导航的能力，由结构化的 `MEMORY` 提供支撑，对于在不被淹没的前提下分析并理解复杂性非常强大。这种往返能力在实际的逆向工程场景中必不可少。

### 更多算子示例（概念性）

除隐藏之外，KRA 算子还提供多样化的变换：

*   **等价（`ωeq...`）：**想象在电子商务 P-Set 的 `latency` 属性上使用 `ωeqattrval`。不再是 `12.5`、`35.1` 等具体值，而是把范围内的值等价化（例如，`[0, 50]` 归为 'Low'，`[51, 100]` 归为 'Medium'）。`meth_eqattrval` 会修改 'A' 组件，把相关服务的具体延迟值替换为这些抽象类别。
*   **组合（`ωgroup`）：**利用电子商务 P-Set，你可以对 `ωgroup` 施加一个条件，把属于"订单履行过程"的服务和数据库归为一组（例如 'OrderService'、'InventoryService'、'OrderDB'）。`meth_group` 会在 'O' 中创建一个代表该组的*新的*抽象对象（例如 'OrderFulfillmentComponent'），并可能在 'A'、'F' 和 'R' 中创建新的聚合属性、函数和关系来概括被分组的元素。
*   **层次（`ωhier...`）：**在图形示例上，`ωhiertype` 可以把 'triangle'、'square'、'circle'、'rectangle' 归入父类型 'Polygon' 或 'Shape' 之下。`meth_hiertype` 会更新 'O' 组件以反映这一层次结构，从而可能允许在抽象的 'Shape' 类型上定义关系。

这些示例说明了不同算子如何通过作用于 P-Set 实现各种形式的抽象，每一种都服务于与查询（Q）相关的特定目的。

## 在逆向工程中的应用

把这种结构化、可操作的方法应用于涉及高度复杂性的现实问题时，它有着难以估量的价值，例如通过逆向工程进行软件系统分析 [2, 3]。逆向工程工具会产生关于类、方法、依赖、配置等的海量数据（犹如消防水管喷涌而出）。这些输出就是我们的基准 P-Set（$P_g$），可能还有其他数据结构。挑战在于让这些海量细节对特定分析任务（查询 $Q$）变得可以理解，例如理解架构依赖、识别重构候选对象或分析安全缺陷。


![IT 系统复杂性]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/IT_system_complexity.jpg)

*图：IT 系统复杂性的可视化展示。此图展示了大规模 IT 系统中常见的令人望而生畏的复杂度。图中包含数百个相互连接的组件，凸显了底层系统表示几乎会变得无法解读——这正说明了为什么需要抽象和高层建模，才能有效地管理和推理这种复杂性。（注：原图分辨率较低：https://rickrobinson.files.wordpress.com/2012/10/it-systems.jpg）*

KRA 算子为此提供了系统化的工具。我们可以使用：

*   **隐藏（`ωh...`）：**清除噪声——隐藏私有方法、特定属性值（如行号）或与当前查询无关的底层依赖。
*   **等价（`ωeq...`）：**简化词汇表——把不同的集合类型等同起来、把错误代码归入类别，或在特定分析中把具体的 DTO 实例视为单一概念。
*   **层次生成（`ωhier...`）：**构建架构视图——把类归组为组件（常与组合类的 `ωaggr` 或 `ωgroup` 配合使用）、为模块创建抽象属性（如从文件变更派生的稳定性或流失度），或定义由类级调用派生的组件间关系。
*   **组合（`ωcoll`、`ωaggr`、`ωgroup`、`ωconstr`）：**显式地把元素归组——把类聚合为服务、按所有权或迭代冲刺对文件分组，或从已有关系构造新关系（例如，由多个类级依赖派生出 'ServiceA_depends_on_ServiceB'）。
*   **逼近（`pid...`、`prepl`）：**在可以为清晰性牺牲一些精度的场合简化属性取值范围或元素类型——用类别（'major'、'minor'）替换具体版本号，或用有代表性的样本替换庞大的对象集合。

与临时拼凑的脚本相比，为此使用 KRA 这样的形式化模型具有明显的优势。它是可操作且可重复的。它鼓励捕获可复用的抽象模式。内置的记忆与具体化能力支持下钻，能在不淹没初始视图的情况下提供必要的上下文。这是一种有原则、结构化的方法，有助于确保抽象是有意义的、切合目的的，甚至为处理现实系统固有的不一致性提供了办法。

### 在 KRA 框架内利用 AI

由此我们来到在 KRA 框架内利用 AI 这一令人兴奋的前景。KRA 给了我们*做什么*（算子）和*怎么做*（把算子应用到 P-Set 上的方法）。AI 则可以搞定*选哪个*和*何时用*——决定应用哪些具体算子、带什么参数、按什么顺序，才能最好地处理用户针对复杂系统表示（$DS_g$，实际上就是我们的 $P_g$）的查询（Q）。

AI 首先可以协助把原始的逆向工程输出初步结构化为 KRA 组件（$O_g, A_g, F_g, R_g$）。更重要的是，AI 可以**学习抽象策略**。给定用户的查询（Q）和基准 P-Set（$P_g$），目标是产出一个相关的抽象 P-Set（$P_a$）。

*   **学习算子选择：**监督学习可以利用专家架构师或分析师针对类似任务成功抽象类似系统的示例来训练模型。AI 可以学习把查询的特征（Q 关键词、分析目标）和 $P_g$ 的特征（某些类型的对象数量、关系密度、属性的取值范围）映射到 KRA 算子序列及其参数。决策树、规则学习系统，甚至序列到序列模型都可能派上用场。
*   **优化抽象：**强化学习可以让 AI 智能体探索算子应用的巨大可能空间。给定一个 Q，智能体根据量化所得 $P_a$ 质量的指标（如规模缩减程度、与 Q 的相关性、结构清晰度、经由记忆的可导航性）获得奖励。智能体学习按顺序应用算子以最大化该奖励的策略。
*   **参数发现：**聚类或模式挖掘等 AI 技术可以自动发现算子的有用参数。基于属性或关系对 $P_g$ 中的对象进行聚类，可以为 `ωeqobj` 或 `ωgroup` 建议分组。分析属性值分布可以为 `ωhattrval` 或 `ωeqattrval` 的参数选择提供依据。在 F 或 R 覆盖中识别常见的图模式，可以为 `ωconstr` 提供候选。
*   **工作流自适应：**基于案例的推理可以存储并调整过去成功的抽象工作流（带参数的算子序列），以应对相似的查询或系统结构。


![IT 系统复杂性与抽象]({{ site.baseurl }}/images/ai-powered-approach-to-structural_abstraction/IT_system_complexity_with_abstraction.jpg)

*图：IT 医疗系统的抽象视图：作为一个示意，这是对一个复杂医疗 IT 生态系统的以用户为中心的抽象。通过应用 AI 驱动的抽象算子——例如把相关功能分组（`ωgroup`）、隐藏底层技术细节（`ωh...`）并突出关键用户流程——系统变得一目了然。它展示了抽象如何把令人不堪重负的技术复杂性转化为直观、可操作的地图，从而支持清晰的认知、导航与决策。（图片来源：Christina Wodtke [5] (http://architecture.31bio.org/information-architecture-concept-model/))*

这一愿景最终汇聚为一个 **AI 驱动的 KRA 工作台**——一个把逆向工程工具与 KRA 表示连接起来的集成平台。AI 组件解读用户的查询和系统结构（$P_g$），然后推荐或自动应用 KRA 算子序列。它在 $P_g$ 上执行相应的方法，产出一个简化后的 $P_a$。用户与这个抽象视图交互，平台则利用 KRA 记忆，支持按需下钻到细节。系统从用户的反馈和交互中学习，随时间不断改进其抽象推荐。

## 结语

总而言之，被复杂性淹没的感受无处不在。抽象是我们从表层包装中提炼本质、把不可理解之物变得可驾驭的最有力工具。KRA 模型 [1] 提供了一个严谨、可操作的框架，通过定义好的算子和方法对可观测数据（P-Set）执行这种抽象。通过利用 AI——借助机器学习、聚类、模式识别和智能推荐系统——我们可以基于用户查询和底层数据，把 KRA 算子的选择和应用自动化并加以优化。形式化的 KRA 结构与 AI 学习能力的这种协同，蕴藏着从根本上改变我们分析和交互大型复杂系统方式的巨大潜力——不仅限于软件逆向工程 [2, 3]，还可能适用于任何这样的领域：直接对着原始细节的"消防水管"狂饮，只会阻碍真正的理解。留给我们的问题是：我们需要从身边的复杂性中提炼出怎样的本质？

## 参考文献

1.  Saitta, Lorenza, and Jean-Daniel Zucker. *Abstraction In Artificial Intelligence And Complex Systems*（《人工智能与复杂系统中的抽象》）. Springer, May 2013. DOI: 10.1007/978-1-4614-7052-6_1
    * 本书提出了一种抽象模型——KRA（Knowledge Representation & Abstraction，知识表示与抽象）模型，但这并不是全书的核心。本书范围有限，主要有两个目的：一方面，它表明此前多种抽象理论提案有着共同的根源，可以在一个统一框架内加以处理；另一方面，它提供了一个计算环境，通过应用一组现成的、与领域无关的算子（程序）来执行抽象。
2.  Paolo Tonella, Marco Torchiano, Bart Du Bois Tarja Systä, [Empirical studies in reverse engineering: state of the art and future trends](https://www.researchgate.net/publication/220277794_Empirical_studies_in_reverse_engineering_State_of_the_art_and_future_trends)（逆向工程中的实证研究：现状与未来趋势）, Empir Software Eng, Springer, 2007. DOI 10.1007/s10664-007-9037-5.
    * 逆向工程始于让遗留系统（它们往往以陈旧的编程语言编写）现代化的目标，如今其适用范围已扩展到几乎各类软件系统。此外，最初为恢复目标系统图示化高层视图而设计的方法，也已被扩展用于解决程序员在需要理解和修改现有软件时面临的诸多其他问题。作者的观点是，这一学科的下一阶段发展必然建立在对方法的实证评估之上。事实上，这种评估既是获得关于应用某一给定方法的实际效果的知识所必需的，也是让最终用户相信其成本-收益权衡是积极的所必需的。本文对现有技术水平的贡献是一份该领域未来研究的路线图，其中包括：明确研究范围、定义参考分类法，以及采用一个执行实验的通用框架。
3.  Tonella, Paolo, and Alessandra Potrich. *Reverse Engineering of Object Oriented Code*（《面向对象代码的逆向工程》）. Vol. 1. Springer, 2005.
    * 本书描述的算法处理以下几种图的逆向工程：类图、对象与交互图、状态图、包图。（为简洁起见略去了描述，但原文中可以找到）。
4. Robert, Goldstone and Lawrence Barsalou, Reuniting perception and conception（重新统一感知与概念）, Cognition 65 (1998) 231–262 
    * 哲学和心理学领域的研究主张，在概念思维中，基于感知的相似性与更高层的规则是彼此分离的。尽管这种分离在某些时候是有道理的，但我们的目标是阐明概念处理如何植根于感知——无论对感知相似性还是对抽象规则而言。
5. Christina Wodtke, [Five Models for Making Sense of Complex Systems](https://cwodtke.medium.com/five-models-for-making-sense-of-complex-systems-134be897b6b3)（理解复杂系统的五种模型）, Medium, 11 Feb 2017
    * 介绍了五种可视化框架，帮助个人和团队驾驭并理解复杂系统。这些模型既是探索的工具，也是沟通的工具，通过视觉思维促进对复杂系统的更深入理解。
6. Ruixin Hong, Hongming Zhang, Xiaoman Pan, Dong Yu, Changshui Zhang, [Abstraction-of-Thought Makes Language Models Better Reasoners](https://arxiv.org/pdf/2406.12442)（"思考的抽象"让语言模型成为更出色的推理者）, [arXiv:2406.12442](https://arxiv.org/abs/2406.12442) [cs.CL], 18 Jun 2024.
    * 抽象推理，即从问题的抽象本质出发进行推理的能力，是人类推理中实现泛化的关键。然而，如何引导语言模型进行带有抽象的推理仍未被充分探索。本文通过引入一种名为 Abstraction-of-Thought（AoT）的新型结构化推理格式来弥合这一差距。AoT 的独特之处在于它明确要求推理过程包含不同层次的抽象。这种方法可以引导语言模型先在抽象层面思考，再纳入具体细节——而这正是流行的逐步式 Chain-of-Thought（CoT）方法所忽视的。为使模型与 AoT 格式对齐，我们提出了 AoT Collection，这是一个通过自动化、可扩展的流程收集而来的通用微调数据集，包含 348k 条带有 AoT 推理过程的高质量样本。我们用 AoT Collection 微调了多种语言模型，并在具有挑战性的 Big-Bench Hard 基准的 23 个未见任务上进行了广泛评估。实验结果表明，与 AoT 推理格式对齐的模型在许多推理任务上显著优于与 CoT 对齐的模型。
