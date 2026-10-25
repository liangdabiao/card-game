---
layout: post
title: 面向 NLP 系统的神经到符号知识综合
date: 2020-09-13 12:00:00.000000000 -00:00
permalink: synthesis-neural-symbolic-knowledge-nlp/
author: Benny Cheung
artwork:
  author: M.C. Escher - Metamorphosis II
tags:
- AI
- Deep Neural Network
- Neural-Symbolic
- NLP
- Prolog
- Python
category: post
comments: true
image: images/synthesis-neural-symbolic-knowledge-nlp/cover2x.jpg
images:
  cover: images/synthesis-neural-symbolic-knowledge-nlp/cover.jpg
  header: images/synthesis-neural-symbolic-knowledge-nlp/header.jpg
published: true
---

<!--excerpt.start-->
人类的大部分知识都以书面语言的形式被收集起来。从自然语言的文本形式中直接提取知识，是自然语言处理（NLP）自 AI 研究开端以来的宏愿之一。最近利用深度神经网络的 NLP 进展，已有效地把自然语言的解析与理解自动化。使用深度神经网络的 NLP 之所以成功，是因为 DNN 具备自适应学习能力，能够处理那些难以用传统符号规则描述的真实世界数据。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/Podcast_Neural_Symbolic_AI.mp3
{% include open-embed.html %}
----

我们过去对 "[Deep Learning on Text Data]({{ site.baseurl }}/deep-learning-on-text-data)"（文本数据上的深度学习）的探索演示了用 DNN 做 NLP 的基础。
本文中，我们将再向前跃进一步，迈入 NLP 的下一阶段。在 NLP 准确解析文本之后，可以计算出一棵依存图（dependency graph）。
为了达到人类水平的理解，我们可以使用传统的符号推理。

符号 AI 与新近从深度神经网络研究中演化而来的 NLP 工具的结合正走向成熟。
我们相信，这种高层符号推理与低层统计学习是互补的，AI 专家们也持此观点 [[Launchbury17]](#Launchbury17)。
让两者协同工作，自然语言*理解*将迈出重大的一步。
随后，人类可以借助符号解释来理解 AI 模型的推理，并改进人机交互。

![神经到符号 NLP 系统架构]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/Neural_Symbolic_NLP_System_Architecture.png)
*图：神经到符号 NLP 系统架构，展示了低层 NLP 与高层符号处理器之间的协同。通过把低层解析后的文本编码为符号表示，可以借助符号推理中可追溯的问答来改进人机交互。*

在"神经到符号 NLP 系统架构图"中，NLP 的未来被组件化为*自然语言处理器*——它的专门职责是用深度神经网络准确而灵活地解析文本。文本集合被转换为事实与规则，使得*符号处理器*可以对这些转换后的事实与规则构成的*知识库*应用高层推理。此外，借助数据挖掘与知识库管理系统（KBMS）的丰富积累，我们懂得如何让大量事实在 AI 推理任务中可用。如果把人类水平的符号事实喂入基于规则的系统，推理引擎就可以通过一组领域特定的规则进行反向链或正向链搜索 [[GiarratanoRiley04]](#GiarratanoRiley04)。最重要的商业价值在于：确保推理步骤可追溯、可解释，并且建立在来自人类语境的原始真实观察之上。

这看起来也许又是一个 AI 白日梦，但我们将迈出实际的工程一步，用 [DeepRank [TarauBlanco20]](#TarauBlanco20) 来展示可能性。我们先从一个简单文档综合神经到符号的知识并执行问答；随后，我们将从一个复杂的 HIPAA 法规文档综合知识，以展示系统更强大的能力。

# 神经到符号知识的综合
知识工程是创建事实与规则并应用于数据的过程，用以模仿人类思考和处理问题的方式。任务及其解法被分解为其结构，基于这些信息，推理引擎判定解法是如何得出的。传统上，知识工程流程需要领域专家与知识工程师协作，把领域事实与规则手工编码进知识库。这个过程通常非常昂贵而缓慢。

幸运的是，在文本文档领域，我们可以依赖隐式的文本数据来陈述关于其内容的事实——内容本身就是领域专家。我们所需做的，只是找到最好的知识表示来陈述这组隐式事实。规则则更棘手；不过，我们始终可以从语言语法和语句上下文所提供的通用文本理解出发。

我们认识到当前所提技术的局限 [[CS224N]](#CS224N)。

* 我们构建和访问记忆/知识的方法仍然很原始。
* 当前模型在制定和执行目标与计划方面几乎一无所有。
* 我们理解和利用句间关系的能力仍然相当不足。
* 我们仍无法大规模地利用常识知识从情境出发展开阐述。

前述清单并非不可攻克，但不在本文讨论范围之内。

## 使用 DeepRank
我们利用的是 [DeepRank [TarauBlanco20]](#TarauBlanco20) 的最新研究。该系统把基于 Python 的文本图处理算法与基于 Prolog 的符号推理引擎结合起来，构成一个统一的高层文本处理器，面向人类接口， capable of 文本问答。DeepRank 的后端作为 Python 客户端连接到 Stanford CoreNLP 服务器 [[ChenManning14]](#ChenManning14)，用它提供依存图的低层文本解析结果。DeepRank 可以生成摘要、关键词短语和其他关系，作为构成文本文档逻辑模型的抽取事实，随后交由符号推理引擎处理。

![DeepRank 系统架构]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRank_System_Architecture.png)
*图：DeepRank 系统架构。该系统开源（见[参考文献](#References)部分）*

作为介绍，我们从简单的 `test.txt` 开始，它用 3 句话描述苏格拉底与柏拉图的关系。

```
Socrates did not write any books.
Plato is a student of Socrates.
Plato wrote books and put words into Socrates mouth.
```

以下各节将描述 DeepRank 系统架构图中工作流的每一步，
* (0) 启动 Stanford CoreNLP 服务器
* (1) DeepRank 文本解析
* (2) 生成 Prolog 事实
* (3) 把事实与规则载入 Prolog
* (4) DeepRank 查询处理
* (5) 在 Prolog 中推理以提供答案

### 0. 启动 Stanford CoreNLP 服务器
Stanford CoreNLP 依存分析器 [[ChenManning14]](#ChenManning14) 格外出色，产出高度精确的依存图。
这些图的顶点是单词及其词性标签，带标签的边指示单词的句法头（如主语、直接宾语）。

下载 [Stanford CoreNLP Java 服务器](http://nlp.stanford.edu/software/stanford-corenlp-full-2018-10-05.zip)
  * 需要版本 3.9.2（2018-10-05 发布）才能工作
  * 解压到目录 `stanford-corenlp-full-2018-10-05`，`start_server.sh` 脚本会在该目录寻找 jar 文件。

我们在 shell 窗口启动基于 Java 的 Stanford CoreNLP 服务器，

```sh
java -mx16g -cp "stanford-corenlp-full-2018-10-05/*" \
    edu.stanford.nlp.pipeline.StanfordCoreNLPServer \
    -preload tokenize,ssplit,pos,lemma,ner,parse,depparse \
    -status_port 9000 -port 9000 -timeout 30000 -quiet
```

请注意我们请求了预载解析结果所需的标注器列表，

```sh
    -preload tokenize,ssplit,pos,lemma,ner,parse,depparse
```

### 1. DeepRank 文本解析
我们可以从导入基于 Python 的 DeepRank 主接口开始。`params` 控制系统配置。

```python
from textcrafts import deepRank as dr
from textcrafts.sim import *
from textcrafts.vis import *
from textcrafts.parser_api import *

class talk_params(dr.craft_params):
  def __init__(self):
    super().__init__()
    self.corenlp = True
    self.quiet = True
    self.summarize = True
    self.quest_memory = 1
    self.max_answers = 3
    self.repeat_answers = 'yes'
    self.by_rank = 'no'
    self.personalize = 30
    self.show=True
    self.cloud=36

params=talk_params()
```

然后，创建 `GraphMaker` 对象并传入 `params`。

```python
fName='test.txt'
fNameNoSuf = 'examples/{}'.format(fName)
gm=dr.GraphMaker(params=params)
gm.load(fNameNoSuf+'.txt')
```

给定文档被 DeepRank 的后端 Stanford CoreNLP 解析之后，它返回依存图和全部请求的标注。

```python
print(gm)
```

要获得依存图的概要洞见，`print(gm)` 显示如下特征，

```
--- GraphMaker object ---
nodes: 20
edges: 41
--- KEYPHRASES ---
Socrates;
student;
book;
Plato;
word;
--- SUMMARY ---
0 :  Socrates did not write any books .
1 :  Plato is a student of Socrates .
2 :  Plato wrote books and put words into Socrates mouth .
--- RELATIONS ---
('Socrates', 'write', 'book', 0)
('Plato', 'write', 'book', 2)
```

#### 用词云对关键词排序
更有趣的是，我们可以用[词云](https://en.wikipedia.org/wiki/Tag_cloud)可视化全部关键词的排名

```python
gm.kshow(params.cloud,file_name="{}_cloud.jpg".format(fName),show=True)

img = mpimg.imread('{}_cloud.jpg'.format(fName))
fig, ax = plt.subplots(figsize=(8, 8))
ax.imshow(img)
```

如预期，Socrates 被识别为排名最高的关键词。

![苏格拉底文档词云关键词排名]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/socrates/test_cloud.jpg)

#### 排名后的依存图
DeepRank 使用 `networkx` 工具包的 PageRank 实现。
在对依存图的句子和词节点进行排名之后，
系统还能用 Python 的 graphviz 库显示只包含最高排名节点的过滤子图。

```python
B = dr.best_line_graph(32,gm)
gshow(B, file_name="{}.gv".format(fName), attr="rel")
```

然后我们可以绘制排名后的依存图，

```python
import matplotlib.pyplot as plt
import matplotlib.image as mpimg

img = mpimg.imread('{}.gv.png'.format(fName))
fig, ax = plt.subplots(figsize=(16, 16))
ax.imshow(img)
```

![排名后的苏格拉底文档依存图]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/socrates/test.gv.png)

依存图的边使用[通用依存树库标签](https://universaldependencies.org)（Universal Dependencies Treebank Tags）标注。

![通用依存树库标签概览]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/Universal_Dependency_Treebank.png)

Stanford CoreNLP 向通用依存（Universal Dependencies）的演进，使依赖它的系统有可能
移植到通用依存工程覆盖的 70 多种语言。

### 2. 生成 Prolog 事实
把依存图和标注一起收集起来，
它们可以被翻译为代表文档内容的 Prolog 事实。

```python
from deep_talk.qpro import *

to_prolog('', gm, gm, fNameNoSuf, params=params)
```

以下谓词以 Prolog 可读代码的形式生成到文件中：

* **keyword**(WordPhrase). —— 抽取的关键词短语

```pl
keyword('Socrates').
keyword('student').
...
```

* **summary**(SentenceId,SentenceWords). —— 抽取的摘要句标识符及句中单词列表

```pl
summary(0, ['Socrates', 'did', 'not', 'write', 'any', 'books', '.']).
summary(1, ['Plato', 'is', 'a', 'student', 'of', 'Socrates', '.']).
...
```

* **dep**(SentenceID,WordFrom,FromTag,Label,WordTo,ToTag). —— 一条依存链接的组成部分，第一个参数标明它们被抽取自哪个句子

```pl
dep(0, 'write', 'VB', 'nsubj', 'Socrates', 'NNP').
dep(0, 'write', 'VB', 'aux', 'did', 'VBD').
...
```

* **edge**(SentenceID,FromLemma,FromTag,RelationLabel,ToLemma,ToTag).—— 带句标识符的边，标明抽取来源，以及边两端的词元及其词性标签

```pl
edge(0, 'write', 'VB', 'nsubj', 'Socrates', 'NNP').
edge(0, 0, 'SENT', 'about', 'Socrates', 'NNP')
...
```

* **rank**(LemmaOrSentenceId,Rank). —— 为每个词元计算的排名

```pl
rank('Socrates', 0.13106262484911943).
rank('student', 0.10238177148979816).
...
```

* **w2l**(Word,Lemma,Tag). —— 把每个单词关联到词元和标签的映射，来自词性标注器

```pl
w2l('Socrates', 'Socrates', 'NNP').
w2l('books', 'book', 'NNS').
w2l('wrote', 'write', 'VBD').
...
```

* **svo**(Subject,Verb,Object,SentenceId). —— 从解析器输入抽取的主谓宾关系，或基于 WordNet 的 is-a 与 part-of 标签位于动词位置的关系

```pl
svo('Socrates', 'write', 'book', 0).
svo('Plato', 'write', 'book', 2)
...
```

* **ner**(SentId,ListOfNamedEntityPairs) - 从命名实体识别（NER）标注抽取

```pl
ner(0, [(0, ('Socrates', 'PERSON'))]).
ner(1, [(0, ('Plato', 'PERSON')), (1, ('student', 'TITLE')), (2, ('Socrates', 'PERSON'))]).
ner(2, [(0, ('Plato', 'PERSON')), (1, ('Socrates', 'PERSON'))])
```

* **sent**(SentenceId,ListOfWords). —— 文档中的句子列表，第一个参数是句标识符，第二个参数是单词列表

```pl
sent(0, ['Socrates', 'did', 'not', 'write', 'any', 'books', '.']).
sent(1, ['Plato', 'is', 'a', 'student', 'of', 'Socrates', '.']).
sent(2, ['Plato', 'wrote', 'books', 'and', 'put', 'words', 'into', 'Socrates', 'mouth', '.']).
```

这些谓词以事实数据库的形式提供文档的关系视图，将支撑构建在其之上的推理机制。

### 3. 把事实与规则载入 Prolog
[PySwip](https://pypi.org/project/pyswip/) 是一个（Python - SWI-Prolog）桥接库，让 Python 程序能够查询 SWI-Prolog。
它提供 SWI-Prolog 外语接口、
一个让 Prolog 查询变简单的工具类，还有一个 Pythonic 接口。

#### Mac 上的安装
我们可以获取 [SWI-Prolog 7.6.4](http://www.swi-prolog.org/Download.html) 并安装。

> 在 Mac 上，pyswip 只能与 SWI-Prolog（threaded, 64 bits, version 7.6.4）配合工作

```
pip install pyswip
```

确保 `swipl` 可执行文件在 `PATH` 上，且包含 `libswipl.dylib` 的目录在
`DYLD_FALLBACK_LIBRARY_PATH` 环境变量中。
例如，如果 SWI-Prolog 安装在 `/Applications/SWI-Prolog.app` 目录，以下设置可能有效：

```sh
export PROLOG_HOME="/Applications/SWI-Prolog.app/Contents"
export PATH="$PATH:${PROLOG_HOME}/swipl/bin/x86_64-darwin15.6.0"
export DYLD_FALLBACK_LIBRARY_PATH="${PROLOG_HOME}/swipl/lib/x86_64-darwin15.6.0":${DYLD_FALLBACK_LIBRARY_PATH}
```

`pyswip` 设置完成后，我们就可以把事实与规则载入 SWI-Prolog 了。
默认的配套规则包含在 [deep_talk/qpro.pro](https://github.com/ptarau/DeepRank/blob/master/deep_talk/qpro.pro) 中。
这些规则用于把查询与文档事实进行匹配与推理。

```python
from pyswip import *
from pyswip.easy import *

# finds absolute file name of Prolog companion rules qpro.pro
def pro() :
    return 'deep_talk/qpro.pro'

def sink(generator) :
    for _ in generator : pass

prolog = Prolog()
sink(prolog.query("consult('" + pro() + "')"))
sink(prolog.query("load('"+fNameNoSuf+"')"))
```

执行之后，事实与规则已载入 SWI-Prolog。

### 4. DeepRank 查询处理
我们可以向知识库提出问题（即查询）。例如，问题：

```
Who is Plato's teacher?
```

查询将被解析。

```python
question="Who is Plato's teacher?"

qgm=dr.GraphMaker(params=params)
qgm.digest(question)
qfName=fNameNoSuf+'_query'

to_prolog('query_', gm, qgm, qfName,params=params)
```

和文档一样，我们可以可视化查询词云的关键词排名。

```python
img = mpimg.imread('query_cloud.jpg')
fig, ax = plt.subplots(figsize=(8, 8))
ax.imshow(img)
```

如预期，查询围绕 Plato 关键词为中心。

![苏格拉底文档查询词云关键词排名]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/socrates/query_cloud.jpg)

查询将被解析并生成查询对应的依存图

```python
img = mpimg.imread('query_graph.gv.png')
fig, ax = plt.subplots(figsize=(10, 10))
ax.imshow(img)
```

![排名后的苏格拉底文档查询依存图]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/socrates/query_graph.gv.png)


### 5. 在 Prolog 中推理以提供答案
它会激活一个 Prolog 进程，Python 向其交互式地发送关于所选文档的查询。
答案由 Prolog 计算得出，然后，如果 `say` 这个操作系统级功能可用（在 OS X 和 Linux 机器上），系统将真的把答案说出来。

```python
import subprocess

def say(what) :
  print(what)
  subprocess.run(["say", what])

rs=prolog.query("ask('" + fNameNoSuf + "'"  + ",Key)")
answers=[pair['Key'] for pair in rs]
    
if not answers: say("Sorry, I have no good answer to that.")
else :
  for answer in answers :
    say(answer)
```

最后，DeepRank 将返回排名最高的答案。结果合情合理。

```
1 : Plato is a student of Socrates . 
2 : Plato wrote books and put words into Socrates mouth . 
```

## HIPAA 法规
我们将使用美国《健康保险流通与责任法案》（HIPAA）作为更复杂的示例。HIPAA 根据披露类型与患者同意情况来规范患者医疗信息的传输。
我们不仅会看到知识工程流程有能力处理一份复杂交织的法规文档，
如今还可以解释答案是如何得出的。

### 示例：HIPAA 规则 164.508
为了帮助后文的描述，让我们阅读 HIPAA 规则 164.508——需要授权的使用与披露——的一小段节选。全文见 [Cornell19](https://www.law.cornell.edu/cfr/text/45/164.508)。

```
(a) Standard: Authorizations for uses and disclosures -
        (1) Authorization required: General rule. Except as otherwise permitted or required by this subchapter, a covered entity may not use or disclose protected health information without an authorization that is valid under this section. When a covered entity obtains or receives a valid authorization for its use or disclosure of protected health information, such use or disclosure must be consistent with such authorization.
        (2) Authorization required: Psychotherapy notes. Notwithstanding any provision of this subpart, other than the transition provisions in § 164.532, a covered entity must obtain an authorization for any use or disclosure of psychotherapy notes, except:
            (i) To carry out the following treatment, payment, or health care operations:
                (A) Use by the originator of the psychotherapy notes for treatment;
                (B) Use or disclosure by the covered entity for its own training programs in which students, trainees, or practitioners in mental health learn under supervision to practice or improve their skills in group, joint, family, or individual counseling; or
                (C) Use or disclosure by the covered entity to defend itself in a legal action or other proceeding brought by the individual; and
            (ii) A use or disclosure that is required by § 164.502(a)(2)(ii) or permitted by § 164.512(a); § 164.512(d) with respect to the oversight of the originator of the psychotherapy notes; § 164.512(g)(1); or § 164.512(j)(1)(i)
...
```

沿用入门示例的同一工作流，我们用 DeepRank 摄取这份 HIPAA 文档。
词云很好地总结了文档中的重要关键词。

![HIPAA 文档词云关键词排名]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/hipaa/hipaa_cloud.jpg)

当然，依存图看起来密集得多：更多节点与边，以及它们之间的互连。

![排名后的 HIPAA 文档依存图]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/hipaa/hipaa.gv.png)


让我们用一个相关的 HIPAA 问题测试系统：

```
What must be done before disclosing protected health information?
```

查询的词云很好地总结了寻找答案所需的重要关键词。

![HIPAA 查询词云关键词排名]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/hipaa/query_cloud.jpg)

生成的查询依存图将被翻译为 Prolog 查询，与文档事实进行匹配。

![排名后的 HIPAA 查询依存图]({{ site.baseurl }}/images/synthesis-neural-symbolic-knowledge-nlp/DeepRankExample/hipaa/query_graph.gv.png)

最后，答案都是贴切而良好的，都围绕"披露之前必须先获得授权"这一要求。

```
2 : Except as otherwise permitted or required by this subchapter , a covered entity may not use or disclose protected health information without an authorization that is valid under this section . 
9 : ( 4 ) Authorization required : Sale of protected health information . 
43 : ( iii ) The potential for information disclosed pursuant to the authorization to be subject to redisclosure by the recipient and no longer be protected by this subpart .
```

## 结语

使用 DeepRank 令人兴奋，它演示了从任意文档综合神经到符号知识的可能。我们可以看到，把深度神经网络 NLP 与符号推理相结合的策略是有效的，尽管它存在局限。要迈向更宏大的 AI 自然语言理解目标，系统需要在以下方面改进，

* 理解和利用句间关系的能力。
* 利用常识知识从情境出发展开阐述的能力。

提供 [Stanford CoreNLP](https://stanfordnlp.github.io/CoreNLP/) 的研究团队，正在用最新的神经网络技术切实改进句间关系的构建。要在知识库中构建更好的常识——支持泛化概念和更广泛的推理规则——我们可以从 Douglas Lenat 的 [Cyc](https://en.wikipedia.org/wiki/Cyc) 项目寻找研究洞见。


## 参考文献

* <a name="ChenManning14">[ChenManning14]</a> Danqi Chen and Christopher Manning, A Fast and Accurate Dependency Parser using Neural Networks, 2014, In Proceedings of the 2014 Conference on Empirical Methods in Natural Language Processing (EMNLP). Association for Computational Linguistics, 740–750.
  * 论文：<https://nlp.stanford.edu/pubs/emnlp2014-depparser.pdf>

* <a name="Cornell19">[Cornell19]</a> Cornell Law School (2019), Legal Information Institute, Digital version of 164.508
  * 网站：<https://www.law.cornell.edu/cfr/text/45/164.508>

* <a name="CS224N">[CS224N]</a> Christopher Manning, Stanford CS224N: NLP with Deep Learning, Winter 2019
  - 极佳的深度学习 NLP 讲座系列。
  * 网站：<http://www.stanford.edu/class/cs224n/>
  * 视频：<https://www.youtube.com/watch?v=8rXD5-xhemo&list=PLoROMvodv4rOhcuXMZkNm7j3fVwBBY42z>

* <a name= "GiarratanoRiley04">[GiarratanoRiley04]</a> Giarratano J, Riley G. (2004), Expert Systems: Principles and Programming, Fourth Edition, Course Technology, ISBN: 978-0534384470

* <a name="Launchbury17">[Launchbury17]</a> Launchbury J. (2017), A DARPA Perspective on Artificial Intelligence
  * 论文：<https://www.darpa.mil/attachments/AIFull.pdf>

* <a name="LamMitchell18">[LamMitchell18]</a> Lam P.E., Mitchell J.C., Sundaram S. (2018) A Formalization of HIPAA for a Medical Messaging System. In: Fischer-Hübner S., Lambrinoudakis C., Pernul G. (eds) Trust, Privacy and Security in Digital Business. TrustBus 2018. Lecture Notes in Computer Science, vol 5695. Springer.
  * 论文：<http://theory.stanford.edu/people/jcm/papers/hipaa_formalization.pdf>

* <a name="TarauBlanco20">[TarauBlanco20]</a> Paul Tarau and Eduardo Blanco, Interactive Text Graph Mining with a Prolog-based Dialog Engine, July 2020, Intl. Symposium on Practical Aspects of Declarative Languages, pp 3-19.
  * 论文：<https://arxiv.org/pdf/2008.00956>
  * 该集成系统的开源代码见 <https://github.com/ptarau/DeepRank>
