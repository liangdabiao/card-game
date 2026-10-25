---
layout: post
title: 用 LangChain 与 OpenAI 向一本书提问
date: 2023-03-12 12:00:00.000000000 -00:00
permalink: ask-a-book-questions-with-langchain-openai/
author: Benny Cheung
artwork:
  author: Stable Diffusion AI
tags:
- AI
- OpenAI
- LangChain
- NLP
- Python
category: post
comments: true
image: images/ask-a-book-questions-with-langchain-openai/post_cover.jpg
images:
  cover: images/ask-a-book-questions-with-langchain-openai/cover2x.jpg
  header: images/ask-a-book-questions-with-langchain-openai/header.jpg
published: true
---

<!--excerpt.start-->
读一本书可以是一段充实的体验：它带你前往新的世界、向你介绍新的人物、让你接触新的概念与思想。然而，读完之后，你也许发现自己有一堆想讨论的问题。也许身边没有读过这本书或有兴趣讨论的人，也许你只是想按自己的节奏探索这本书。在这种情况下，你可能会疑惑：要完全消化这本书、回答自己的问题需要多久？没有导师或朋友提供引导与讨论，你可能需要采取更有思考性、更内省的阅读方式。
<!--excerpt.end-->

Mortimer Adler 在他的经典著作["How to Read a Book"](https://www.amazon.ca/How-Read-Book-Mortimer-Adler/dp/0671212095)中给出过著名的忠告，

> "_阅读一本书，应该是你与作者之间的一场对话。_"

![向书提问的 AI 导师]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Steampunk_Tutor.jpg)

*图：想象我们有一位不做评判的 AI 导师，辅助对一本书进行问答。（图片来源：Stable Diffusion 生成 artwork）*

想象我们有一位不做评判的 AI 导师来辅助问答过程，这会大有裨益，尤其是在探索和运用一本书提出的观点时。AI 可以对书的主题和概念提供不带偏见、客观的洞见，帮助你理解作者对主题的立场。在 AI 的协助下，你可以提出更深入、更有意义的问题，并得到富有思想、信息量充足的回应，帮助你把书中的观点与你自己的经验和信念联系起来。这能让阅读前后的体验都更加丰富。

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Podcast_Ask_a_Book_Questions.mp3
{% include open-embed.html %}
----

## 如何构建一个 AI 问答系统？
本文采取动手构建问答系统的实践路线。在这个过程中，我们讲解如何使用 [OpenAI](https://github.com/openai/openai-python)、[LangChain](https://langchain.readthedocs.io/en/latest/) 和 [Pinecone](https://www.pinecone.io/)（一个外部向量存储）对一本书进行语义搜索和查询。书被拆分为更小的文档，用 OpenAI embeddings 把它们转换为向量，然后使用 Pinecone 存储到外部。

![向书提问工作流]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow.jpg)

*图：本文将走完以下过程：(1) 提取书籍内容，(2) 把书拆分为更小的块，(3) 构建语义索引，(4) 向书提问（红色箭头表示提问流，绿色箭头表示回答流）。*

### 选定书籍
我们使用一本有趣且免费的在线图书：[60 Leaders on Artificial Intelligence](https://www.60leaders.com/_files/ugd/e7ba2f_e965e98b5ab3423f8b131d1a120dc1bf.pdf?index=true) 来演示整个过程。这是一本 PDF 格式的书，共 236 页，包含大量图表。如果我们能自动提取非结构化文本并构建索引，随后就能对内容进行查询和摘要。

![60 Leaders on Artificial Intelligence]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Book_60_Leaders_on_AI_cover.jpg)

*图：使用《60 Leaders on Artificial Intelligence》进行实现。*

这个示例演示了如何用自然语言提问并获得回答。这种方法不限于书籍，也可用于内部文档或外部数据集。按照列出的步骤，读者将能够对大量文本进行精细的搜索，这有助于回答读完这本书后可能产生的问题。

## 安装

秉持"干中学"的理念，我们将以实践的方式演示如何安装构建系统所需的全部 Python 模块。最新的 LangChain 具备处理多种[非结构化文档格式](https://langchain.readthedocs.io/en/latest/modules/document_loaders/how_to_guides.html)（包括 PDF 和 Microsoft Word）的全部能力，要求 Python >= 3.8.1。首先，我们用 `python==3.9` 创建一个名为 `nlp` 的 virtualenv。

```
conda create -n nlp python==3.9
conda activate nlp
```

激活 `nlp` virtualenv 之后，我们可以安装带"all"模块（全部集成所需）的 LangChain：

```
pip install -U langchain[all]
```

我们还要添加 OpenAI 和 Pinecone 支持：

```
pip install openai
pip install pinecone-client
```

### Mac 平台运行要求
我们使用 Mac M1 Pro 运行实验。还需要一些额外的 `brew` 软件包。（显然，我们无法提供 Windows 上的安装说明。）

> - [poppler](https://poppler.freedesktop.org/) 是一个用于渲染[便携式文档格式](https://en.wikipedia.org/wiki/Portable_Document_Format "Portable Document Format")（PDF）文档的自由软件工具库。
> - [tesseract](https://pypi.org/project/pytesseract/) 是一个 Python 的光学字符识别（OCR）工具。也就是说，它能识别并"读出"嵌入图像中的文本。

```
# Install other dependencies
# https://github.com/Unstructured-IO/unstructured/blob/main/docs/source/installing.rst
brew install libmagic
brew install poppler
brew install tesseract
# If parsing xml / html documents:
brew install libxml2
brew install libxslt
```

### Unstructured 文件加载器
[LangChain Unstructured](https://langchain.readthedocs.io/en/latest/modules/document_loaders.html) 讲解了如何加载多种类型的文件。Unstructured 目前支持加载文本文件、PowerPoint、HTML、PDF、图像等。

其他需要安装的依赖见 https://langchain.readthedocs.io/en/latest/modules/document_loaders/examples/unstructured_file.html

> - [unstructured](https://github.com/Unstructured-IO) 让准备 PDF、HTML 和 Word 文档等非结构化数据以供下游数据科学任务使用变得容易。
> - [layoutparser](https://layout-parser.github.io/) 旨在提供广泛的工具，简化文档图像分析（DIA）任务。
> - [detectron2](https://github.com/facebookresearch/detectron2) 是 Facebook AI Research 的下一代库，提供最先进的检测与分割算法。

```
pip install "unstructured[local-inference]"
pip install "detectron2@git+https://github.com/facebookresearch/detectron2.git@v0.6#egg=detectron2"
pip install layoutparser[layoutmodels,tesseract]
```

如果你有毅力装好全部依赖，就可以开始实际的实现流程了。

---------------
## 实验实现

假设本文读者具备 Python 编程语言的基础知识，并对自然语言处理（NLP）有基本了解。这将帮助他们跟上文章的技术细节，理解问答系统实现中使用的概念与技术。不过，我们会努力以清晰简洁的方式解释关键概念，确保所有背景的读者都能从本文受益。

> 特别感谢 [Data Independent](https://www.youtube.com/@DataIndependent) 的视频和 notebook，它们启发了这个实现，见**参考文献**部分。

### 1. 提取书籍内容
这段代码提供了一个基本示例：如何使用 LangChain 库从 PDF 文件提取文本数据，并显示该文件内容的一些基本信息。

![向书提问工作流——第 1 步]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow_Ext_Step1.jpg)

*图：展示第 (1) 步提取书籍内容（红色高亮）。*

使用 LangChain 库的 `document_loaders` 和 `text_splitter` 模块。这些库包含让用户访问和操作来自不同来源的文本数据的函数和类。具体而言是 `UnstructuredPDFLoader`，用于加载 PDF 文件并提取文本。PDF 文件的路径指定为 "data/60 Leaders on AI.pdf"。

```python
from langchain.document_loaders import UnstructuredPDFLoader, OnlinePDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader = UnstructuredPDFLoader("data/60 Leaders on AI.pdf")
data = loader.load()
print (f'You have {len(data)} document(s) in your data')
print (f'There are {len(data[0].page_content)} characters in your document')
```

加载 PDF 文件后，它会显示第一个文档页面内容中的字符数。

```
You have 1 document(s) in your data
There are 533071 characters in your document
```

### 2. 把书拆分为更小的块
我们将把加载的 PDF 文档划分为每块 1000 字符的较小"页面"。这样做的原因是：当我们向 OpenAI 提问时，为其提供上下文信息。这是因为 OpenAI embeddings 对较短的文本效果最好。与其每次提问都让 OpenAI 读整本书，不如给它一小段相关信息去处理，这样更高效、更划算。

![向书提问工作流——第 2 步]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow_Ext_Step2.jpg)

*图：展示第 (2) 步把书拆分为更小的块（红色高亮）。*

```python
text_splitter = RecursiveCharacterTextSplitter(
  chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(data)

print (f'Now you have {len(texts)} documents')
```

```
Now you have 701 documents
```

现在，我们已完成文本准备，可以进入下一步。

### 3. 构建语义索引
为文档创建 embeddings，为语义搜索做好准备。我们把这些向量在线存储到 Pinecone 向量存储中，这样以后可以把更多书加入语料库，而无需每次重新读取 PDF。我们还在索引中分配了一个 `book` 命名空间。

![向书提问工作流——第 3 步]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow_Ext_Step3.jpg)

*图：展示第 (3) 步构建语义索引（红色高亮）。*

```python
from langchain.vectorstores import Chroma, Pinecone
from langchain.embeddings.openai import OpenAIEmbeddings
import pinecone

embeddings = OpenAIEmbeddings(openai_api_key=OPENAI_API_KEY)

# initialize pinecone
pinecone.init(
    api_key=PINECONE_API_KEY,  # find at app.pinecone.io
    environment=PINECONE_API_ENV # next to api key in console
)
index_name = "langchain-openai"
namespace = "book"

docsearch = Pinecone.from_texts(
  [t.page_content for t in texts], embeddings,
  index_name=index_name, namespace=namespace)
```

完成 Pinecone 和 OpenAI API 的双重认证之后，我们可以使用 `from_texts()` 函数把每个文档转换为向量。

![LangChain 创建 Pinecone 索引并插入文本]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/LangChain_Create_Pinecone_Index_and_insert_text.jpg)

*图：展示建好的 Pinecone 索引，其索引统计显示 701 个文本向量。*

至此，我们已把书拆分为更小的文档，用 OpenAI embeddings 把每个文档转换为向量，并使用 Pinecone 把这些向量存储到外部。我们可以进入下一步——构建实际的问答组件！

### 4. 向书提问
索引建好后，我们就可以查询这些文档以获得答案。

![向书提问工作流——第 4 步]({{ site.baseurl }}/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow_Ext_Step4.jpg)

*图：展示第 (4) 步向书提问（红色高亮）。*

```python
from langchain.llms import OpenAI
from langchain.chains.question_answering import load_qa_chain

llm = OpenAI(temperature=0, openai_api_key=OPENAI_API_KEY)
chain = load_qa_chain(llm, chain_type="stuff")

query = "How to explain AI to a 5 years old?"
docs = docsearch.similarity_search(query,
  include_metadata=True, namespace=namespace)

chain.run(input_documents=docs, question=query)
```

我们来提问，

> _How to explain AI to a 5 years old?_（如何向 5 岁小孩解释 AI？）

QA 链会取得排序后的结果文档，交给 OpenAI，严格地_仅从书籍内容_进行摘要！最后，我们无需读完整本书，就能得到一个易懂的答案。

```
You can explain AI to a 5-year-old by telling them that AI is like a computer
having a small brain. AI is like enabling a computer to have human-like intelligence,
not in terms of simple repetitive tasks but of human cognitive functions.
For example, AI can help a computer recognize people, objects, animals, or places.
```

针对这本书的更多摘要问题，

> _Who are the AI leaders in the book?_（书中有哪些 AI 领袖？）

```
The book features 60 leaders from a variety of backgrounds and standpoints,
including academics, business leaders, technologists, authors, and researchers.
Some of the AI leaders featured in the book include Professor Andy Pardoe,
Harry Mamangakis, Agnis Stibe, Richard Benjamins, Jair Ribeiro, Jordi Guitart, PhD,
Dr. Sunil Kumar Vuppala, and John Thompson.
```

> _What is the most unusal ideas that can disrupt the society?_（书中最可能颠覆社会的非同寻常的想法是什么？）

```
The idea of digitally modified citizens and intelligently-controlled societies,
enabled by advancements in Artificial Intelligence, could result in a paradigm shift
in our society.
```

### 恢复问答
如果我们中途暂停、稍后返回，不必从头开始访问 Pinecone 索引。我们可以直接连接已有索引来继续。

```python
# use existing Pinecone index
docsearch = Pinecone.from_existing_index(index_name, embeddings, namespace=namespace)
```

到目前为止，我们介绍的只是构建理解一本书的问答系统的初始步骤。还有很多其他步骤可以改进系统、让它更有效。例如，我们可以用更大的问答数据集训练系统以提高准确性。我们还可以针对特定领域（如历史、科学等）微调 OpenAI 模型，以提升其回答该领域相关问题的表现。


## 结语
现代 AI 技术极大地扩展了我们从书籍或文档等海量文本数据中分析并提取有价值洞见的能力。借助 OpenAI、LangChain 和 Pinecone 等工具，我们可以对这些数据进行精细的搜索，快速高效地找到所需信息。此外，通过把数据分块为更小、更易管理的片段，我们可以为 AI 模型提供上下文信息，提高其处理和分析我们所呈现信息的能力。

虽然本文提供了一个成功的问答实验系统，但要构建一个高性能、健壮、能应对广泛用例并提供更佳准确性与推理能力的系统，需要像我们这样经验丰富的系统开发者的专业能力。

凭借我们在这一领域渊博的知识与经验，我们提供量身定制的解决方案，不仅满足、更超越企业客户的期望。所以，如果你想把项目提升到新的水平，请随时[联系我们](btscheung@gmail.com)。记住，提问永远重要且受欢迎。我们在这里帮你构建最好的系统！

## 参考文献


- Data Independent, [LangChain101: Question A 300 Page Book](https://www.youtube.com/watch?v=h0DHDp1FbmQ), video, Feb 2023.
  - 本文深受这个视频的启发。它展示了如何加载一本 PDF 书、拆分为文档、为文档获取向量 embeddings，然后提问。

- Matthew MacFarquhar, [A.I. Wonderland: SparkNotes.ai with Langchain and OpenAI](https://blog.devgenius.io/a-i-wonderland-sparknotes-ai-with-langchain-and-openai-b4d3799d0173), Medium, Mar 2023.
  - 创建一个自动 SparkNotes 机器人：只用提供的文本回答问题，不会编造文本中不存在的内容。它使用了内存中的 Chroma DB 作为索引；这意味着索引不持久化，每次都需要重建。

- Adler Mortimer and Charles Van Doren, [How to Read a Book](https://www.amazon.ca/How-Read-Book-Mortimer-Adler/dp/0671212095), Touchstone, Revised and Updated edition, Aug 1972, isbn: 978-0671212094.

- Big Think, [The art of asking the right questions](https://www.youtube.com/watch?v=tYnGiWlwcj4), video, Sep 2020.
  - 好问题与坏问题的区别不只在难度。在这期视频中，地球生物学家 Hope Jahren、记者 Warren Berger、实验哲学家 Jonathon Keats 和投资人 Tim Ferriss 讨论创造力的力量，以及提出幼稚甚至"愚蠢"问题的价值。

- Online Book, [60 Leaders on Artificial Intelligence](https://www.60leaders.com/_files/ugd/e7ba2f_e965e98b5ab3423f8b131d1a120dc1bf.pdf?index=true), 2022.

### 其他博文

- Benny Cheung, [Deep Learning on Text Data]({{ site.baseurl }}/deep-learning-on-text-data), Benny's Mind Hack, Jul 2018.

- Benny Cheung, [Synthesis of Neural to Symbolic Knowledge for NLP System]({{ site.baseurl }}/synthesis-neural-symbolic-knowledge-nlp), Benny's Mind Hack, Dec 2020.

### AI 工具

- [OpenAI Python Module](https://github.com/openai/openai-python)
- [LangChain Python Module](https://langchain.readthedocs.io/en/latest/)
- [Pinecone Vector Index](https://www.pinecone.io/)
