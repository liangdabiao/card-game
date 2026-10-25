---
layout: post
title: 使用 Python 的 Dempster-Shafer 分类理论
date: 2020-08-21 00:00:00.000000000 -00:00
permalink: dempster-shafer-theory-for-classification/
author: Benny Cheung
artwork:
  author: Gijs Van Vaerenbergh
tags:
  - Dempster-Shafer Theory
  - Machine Learning
  - Classification
  - Python
category: post
comments: true
image: images/dempster-shafer-theory-for-classification/cover2x.jpg
images:
  cover: images/dempster-shafer-theory-for-classification/cover.jpg
  header: images/dempster-shafer-theory-for-classification/header.jpg
published: true
---

<!--excerpt.start-->
机器学习由 ANN（Automated Neural Network，自动神经网络）主导，它需要大规模带标注的数据集来学习一个分类模型。
当只有小数据集可用时，决策树及其变体随机森林便主导了分类任务。
在本文中，我们将探索把 Dempster-Shafer 理论作为小数据集上分类器的理论基础，其分类运作遵循组合证据片段的原则。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以把本文当作**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主播既风趣又有见地。

{{ site.baseurl }}/images/dempster-shafer-theory-for-classification/Podcast_Dempster_Shafer_Theory_for_Classification.mp3
{% include open-embed.html %}
----

![鸢尾花样本图像]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/iris_collage_images.jpg) 

*Figure. 鸢尾花（Iris）物种的图像。本文将尝试利用花瓣宽度、花瓣高度、花萼宽度、花萼长度的测量值，对 Iris Setosa、Iris Versicolour 或 Iris Virginica 这几个类别进行分类。* 

Dempster-Shafer 理论 [[DST]](#DST)[[GS76]](#76)[[GS90]](#GS90) 是一种关于证据的数学理论，
为不确定性的数学表示提供了不同于传统概率论的另一种选择。
这一框架的重大创新在于，它允许把概率质量分配给集合或区间，
而不是互斥的单个元素（singleton）。相比之下，贝叶斯推断需要一定的先验知识，且无法为“无知”分配概率。
当无法通过实验获得精确测量，或者知识来自专家引出（expert elicitation）时，D-S 理论是在工程应用中评估风险与可靠性的潜在宝贵工具。
D-S 理论的一个重要方面，是组合来自多个来源的证据并对它们之间的冲突进行建模。

选择 D-S 理论的动机可以归纳为以下几点 [[SF02]](#SF02)：

* 在各种刻画不确定性的非传统理论中，它的理论发展程度相对较高。
* Dempster-Shafer 理论与传统概率论和集合论之间的联系。
* 过去 Dempster-Shafer 理论在工程应用中已有大量应用实例。
* Dempster-Shafer 理论具有表示并组合来自多个来源的不同类型证据的通用性。

秉持“做中学”的理念，我们将采用实践方式来演示如何使用 Python 的 `pyds` Dempster-Shafer 模块。
先从 Dempster-Shafer 信度函数的一些实验开始，然后逐步深入到使用 D-S 理论对“鸢尾花植物数据集”（Iris Plant Dataset）[[IPD]](#IPD) 进行分类。最终达到了 96% 的准确率，与其他机器学习模型相当。

## Python 安装

### 虚拟环境
使用隔离的 Python 虚拟环境可以让你免受安装带来的麻烦与灾难。
`dst`（或你自选的名称）是虚拟环境的名称，`python=3.5` 是 Python 版本。

```bash
conda create -n dst python=3.5
```

按 y 继续。这会安装该 Python 版本以及所有相关的 `anaconda` 打包库，位置在 `{path_to_anaconda_location}/envs/crypto

然后通过以下命令激活虚拟环境，

```
source activate dst
```

### pyds 模块
`pyds` 是一个用于执行 Dempster-Shafer 证据理论计算的 Python 库。它在以下方面是最出色、最全面的：
* 支持归一化与未归一化的信度函数
* 多种用于组合信度函数的蒙特卡洛（Monte-Carlo）算法
* 与广义贝叶斯定理相关的各种方法
* 不确定性度量
* 从数据构造信度函数的方法

确保你处于虚拟环境（virtualenv）中，

```
pip install pyds
```

或者从下载的源码安装也同样简单：<https://github.com/reineking/pyds>

```
cd pyds; python setup.py install
```

### 其他模块
运行我们的实验还需要以下模块，

```
pip install numpy
pip install pandas
pip install matplotlib
pip install seaborn
```

这些模块很重要，但由于本文聚焦于 D-S 理论，这里不做详细介绍。我们只把它们当作处理和展示的工具来使用。

## Dempster-Shafer 证据理论
当专家对某些证据做出观察时，他可能会说，
> 专家：“我可以相当肯定，证据要么是 $$a$$，要么是 $$b$$。很可能是 $$a$$，但也可能是 $$b$$。不过我也可能弄错。”

关于“很可能是 $$a$$ 或 $$b$$”的陈述可以转化为
$$A = \{a, b\}$$，即不对 $$a$$ 是否比 $$b$$ 更可能做出任何断言。
相反，这样的赋值表达了对该问题的完全无知。
在 D-S 信度函数框架内，有几种等价的表示可用于量化信度。最重要的四种表示是 [[RT14]](#RT14)，

![信度函数表示]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/DS_Belief_Functions.png)

*Figure. 不同信度函数表示的示意图。本例中辨识框架 $$\Theta$$ 由三个元素 $$a, b, c$$ 组成。三角形包含 $$\Theta$$ 中除 $$\emptyset$$ 之外的所有子集。图中标示的集合对应于与集合 $$\{a,b\}$$ 相关联的不同信度表示 $$m, bel, pl,$$ 和 $$q$$ 的质量。*

假设辨识框架为 $$\Theta$$，$$\Theta$$ 的所有可能子集构成 U（$$\Theta$$ 的幂集）。

* **质量函数（mass function）**，记作 `m`：
质量函数（也称为基本信度分配或基本概率分配）
在许多方面是最基本的信度表示，所有其他表示都可以很容易地从质量函数导出。形式上，
质量函数 $$m$$ 是一个映射 $$m : P(U) \rightarrow [0,1]$$，为辨识框架 $$\Theta$$ 中属于 $$U$$ 的每个
假设 $$A$$ 分配一个质量值。
值 $$m(A)$$ 是严格分配给假设 $$A$$ 的信度数量。

* **信度函数（belief function）**，记作 `bel`：
分配给 $$U$$ 的子集假设 $$A$$ 的信度总量（包括 $$A$$ 的所有
子集 $$B$$）记作 $$bel(A)$$。函数 $$bel : P(U) \rightarrow [0,1]$$ 称为
信度函数，它可以直接由质量函数 $$m$$ 计算得到。
信度函数 $$bel$$ 有时被解释为为未知概率函数 $$P$$ 定义了一个“下界”。

* **似真度函数（plausibility function）**，记作 `pl`：
似真度 $$pl(A)$$ 是未被严格分配给补集 $$\lnot A$$ 的信度数量。因此，它表达了假设 $$A$$ 有多可信，即
有多少信度质量潜在地支持 $$A$$。
前面提到 $$bel$$ 可以被视为未知概率
函数 $$P$$ 的下界。在上下概率的解释下，似真度 $$pl$$
可以被视为“上界”。

* **公共度函数（commonality function）**，记作 `q`：
公共度 $$q(A)$$ 表示总共分配给 $$A$$ 以及所有以 $$A$$ 为子集的超集 $$A^s$$ 的质量有多少。
因此，公共度 $$q(A)$$ 表达了有多少质量潜在地支持整个集合。

让我们用 pyds 的 `MassFunction` 来测试各种信度函数，

```python
from pyds import MassFunction, powerset

m1 = MassFunction({'a':0.4, 'b':0.2, 'ab':0.1, 'abc':0.3})
m2 = MassFunction({'b':0.5, 'c':0.2, 'ac':0.3, 'a':0.0})
print("m1:",m1)
print("m1: bpa of {'a','b'}=", m1['ab'])
print("m1: belief of {'a','b'}=", m1.bel('ab'))
print("m1: plausibility of {'a','b'}=", m1.pl('ab'))
print("m1: commonality of {'a','b'}=", m1.q('ab'))
print("m2:",m2)
print("m2: bpa of {'b'}=", m2['b'])
print("m2: belief of {'b'}=", m2.bel('b'))
print("m2: plausibility of {'b'}=", m2.pl('b'))
print("m2: commonality of {'b'}=", m2.q('b'))
```

结果如下，
```
m1: { {'a'}:0.4; {'c', 'a', 'b'}:0.3; {'b'}:0.2; {'a', 'b'}:0.1 }
m1: bpa of {'a','b'}= 0.1
m1: belief of {'a','b'}= 0.7000000000000001
m1: plausibility of {'a','b'}= 1.0
m1: commonality of {'a','b'}= 0.4
m2: { {'b'}:0.5; {'c', 'a'}:0.3; {'c'}:0.2; {'a'}:0.0 }
m2: bpa of {'b'}= 0.5
m2: belief of {'b'}= 0.5
m2: plausibility of {'b'}= 0.5
m2: commonality of {'b'}= 0.5
```

你可以试着检验自己对所有信度函数的理解，与这些计算值进行核对。

此外，`MassFunction` 还可以用许多不同的方式构造，例如，

```python
m1_1 = MassFunction([(('a',), 0.4), (('b',), 0.2), (('a', 'b'), 0.1), (('a', 'b', 'c'), 0.3)])
if (m1_1==m1): print("m1_1 Equal to m1")
m1_2 = MassFunction([('a', 0.4), ('b', 0.2), ('ab', 0.1), ('abc', 0.3)])
if (m1_2==m1): print("m1_2 Equal to m1")
```

我们可以通过以下方式获取 `MassFunction` 的辨识框架和幂集，

```python
print("frame of discernment", m1.frame())
print("powerset of frame", list(m1.all()))
```

结果如下，
```
frame of discernment frozenset({'c', 'a', 'b'})
powerset of frame [frozenset(), frozenset({'c'}), frozenset({'a'}), frozenset({'b'}), frozenset({'c', 'a'}), frozenset({'c', 'b'}), frozenset({'a', 'b'}), frozenset({'c', 'a', 'b'})]
```

可以看到 pyds 的 `MassFunction` 使用了 `frozenset`，“冻结集合（frozen set）”只是 Python set 对象的不可变版本。集合的元素可以随时修改，而冻结集合的元素在创建之后保持不变。正因如此，冻结集合可以用作字典（Dictionary）的键，或者作为另一个集合的元素。但与集合一样，它是无序的（元素可以被放到任意索引位置）。

## Dempster 组合规则
为了解决推断问题，代表不同证据片段的信度函数需要以有意义的方式组合起来。这正是组合规则成为
D-S 证据理论重要基石的原因。通常，每条证据由一个单独的信度函数表示。
然后用组合规则逐步融合所有这些信度函数，从而获得
一个代表全部可用证据的信度函数。

具体而言，组合结果（称为*联合质量*）由两组质量 $$m_1$$ 和 $$m_2$$ 按如下方式计算：

* $$m_{1,2}(\emptyset) = 0$$

* $$m_{1,2}(A) = (m_1 \oplus m_2) (A) = \frac 1 {1 - K} \sum_{B \cap C = A \ne \emptyset} m_1(B) m_2(C)$$

其中

* $$K = \sum_{B \cap C = \emptyset} m_1(B) m_2(C)$$

$$K$$ 衡量的是两组质量之间冲突的大小。

尽管我们使用的是 pyds，它的 `MassFunction` 已内置组合规则，
但阅读下面的 `DempsterRule` 脚本仍然值得，它仅用 10 行代码就把组合规则清楚地呈现了出来。

```python
from numpy import *

# 不要直接使用，仅用于演示 D-S 组合规则的实现
def DempsterRule(m1, m2):
    ## 提取辨识框架      
    sets=set(m1.keys()).union(set(m2.keys()))
    result=dict.fromkeys(sets,0)
    ## 组合过程
    for i in m1.keys():
        for j in m2.keys():
            if set(str(i)).intersection(set(str(j))) == set(str(i)):
                result[i]+=m1[i]*m2[j]
            elif set(str(i)).intersection(set(str(j))) == set(str(j)):
                result[j]+=m1[i]*m2[j]
                 
    ## 对结果进行归一化
    f= sum(list(result.values()))
    for i in result.keys():
        result[i] /=f
    return result

m1 = {'a':0.4, 'b':0.2, 'ab':0.1, 'abc':0.3}
m2 = {'b':0.5, 'c':0.2, 'ac':0.3, 'a':0.0}
print(DempsterRule(m1, m2))}
```

组合的结果是，
```
{'ac': 0.15789473684210523, 'c': 0.10526315789473682, 'b': 0.5263157894736842, 'ab': 0.0, 'abc': 0.0, 'a': 0.21052631578947364}
```

现在，回到使用 pyds 的 D-S 组合规则 `combine_conjunctive()`（为了方便，它也被定义为 AND（&）运算符），

```python
print("Combine m1 and m2:", m1 & m2)
print("Combine m1 and m2 using sampling:", m1.combine_conjunctive(m2, sample_count=10000))
```

组合得到的 `MassFunction` 结果会自动归一化，使总和为 1。

```
Combine m1 and m2: { {'b'}:0.5; {'a'}:0.24999999999999994; {'c', 'a'}:0.14999999999999997; {'c'}:0.09999999999999998 }
Combine m1 and m2 using sampling: { {'b'}:0.5015825420622867; {'a'}:0.2518740629685215; {'c', 'a'}:0.14842578710645304; {'c'}:0.09811760786273886 }
```
## Dempster-Shafer 分类器
有了 D-S 理论知识以及 pyds `MassFunction` 的使用经验，
下面几节将介绍如何使用 Dempster-Shafer 理论对“鸢尾花植物数据集”（Iris Plant Dataset）[[IPD]](#IPD) 进行分类。

### 鸢尾花植物数据集（Iris Plant Dataset）
鸢尾花植物数据集（Iris Plant Dataset）[[IPD](#IPD)] 是 UCI 数据集中的另一个标准基准问题，是分类测试的热门选择。该数据集包含 150 个数据项，具有以下四个数值属性：花萼长度（Sepal Length）、花萼宽度（Sepal Width）、花瓣长度（Petal Length）和花瓣宽度（Petal Width）（单位均为厘米）。类别即植物类型，分别是 Iris Setosa、Iris Versicolour 和 Iris Virginica，每个类别包含 50 个样本。

![花瓣与花萼]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/iris_petal_sepal.jpg)

IPD 的辨识框架 $$\Theta$$ 为 {Setosa, Versicolour, Virginica}，
而所选定的初始分类步骤允许七个可能的假设（即 $$\Theta - \emptyset$$ 的幂集）：

* {Setosa} = 类别 1
* {Versicolour} = 类别 2
* {Virginica} = 类别 3
* {Setosa, Versicolour}
* {Setosa, Virginica}
* {Versicolour, Virginica}
* {Setosa, Versicolour, Virginica}

接着导入常用的分析模块，对 IPD 进行一些基本的数据分析，

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd
from pandas import DataFrame, read_csv
import matplotlib
matplotlib.use('Agg')
matplotlib.style.use('ggplot')
```

将鸢尾花植物数据集 `data/iris.csv` 加载到 pandas DataFrame `df` 中

```
data_file = r'data/iris.csv'
df = pd.read_csv(data_file).dropna()
print('data shape:', df.shape)
df.sample(10)
```

从 IPD DataFrame 中随机选取 10 个样本，以对数据有一个直观感受；注意 `Class` 列就是期望分类出的鸢尾花类别。

![IPD 数据框样本]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/IPD_Data_Frame_samples.jpg)

对于每个属性，我们将绘制各个类别的数据分布图以便进一步观察。

```python
f, axes = plt.subplots(2, 2, figsize=(12, 10), sharex=True)
for fd, ax in [('SepalLength',axes[0,0]), ('SepalWidth',axes[0,1]), ('PetalLength',axes[1,0]), ('PetalWidth',axes[1,1])]:
    for c in df.Class.unique():
        sns.distplot(df[df.Class == c][fd], ax=ax, hist=False, label=c, axlabel=fd)
```

![IPD 数据分布]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/iris_distribution_plots.jpg)

从分布图中可以得出结论：利用 PetalLength 和/或 PetalWidth 属性就能轻松地将 Setosa 类别分离出来。然而，可以预期 Versicolor 和 Virginica 类别将给分类器带来最大的挑战。

### 将 Dempster-Shafer 用作分类器
按照 [[QAUC14]](#QAUC14) *Data Classification Using the Dempster-Shafer Method*（使用 Dempster-Shafer 方法进行数据分类）所概述的策略，分类可以表示为一系列步骤：

1. **定义辨识框架（$$\Theta$$）**：这是与给定数据集相关的所有可能假设的集合，并确定数据必须被指派到的类别。
2. **确定类别归属**：确定哪些数据属性对于建立类别归属是重要的，并舍弃其他属性。一般而言，辨识框架和所选属性（它们的数量和数据类型）将为设计质量函数和 DRC 组合结构提供粗略的指导。
3. **指派质量函数**：在数据的一个子集内考察所选属性及其数据值，以便为每个属性设计质量函数。这些函数将用于根据测试数据的属性值为相应的假设指派质量值。
4. **设计 DRC 策略**：基于数据结构设计 DRC 组合策略。一次 DRC 应用会对每个数据项组合各属性的质量值，但可以进行多次应用，DRC 还可用于组合此前应用的结果。
5. **分类数据**：组合完成后，选择一个将结果转化为决策的规则。可以在不同步骤中使用多个规则，但最终的规则决定数据的分类。

[[QAUC14]](#QAUC14) 并未提供任何实现。我们将沿着他们的研究方向用 Python 实现一个方案来进行测试。

### 1. 定义辨识框架
IPD 的辨识框架（$$\Theta$$）来自 DataFrame `df` 中所有不重复的 `Class` 值。

```python
pset = dict.fromkeys(powerset(df.Class.unique()), 0)
print(pset)
```

下面是辨识框架的幂集，其中 frozenset() 是空集。
```
{frozenset(): 0,
 frozenset({'Virginica'}): 0,
 frozenset({'Setosa', 'Virginica'}): 0,
 frozenset({'Versicolor'}): 0,
 frozenset({'Versicolor', 'Virginica'}): 0,
 frozenset({'Setosa'}): 0,
 frozenset({'Setosa', 'Versicolor'}): 0,
 frozenset({'Setosa', 'Versicolor', 'Virginica'}): 0}
```

### 2. 确定类别归属
由于 IPD 的属性都是数值数据，确定类别归属最简单的方法就是利用属性的最小值和最大值。稍后，我们将使用这些属性数据范围来指派类别归属。

```python
classRange = {}
for c in df.Class.unique():
    fieldRange = {}
    for f in fields:
        fieldRange[f] =(df[df.Class == c][f].min(), df[df.Class == c][f].max())
    classRange[c] = fieldRange
print(classRange)
```

对于每个属性，我们得到了各类别的 [min, max] 数据范围，

```
{
  'Setosa': {
    'PetalLength': (1.0, 1.9),
    'PetalWidth': (0.1, 0.6),
    'SepalLength': (4.3, 5.8),
    'SepalWidth': (2.3, 4.4)},
 'Versicolor': {
    'PetalLength': (3.0, 5.1),
    'PetalWidth': (1.0, 1.8),
    'SepalLength': (4.9, 7.0),
    'SepalWidth': (2.0, 3.4)},
 'Virginica': {
    'PetalLength': (4.5, 6.9),
    'PetalWidth': (1.4, 2.5),
    'SepalLength': (4.9, 7.9),
    'SepalWidth': (2.2, 3.8)}
}
```

### 3. 指派质量函数
这一步提供了一种粗略的数据分组方法。如图所示，某些数据项可能无法被归入单一类别。例如，如果一个属性值小于 min(类别 2) 且大于等于 min(类别 1)，那么该数据项必定属于类别 1。但如果该值小于 min(类别 3) 且大于等于 min(类别 2)，那么该数据项可能属于类别 1 或类别 2。对每个属性都重复这一过程，并按以下方式指派质量值。对于只被指派到一个类别的数据项：

![为每个类别属性指派质量函数]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/DS_Classification_three_class_overlap_for_1_attribute.png)
*图片来源：[[QAUC14]](#QAUC14)*

* $$m(\text{class x}) = 0.9, m(\Theta) = 0.1,$$

即不确定性 m($$\Theta$$) 固定为 0.1。对于被指派到两个可能类别 x 和 y 的数据项：

* $$m(\text{class x} \cup \text{class y}) = 0.9, m(Θ) = 0.1$$

对于被指派为可能属于全部三个类别的数据项：

* $$m(\Theta) = 1$$

即不确定性最大的情形，$$m(\Theta)$$ 为 1，对具体假设毫无所知。

```python
def hypothesis(df, classRange, fieldName, value):
    hset = set()
    for c in df.Class.unique():
        if (classRange[c][fieldName][0] <= value and value < classRange[c][fieldName][1]):
            hset.add(c)
    return frozenset(hset)

def hypothesisCounts(hypothesisCount, h, singletons, frame):
    if h in singletons:
        hypothesisCount[h] += 0.9
        hypothesisCount[frame] += 0.1
    elif h == frame:
        hypothesisCount[frame] += 1.0
    else:
        hypothesisCount[h] += 0.9
        hypothesisCount[frame] += 0.1

hypothesisCount = dict.fromkeys(powerset(df.Class.unique()), 0)
singletons = MassFunction(hypothesisCount).singletons()
frame = frozenset(df.Class.unique())

m = {}
for f in fields:
    hypothesisCount = dict.fromkeys(powerset(df.Class.unique()), 0)
    for index, row in df.iterrows():
        h = hypothesis(df, classRange, f, row[f])
        hypothesisCounts(hypothesisCount, h, singletons, frame)
    m[f] = MassFunction(hypothesisCount).normalize()
    print(f, 'MassFunction =', m[f])
```

得到的 `MassFunction` 结果如下，

```
SepalLength MassFunction = {
  {'Versicolor', 'Setosa', 'Virginica'}:0.4446680080482897; {'Versicolor', 'Virginica'}:0.3863179074446678; {'Setosa'}:0.0965794768611671; {'Virginica'}:0.07243460764587531; {'Versicolor'}:0.0; {'Setosa', 'Virginica'}:0.0; {'Versicolor', 'Setosa'}:0.0
  }
SepalWidth MassFunction = {
  {'Versicolor', 'Setosa', 'Virginica'}:0.7645875251509054; {'Setosa', 'Virginica'}:0.1448692152917505; {'Setosa'}:0.06639839034205235; {'Versicolor', 'Virginica'}:0.018108651911468817; {'Versicolor'}:0.006036217303822939; {'Virginica'}:0.0; {'Versicolor', 'Setosa'}:0.0
  }
PetalLength MassFunction = {
  {'Setosa'}:0.29327902240325865; {'Virginica'}:0.25050916496945014; {'Versicolor', 'Virginica'}:0.1771894093686355; {'Versicolor'}:0.1771894093686355; {'Versicolor', 'Setosa', 'Virginica'}:0.10183299389002021; {'Setosa', 'Virginica'}:0.0; {'Versicolor', 'Setosa'}:0.0
  }
PetalWidth MassFunction = {
  {'Setosa'}:0.3012295081967213; {'Virginica'}:0.264344262295082; {'Versicolor'}:0.1721311475409837; {'Versicolor', 'Virginica'}:0.15983606557377059; {'Versicolor', 'Setosa', 'Virginica'}:0.10245901639344246; {'Setosa', 'Virginica'}:0.0; {'Versicolor', 'Setosa'}:0.0
  }
```

### 4. 设计 DRC 策略
系统使用 DRC 组合来自全部四个属性的质量值，从而为所有假设产生总体质量值，并使用信度值最高的假设来对数据项进行分类。如果该假设并不代表单一类别，则需要进行第二步。

在第二步中，针对每个属性，分别对所涉及的各个类别以及这些类别的并集计算标准差（$$sd$$），然后据此计算特征选择值（$$FSV$$）：

* $$FSV = \frac{sd(class_1)×sd(class_2)×⋅⋅⋅×sd(class_n)}{sd(class_1 \cup class_2 \cup ⋅⋅⋅ \cup class_n)}$$

其中 n 是表示类别数的自然数。选择具有最小 $$FSV$$ 的属性 a，并对每个类别计算数据项的 $$a$$ 值与平均 $$a$$ 值之间的绝对差 $$d$$（距离），

* $$d_i = |a_i − mean(a_i)|, i = 1,...,n$$

数据项被分类为具有最小 $$d$$ 值的类别 $$x$$。

```python
import functools

def min_fsv_attr(sdf, fields):
    fsv = {}
    for f in fields:
        l = list(sdf.loc[:, f]['std'])
        fsv[f] = functools.reduce(lambda a,b: a*b, l) / df[f].std()
    return min(fsv.keys(), key=(lambda k: fsv[k]))

def min_dist_attr(sdf, fsv_attr, candidates, sample):
    dist = {}
    for c in candidates:
        dist[c] = abs(sample[fsv_attr] - sdf.loc[c, fsv_attr]['mean'])
    return min(dist.keys(), key=(lambda k: dist[k]))

sdf = df.groupby('Class').describe()
fsv_attr = min_fsv_attr(sdf, fields) 
print(fsv_attr)
```

调用 `min_fsv_attr` 之后，我们知道具有最小 $$FSV$$ 的最佳属性是 `PetalWidth`。

### 5. 分类数据
完整的分类流程如流程图所示，

![使用 Dempster-Shafer 理论进行 IPD 分类的策略]({{ site.baseurl }}/images/dempster-shafer-theory-for-classification/DS_Classification_Iris_Plant_Data_process.png)
*图片来源：[[QAUC14]](#QAUC14)*

把前面的代码片段与下面的代码结合起来，D-S 分类器的实现就完成了。

```python
def predict(df, combine):
    best = 0.0
    bestClass = frozenset({})
    for c in combine.all():
        if (combine[c] > best):
            best = combine[c]
            bestClass = c
    return (bestClass, best)

countFail = 0
countPredict = 0
ms = {}
for index, row in df.iterrows():
    sample = row
    for f in fields:
        hypothesisCount = dict.fromkeys(powerset(df.Class.unique()), 0)
        h = hypothesis(df, classRange, f, sample[f])
        hypothesisCounts(hypothesisCount, h, singletons, frame)   
        ms[f] = MassFunction(hypothesisCount).normalize()
        
    # 相当于执行 - ms['SepalLength'] & ms['SepalWidth'] & ms['PetalLength'] & ms['PetalWidth']
    combine_mf = functools.reduce(lambda a,b: a & b, ms.values())

    (bestClass, best) = predict(df, combine_mf)
    if (bestClass in singletons) and (best > 0.8):
        # 如果是单元素类别且置信度大于 80%
        countPredict += 1
        if bestClass != frozenset({sample.Class}):
            # print('failed predict:', set(bestClass), best, 'expect:', set({sample.Class}))
            countFail +=1
    else:
        # 不是单元素类别，用第二步策略查找
        # print(set(bestClass), best)
        fsvClass = min_dist_attr(sdf, fsv_attr, list(bestClass), sample)
        if (best > 0.8):
            countPredict += 1
            if fsvClass != sample.Class:
                # print('FSV failed predict:', set({fsvClass}), best, 'expect:', set({sample.Class}))
                countFail +=1
    
print('total rows', df.Class.count())
print('total predicted', countPredict)
print('total fail predicted', countFail)
print('accuracy rate', (countPredict - countFail)/(countPredict*1.0))
```

报告的预测准确率是 96%！

```
total rows 150
total predicted 150
total fail predicted 6
accuracy rate 0.96
```

结果表明，在对具有多个类别和较少属性的数据项进行分类的能力上，这个特定的 D-S 分类器与其他领先方法相当。

## 结语
基于 Dempster-Shafer 的分类器在设计上具有灵活性，可以针对任何给定问题进行构建。
特别地，我们可以决定质量函数应如何构造，以及数据组合应如何组织。
人们可以按需构建或简单或复杂的质量函数，并且可以采用不同的策略多次应用 Dempster 组合规则。
这是一个显著的优势，因为它允许创建针对特定问题领域定制的系统；但也可以说这是一个缺点，因为它无法把结果推广到所有领域。

D-S 的理论与实践优势已在许多 D-S 应用中被反复提及。D-S 不需要先验知识（数据标签），因此适合无监督分类——有可能对从未见过的新信息进行分类，例如，目标是检测计算机系统中从未出现过的网络攻击。此外，还可以对“无知”值进行建模，从而提供关于观察结果不确定性的信息。

## 参考文献
### 视频教程
* Richard Bowles，*Dempster-Shafer Theory*（Dempster-Shafer 理论），视频：<https://www.youtube.com/watch?v=51ssBAp_i5Y>
    * 这是互联网上对 D-S 理论最好的讲解，配有说明性示例和足以帮助理解的数学内容。
    * 幻灯片下载：<http://www.richardbowles.co.uk/ai_with_js/code11/#slides>

### 技术参考文献
* <a name="DST">[[DST]](https://en.wikipedia.org/wiki/Dempster%E2%80%93Shafer_theory)</a> Dampster-Shafer Theory Wikipedia（Dempster-Shafer 理论的维基百科条目）
* <a name="GS76">[[GS76]]()</a> Glenn Shafter, *A Mathematical Theory of Evidence*（《证据的数学理论》）, 1976, Princeton University Press, Princeton, N.J
* <a name="IPD">[[IPD]](https://archive.ics.uci.edu/ml/datasets/iris)</a> Iris Plant Dataset（鸢尾花植物数据集）, UCI Machine Learning Repository
* <a name="GS90">[[GS90]]()</a> Jean Gordon and Edward H. Shortliffe, *Chapter 13: The Dempster-Shafer theory of evidence*（《第 13 章：Dempster-Shafer 证据理论》）, Rule-Based Expert Systems: The MYCIN Experiments of the Stanford Heuristic Programming Project, June 1990, isbn:9780201101720
* <a name="PYDS">[[PYDS]](https://github.com/reineking/pyds)</a> Python Dempster-Shafer `pyds` module（Python Dempster-Shafer `pyds` 模块）
* <a name="QAUC14">[[QAUC14]](https://arxiv.org/pdf/1409.0763.pdf)</a> Qi Chena, Amanda Whitbrooka, Uwe Aickelina, Chris Roadknightab, *Data Classification Using the Dempster-Shafer Method*（《使用 Dempster-Shafer 方法进行数据分类》）, (2014) Data classification using the Dempster-Shafer method. J Exper Theor Artif Intell 26:493–517.
* <a name="SF02">[[SF02]](https://prod.sandia.gov/techlib-noauth/access-control.cgi/2002/020835.pdf)</a> Karl Sentz and Scott Ferson, *Combination of Evidence in Dempster-Shafer Theory*（《Dempster-Shafer 理论中的证据组合》）, Sandia National Laboratories Report, April 2002
* <a name="RT14">[[RT14]](https://d-nb.info/1072158280/34)</a> Thomas Reineking, *Belief Functions: Theory and Algorithms*（《信度函数：理论与算法》）, Thesis, Feb 2014.
