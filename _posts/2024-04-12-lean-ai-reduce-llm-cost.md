---
layout: post
title: 精益 AI - 如何降低 LLM 成本？
date: 2024-04-12 12:00:00.000000000 -00:00
permalink: lean-ai-reduce-llm-cost
author: Benny Cheung
artwork:
  author: Stable Diffusion
tags:
- Lean AI
- LLM
- Cost
- Optimization
category: post
comments: true
image: images/lean-ai-reduce-llm-cost/cover2x.jpg
images:
  cover: images/lean-ai-reduce-llm-cost/cover.jpg
  header: images/lean-ai-reduce-llm-cost/header.jpg
published: true
---

<!--excerpt.start-->
在生成式 AI（GenAI）蓬勃发展的领域，初创公司如春雨后的野花般涌现。统计数据令人震惊，仅去年一年就出现了名副其实的井喷。但在这片增长之中，一个紧迫的问题挥之不去：我们如何引导这些 GenAI 产品走向商业成功？这就像驾驶一艘船穿越险恶的水域，而精湛的领域知识、创新的理念和有效的成本管理，正是我们设定航向所依靠的星辰。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以把这篇文章当作**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主持人既风趣又富有洞见。

{{ site.baseurl }}/images/lean-ai-reduce-llm-cost/Podcast_Lean_AI.mp3
{% include open-embed.html %}
----

![LeanAI 初创公司气球 2023]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Startups_Balloon_2023.jpg)

_图：2023 年，仅北美一地就拥有超过 7000 家 AI 初创公司，数量惊人。（图片来自 [2023 年 AI 初创公司统计数据](https://contentdetector.ai/articles/ai-startups-statistics)）_

我们无法预测哪些企业能在未来两年存活下来，但可以肯定的是，少数初创公司将通过对关键业务要素的精准整合而完美胜出。在可预见的未来，这些先行者有望实现显著增长，它们的成功建立在独特的产品创新之上——这些创新切实解决了细分市场中的重大挑战——再加上它们管理运营成本、实现盈利结果的能力。

我们常常关注最大、最快、能力最强的 LLM。然而在商业领域，节省成本几乎总是商业领袖们心头最靠前的事。问题在于：我们如何既能收获 AI 的红利，又能让组织维持合理的成本。无论你是希望精简运营的技术领导者，还是对 AI 经济性感兴趣的开发者，本文都非常值得一读。

## LLM 特性的演进

大语言模型（LLM）的演进，是一部关于快速进步、平民化与专业化的故事。[Huggingface 上如今已有 50 万个模型](https://huggingface.co/models)——这是一个共享模型、算法和数据集的开源仓库——整个领域蕴藏着丰富的潜力。然而对商业领袖而言，这种快节奏的环境是一把双刃剑。紧跟新特性并确保技术债不致累积，就好比以短跑选手的速度去跑马拉松。模型的维护与保养，才是长久生存的关键。

商业领袖需要紧跟这些特性，并懂得如何加以利用——技术债会迅速累积——模型的维护与保养才是关键。

- **更长的上下文窗口**：可扩展至超过 100 万 token，用于对大规模数据进行深入分析。
- **更强的推理能力**：提升回答的可靠性与准确性，以适用于更广泛的行业场景。
- **更快的推理速度与更低的延迟**：追求更迅捷的响应，实现更流畅自然的交互。
- **更强的记忆能力**：提升记忆保持能力，实现连贯的长期交互。
- **更强的视觉能力**：改进图像与视频分析工具，提升成本效益。
- **多模态**：加强对多种输入（文本、图像、音频、视频）的处理，支持多样化的应用。
- **更高的个性化程度**：利用单个用户的数据定制回答，实现个性化交互。

这些正在塑造 LLM 未来的特性，只是有望重新定义我们与技术交互方式的众多进步中的一小部分。然而，能力越大，责任越大——需要驯服的成本也越大。

## 成本意识

在了解了 LLM 的未来趋势和我们产品所需的特性之后，选择合适的 LLM 就成了一种平衡的艺术。我们必须在上市时间与准确性和质量之间权衡，在性能与成本和可扩展性之间权衡，在隐私与保护知识产权的需求之间权衡。这是一支步步为营的舞蹈，每一步都必须审慎而精准。

![LLM 特性的平衡表演]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Balancing_Act_of_LLM_Features.jpg)

_图：用三角框架来表示我们必须平衡的关键因素：准确性、性能与隐私。_

让我们用一个三角框架来表示选择合适 LLM 时涉及的关键因素。三角形的中心是**上市时间（Time to Market）**，凸显部署的紧迫性。三角形的每个角代表不同的优先事项：

- **准确性与质量**：以最先进（SoTA）的大模型及其广泛的能力谱系为代表。
- **性能**：以低延迟、成本效益和可扩展性为特征。
- **隐私**：强调保护知识产权、不泄露敏感信息的重要性。

我们明白，LLM 的成本并非固定不变，它像天气一样多变。要在这些流沙中航行，我们必须理解背后的作用力：准确性、性能与隐私。平衡好这三种力量，才能在不妥协的前提下及时交付市场。关键在于知道该拉动哪些杠杆——本文将深入剖析其中一个可控的杠杆：降低成本。

## 成本估算

延续前面预测天气的比喻，LLM 的成本估算同样复杂且牵涉多个层面。从简单的问答交互，到复杂的提示工程以及解决复杂问题的多智能体交互，token 消耗可能会失控式飙升。理解 token 经济学，是迈向经济高效使用 AI 的第一步。

![token 增多导致成本攀升]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Cost_Escalation_with_More_Tokens.jpg)

_图：这张插图强调，随着 AI 交互复杂度的不断提升——从简单对话到精细的多智能体系统——token 消耗会不断攀升，因此需要以战略性方法来管理成本。_

AI 交互中的 token 用量可能大幅增长，这表明要经济高效地使用 AI，就必须理解 token 经济学。

- **问答**：复杂的问题需要更长、更详细的回答，因而消耗更多 token，尤其是在需要追问或澄清的时候。
- **提示工程消耗更多 token**：为了生成特定输出或引导 AI 的回答而编写提示，需要撰写更长、更详细的指令，从而导致更多的 token 消耗。
- **RAG 带来更多上下文**：指检索增强生成（RAG）这类系统，它们会引入额外的 token，以便从外部来源提供上下文，从而推高 token 数量；为保证质量并与用户意图对齐而实施的步骤，也会给流程增加 token。
- **多智能体协作**：描绘了多个 AI 智能体协作解决问题或生成回答的场景，智能体之间的每次交互都会消耗 token，尤其在一个智能体的输出充当另一个智能体输入的系统中，会形成反馈循环。

所有这些因素共同表明，token 用量预计将随着 LLM 能力的提升而扩展。要应对这一趋势，深入理解 token 经济学至关重要。企业和开发者需要在设计与执行中优先考虑效率，以确保不断增长的 token 用量不会导致令人望而却步的成本，从而确保通往更先进 AI 的进程既可持续，又人人可及。

### 分词（Tokenization）——度量单位

分词（Tokenization）是大语言模型（LLM）运行中的一个基础过程：模型把单词转换为 token，本质上是将文本拆分成 AI 便于处理的小块。通常一个 token 大约相当于四个字符，也就是说一个单词平均约等于 1.5 个 token。LLM 的使用量就是按这些 token 来计量的，这直接影响我们使用和支付 AI 服务的方式。大多数 [LLM API 按每百万 token 计价](https://docs.google.com/spreadsheets/d/1NX8ZW9Jnfpy88PC2d6Bwla87JRiv3GTeqwXoB4mKU_s/edit#gid=0)，例如 GPT4 约为 30.00 美元/百万 token，而 Llama2-70b 约为 2.00 美元/百万 token，因此分词成为使用量指标和定价结构中的关键因素。

分词是 LLM 使用量与定价的度量标尺。并非所有分词器都生而平等，它们将单词转换为 token 的效率各不相同。掌握了 token 经济学，我们就打开了通往最具成本效益的 AI 解决方案的大门。

### 成本可观测性

监测这些系统中的成本低效之处，就好比在桅杆顶端安排一名瞭望员：时刻保持警觉，扫视地平线，寻找精简航程的方法，确保我们的旅程不仅开创性十足，而且成本划算。

于是，LangSmith 和 LLMStudio 这类工具登场了。它们就是瞭望员手中的望远镜，让我们更清晰地看到 AI 开支的去向。有了这些工具，我们就不再只是在费用巨浪拍上船头时被动应对，而是能够预见风浪，规划出一条既避免不必要开支、又不牺牲创新速度的航线。

发现 AI 运营中的低效之处，就像在地平线上发现一场风暴。有了早期预警，我们就可以调整船帆，进行必要的航向修正，然后继续前行——不仅审慎，而且精准。实施变革以促成高成本效益的运营，不仅仅是财务管理的需要，更是可持续创新的关键策略。

## 成本优化

在前面的讨论奠定基础之后，让我们来探讨八种成本优化方法：

![8 种成本优化方法]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Cost_Optimization_8_Methods.jpg)

_图：在成本可观测性和成本指标的基础上，我们将展开介绍这 8 种成本优化方法。_

8 种成本优化方法清单如下：

- **微调（Fine-tuning）**：使用大量特定任务的数据对预训练 LLM 进行定制，以提升其在专门任务上的表现。
- **模型级联（Model Cascade）**：采用分层策略，用较简单的模型处理基础任务，并按需升级到更复杂的模型，以提高效率。
- **模型路由（Model Router）**：将任务分配给最适合处理其复杂度的模型，确保查询得到智能路由。
- **输入压缩（Input Compression）**：在输入数据到达 LLM 之前对其进行预处理，以降低其规模和复杂度，节省 token 用量和计算成本。
- **记忆摘要（Memory Summarization）**：通过总结过往交互来优化聊天机器人的对话记忆，高效管理记忆负载。
- **模型量化（Model Quantization）**：降低 LLM 参数的精度以减少计算需求，从而降低成本、加快处理速度。
- **智能体调用缓存（Agent-Call Cache）**：缓存多智能体协作的响应，减少重复计算的需要，从而节省成本。
- **精简组件（Lean Components）**：精简 AI 组件以减少不必要的 token 消耗，优化输入/输出过程。

### 1. 微调——针对专门任务

用一个强大的模型来发布产品，似乎有点大材小用，不是吗？这就像用大锤砸核桃。但这种看似疯狂的做法自有其道理。这个想法是把初始阶段当作一次数据收集练习。你实际上是在撒一张大网，尽可能捕获相关的数据。这些数据——包括用户对哪些做法有效、哪些无效的反馈——将成为下一个关键步骤——微调——的金矿。

![方法 1：微调]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_01_Fine-tuning.jpg)

_图：LLM 微调的过程：使用吉字节到太字节（GB-TB）级的特定任务标注样本（以提示-补全对的形式呈现）来增强预训练 LLM，从而创建一个在特定任务上表现更佳的微调后 LLM。_

微调正是奇迹发生的地方。这个过程听起来很技术化——确实如此——但它的核心，是让某个东西更适合其用途。想象一下，把一个通才——一个多面手——培养成专家。当你用初始阶段收集的数据，对 Mistral 或 Llama 2 这样的小型 LLM 模型进行微调时，就会发生同样的事情。这些小模型在获得特定领域的洞察之后，执行任务的精准度可以媲美原来的巨无霸 GPT-4，而成本只是它的零头。

节省的幅度令人震惊：与从头构建基础模型相比，成本可以降低 98% 以上。然而，这并非放之四海而皆准的方案。微调在专门任务上表现最出色。当你把一个模型调教得把某一件事做得极好时，一旦让它跨出舒适区，它可能就会磕磕绊绊。对于聊天机器人这类需要应对广泛查询的应用，微调后的模型未必总能命中目标。

### 2. 级联模型——为效率而生

以提高效率为目的的级联模型，其概念既简单又巧妙。它就像一连串过滤器，每一层都被设计用来拦截特定类型的问题。第一层过滤器最简单，负责处理最直接的任务。如果有问题通过了它，说明它需要更严格的审查，于是进入下一层稍微复杂的过滤器，以此类推。这一过程持续进行，直到任务找到自己的归属——一个能够驾驭其复杂度的模型。

![方法 2：模型级联]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_02_Models_Cascade.jpg)

_图：LLM 级联架构示意图（图片来自[原始论文](https://openreview.net/pdf?id=6okaSfANzh)）_

举个例子，假设有一个问题需要回答。最初，它被交给一个小模型，也许是 Mistral 这样的轻量级模型。Mistral 试了一下，但如果它力不从心——以 LLM 级联决策器给出的低置信度分数为标志——这个问题不会被直接丢弃，而是沿着链条向上传递给更强大的模型。这种逐级上升会一直持续，直到我们找到那个“刚刚好”的黄金地带：不过于简单，也不过于复杂，恰好适合眼前的问题。

这种方法的亮点在于成本效益。众所周知，算力是有价的。从较便宜的模型开始，只在必要时才请出重量级选手，我们就能优化资源使用。我们不是在用大锤砸核桃，而是在每一步都使用恰到好处的力量。在一个效率至上的世界里，级联模型提供了一种在成本与计算需求之间取得平衡的聪明做法。

### 3. 模型路由——为任务分发而生

用于任务分发的 LLM 路由，是人工智能领域一项引人入胜的发展。其核心是让一个大语言模型（LLM）充当守门人，分析传入的查询，判断它们可以由较简单的模型处理，还是必须借助更先进的模型。Hugging Face 对这一理念的实现——由一个 LLM 充当控制器，将用户查询拆解为更小、更易管理的子任务，再把它们分配给专门化模型——正是这一创新策略付诸实践的典范。

![方法 3：模型路由]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_03_Models_Router.jpg)

_图：想象一个由大语言模型（LLM）驱动、通过语义路由优化的数字旅行助手（图片来自 [semantic router 库](https://github.com/aurelio-labs/semantic-router/tree/main)）_

采用 LLM 路由进行任务分发的优势十分显著。通过将请求智能地路由到最适合特定任务的模型，这种方法不仅有望提升查询处理的效率，还能优化计算资源的使用。这可以带来更快的响应时间和更准确的答案，改善整体用户体验。此外，它还让我们得以更明智地使用强大的模型，把它们的能力留给真正需要它们的任务，而较简单的请求则由较小的模型高效处理。这种分层解决问题的思路，可能为更具可扩展性和可持续性的 AI 系统铺平道路，让先进的 AI 在更广泛的应用场景中变得触手可及、行之有效。

下面来看看这样一个 AI 旅行助手的一天可能是如何展开的：

用户以一句“我想计划一次希腊度假”开启对话。语义路由系统会迅速分析查询的意图，并将其路由到相应的响应路径。如果用户的请求不符合服务准则，系统会因提示被拒（用于品牌保护）而迅速给出默认回复。但在本例中，该查询是有效的，被路由到“目的地（destination）”路径。

随着对话继续，AI 同时利用 LLM 和检索增强生成（RAG），从庞大的信息数据库中提取内容，提供一份精心整理的希腊必游景点清单。当用户询问“飞往雅典的航班”时，系统将请求路由到“航班（flights）”路径。最后，当用户问起“雅典的热门景点”时，查询则沿着“景点（attraction）”路径发送。

### 4. 输入压缩——处理输入数据

当微软推出 LLMLingua 时，它解决了一个许多人尚未清晰表述的问题：把未经处理的原始数据直接喂给复杂模型是一种低效做法。这就像让人在干草堆里找针，却不先用磁铁把金属碎屑吸走。LLMLingua 就像是数据的磁铁，剔除不必要的部分，只留下精华供更精密的模型检视。

![方法 4：输入压缩]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_04_Input_Compression.jpg)

_图：展示 LLMLingua 如何处理原始文本、提取重要（高亮）内容，生成压缩后的文本，然后再把请求发送给 LLM。（图片来自微软 LLMLingua-2：[https://llmlingua.com/llmlingua2.html](https://llmlingua.com/llmlingua2.html)）_

这种方法的意义不仅在于效率，它更是我们思考信息处理方式的一次根本转变。通过减少 token 数量，LLMLingua 不仅节省了计算成本，还让我们的关注点更加聚焦。它迫使我们思考：在请最强大的工具分析数据之前，数据中真正重要的到底是什么。

看看下面的 Python 示例代码，`llmlingua` 的使用非常简单。

```python
from llmlingua import PromptCompressor

llm_lingua = PromptCompressor()
compressed_prompt = llm_lingua.compress_prompt(prompt, instruction="", question="", target_token=200)
```

在内容摘要或从庞大数据集中提取特定数据点等应用中，LLMLingua 证明了自己的价值。它提醒我们：在追求更强大解决方案的过程中，简化输入有时同样是一种创新。微软在这一开发中的实践，是一堂关于效率之美的课——通往高级分析的道路并不总在于增加复杂性，而往往在于巧妙地减少复杂性。

### 5. 记忆优化——针对对话

当我们与人交谈时，并不会记住对方说过的每一个字。我们的大脑会总结过去的对话，记住要点，放下细枝末节。聊天机器人的开发者正试图模仿这一自然过程，以优化对话智能体的记忆使用。

![方法 5：记忆优化]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_05_Memory_Optimization.jpg)

_图：展示多种内置的对话缓冲记忆（Conversation Buffer Memory）方法，以图表呈现内存消耗随交互次数的变化。（图片来自 Jame Briggs 的 [Conversational Memory for LLMs with Langchain](https://www.pinecone.io/learn/series/langchain/langchain-conversational-memory/)，Pinecone 博客）。_

挑战很直接：我们如何管理对话记忆，让它不必复述过往交互的每一个细节，却仍能生成相关且连贯的回答？两种策略已展现出良好的前景：对话摘要记忆（Conversation Summary Memory）和摘要缓冲记忆（Summary Buffer Memory）。

对话摘要记忆就像拥有一个只记得你所说内容要点的朋友。它把过去的对话压缩成摘要，大幅减少聊天机器人需要保存的数据量（或 token 数量）。这种方式类似于我们向别人介绍一本读过的书：只讲主要情节，而不复述每个场景。

摘要缓冲记忆则采用略微不同的方式。它就像一台相机，把近期事件拍得纤毫毕现，而把较早的记忆存成褪色的老照片。这种方法让最近的对话保持完整细节，同时逐步对较早的交互进行摘要。这是在保留有用信息与高效管理记忆之间的一种平衡。

两种策略都有一个共同的目标：在不损害交互质量的前提下，减轻聊天机器人的记忆负载。采用这些方法后，开发者打造出的聊天机器人不仅省钱，而且在展开有意义的对话时表现得极其类人。正如我们不需要记住对话中的每个字也能理解并恰当地回应一样，聊天机器人如今也能做到这一点，这标志着它们在成为更亲切、更高效的沟通者方面迈出了重要一步。

### 6. 模型量化——降低 GPU 成本

模型量化通过降低精度来缩小模型体积、提升效率。在 LLM 的发展历程中，这个领域常被视为拥有最强大、最昂贵硬件者的游乐场。这些模型的计算需求不仅消耗大量能源，还带来高昂成本，限制了它们的可及性。然而，一种被称为量化（Quantization）的变革性方法正在改变这一格局。

![方法 6：模型量化]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_06_Model_Quantization.jpg)

量化通过降低 LLM 所使用数值的精度来简化其复杂度。想象一下，你在作画时决定改用一块更小的调色板。出人意料的是，画作的神韵犹存，而你用的颜料更少、耗时更短。同样，量化能在基本不牺牲性能的前提下，显著缩小 LLM 的体积并提高其效率。

体积的缩减和速度的提升，带来更快的推理时间和更低的能耗。这对更广泛的社区意味着什么？它让先进 AI 技术的获取变得民主化。硬件不那么强大的小公司和个人，如今也可以实验并部署那些以前遥不可及的复杂 AI 模型。

### 7. 智能体调用缓存——降低成本

以降低成本为目标的多智能体系统概念，宛如一盏照亮高效与务实之路的明灯。想象这样一个场景：任务不是由单个全能模型完成，而是由一支各有专长、成本各异的智能体团队来承担。这并非遥远的未来，而是今天已在落地的实用方案。

![方法 7：智能体调用缓存]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_07_Agent-call_Cache.png)

_图：EcoAssistant：该系统包含两个智能体，一个是执行代码的执行器智能体，另一个是由 LLM 支持的助手智能体，负责建议代码以获取信息并回应用户查询。查询-代码数据库保存以往成功的查询与代码对。当新查询到来时，系统会检索数据库中最相似的查询，并在初始提示中连同相关代码一并示范。对话会首先调用最具成本效益的助手，只有当当前的助手失败时，才尝试助手层级中更昂贵的那个。（图片来自 [Autogen Ecoassistant](https://microsoft.github.io/autogen/blog/2023/11/09/EcoAssistant/)）_

这一方法的核心是一条简单而深刻的策略：先从小处着手，再逐级升级。任务最初分配给更经济的模型。只有当这些模型力不从心时，我们才转向更昂贵的同类。这种分层方式确保大多数任务由成本较低的智能体高效完成，把重量级选手留给真正棘手的问题。

以 EcoAssistant 框架为例，它是这一理念付诸实践的典范。它致力于在不烧钱的前提下，根据外部查询生成准确的代码。通过采用语言模型助手的层级结构，EcoAssistant 同时优化了成本与效果。先由较便宜的模型尝试，仅在需要时才升级到更精密的模型。此外，通过保存成功的输出结果，系统得以自我改进，让较便宜的模型随着时间的推移变得愈发能干。

### 8. 精简组件——优化输入/输出 token

当我们思考如何针对输入/输出 token 优化工具时，本质上谈论的是一种效率。它意味着与 LLM 通信时，我们不是把所有东西一股脑扔过去、看看什么能奏效，而是对发送和接收的信息进行深思熟虑。

![方法 8：精简组件]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Method_08_Lean_Components.png)

设想一个智能体组织结构，分为两个主要团队：数据分析团队和数据可视化（Data Viz）团队，后者被标注为“已解雇（FIRED）”。数据可视化团队之所以被裁撤，是因为其效率低下——尤其是在调用 LLM 执行任务时消耗了过多的 token。它们在生成 JSON、YAML 或 TEXT 输出方面所扮演的角色被评估为不合算，于是决定用更简单的、不依赖 LLM 的方案来取代其职能。

这种方法的好处有两个方面。第一，它减轻了 LLM 的负载。这些模型虽然强大，但也非常消耗资源。通过精简组件，我们不仅节省了计算资源，还可能降低成本。第二，或许也是更重要的一点，这种方法让系统聚焦于真正适合 LLM 的相关用例，从而提升系统的质量。

归根结底，针对输入/输出 token 优化工具，就是要聪明地使用资源。它提醒我们：数据并非越多越好。有时，与 LLM 有效沟通的关键在于简洁与相关性。

### 方法采纳

这一战略模型勾勒出为 AI 服务设计的八种成本优化方法，并依据其采纳成熟度及对产品的潜在影响，将它们归入不同的颜色编码类别：“防守（Defend）”“差异化（Differentiate）”和“颠覆（Upend）”。

![8 种成本优化方法的采纳]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Cost_Optimization_8_Methods_Adoption.jpg)

属于“防守（Defend）”类别的方法必不可少且相对容易实施，包括：

- 模型级联：通过模型分层来高效过滤和升级任务，减少不必要的处理。
- LLM 路由：将查询动态路由到最合适的模型，高效应对不同的需求。
- 输入压缩：减小输入数据的规模，以节省处理时间和成本。
- 记忆摘要：将信息浓缩为紧凑的表示形式，最大限度降低存储与检索成本。

“差异化（Differentiate）”类方法更具挑战性，但能显著增强我们的产品：

- 微调：针对特定任务定制模型，在不带来过多计算开销的情况下提升性能。
- 模型量化：降低模型参数的精度，以减小模型体积、提升推理速度，而准确性不会有明显损失。

最后是“颠覆（Upend）”类别，这些方法具有前瞻性，如果我们采用智能体式（Agentic）AI 模型，它们有可能给产品带来革命性变化：

- 智能体调用缓存：缓存以往的交互，加快响应速度并减少重复计算。
- 精简组件：将 AI 组件精简至必不可少的部分，从而大幅降低复杂度和成本。

这些方法各自代表 AI 优化之旅的不同阶段：从立即可行、可付诸行动的“防守”，到具有创新性乃至可能颠覆行业的“颠覆”。

## GenAI 服务平台

我们提出的 GenAI 服务平台旨在提供高效且智能的解决方案。这套架构的核心是内置模型级联的 LLM 路由器——一个动态枢纽，将传入的用户查询导向合适的资源。这种路由机制对保持高性能至关重要：它确保回答不仅准确，而且能及时送达，同时把运营成本降到最低。

![一体化服务平台]({{ site.baseurl }}/images/lean-ai-reduce-llm-cost/LeanAI_Integrated_Service_Platform.jpg)

_图：展示所提出的 GenAI 服务平台，它将多种成本优化方法整合为一个统一的解决方案。_

为了在不牺牲服务智能水平的前提下实现成本效益，该平台集成了多种成本优化方法。它能够处理跨领域的多样化用户咨询，展现出应对广泛主题所需的灵活性与可扩展性。通过维护一个领域专属模型的仓库，平台可以为给定的查询选择最合适的 LLM，确保回答具备高度的准确性和相关性。

此外，与检索增强生成（RAG）系统和向量数据库（Vector DB）的集成至关重要。这些组件协同工作，优化资源利用。RAG 系统利用外部知识库来支持 LLM 生成更有依据的回答，而 Vector DB 则高效地存储和检索信息，有助于降低计算开销。

这一设计凸显了我们的目标：不仅要提供智能的回答，还要以可持续的方式来实现。通过只在必要时启用专门化模型，并借助 RAG 系统提供支持，该架构将资源消耗——进而将成本——降到最低。这种方式确保 AI 服务在性能上始终一流，在长期运行中保持可持续，与我们追求高性价比智能的目标相一致。

## 结语

当我们谈论 LLM 的成本优化时，实际上谈论的是一系列权衡。这些优化手段——微调、级联和路由——的核心，是在让模型更高效与保持其有效性之间求得平衡。

微调是调整 LLM，使其更好地执行特定任务。它就像磨刀：这个过程让工具更适合手头的工作。级联则是让问题经过一系列越来越专门化的模型进行过滤。想象一连串筛子，一个比一个细密，确保只有最相关的颗粒——或者说数据——能够通过。路由则根据查询内容将其导向最合适的处理路径，就像一座井然有序的图书馆，每本书都有自己的位置，确保快速而准确的检索。

还有模型量化的问题——通过降低模型计算的精度来节省计算资源。这有点像压缩文件以节省硬盘空间：你希望质量的损失不那么明显，但权衡总是存在的。

随着我们对这些成本优化手段理解的加深，也会碰到一些现实问题，比如节省成本的措施，以及始终萦绕的隐私与安全。优化一个 LLM 是一回事，以不损害用户数据的方式进行优化则是另一回事。

> 你可以为思想标上价格。有的昂贵，有的便宜。
>    而一个人该如何为思想付费？
>   我认为，答案是：用勇气。
>
> ——路德维希·维特根斯坦（Ludwig Wittgenstein），《文化与价值》（Culture and Value）

这段 LLM 优化之旅不仅关乎技术功力，更关乎敢于实验、敢于突破边界。路德维希·维特根斯坦曾思索思想的价值。在 LLM 的领域里，显而易见，正是我们创新的意愿和冒险的精神推动着进步。当我们在这片复杂的地形中穿行时，最终决定成败的，是我们探索新路径的勇气。

## 参考文献

### 资源

- Martin Keen, [The most important AI trends in 2024](https://www.youtube.com/watch?v=sGZ6AlAnULc&ab_channel=IBMTechnology), IBM Technology, video, 6 Mar 2024.
- Lingjiao Chen, Matei Zaharia, James Zou, [FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance](https://arxiv.org/pdf/2305.05176.pdf), [arXiv:2305.05176](https://arxiv.org/abs/2305.05176) [cs.LG], 9 May 2023.
- Synthetic Intelligence Forum, [FrugalGPT: Reduce Cost and Improve Performance of Large Language Models](https://www.youtube.com/watch?v=pH7I32-VrZ0), James Zou, 5 Jun 2023.
- TensorOps, [Analyzing the Costs of Large Language Models in Production](https://www.youtube.com/watch?v=WmF0hhf3hBE&ab_channel=TensorOps), video, 6 Dec 2023.
- Jame Briggs, [Conversational Memory for LLMs with Langchain](https://www.pinecone.io/learn/series/langchain/langchain-conversational-memory/), Pinecone blog.
- Greg Kamradt, [Workaround OpenAI's Token Limit With Chain Types](https://www.youtube.com/watch?v=f9_BWhCI4Zo), Data Independence video, 1 Mar 2023.
- Dylan Patel and Afzal Ahmad, [The Inference Cost Of Search Disruption – Large Language Model Cost Analysis](https://www.semianalysis.com/p/the-inference-cost-of-search-disruption), semianalysis Blog, 9 Feb 2023.
- Jieyu Zhang, et al., [EcoAssistant: Using LLM Assistant More Affordably and Accurately](https://arxiv.org/pdf/2310.03046.pdf), [arXiv:2310.03046v1](https://arxiv.org/abs/2310.03046v1) [cs.SE], 3 Oct 2023.

### 工具

- Tokenizer Playground: [https://huggingface.co/spaces/Xenova/the-tokenizer-playground](https://huggingface.co/spaces/Xenova/the-tokenizer-playground)
- Microsoft LLMLingua-2: [https://llmlingua.com/llmlingua2.html](https://llmlingua.com/llmlingua2.html)
- Autogen Ecoassistant: [https://microsoft.github.io/autogen/blog/2023/11/09/EcoAssistant/](https://microsoft.github.io/autogen/blog/2023/11/09/EcoAssistant/)
- LangSmith: [https://www.langchain.com/langsmith](https://www.langchain.com/langsmith)
- Arize Phoenix: https://phoenix.arize.com/ 
