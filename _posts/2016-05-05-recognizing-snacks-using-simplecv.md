---
layout: post
title: 用 SimpleCV 识别零食
date: 2016-05-05 12:00:00.000000000 -05:00
permalink: recognizing-snacks-using-simplecv/
author: Benny Cheung
tags:
  - SnackWatcher
  - SnackClassifier
  - Python
  - SimpleCV
  - Computer Vision
  - Machine Learning
category: post
comments: true
image: images/recognizing-snacks-using-simplecv/cover2x.jpg
images:
  cover: images/recognizing-snacks-using-simplecv/cover.jpg
  header: images/recognizing-snacks-using-simplecv/header.jpg
published: true
series: "Food Detection"
series_order: 2
---

<!--excerpt.start-->
本文旨在提供如何用 Python 和 SimpleCV 识别零食的基础知识。读者将通过
[Snack Classifier](https://github.com/jonahgroup/SnackClassifier) 开源项目所附
Python 脚本的动手实验，获得实用的编程知识。

以一个零食识别应用为例，[Snack Watcher](https://github.com/jonahgroup/SnackWatcher)
监视零食桌上出现的任何零食。
为了判断是否发生了有趣的事件，Snack Watcher 需要把图像处理为一组图像"斑点"（Blob）。对每个"Blob"，Snack Watcher
把"Blob"与其先前状态进行比较，以判定该"Blob"是新增、移除还是静止。
<!--excerpt.end-->

更有趣的是，Snack Watcher 可以配置为识别某个"Blob"看起来是否像某种特定的零食（例如饼干或糖果）。
在下图 Snack Watcher 捕获的图像中，可以看到零食已被分类，
结果都标注了各自的类别，并向消息频道报告"NEW SNACKS DETECTED!"（检测到新零食！）事件。

![Snack Watcher 零食分类与标注示例](images/recognizing-snacks-using-simplecv/Snack_Watcher_marvin_slack_bot.png)

## 视觉与分类的魔法
Python 计算机视觉框架 [SimpleCV](http://simplecv.org/) 是 OpenCV 的封装，
它让图像处理、检测和展示变得简单，我们无需投入巨大精力就能获得有趣的结果。
在机器学习和数据分类方面，SimpleCV 构建在 [Orange](http://orange.biolab.si/) 框架之上。
Orange 实现了丰富的图像特征提取算子和
机器学习算法，以支持我们的目标——零食分类。
关于为树莓派项目安装 OpenCV、SimpleCV 和 Orange，请参考这篇博文
[用树莓派 3 做计算机视觉]({{ site.baseurl }}/raspberry-pi-3-for-computer-vision)
获取说明。

学习零食分类的最好方式就是亲手一试。

[Snack Classifier](https://github.com/jonahgroup/SnackClassifier) 是一个
零食分类的开源项目。它附带两个 Python 脚本，帮助你实验这套系统。通过脚本实验确认概念之后，
Snack Classifier 可以作为一个*微服务*（Microservice）来支持你的零食分类任务；
从而扩展"有效监视零食"这一总体目标！

### 用于监督式机器学习的 Python 脚本
本文描述的 Python 脚本可以用作*监督式机器学习*工具。
典型的监督式机器学习要经历两个阶段（见下图）。
第一阶段 `(a) 训练`（Training）中，
向计算机呈现示例输入及其期望输出 `label`（又称类别，如 cookies），
经过一组特征提取器把输入概括为特征向量。
在 SimpleCV 机器学习算法的支持下，
系统生成一个能够有效分类训练输入的分类器模型。

第二阶段 `(b) 预测`（Prediction），可以接收未知的输入样本，
经过同一组特征提取器把未知样本概括为
特征向量。借助训练得到的分类器模型，未知
输入可以被分类为结果 `label`（例如 cookies）。

![监督式机器学习]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/supervised_classification.png)

#### (a) 零食训练（Python 脚本）
该脚本的灵感来自这篇文章，
[A fruit image classifier with Python and SimpleCV](http://jmgomez.me/a-fruit-image-classifier-with-python-and-simplecv/)。
我们增强了文章的代码片段，做成易于使用的
零食训练与分类脚本。

[snack-trainer.py](https://github.com/jonahgroup/SnackClassifier/blob/master/scripts/snack-trainer.py)

```bash
Usage: snack-trainer.py [options] -c <classes> -m <method>

Options:
  -h, --help            show this help message and exit
  -g, --debug           debugging mode
  -c CLASSES, --class=CLASSES
                        detect classes, comma seperated
  -a TRAIN_PATH, --train=TRAIN_PATH
                        training samples path
  -t TEST_PATH, --test=TEST_PATH
                        testing samples path
  -r RESULT_PATH, --result=RESULT_PATH
                        testing results path
  -s CLASSIFIER, --classifier=CLASSIFIER
                        using classifier (svm|tree|bayes|knn)
  -f FEATURE_PATH, --feature=FEATURE_PATH
                        save training features into file
  -e CLASSIFIER_FILE, --save=CLASSIFIER_FILE
                        save classifier into file
```

包含全部训练图像的默认路径是 `train`，
包含全部测试图像的默认路径是 `test`，
结果写入的默认路径是 `result`。当然，你可以
用相应的命令行选项更改这些目录。

所有图像类别必须分门别类地放入各自的目录。
类别标签必须与目录一一对应。
例如，如果我们有 2 个类别 `cookie` 和 `other`，目录布局应该像这样，

```
  train
  |____cookie
  | |____cookie_001.png
  | |____cookie_002.png
  | |____ ...
  |____other
  | |____other_001.png
  | |____other_002.png
  | |____ ...

  test
  |____cookie
  | |____test_001.png
  | |____test_002.png
  | |____ ...
  |____other
  | |____test_003.png
  | |____test_004.png
  | |____ ...
```

设置好全部图像目录之后，我们可以这样执行 `snack-trainer.py`，

```bash
python snack-trainer.py -g -c "cookie,other" -s tree
```

在这个例子里，我们分类 `cookie` 和 `other`。我们指定使用 `tree` 分类器来分类 `test` 图像。带标注的结果图像会保存到 `result`。

此外，训练好的分类器参数会被保存，默认以分类器名命名，例如 `tree.dat`。
训练完成后，我们可以用这个分类器文件执行
`snack-classify.py` 做进一步的图像分类。

#### (b) 零食预测（Python 脚本）
如果我们使用 `tree` 分类器做训练，训练好的分类器默认保存为 `tree.dat`（注意：如有需要，`snack-trainer.py` 提供了控制分类器保存文件名的选项）。
随后，我们可以用 `tree.dat` 重建分类器，对其他图像分类。`snack-classfy.py` 就是执行进一步图像分类的工具程序。

[snack-classify.py](https://github.com/jonahgroup/SnackClassifier/blob/master/scripts/snack-classify.py)

```bash
Usage: snack-classify.py [options] -c <classifier_file> -i <image>

Options:
  -h, --help            show this help message and exit
  -g, --debug           debugging mode
  -c CLASSIFIER_FILE, --classifier=CLASSIFIER_FILE
                        load classifier from file
  -i IMAGE_FILE, --image=IMAGE_FILE
                        classify this image file
```

如果我们想对 `something.png` 图像分类，可以执行，

```bash
python snack-classify.py -c tree.dat -i something.png
```

它会返回给定图像的分类名称。

## SimpleCV 软件包
为了深化图像训练与分类的实践知识，
本节旨在展示一系列 Python 代码片段。
这里并不打算给出 SimpleCV 机器学习软件包的详尽教程，
完整细节请参阅 [SimpleCV Documentation](http://simplecv.sourceforge.net/doc/)。

本节展示如何在训练与分类零食的语境中使用 SimpleCV 机器学习软件包。它演示了 SimpleCV 方法的实际用法；
我们可以了解 Snack Classifier 的*微服务*是如何构建的。


对于代码片段，我们使用这张样例
[Cookie 图像]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/cookie_001.png)，
下载它并跟着下面的代码片段操作。

![Cookie 图像]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/cookie_001.png),

### SimpleCV 特征提取
本节我们试用 [SimpleCV Feature Package](http://simplecv.sourceforge.net/doc/SimpleCV.Features.html)。

#### 边缘特征提取器
这个提取器方法接收一张图像，施加边缘检测器，并计算图像中线条的长度和方向。
线条的值累积到 bin 中，形成 1D 边缘长度直方图和 1D 边缘角度直方图。

![对测试 cookie 图像施加边缘提取器]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/cookie_001_edge.png)

例如，我们想从 `cookie_001.png` 图像提取边缘直方图特征。特征被分入例如 5 个 bin。然后，我们打印特征字段名及其值。

```python
edge = EdgeHistogramFeatureExtractor(5)
img = Image('cookie_001.png')

edge_vecs = edge.extract(img)
edge_fields = edge.getFieldNames()
for i in range(0, edge.getNumFields()):
    print edge_fields[i], '=', edge_vecs[i]
```

结果类似，

```
Length0 = 1.80579216354
Length1 = 1.66950596252
Length2 = 0.834752981261
Length3 = 0.45996592845
Length4 = 0.229982964225
Angle0 = 0.0
Angle1 = 0.00384859186815
Angle2 = 0.00635017658245
Angle3 = 0.00369012043829
Angle4 = 0.0
```

审视边缘提取的统计结果，可以发现结果很符合直觉：小线段数量远超长线段。边缘的角度值呈正态分布。

#### 色调（Hue）特征提取器
这个特征提取器接收一张图像，获取色调通道，按特定色调值统计像素数量并分入 bin。
色调值累积到 bin 中，形成 1D 色调直方图。

![对测试 cookie 图像施加色调提取器]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/cookie_001_hue.png)

例如，我们想从 `cookie_001.png` 图像提取色调直方图特征。特征被分入例如 5 个 bin。然后，我们打印特征字段名及其值。

```python
hue = HueHistogramFeatureExtractor(5)
img = Image('cookie_001.png')

hue_vecs = hue.extract(img)
hue_fields = hue.getFieldNames()
for i in range(0, hue.getNumFields()):
    print hue_fields[i], '=', hue_vecs[i]
```

结果类似，

```
Hue0 = 0.0183470315515
Hue1 = 0.000665208703398
Hue2 = 0.000427097170245
Hue3 = 0.000168505712084
Hue4 = 0.0
```

结果表明"偏红"的色调主导了这张样例 cookie 图像。

#### Haar 特征提取器
该提取器从图像生成类 Haar 特征。这些类 Haar 特征被机器学习的分类器用来根据分割对比度帮助识别图像中的对象或物体——在本例中就是类 Haar 特征。有兴趣的读者可以深入了解[类 Haar 特征](http://en.wikipedia.org/wiki/Haar-like_features)

例如，我们将使用 SimpleCV 的 [haar.txt](https://github.com/jonahgroup/SnackClassifier/blob/master/static/trainer/haar.txt) 文件定义`类 Haar 特征`提取参数集。然后，我们打印特征字段名及其值。由于该提取器产出大量特征，结果在这里做了截断显示。

```python
haar = HaarLikeFeatureExtractor(fname='haar.txt')
img = Image('cookie_001.png')

haar_vecs = haar.extract(img)
haar_fields = haar.getFieldNames()
for i in range(0, len(haar_fields)):
    print haar_fields[i], '=', haar_vecs[i]
```

结果类似，

```
feature2x1_1 = 2242356.0
feature2x1_2 = 498204.0
feature2x2_1 = -1306748.0
feature3x1_1 = 22915652.0
...
Angle_feature2x1_1 = 2242356.0
Angle_feature2x1_2 = 498204.0
Angle_feature2x2_1 = -1306748.0
Angle_feature3x1_1 = 22915652.0
...
```

### SimpleCV 分类（即机器学习）
本节我们将探索 [SimpleCV Machine Learning Package](http://simplecv.sourceforge.net/doc/SimpleCV.MachineLearning.html)。
本节介绍四种分类器：SVM、Tree、朴素贝叶斯和 KNN。

#### SVM（支持向量机）分类器
给定一组训练样本，每个样本都标记为属于两个类别之一，SVM 训练算法构建一个把新样本归入其中一个类别的模型，使之成为一个非概率的二元线性分类器。SVM 模型把样本表示为空间中的点，映射方式是让不同类别的样本被一条尽可能宽的清晰间隔分开。
延伸阅读：<http://en.wikipedia.org/wiki/Support_vector_machine>

#### Tree 分类器
决策树是一种类似流程图的结构：每个内部节点表示对某个属性的"测试"（例如抛硬币是正面还是反面），每条分支表示测试的结果，每个叶节点表示一个类别标签（在计算全部属性之后做出的决策）。从根到叶的路径表示分类规则。
延伸阅读：<http://en.wikipedia.org/wiki/Decision_trees>

#### 朴素贝叶斯分类器
朴素贝叶斯是一种构建分类器的简单技术：这类模型为以特征值向量表示的问题实例指派类别标签，类别标签取自某个有限集合。训练这类分类器并非只有单一算法，而是基于共同原理的一族算法：所有朴素贝叶斯分类器都假设，在给定类别变量的前提下，某个特定特征的值独立于任何其他特征的值。例如，如果一种水果是红色、圆形、直径约 10 厘米，就可以认为它是苹果。朴素贝叶斯分类器认为这些特征中的每一个都独立地贡献于"该水果是苹果"的概率，而不考虑颜色、圆度和直径特征之间任何可能的相关性。
延伸阅读：<http://en.wikipedia.org/wiki/Naive_bayes>

#### KNN（K 近邻）分类器
在 k-NN 分类中，输出是类别归属。一个对象由其邻居的多数投票来分类，对象被指派到其 k 个最近邻居中最常见的类别（k 是正整数，通常很小）。如果 k = 1，对象就被简单地指派为那唯一最近邻居的类别。
延伸阅读：<http://en.wikipedia.org/wiki/K-nearest_neighbor_algorithm>

### 图像特征提取与分类
有了累积至今的提取器与分类器，
我们已准备好把这些知识整合进一个完整的
图像提取与分类示例。

```python
def getExtractors():
    hue = HueHistogramFeatureExtractor(10)
    edge = EdgeHistogramFeatureExtractor(10)
    haar = HaarLikeFeatureExtractor(fname='haar.txt')
    return [hue, edge, haar]

def getClassifiers(extractors):
    svm = SVMClassifier(extractors)
    tree = TreeClassifier(extractors)
    bayes = NaiveBayesClassifier(extractors)
    knn = KNNClassifier(extractors)
    return {
        'svm': svm,
        'tree': tree,
        'bayes': bayes,
        'knn': knn
    }

extractors = getExtractors()
classifiers = getClassifiers(extractors)
```

我们用一组提取器构造出分类器。把代码片段拆解得更细一些：
我们创建了一组提取器，即 `hue`、`edge` 和 `haar` 提取器。这些提取器能够
接收一张图像并从中提取本质特征。
`hue` 提取器会关注像素的色调并把值分入 bin。
类似地，`edge` 和 `haar` 提取器会对图像执行它们各自的特殊计算。

从数学上说，这些提取器把图像特征**向量化**为一组特征值。
例如，"红色色调"这样的特征，可以由字段名 `Hue0` 中的值 0.018 来刻画。
我们可以把**特征向量**视为一种图像的形式化描述。
训练集中的每张图像都将被刻画为一个特征向量。

```python
img = Image('cookie_001.png')
hue_feature_vec = extractors[0].extract(img)
print hue_feature_vec
```

`hue_feature_vect` 的输出类似，

```
  [0.03549028421988077,
   0.001203778883175249,
   0.00035178807892081684,
   0.0009786293278747852,
   0.00033078967479953006,
   0.000523404665689852,
   0.00028308959630179216,
   5.3921827867008034e-05,
   0.0,
   0.0]
```

我们读取了图像 `cookie_001.png`，并用色调提取器（示例代码中的 `extractors[0]`）
提取了带有分箱值的色调特征向量。

```python
  img = Image('cookie_001.png')

  # iterating through all the extractors, and combine their extracted vectors
  feature_name = []
  feature_vec = []
  for i in range(0, len(extractors)):
      feature_name += extractors[i].getFieldNames()
      feature_vec += extractors[i].extract(img)

  # we can see how many features are extracted
  print "Total number of features:", len(feature_name)
  print "Total number of values:", len(feature_vec)
  print

  # display some random sample features
  for x in range(0, 8):
      i = int(random.random()*len(feature_name))
      print feature_name[i], '=', feature_vec[i]
```

这段代码列出 8 个随机特征字段及其值作为演示。

```
Total number of features: 260
Total number of values: 260

Angle_feature4x2_8 = 1662682.0
Angle_feature4x4_10 = 5007468.0
Angle0 = 0.0
Angle_feature5x1_8 = 16554016.0
Angle2 = 0.00310151227022
feature3x1_4 = 23657996.0
Angle_feature5x1_12 = -181856.0
Angle_feature4x1_1 = 43748.0
```

分类器将使用这些提取器获得特征向量，
训练自身以产出分类器模型。

### 用分类器模型分类零食
分类器训练完成后，我们可以把分类器模型保存到文件。
例如，我们已把 `tree` 分类器保存为 `tree.dat`。我们的示例 `tree` 分类器
已用标签 `[brownie, candy, cookie, other]` 训练。导入之后，
我们就可以用分类器模型对未知图像分类。

测试图像：

- [test_001.png]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/test_001.png)
- [test_002.png]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/test_002.png)
- [test_003.png]({{ site.baseurl }}/images/recognizing-snacks-using-simplecv/test_003.png)

```python
classifierFile = 'tree.dat'
classifier = TreeClassifier.load(classifierFile)

test_img1 = Image('test_001.png')
test_img2 = Image('test_002.png')
test_img3 = Image('test_003.png')

print "test_001.png is", classifier.classify(test_img1)
print "test_002.png is", classifier.classify(test_img2)
print "test_003.png is", classifier.classify(test_img3)
```

结果会告诉我们它们是什么。

```
test_001.png is brownie
test_002.png is candy
test_003.png is cookie
```

动手吧，你已经可以配置 Snack Watcher 去学习和分类一些零食了！
