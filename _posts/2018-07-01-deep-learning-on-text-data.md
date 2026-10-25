---
layout: post
title: 文本数据上的深度学习
date: 2018-07-01 00:00:00.000000000 -00:00
permalink: deep-learning-on-text-data/
author: Benny Cheung
artwork:
  author: Style of text data transfer to Malevich's Bureau and Room
tags:
  - NLP
  - Deep Learning
  - Sentiment Analysis
category: post
comments: true
image: images/deep-learning-on-text-data/cover2x.jpg
images:
  cover: images/deep-learning-on-text-data/cover.jpg
  header: images/deep-learning-on-text-data/header.jpg
published: true
---

<!--excerpt.start-->
人类的绝大部分交流都是以自然语言书写的文本形式进行的。机器学习领域的最新进展证实，有意义的知识可以被有效地提取出来。一旦将自然语言处理的通用技术与机器学习相结合，就可以设想出广泛的实用企业应用。
<!--excerpt.end-->

----
> [2024/10/07] 我们可以把本文当作**播客讨论**（Podcast Discussion）来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主持人既风趣又富有洞见。

{{ site.baseurl }}/images/deep-learning-on-text-data/Podcast_Deep_Learning_on_Text.mp3
{% include open-embed.html %}
----

![文本之梦]({{ site.baseurl }}/images/deep-learning-on-text-data/StarryNight_Newspaper.jpg)

*图：将文本数据投影到梵高的《星夜》画作上，以此比喻从迷惑性的混沌中发现规律的梦想。*

我们想要学习的是如何把机器学习技术——深度学习应用于文本数据。不出所料，业务中拥有大量的文本数据，它们通常以非结构化文本的形式存在，可能是电子邮件、评论或文档，其原始形式是混乱无序的。对机器学习而言，从这种迷惑性的混沌中自动发现规律，是人工智能的工程梦想。为了应对这种自然语言的混沌，ML 开发者拥有自然语言处理（NLP）技术来发现规律。在本文中，我们将从 NLP 应用于 ML 的理论知识开始，然后在 Twitter 情感分析问题上实践这些技术，并应用深度学习来建模和预测推文的情感。

## 使用机器学习进行情感分析

情感分析是 NLP 领域中一个相当有趣的问题。每当有电子邮件进入客服收件箱时，业务方都希望能够识别客户的情感；如果客户的情感是负面的，我们希望把它发送到客服文件夹以引起注意；否则，我们希望把它发送到正面反馈收件箱。解决这个问题的一种方法是手工设计一套复杂的规则，然后实际实现一个程序来检查这些规则是否被满足。当一封电子邮件到达时，它会经过一个检查静态规则的算法，其输出是一个标签，用来说明这封邮件的情感是正面还是负面。

静态规则可以是例如一种检查，用于查看电子邮件中是否存在特定关键词。如果这些关键词存在，那么就把它发送到客服文件夹。对于某些业务来说，这种方法可能完全可行。但考虑到自然语言的复杂性和问题的类型，即使是人类专家也可能发现难以用规则来表达。在这种情况下，如果业务方恰好拥有大量的历史电子邮件，也就是说基本上已经有一大块文本可供使用；此外，如果业务方试图建模的模式或关系在动态且持续地变化，那么使用机器学习方法将是比基于规则的方法更好的选择。

在机器学习方法中，我们会这样设计系统的某个部分：一封电子邮件到来，它会经过一个检查邮件情感的已训练模型，模型会返回一个说明正面或负面的输出。这里唯一的区别在于，用于检查电子邮件情感是正面还是负面的规则，依赖于历史电子邮件。因此，ML 算法基本上会查看历史电子邮件来推导规则，并且随着业务积累越来越多的历史电子邮件，它会不断更新这些规则。这些已被标记为正面和负面情感的历史电子邮件，内嵌了大量信息，表明是什么让一封邮件成为正面或负面情感。ML 算法能够查看历史电子邮件，自行推断出那些规则是什么，并使用这些规则去检查任何新邮件；随后，把它正确地分类为正面或负面情感。

## NLP 中机器学习的类型

在解决任何 NLP 问题时，我们可以采取两种方法。基于规则的方法需要程序员要么知道规则是什么，要么通过数据分析和探索凭经验识别规则。在机器学习方法中，规则将由 ML 算法来识别。但首先需要建立起一套工作流程。

机器学习问题通常属于特定的几类。你首先要识别你试图解决的问题属于哪种类型或哪个类别。一旦确定了问题的类别，就需要使用数值属性来转换和表示数据。然后，我们会在这些数值数据上应用标准的 ML 算法。

如前所述，机器学习问题通常属于宽泛的几大类别。这些类别包括*分类聚类*、*推荐*和*回归*。由于我们试图解决的邮件情感问题属于分类，我们将研究集中在*分类聚类*上。

![NLP 中机器学习的类型]({{ site.baseurl }}/images/deep-learning-on-text-data/types_of_machine_learning.jpg)

*图：展示机器学习中聚类与分类的关键区别。*

### 分类
分类是机器学习中最常见的 NLP 任务类型之一，例如，执行情感分析把一段文本分类为正面或负面情感，或者把一批文章分类到不同体裁集合中的某一类。例如，一封电子邮件或一条推文可以是目标问题实例。这个问题实例需要被分类到若干类别之一。我们需要为问题实例分配一个类别或标签。在情感分析问题中，标签是正面（数值为 1）和负面（数值为 0）。

帮助执行分类的 ML 算法被称为**分类器**（Classifier）。要使用分类器，有一个前提条件。前提条件是拥有一组已知正确类别归属的实例。如果你在执行情感分析，就是一组已经被分类为正面或负面的推文。这组推文被称为训练数据，分类器正是从这些数据中推断出规则或模式，从而真正帮助把新推文分类为正面或负面。

### 聚类
另一类在 NLP 中相当常见的机器学习问题是**聚类**（Clustering）。例如，需要根据内容识别一组文章的主题或话题。具有某些共同属性的文章被识别为彼此相关。与分类相比，这里的关键在于：在开始之前，文章的分组是未知的。因此，使用聚类算法，可以把大量的文章聚类成较小的组。一旦我们得到了这些簇，这些组就具有某些共享的属性和含义，代表特定的主题或话题。聚类通常只用于探索数据、探索文本语料，并识别该集合中存在的模式。

以这些已识别的模式为指导，我们可以确定最适合应用的机器学习工作流程。

## 机器学习处理文本数据的机制

在确定了问题类型之后，下一步是从文本中识别数值属性。数值属性用于表示每一段文本。由于 ML 算法只接受数值数据作为输入，输入文本需要被转换为其数值表示。这些数值属性被称为特征。把文本转换为这些数值属性有多种不同的方法。

一个例子是词频表示（term frequency），它关注文本中出现的词的频率。另一个例子是词频-逆文档频率（term frequency inverse-document frequency）。在继续进行示例的过程中，我们将逐一考察这两种方法。

一旦数据以数值形式被读入，你就可以选取一个标准算法并将其应用于数据。算法的任务是从历史数据中发现模式。它接收历史数据——可能是你以数值属性形式输入给算法的一大块文本——并识别出量化这些数值属性之间关系的规则。

ML 算法识别出的这些规则共同构成一个**模型**（Model）。模型只是算法所识别出的模式的一种表示。它可以是一个数学方程那样的东西，也可以是一组以 if-then-else 语句形式表示的规则。就神经网络而言，它就是网络中的神经元权重集合。

* 对于分类问题，可以应用朴素贝叶斯（Naive Bayes）、支持向量机（Support Vector Machines）或深度学习等算法。
* 或者，对于聚类问题，可以选择 K-means 聚类或层次聚类（Hierarchical Clustering）等算法。


--------------
## Twitter 情感分析

机器学习是解决文本数据问题的极其有用的工具。我们可以在文本数据上应用基本的 ML 工作流程：

1. 准备数据 - 使用分词（tokenization）、停用词去除、词义消歧等。
2. 训练模型 - 使用无监督聚类、有监督分类等 ML 算法。
3. 评估模型 - 度量模型性能以持续改进模型。

### 准备数据
文本数据通常要经过一系列预处理任务，把一大块文本分解成更小且有意义的组成部分。根据数据集的要求，会应用以下任务的组合：

* **分词（Tokenization）** - 把单个句子或单个词本身分解开来，这个过程称为*分词*。
* **停用词（Stopwords）** - 在隔离出文本中的单个词或词元时，有些词并不为文本增加信息或含义，而另一些词的存在只是为了增加结构或出于语法目的。这些被称为*停用词*，例如 “is” 和 “the” 这样的词，停用词列表可以在进一步处理文本之前被过滤掉。
* **N-Gram** - 识别文本中最常出现的词是哪些，因为它们是文本中最重要的词；找出经常一起出现的词组，这是 *n-gram* 的任务，可以从文本中提取出更多含义。
* **词干提取（Stemming）** - 只提取词的词根，例如 'closed' 变成 'close'，因为它们具有相同的含义，这称为*词干提取*。

![文本预处理工作流程]({{ site.baseurl }}/images/deep-learning-on-text-data/text_preprocessing_workflow.jpg)

*图：文本预处理工作流程示例及其处理结果*

把文本数据转换为其他表示形式的技术有：

* **消歧（Disambiguation）** - 根据词出现的上下文来识别词的含义。通过解析句子来识别词性，判断某个词属于哪一类，是名词还是副词等等。
* **Word2Vec** - 基于词在文本组中的上下文和含义，把词向量化到多维空间中。


### 准备推文数据
推文数据可以从 [Twitter 情感分析数据集](http://thinknook.com/wp-content/uploads/2012/09/Sentiment-Analysis-Dataset.zip)下载。它是一个包含 160 万行的 csv 文件。除其他内容外，每行包含推文的文本和对应的情感。每条推文被标记为 `1` 表示*正面情感*，`0` 表示*负面情感*。

原始下载的情感推文需要进行一些预处理来清理 csv 文件。特别是，`SentimentText` 列需要被正确地加上引号。下面的脚本会用引号包围原始推文，并清理外层引号内部的引号，使数据格式可以被 Pandas 的 csv 解析器消费以构建 dataframe。

```python
fname = "tweets_download.csv"
with open(fname) as f:
    content = f.readlines()

content = [x.strip().split(',', 3) for x in content]
f.close()

fname = "tweets.csv"
with open(fname, 'w') as f:
    f.write('"ItemID","Sentiment","SentimentSource","SentimentText"\n')
    for x in content[1:]:
        f.write('"' + x[0] + '"')
        f.write(',')
        f.write('"' + x[1] + '"')
        f.write(',')
        f.write('"' + x[2] + '"')
        f.write(',')
        y = x[3].replace('"', '')
        f.write('"' + y + '"')
        f.write('\n')

f.close()
```

清理完 `"` 引号问题之后，我们可以把结果文件 `tweets.csv` 中的推文加载到 Pandas dataframe 中。

```python
import pandas as pd

tweets = pd.read_csv("tweets.csv")
```

可以检查 dataframe `tweets` 以深入了解推文数据，例如，从显示前 10 行开始，等等。

![推文数据框]({{ site.baseurl }}/images/deep-learning-on-text-data/tweets_data_table_head10.jpg)

### 推文文本预处理
我们希望把每条推文分解成有意义的词。

`tokenizer_tweet()` 函数会调用 `tokenizer()`，用于推文的*分词*、*停用词*和*标点去除*。推文特有的过滤逻辑被隔离在 `tokenizer_tweet()` 中，同时保持 `tokenizer()` 对任何文本数据都是通用的，以便潜在复用。

```python
import re
import nltk
from nltk.tokenize import word_tokenize, sent_tokenize
from nltk.corpus import stopwords
from string import punctuation

# 停用词列表，如冠词、介词
stop = set(stopwords.words('english'))

def tokenizer(text):
    try:
        tokens_ = [word_tokenize(sent) for sent in sent_tokenize(text)]
        
        tokens = []
        for token_by_sent in tokens_:
            tokens += token_by_sent

        tokens = list(filter(lambda t: t.lower() not in stop, tokens))
        tokens = list(filter(lambda t: t not in punctuation, tokens))
        
        filtered_tokens = []
        for token in tokens:
            if re.search('[a-zA-Z]', token):
                filtered_tokens.append(token)

        filtered_tokens = list(map(lambda token: token.lower(), filtered_tokens))

        return filtered_tokens
    except Exception as e:
        print(e)
        
def tokenizer_tweet(tweet):
    tweet = unicode(tweet.decode('utf-8').lower())
    tokens = tokenizer(tweet)
    tokens = filter(lambda t: not t.startswith('@'), tokens)
    tokens = filter(lambda t: not t.startswith('#'), tokens)
    tokens = filter(lambda t: not t.startswith('http'), tokens)
    return tokens
```

### 发现重要词语
文本预处理之后，探索永远不会停止。

有了分好词的词汇，我们必须继续回答这个问题：*对于问题领域而言，哪些是重要的词？*首先，我们展示一个典型的文本分析步骤，使用 tf-idf（词频-逆文档频率）来找出重要的词。理解 tf-idf 概念所付出的努力对于一般文本分析仍然很重要；不过，我们将说明为什么 tf-idf 词并不适合情感分析。

然后，我们将展示简单的词频统计就足以识别出推文中的重要词。

### （1）通过 tf-idf
要分析一个文本语料库，我们想知道哪些是重要的词。**tf-idf** 代表*词频-逆文档频率*（term frequency-inverse document frequency）。它是一种数值统计量，旨在反映一个词对某个文档或语料库（即文档集合）的重要程度。

对照本文而言，词对应于词元（token），文档对应于描述（description）。因此，语料库就是描述的集合。

词 t 在文档 d 中的 tf-idf 与词 t 在文档 d 中出现的次数成正比，但也会被词 t 在语料库文档集合中的频率所抵消。这有助于调整某些词总体上出现得更频繁、并不特别带有含义的事实。

因此，tf-idf 充当一种加权方案，用于提取文档中的相关词。

$$tfidf(t,d) = tf(t,d) . idf(t) $$

$$tf(t,d)$$ 是词 t 在文档 d 中的词频（即词元 t 在描述 d 中出现的次数）

$$idf(t)$$ 是词 t 的逆文档频率。它由以下公式计算：

$$idf(t) = log(1 + \frac{1 + n_d}{1 + df(d,t)}) $$

- $$n_d$$ 是文档数量
- $$df(d,t)$$ 是包含词 t 的文档（或描述）数量

计算 tfidf 矩阵是使用 scikit-learn 的 `TfidfVectorizer()` 方法完成的。让我们看看如何做到这一点：

```python
from sklearn.feature_extraction.text import TfidfVectorizer

# min_df 是包含词 t 的文档的最小数量
# max_features 是我们要考虑的唯一词元（跨文档）的最大数量
# TfidfVectorizer 使用我们上面定义的 tokenizer 对描述进行预处理
vectorizer = TfidfVectorizer(min_df=10, max_features=10000, tokenizer=tokenizer, ngram_range=(1, 2))
vz = vectorizer.fit_transform(list(tweets['SentimentText']))
```

vz 是一个 tfidf 矩阵。

- 它的行数是文档（描述）的总数
- 它的列数是这些文档（描述）中唯一词（词元）的总数

$$x_{dt}  = tfidf(t,d)$$，其中 $$x_{dt}$$ 是矩阵中索引 (d,t) 处的元素。

让我们创建一个字典，把词元映射到它们的 tfidf 值

```python
tfidf = dict(zip(vectorizer.get_feature_names(), vectorizer.idf_))
tfidf = pd.DataFrame(columns=['tfidf']).from_dict(dict(tfidf), orient='index')
tfidf.columns = ['tfidf']
```

我们可以用直方图来可视化 tfidf 分数的分布

```
tfidf.tfidf.hist(bins=50, figsize=(15,7))
```

![tf-idf 直方图分布]({{ site.baseurl }}/images/deep-learning-on-text-data/tfidf_Histogram_Distribution.jpg)

*图：tf-idf 分数直方图，高的 tf-idf 分数意味着区分度更高的词；反之，低的 tf-idf 意味着区分度更低的词。*

我们尝试查看一些明显的情感词的 tf-idf 分数。不出所料，这些词的区分度并不高，因为它们在语料库中被非常频繁地使用。

```python
tfidf.loc[['sorry','happy','sad']]
```

| | tfidf |
|-|-------|
| sorry | 5.710631 |
| happy | 5.092447 |
| sad | 4.721503 |

区分度最高的几个词可以通过以下方式找到：

```python
tfidf.sort_values(by=['tfidf'], ascending=False).head(3)
```

| | tfidf |
|-|-------|
| window | 7.812545 |
| maths | 7.812545 |
| grandma | 7.812545 |

这告诉我们 {window, maths, grandma} 是推文中很少使用的词。如果一条推文包含这些词，那就比包含其他词显得特殊得多。但对于情感分析来说，它并不是要找出区分度极高的词；相反，那些被普遍使用且被公认为带有情感的词应该排在更高的位置。

### （2）通过词频统计

在无法使用更复杂、更流行的 tf-idf 技术来发现重要词的情况下，我们测试了朴素的词频统计技术。

```python
import collections

counter = collections.Counter()
maxlen = 0
for index, tweet in tweets.iterrows():
    try:
        words = [x.lower() for x in tokenizer_tweet(tweet["SentimentText"])]
        if len(words) > maxlen:
            maxlen = len(words)
        for word in words:
            counter[word] += 1
    except:
        pass
```

令人惊讶的是，仅仅通过检查推文中最常用的前 10 个词，我们就发现情感词就在最常用的词之列。

```python
counter.most_common(10)

[(u"'s", 179509),
 (u"n't", 173837),
 (u"'m", 130868),
 (u'good', 88554),
 (u'day', 82165),
 (u'get', 81338),
 (u'like', 77753),
 (u'go', 72547),
 (u'quot', 71387),
 (u'got', 69689)]
```

我们决定采用简单的词频统计技术来构建用于机器学习的词汇表。我们的词汇表将是最常用的词，取自推文的 `VOCAB_SIZE`（=5000）个词。顺带一提，我们把词索引 `0` 保留给未知的 `_UNK_`，对于任何没有出现在我们词汇表中的其他词，我们将把它们指定为未知。因此，深度学习会把那些词当作匿名的 `_UNK_`。

```python
VOCAB_SIZE = 5000

word2index = collections.defaultdict(int)
for wid, word in enumerate(counter.most_common(VOCAB_SIZE)):
    word2index[word[0]] = wid + 1
vocab_sz = len(word2index) + 1
index2word = {v:k for k, v in word2index.items()}
index2word[0] = "_UNK_"
```

### 使用 word2vec 把词转换为数值表示
**word2vec** 是最近才发明的先进技术，用于把词转换为数值表示。word2vec 是由 Google 开发的一组深度学习模型，旨在捕捉词的上下文，同时又提出了一种非常高效的原始文本数据预处理方法。该模型以推文或新闻文章等大型文档语料库作为输入，生成一个通常为几百维的向量空间。语料库中的每个词都会在向量空间中被分配一个唯一的向量。

例如，如下面这张来自 [Tomas Mikolov 在 NIPS 2013 上的演讲](https://pdfs.semanticscholar.org/presentation/79dc/45651d6c173fe6132229203bcb9e8c84b555.pdf)的图所示，连接含义相似但性别相反的词的向量，在降维后的 2D 空间中近似平行，我们经常可以通过对词向量做算术运算得到非常直观的结果。该演讲提供了许多其他例子。

![word2vec 向量空间邻近性示例]({{ site.baseurl }}/images/deep-learning-on-text-data/word2vec_Vector_Space_Proximity_Examples.jpg)

*图：（左）展示 word2vec 向量与含义相似的词距离接近。（右）展示 word2vec 向量在不同抽象层次上检测多种特征。*

出于我们的目的，下载 [word2vec GoogleNews 向量](https://github.com/mmihaltz/word2vec-GoogleNews-vectors)作为起点，这些数据已经使用 Google News 语料库（30 亿词）的词向量模型完成了预训练（包含 300 万个 300 维英文词向量）。

下载 word2vec 数据之后，我们可以使用 `gensim.models.KeyedVectors` 加载它。

```python
from gensim.models import KeyedVectors

WORD2VEC_MODEL = "data/GoogleNews-vectors-negative300.bin.gz"
word2vec = KeyedVectors.load_word2vec_format(WORD2VEC_MODEL, binary=True)
```

为了对 word2vec 获得一些直观认识，例如，词 `man` 被向量化为一个 300 维的点，如下所示。

```python
print word2vec['man']

[ 0.32617188  0.13085938  0.03466797 -0.08300781  0.08984375 -0.04125977
 -0.19824219  0.00689697  0.14355469  0.0019455   0.02880859 -0.25
 -0.08398438 -0.15136719 -0.10205078  0.04077148 -0.09765625  0.05932617
  0.02978516 -0.10058594 -0.13085938  0.001297    0.02612305 -0.27148438
  0.06396484 -0.19140625 -0.078125    0.25976562  0.375      -0.04541016
  ...
  0.07177734  0.13964844  0.15527344 -0.03125    -0.20214844 -0.12988281
 -0.10058594 -0.06396484 -0.08349609 -0.30273438 -0.08007812  0.02099609 ]
```

word2vec 背后的强大概念是：在向量空间中彼此接近的词向量，所代表的词不仅含义相同，而且上下文也相同。当我们查找最相似的词时，word2vec 给出如下结果。

```python
word2vec.most_similar('man')

[(u'woman', 0.7664012908935547),
 (u'boy', 0.6824870705604553),
 (u'teenager', 0.6586930751800537),
 (u'teenage_girl', 0.6147903800010681),
 (u'girl', 0.5921714901924133),
 (u'suspected_purse_snatcher', 0.571636438369751),
 (u'robber', 0.5585119128227234),
 (u'Robbery_suspect', 0.5584409236907959),
 (u'teen_ager', 0.5549196600914001),
 (u'men', 0.5489763021469116)]
```

我们发现词的向量表示的有趣之处在于，它自动嵌入了通常必须手工设计的若干特征。由于 word2vec 依靠深度神经网络来检测模式，我们也可以依靠 word2vec 在不同抽象层次上检测多种特征。

> 注意一个有趣的现象：“man”与“robber”和“Robbery_suspect”最为相似。这一定是受到了构建 word2vec 时所用的那组新闻来源的影响。

对于每个词索引，对应的 word2vec 向量被赋值到 `embbeding_weights` 矩阵中。这个矩阵稍后将用于把词转换为 word2vec 表示。

```python
import numpy as np

embedding_weights = np.zeros((vocab_sz, EMBED_SIZE))
for word, index in word2index.items():
    try:
        embedding_weights[index, :] = word2vec[word]
    except KeyError:
        pass
```

例如，如果词 `man` 在词汇表中为 `word2index['man']=123`，那么向量化后的词 `man` 就由 `embbeding_weights[123]` 表示。

### 使用 Keras 训练模型

为了保持重点，我们不会偏离主题去解释 keras 深度学习框架。感兴趣的读者可以查阅 Antonio Gulli 与 Sujit Pal 所著的 [Deep Learning with Keras](https://www.packtpub.com/big-data-and-business-intelligence/deep-learning-keras) 一书。

### 准备数据集
还要做更多的数据准备！我们需要把推文重新格式化为适合 keras 各层消费的向量格式。

将构造出以下向量：

* `X` 是输入的推文词向量
* `Y` 是情感标签，0 为负面，1 为正面情感
* `Text` 是推文的 “SentimentText”（稍后我们将使用这个向量来显示推文及预测结果）

```python
from keras.preprocessing.sequence import pad_sequences
from keras.utils import np_utils

ts = []
xs = []
ys = []
for index, tweet in tweets.iterrows():
    try:
        ts.append(tweet["SentimentText"])
        ys.append(tweet["Sentiment"])
        words = [x.lower() for x in tokenizer_tweet(tweet["SentimentText"])]
        wids = [word2index[word] for word in words]
        xs.append(wids)
    except:
        print tweet

Text = ts
X = pad_sequences(xs, maxlen=maxlen)
Y = np_utils.to_categorical(ys)
```

### 划分数据集
通常的数据划分建议是训练/测试数据集按 70/30 划分。我们使用 `sklearn.model_selection` 工具中的 `train_test_split()` 函数来随机化划分。注意参数 `test_size=0.3`，意味着 30% 将被分配给测试数据集。我们把初始 `random_state=42` 设置为数据集可复现，以支持使用相同划分序列的多次运行。

```python
from sklearn.model_selection import train_test_split

Xtrain, Xtest, Ytrain, Ytest = train_test_split(X, Y,
                                                test_size=0.3, 
                                                random_state=42)
# 对相应的 Text 向量执行完全相同的划分
Xtrain, Xtest, Ttrain, Ttest = train_test_split(X, Text,
                                                test_size=0.3, 
                                                random_state=42)
```

### 深度学习
我们使用 keras 定义并训练深度学习神经网络。

#### （1）定义深度神经网络
我们使用上一节构建的 `embedding_weights` 矩阵来初始化嵌入层的权重。

模型使用 `binary-crossentropy` 损失函数（因为我们只有 2 个类别）和 `adam` 优化器进行编译。

```python
from keras.layers.core import Dense, Dropout
from keras.layers.convolutional import Conv1D
from keras.layers.embeddings import Embedding
from keras.layers.pooling import GlobalMaxPooling1D
from keras.models import Sequential
from keras import regularizers

EMBED_SIZE = 300
NUM_FILTERS = 128
NUM_WORDS = 3

model = Sequential()
model.add(Embedding(vocab_sz, EMBED_SIZE, input_length=maxlen,
                    weights=[embedding_weights],
                    embeddings_regularizer=regularizers.l2(0.01),
                    trainable=True))
model.add(Dropout(0.5))
model.add(Conv1D(filters=NUM_FILTERS, kernel_size=NUM_WORDS,
                 activation="relu"))
model.add(Dropout(0.5))
model.add(GlobalMaxPooling1D())
model.add(Dense(sentiment_len, activation="sigmoid"))

model.compile(optimizer="adam", loss="binary_crossentropy",
              metrics=["accuracy"])
```

深度学习网络通过 `summary()` 函数来汇总。

```python
model.summary()
```

![推文 Keras 网络摘要]({{ site.baseurl }}/images/deep-learning-on-text-data/tweets_keras_network_summary.jpg)

#### （2）训练深度神经网络
我们将通过 `model.fit()` 调用网络训练，以批量大小 16 训练网络 20 个轮次（epoch），然后评估训练好的模型。

```python
BATCH_SIZE = 16
NUM_EPOCHS = 20
history = model.fit(Xtrain, Ytrain, batch_size=BATCH_SIZE,
                    epochs=NUM_EPOCHS,
                    validation_data=(Xtest, Ytest))
```

经过 20 个轮次的训练后，准确率收敛到约 72%。

```python
Train on 1104929 samples, validate on 473542 samples
Epoch 1/20
1104929/1104929 [==============================] - 1966s 2ms/step - loss: 1.4675 - acc: 0.7124 - val_loss: 0.8175 - val_acc: 0.7212
...
Epoch 20/20
1104929/1104929 [==============================] - 1941s 2ms/step - loss: 0.8374 - acc: 0.7174 - val_loss: 0.8260 - val_acc: 0.7221
```

![推文训练准确率图表]({{ site.baseurl }}/images/deep-learning-on-text-data/tweets_training_accuracy_chart.jpg)

*图：图表显示了训练准确率和验证准确率。训练准确率很快收敛。*

#### （3）评估训练好的模型
经过 20 个轮次的训练后，模型在测试集上给出了 72.2% 的准确率。

```python
score = model.evaluate(Xtest, Ytest, verbose=1)
print("Test score: {:.3f}, accuracy: {:.3f}".format(score[0], score[1]))

473542/473542 [==============================] - 85s 180us/step
Test score: 0.826, accuracy: 0.722
```

### 评估模型以进一步改进
模型的预测准确率不错，但并不出色。由于 keras 可以为每个分类返回预测向量，在我们的例子中有 2 个类别（`0` 负面和 `1` 正面情感），我们可以进行进一步的分析，比较实际结果与预测结果；希望这些分析能够揭示改进模型的线索。

```python
Ypredict = model.predict(Xtest, batch_size=32, verbose=1)
print Ypredict[:5]
```

我们可以看到预测概率向量，每一行（一条推文）对应每一列（每个类别的概率）。每一行的概率总和应该为 1。

```python
array([[ 0.25863048,  0.74136955],
       [ 0.69782579,  0.30217424],
       [ 0.20225964,  0.79774034],
       [ 0.16227043,  0.83772963],
       [ 0.74099219,  0.25900784]], dtype=float32)
```

### （0）找出对应的实际情感与预测情感
这是为支持分析而构造向量的数据准备工作。我们可以为每条推文收集 `actual_sentiment` 标签和 `predict_sentiment` 标签。对于情感分类，选择概率最高的索引，即可得到预测标签。

```python
actual_sentiment_idx = np.argmax(Ytest, axis=1)
actual_sentiment = [id2sentiment[k] for k in actual_sentiment_idx]
predict_sentiment_idx = np.argmax(Ypredict, axis=1)
predict_sentiment = [id2sentiment[k] for k in predict_sentiment_idx]
predict_probs = np.array([Ypredict[k][predict_sentiment_idx[k]] for k in np.arange(len(predict_sentiment_idx))])
```

### （1）随机查看预测正确的标签
我们可以随机查看预测正确的标签，并连同推文一起打印出来。

```python
correct = np.where(predict_sentiment_idx==actual_sentiment_idx)[0]
print("Found %d correct labels of %d" % (len(correct), len(predict_sentiment_idx)))
idx = permutation(correct)[:5]
for i in idx:
    print("[%d] predict '%s' -> actual '%s': '%s'" %(i, predict_sentiment[i], actual_sentiment[i], Ttest[i]))
```

结果展示了预测以及对应的推文。

```
Found 341950 correct labels of 473542
[61746] predict '0' -> actual '0': 'I'm off to work, be back around 11pm  I hate long work days'
[25913] predict '1' -> actual '1': 'Jus woke up!!!!!!! Crazy night'
[85662] predict '1' -> actual '1': '&quot;Kids&quot; on repeat. Absolutely amazing! So fatigued right now but we're leaving 4 Norway today'
[202167] predict '0' -> actual '0': '@RoxygirlSLB nah didn't say that but she probably would have one! Hehe I want a black one, but they don't do them in UK'
[165979] predict '1' -> actual '1': 'Aww! Zack &amp; Vanessa are so cute together.'
```

### （2）随机查看预测错误的标签
我们可以随机查看预测错误的标签，并连同推文一起打印出来。

```python
incorrect = np.where(predict_sentiment_idx!=actual_sentiment_idx)[0]
print("Found %d incorrect labels of %d" % (len(incorrect), len(predict_sentiment_idx)))
idx = permutation(incorrect)[:5]
for i in idx:
    print("[%d] predict '%s' -> actual '%s': '%s'" %(i, predict_sentiment[i], actual_sentiment[i], Ttest[i]))
```

结果展示了错误的预测以及对应的推文。这些失败展示了推文的微妙之处，例如 'Oh happy day. Not'，网络错误地预测为正面情感。相距较远的词 'Not' 似乎对推文中的 'happy' 一词构成了否定因素。

```
Found 131592 incorrect labels of 473542
[241731] predict '0' -> actual '1': '@astynes Yr norty!  When we try copy a big disc like that we have to forfeit some of the quality 2 fit it on. It's a gr8 disc 2 watch tho!'
[235609] predict '1' -> actual '0': 'Oh happy day. Not'
[220989] predict '0' -> actual '1': 'My first day in Twitter It will be interesting I think So do you want to see my hometown?  http://www.admkrsk.ru/doc.asp?id=12'
[315661] predict '0' -> actual '1': 'Working on transfer papers, then lunch with Luke.'
[340588] predict '1' -> actual '0': 'business business business  bye bye awards
```

### （3）查看置信度最高且预测正确的标签
对于每个类别，我们可以查看置信度最高且预测正确的标签。

```python
for sent in range(0,sentiment_len):
    correct_sent = np.where((predict_sentiment_idx==sent) & (predict_sentiment_idx==actual_sentiment_idx))[0]
    print("Found %d confident correct %d:'%s' category" % (len(correct_sent), sent, id2sentiment[sent]))
    if (len(correct_sent) > 0):
        most_correct_sent = np.argsort(predict_probs[correct_sent])[::-1][:4]
        for k in most_correct_sent:
            print("[%d] predict '%s' with %.3f confidence: '%s'" %(correct_sent[k], predict_sentiment[correct_sent[k]], predict_probs[correct_sent[k]], Ttest[k]))
```

置信度最高的预测结果连同推文一起展示如下。

```
Found 185735 confident correct 1:'1' category
[57266] predict '1' with 0.974 confidence: '@n8moses What fun! Hope you brought yummies for them'
[443611] predict '1' with 0.973 confidence: 'can't say I've had a Friday go this bad this fast before!'
[100613] predict '1' with 0.973 confidence: 'OMG my dog just attacked the baby possum  we think its ok but not positive.... -fingers crossed-'
[253526] predict '1' with 0.969 confidence: 'E3 looks awsome this year i missed microsofts keynote though  damn!'
```
### （4）查看置信度最高但预测错误的标签
对于每个类别，我们可以查看置信度最高但预测错误的标签。

```python
for sent in range(0,sentiment_len):
    incorrect_sent = np.where((predict_sentiment_idx==sent) & (predict_sentiment_idx!=actual_sentiment_idx))[0]
    print("Found %d confident incorrect %d:'%s' category" % (len(incorrect_sent), sent, id2sentiment[sent]))
    if (len(incorrect_sent) > 0):
        most_incorrect_sent = np.argsort(predict_probs[incorrect_sent])[::-1][:4]
        for k in most_incorrect_sent:
            print("[%d] predict '%s' but actual '%s' with %.3f confidence: '%s'" %(incorrect_sent[k], predict_sentiment[incorrect_sent[k]], actual_sentiment[incorrect_sent[k]], predict_probs[incorrect_sent[k]], Ttest[k]))
```

置信度最高的错误预测结果连同推文一起展示如下。

```
Found 80332 confident incorrect 0:'0' category
[398966] predict '1' but actual '0' with 0.962 confidence: 'I cnt bel eive im saying this an ino i sudnt but i kinda love lois'
[425603] predict '1' but actual '0' with 0.961 confidence: 'was almost in a wreck and mcdonalds was too crowded for me to stop, so now i'm shaking AND hungry'
[340460] predict '1' but actual '0' with 0.961 confidence: '#phpkonferenca anÅ¾e is beeing funny'
[87609] predict '1' but actual '0' with 0.958 confidence: 'Also my niece has better brand names than me. Girl is rockin pink baby phat blankets'
```

### （5）通过混淆矩阵查看
分析分类模型结果最常用的方式也许是使用混淆矩阵。scikit-learn 为此提供了一个方便的 `confusion_matrix()` 函数：

```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(actual_sentiment_idx, predict_sentiment_idx)
```

我们可以直接打印出混淆矩阵，也可以展示图形化的视图（后者主要用于类别数量较多的情形）。

```python
import matplotlib.pyplot as plt

def plot_confusion_matrix(cm, classes, normalize=False, title='Confusion matrix', cmap=plt.cm.Blues):
    """
    此函数打印并绘制混淆矩阵。
    可以通过设置 `normalize=True` 来应用归一化。
    （此函数复制自 scikit 文档。）
    """
    plt.figure(figsize=(8, 8), dpi=100)
    plt.imshow(cm, interpolation='nearest', cmap=cmap)
    plt.title(title)
    plt.colorbar()
    tick_marks = np.arange(len(classes))
    plt.xticks(tick_marks, classes, rotation=45)
    plt.yticks(tick_marks, classes)

    if normalize:
        cm = cm.astype('float') / cm.sum(axis=1)[:, np.newaxis]
    print(cm)
    thresh = cm.max() / 2.

    plt.tight_layout()
    plt.ylabel('True label')
    plt.xlabel('Predicted label')
```

调用 `plot_confusion_matrix()` 进行绘图：

```python
plot_confusion_matrix(cm, [id2sentiment[i] for i in range(0,sentiment_len)])
```

混淆矩阵如下：

```
[[156215  80332]
 [ 51260 185735]]
```

![推文混淆矩阵]({{ site.baseurl }}/images/deep-learning-on-text-data/tweets_confusion_matrix.jpg)

该模型更倾向于把负面情感错误地预测为正面情感。

### 进一步改进
对于进一步改进，这些分析无疑给出了一些如何增强模型的线索：

* 增大词汇表规模
* 为那些情感词增加权重
* 忽略不在英语词典中的未知词
* 使用更先进的深度学习层（例如 RNN）改进网络，以获得更大的上下文


## 参考文献
* Antonio Gulli & Sujit Pal，*Deep Learning with Keras*，Packt Publishing，2017 年 4 月，ISBN：978-1-78712-842-2
    * 图书网站：<https://www.packtpub.com/big-data-and-business-intelligence/deep-learning-keras>
* Ahmed Besbes，*Sentiment Analysis on Twitter using word2vec and keras*，2017 年 4 月。
    * 在线阅读：<https://ahmedbesbes.com/sentiment-analysis-on-twitter-using-word2vec-and-keras.html>
* Ahmed Besbes，*How to Mine Newsfeed Data and Extract Interactive Insight in Python*，2018 年 4 月。
    * 在线阅读：<https://ahmedbesbes.com/how-to-mine-newsfeed-data-and-extract-interactive-insights-in-python.html>
* T. Mikolov, I. Sutskever, K. Chen, G. S. Corrado, J. Dean, Q. Le, and T. Strohmann，*Learning Representations of Text using Neural Networks*，NIPS 2013
    * 演讲幻灯片：<https://pdfs.semanticscholar.org/presentation/79dc/45651d6c173fe6132229203bcb9e8c84b555.pdf>

希望你和我们一起享受文本数据上的机器学习之旅！
