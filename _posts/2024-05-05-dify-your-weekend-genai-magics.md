---
layout: post
title: Dify - 你的周末 GenAI 魔法
date: 2024-05-05 12:00:00.000000000 -00:00
permalink: dify-your-weekend-genai-magics
author: Benny Cheung
artwork:
  author: Clara Cheung
tags:
- Generative AI
- LLM
- Education
- AI Platform
category: post
comments: true
image: images/dify-your-weekend-genai-magics/cover2x.jpg
images:
  cover: images/dify-your-weekend-genai-magics/cover.jpg
  header: images/dify-your-weekend-genai-magics/header.jpg
published: true
---

<!--excerpt.start-->
谈到学习，尤其是在 AI 这样复杂的领域，以及在蓬勃发展的生成式 AI 竞技场中，人们天然倾向于深入钻研理论。书籍、论文、讲座——这些是获取知识的传统工具。毫无疑问，它们很有价值，但它们只代表了硬币的一面。真正的魔力发生在我们把理论知识应用到实际项目中的那一刻。正是在这里，抽象变得具体，想法转化为你可以看到、触摸并与之互动的东西。
<!--excerpt.end-->

生成式 AI，尤其是像 ChatGPT 这样的工具，已经彻底改变了这一过程。传统上，在 AI 领域从概念走向执行的旅程令人生畏，可能长达数月甚至数年，成为创新和 AI 技术更广泛普及的重大障碍。但现在，随着大语言模型（LLM）的到来，我们正目睹一场戏剧性的转变。

----
> [2024/10/04] 我们可以把这篇文章当作**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主持人既风趣又富有洞见。

{{ site.baseurl }}/images/dify-your-weekend-genai-magics/Podcast_Dify_GenAI_Magic.mp3
{% include open-embed.html %}
----

![教育中的知识鸿沟]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/knowledge-gap.jpg)

_图：教育中的知识鸿沟（图片来自 [qorrectassess.com](https://qorrectassess.com/en/blog/knowledge-gaps-in-education/)）_

ChatGPT 和类似的工具提供了一种将想法变为现实的新范式。通过所谓的提示工程（Prompt Engineering），我们现在可以用自然语言与这些复杂的 AI 模型交互，用平实的语言描述我们想要什么，模型便会以结构化、可重复的方式生成或补全我们的想法。这不仅仅是一点小改进，而是我们进行 AI 开发方式的一次根本变革。

然而，一个重要的障碍依然存在。把这些精心打磨的提示转化为可行的产品并非易事。通常，单个提示不足以完成一项任务，它需要一系列相互配合的提示。这就需要额外的工具和平台，才能把设想中的产品变为现实。

幸运的是，这一挑战并非个例，已经得到许多人的重视。经过 [腾讯云 CODING DevOps 团队](https://dify.ai/blog/open-source-llmops-platform-define-your-ai-native-apps)成员多年的研究与实践开发，一个有望有效弥合这一鸿沟的解决方案问世了。在众多旨在促进生成式 AI 从概念走向产品的平台和工具之中，Dify 是一个尤为出众的创新方案。

> 注意：Dify 的实用性和人气有目共睹——上线仅 36 小时，就有超过 1500 个应用基于这个开源项目被创建，这一里程碑令人印象深刻。

[Dify](https://dify.ai/)（Do It For Yourselves，为你自己而做）正在开辟一片独特的开发天地。它不仅仅是庞大数字工具箱里的又一个工具；Dify 是一个变革性的生态系统，服务于广泛的用户群体——从初涉 AI 的新手，到经验丰富的开发者，再到大型企业。这个平台以其多功能性和健壮性著称，提供覆盖教育、原型设计、智能体开发和生产部署的解决方案。让我们来了解 Dify 如何在这些领域产生重大影响。

![Dify 作为面向多个方面的平台]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_as_a_platform_for_many_aspects.jpg)

_图：Dify 作为面向多个方面的平台：教育、概念验证、智能体产品和产品部署。_

## Dify 作为教育平台：动手学习生成式 AI 技术与实现技巧

生成式 AI 技术尚处于起步阶段，呈现出一片广阔而未知的图景。乍看之下，其复杂性可能令人望而生畏。然而，理解并运用这些技术的要义，不在于一夜之间征服陡峭的学习曲线，而在于踏上一段循序渐进的发现之旅。

从根本上说，Dify 是通往生成式 AI 技术世界的一扇教育之门。Dify 通过提供可视化学习工具来填补教育的空白。Dify 的独到之处在于其可视化 AI 应用编排工作室，即所谓的低代码/无代码（Low Code/No Code）方式，它让用户能够以可视化方式把 AI 组件拼装成可运行的应用。这大幅降低了缺乏深厚编程功底者的入门门槛，使其成为一流的教育工具。平台的检索增强生成（RAG）管道进一步丰富了学习体验：它为 AI 应用配备了庞大的向量数据库，使其能够产出准确且贴合上下文的输出。这一特性生动展示了 AI 如何利用海量资源来生成有理有据的回答，让人们对先进 AI 功能的实际运作有了直观认识。

### 5 分钟构建 RAG

下面通过 Dify 的工作流组件展示一个简单的 RAG 管道，

![Dify 工作流知识检索聊天机器人]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Workflow_Knowledge_Retrieval_Chatbot.jpg)

_图：使用 Dify 的知识检索 + 聊天机器人模板，展示一个可在 5 分钟内实现的 RAG 管道。_

我们可以从 Dify 的知识检索 + 聊天机器人模板入手。让我们借助 Dify 的组件来探究这个工作流模板的运作方式，重点关注如图所示的最简形式的 RAG：

1. **开始（Start）**：
    - 这是工作流的初始触发点。它代表 AI 应用开始运行的过程起点，通常由用户查询或自动化事件触发。
2. **知识检索（Knowledge Retrieval）**：
    - 这一组件是 RAG 架构的关键。它允许系统查询庞大的数据库或知识库，检索与用户问题相关的文本内容。这一步骤的目的是收集所有可能有助于生成有据回答的信息，确保 AI 能获取超越其预训练模型知识的更广泛数据。
3. **LLM**：
    - 在这里，工作流调用一个强大的语言模型，在本例中是 `gpt-3.5-turbo`。Dify 在设置中提供了许多其他 LLM 的连接选项。该模型利用上一步检索到的信息来理解上下文并生成恰当的回答。模型处理自然语言输入并综合信息，形成的回答不仅切题，还因检索到的知识而在上下文上更加充实。
4. **回答（Answer）**：
    - 工作流的最终输出在这一步生成。回答模块接收 LLM 的响应，并以文本形式交付给最终用户。它可能是对用户查询的直接回答、一项任务的完成结果，或应用设计提供的任何其他类型的响应。

![Dify 模型提供商设置]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Settings_on_Model_Providers.jpg)

_图：Dify 在设置中提供了许多其他 LLM 的连接选项。通常每个模型都需要一个 API 密钥才能访问该服务。_

### 借助 RAG 增强学习体验：

Dify 工作流如何在教育或学习场景中发挥作用？这条 RAG 管道有助于更深入地理解 AI 如何动态利用庞大的数据库来提升回答的相关性和准确性。这一架构展示了先进 AI 功能的实际运作，例如：

- **动态数据检索：**按需从庞大存储库中提取数据的能力，增强了 AI 的回答质量，使其更贴合当前上下文或特定用户查询。
- **上下文理解与回答生成：**借助最先进的 LLM，系统能够理解细微差别，生成不仅事实准确、而且符合上下文的回答。
- **实时学习与适应：**工作流展示了 AI 如何有效地利用检索到的信息，根据新信息或不断演变的上下文调整其回答。

### Dify 提示 IDE

此外，Dify 的提示 IDE（Prompt IDE）作为掌握提示工程的工具大放异彩——提示工程是塑造 AI 回应的一项关键技能。在这个环境中，用户可以试验各种提示，观察调整带来的即时效果，并领会引导 AI 实现特定结果的种种微妙之处。

![Dify 工作流 LLM 组件提示工程示例]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Workflow_LLM_Component_Prompt_Engineering_Example.jpg)

_图：展开 LLM 工作流组件后，我们可以交互式地雕琢提示词，以总结检索到的上下文。_

这张图片展示了 Dify 工作流中 LLM 组件的详细视图，重点是一个 LLM 的配置，用于处理自然语言查询并作出回应。对于理解如何利用 Dify 的提示 IDE 来提升提示工程技巧，这个示例尤其富有启发。以下是对图中所示特性与功能的分解和阐释：

#### LLM 组件概览

- **模型选择：**
  - 该组件设置为使用 `gpt-3.5-turbo`，这是 OpenAI 生成式预训练转换器的一个强大版本。我们也可以选择在设置中配置过的其他模型提供商。
- **调用方式：**
  - 该模型以聊天方式调用，这意味着它针对对话式使用做了优化，处理输入并生成适合对话形式的回应。

#### 配置细节

- **上下文变量：**
   - “设置变量（Set variable）”功能允许工作流存储并传递上下文信息，供 LLM 在生成回应时使用。这对于在对话的多次交互之间保持上下文至关重要。
- **系统指令：**
   - 系统配置中提供的详细提示指令，指导 LLM 应如何处理这些信息：
    - 其中提到使用封装在 `<context></context>` XML 标签中的已学习知识。模型可以引用动态提供的上下文来增强其回应。
    - 指令规定了诸如承认知识不足或请求澄清之类的行为，这些对于打造真实可靠的 AI 交互必不可少。
    - 指令还建议模型避免提及知识的来源（即上下文），使交互更加流畅自然、更像真人。

#### 使用提示 IDE

Dify 的提示 IDE 允许用户交互式地编写和打磨输入 LLM 的提示。这一工具对掌握提示工程至关重要，它让用户能够：

- **试验不同的提示：**用户可以测试不同的提问或提示表述方式，观察它们如何影响 AI 的回应。这对于优化 AI 以处理特定类型的查询，或让它按不同用户需求执行任务至关重要。
- **观察即时效果：**对提示所做的任何更改都可以立即测试，观察它们如何改变 AI 的输出。这种即时反馈对于理解和改进用户与 AI 之间的交互动态极为宝贵。
- **理解微妙之处：**通过调整提示并观察结果，用户可以领会某些措辞或上下文线索如何显著改变 AI 回应的性质。

这一环境不仅帮助开发者构建更精细的 AI 驱动应用，也为 AI 新手提供了一个动手学习工具，帮助他们理解语言细微差别和上下文在 AI 交互中的影响。通过这些详细的配置和工具，Dify 增强了用户开发先进、具备上下文感知能力的 AI 系统的实力，使这些系统在真实场景中健壮、响应迅速且功能强大。

## Dify 作为 PoC 平台：快速原型验证与创意实验

对于渴望把想法变为现实的创新者和开发者，Dify 为快速原型验证和实验提供了宝贵资源。平台将内置工具（Built-in Tools）与基于 API 的工具（API-Based Tools）相结合，简化了开发流程，支持快速的迭代与完善。这种支持性的环境鼓励创造性实验，帮助概念快速转化为实实在在的原型。

![Dify 面向广泛应用的工作区模板]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Workspace_Template_for_Wide_Range_of_Apps.jpg)

_图：Dify 自带丰富的工作区模板，有助于快速启动一个项目。_

这样的环境非常重要，它支持对“不同组件在 AI 应用中如何协同工作”的假设进行测试和验证。用户可以轻松地组装、拆卸和重新配置应用元素，这鼓励了创造性实验，让想法得以从概念快速演进为原型。

### 创建 Dify 应用——用户指南

Dify 提供了一系列教程，指导用户创建流行的生成式 AI 应用。这些教程旨在把 LLM 的能力运用到实用且易用的应用中。

- [智能体助手（Agent Assistant）](https://docs.dify.ai/user-guide/creating-dify-apps/prompt-engineering/agent-assistant)
  - 这项工具利用 LLM 的推理能力，自主设定目标、拆解复杂任务、驾驭各类工具并优化流程，从而独立完成任务。这是让 AI 成为问题解决的积极参与者的一次飞跃。
- [聊天应用（Chat App）](https://docs.dify.ai/user-guide/creating-dify-apps/prompt-engineering/conversation-application)
  - 该应用支持与用户进行流畅的对话，以简单直接的问答方式运作。它旨在让用户保持持续对话，使与 AI 的交互更加自然直观。
- [文本生成器（Text Generator）](https://docs.dify.ai/user-guide/creating-dify-apps/prompt-engineering/text-generation-application)
  - 生成器应用的突出之处在于，它能根据用户提示生成各种格式的高质量文本，从文章摘要到翻译不一而足。该应用展示了 AI 在创意和内容生成任务上的潜力，让人得以窥见自动化写作的未来。

通过这些教程，Dify 不仅揭开了创建生成式 AI 应用的神秘面纱，还为用户探索和创新 AI 技术打开了一个充满可能性的世界。

## Dify 作为智能体平台：学习智能体组件构建与提示工程技巧

作为一个智能体平台，Dify 走在教授用户构建智能体组件——能够自主执行任务的自运行单元——的最前沿。平台的智能体助手（Agent Assistant）功能借助 LLM 的力量独立完成复杂的任务，从进行细致的分析到撰写完整的报告。

![Dify 助手智能体 UI]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Assistant_Agent_UI.jpg)

_图：一个完全在 Dify Studio 中通过 UI 构建的助手智能体应用示例，唯一的前提条件是有一个想法和对 GenAI 的高层次理解，无需任何编程经验。_

Dify 的智能体界面专门为创建助手智能体而量身打造，例如投资分析副驾（Investment Analysis Copilot），这是一个专门为股票提供专家级数据分析的 AI 智能体。以下是对界面中所示各组件和功能的详细分解：

- **指令（Instructions）部分：**
  - 在这里定义智能体的详细职位描述和角色设定。该智能体被描述为一个数据分析副驾，擅长基本面分析、技术分析和市场情绪分析等多个分析领域。
  - 技能被清晰地定义，这有助于构建智能体的能力结构。例如，使用股票代码（Ticker）从 Yahoo Finance 搜索股票信息，或搜索目标公司的最新新闻。
- **变量（Variables）：**
  - 在这里定义诸如 'company' 之类的变量，用于捕获用户输入，并基于动态输入影响智能体的行为和回应。
- **上下文（Context）：**
  - 这一部分暗示了可以导入知识作为上下文的能力，为 AI 的分析和回应提供额外信息，从而提升其输出的准确性和相关性。
- **工具（Tools）：**
  - 图中展示了 Yahoo 的 "Analytics"、"News" 和 "Ticker" 等工具，表明智能体可以借助这些内置能力来执行任务。这些工具旨在连接数据源和 API，以获取所需信息。
- **调试与预览窗格（Debug and Preview Pane）：**
  - 右侧这一区域显示了最终用户将看到的用户界面预览。它提供了潜在用户与 AI 的示例交互，例如分析特定股票、询问某家公司的最新动态，或进行基本面分析。这对于测试和打磨智能体与用户的交互方式至关重要。

### 智能体工具及其集成

在 Dify 项目中，智能体和工作流使用的工具被清晰地划分为两类。第一类称为内置工具（Built-in Tools），直接内嵌于产品核心，经过预编码，无需任何额外调整即可使用。第二类是基于 API 的工具（API-Based Tools），则建立在第三方 API 的肩膀之上。

- **内置工具（Built-in Tools）：**它们是平台不可分割的组成部分，经预编码用于执行特定功能，如拉取股票数据或最新新闻，无需额外编码即可部署。
- **基于 API 的工具（API-Based Tools）：**这类工具旨在与外部服务集成，可以通过 API 进行定制或扩展。它们让智能体能够执行更复杂、更多样的任务，增强其实用性和适应性。

这些工具的设计以易用为先，用户无需直接编码。集成就像通过 OpenAPI、Swagger 乃至 OpenAI 插件等熟悉的格式提供接口定义一样简单。这种二分法不仅简化了用户体验，也拓宽了 Dify 生态系统中可用的定制范围和功能。

## Dify 作为生产平台：部署与监控 AI 产品

当我们着手探讨部署生成式 AI 应用的话题时，其复杂性可能令人望而生畏，尤其是对这一领域的新手而言。本节将深入探讨这些错综复杂的问题，但它是为已经牢牢掌握该主题的读者量身定制的。新手不妨暂时跳过这一节。

在部署与监控的生产要求方面，Dify 通过其 LLMOps 能力提供全面的解决方案。平台的后端即服务（Backend-as-a-Service，BaaS）产品简化了把 AI 功能集成到各类产品中的过程，助力从开发阶段到全面部署的平稳过渡。

### 快速开始

我们可以从 Dify 的 github 仓库克隆项目。
> [Dify.ai](https://dify.ai/) - github 仓库：- langgenius/ [dify](https://github.com/langgenius/dify)

克隆仓库之后，启动 Dify 服务器最简单的方法是运行我们的 [docker-compose.yml](https://github.com/langgenius/dify/blob/main/docker/docker-compose.yaml) 文件。在运行安装命令之前，请确保你的机器上已安装 [Docker](https://docs.docker.com/get-docker/) 和 [Docker Compose](https://docs.docker.com/compose/install/)：

```shell
cd dify/docker
docker compose up -d
```

开源最美妙的一点是，我们可以深入其内部，理解它的运作方式，而不必依赖产品公司发布的碎片化信息。凭借健康的社区，功能在持续增长。如果我们具备开发能力，没有什么能阻止我们用自己的代码扩展这个平台。

### Docker 部署架构

系统采用 Docker 部署，涉及多个服务和数据卷，并展示它们之间的关系。这是现代 Web 应用的典型健壮配置，对这类应用而言，关注点分离和数据持久化至关重要。使用 Docker 和 Docker Compose 有助于以隔离而又互联的方式管理这些服务，使开发、部署和扩展更加高效。

![Dify docker compose]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_docker_compose.jpg)

_图：Dify docker compose 部署架构（图片由 Dify.ai 提供）。_

- **Nginx**：充当反向代理和 Web 服务器。图中展示了配置文件：
    - `nginx/nginx.conf`：主配置文件。
    - `/etc/nginx/nginx.conf`：挂载的卷，用于跨容器持久化或共享配置。
    - `nginx/conf.d`：存放额外配置文件的目录。
- **Weaviate**：向量搜索引擎，配置了用于数据存储的卷：
    - `/volumes/weaviate`：Weaviate 的数据存储。
    - `/var/lib/weaviate`：Weaviate 容器内部的数据挂载点。
- **PostgreSQL（`db`）**：数据库服务及其数据卷：
    - `/volumes/db/data`：PostgreSQL 的数据存储。
    - `/var/lib/postgresql/data`：PostgreSQL 容器内部的挂载点。
- **Redis**：内存数据结构存储，用作数据库、缓存和消息代理：
    - `/volumes/redis/data`：Redis 的数据存储。
- **API（`api`）**：使用 PostgreSQL 和 Redis 的后端 API 服务：
    - `/app/api/storage`：用于存储 API 相关数据的卷。
- **Worker（`worker`）**：与 API 和数据库（PostgreSQL 和 Redis）都有交互的后台工作进程。
- **Web（`web`）**：与 API 通信的前端 Web 服务。

#### 卷映射：
- Nginx、PostgreSQL、Redis 和 Weaviate 的配置文件与数据存储在 Docker 卷中，这些卷是宿主机上挂载进容器的目录。这种设置确保数据在容器重启后依然持久保留，并可在不同容器之间共享配置。

#### 服务交互：
- **Nginx** 把外部请求路由到相应的容器/服务，例如 API 或 Web 界面。
- **API** 连接 PostgreSQL 进行数据存储和检索，还可能使用 Redis 进行缓存或会话存储。
- **Worker** 进程可能处理涉及繁重工作或后台处理的异步任务，与 API 和数据库都有交互。

以下是 docker compose 完成时显示的输出，

```
...                                                                             40.6s
[+] Running 9/9
 ✔ Network docker_default       Created                                                        0.0s
 ✔ Container docker-weaviate-1  Started                                                        0.2s
 ✔ Container docker-web-1       Started                                                        0.2s
 ✔ Container docker-db-1        Started                                                        0.2s
 ✔ Container docker-sandbox-1   Started                                                        0.2s
 ✔ Container docker-redis-1     Started                                                        0.2s
 ✔ Container docker-api-1       Started                                                        0.0s
 ✔ Container docker-worker-1    Started                                                        0.0s
 ✔ Container docker-nginx-1     Started                                                        0.0s
```

运行之后，你可以在浏览器中访问 [http://localhost/install](http://localhost/install)，进入 Dify 仪表盘并启动初始化过程。

### 升级 Dify

Dify 是一个充满活力、不断演进的项目，拥有一个贡献者社区，其更新节奏几乎像时钟的滴答声一样可预测——确切地说，是每周更新。

要把 Dify 更新到最新状态，需要深入其源代码的数字世界，具体来说是 docker 目录。这段旅程从一条作为入口的简单命令开始：`cd dify/docker`。

```
cd dify/docker
git pull origin main
docker compose down
docker compose pull
docker compose up -d
```

最后，一切就绪，`docker compose up -d` 让更新后的 Dify 焕发生机。这条命令让一切运转起来，在后台静默工作（`-d` 表示分离模式），让我们在新版本就位的同时不受打扰地继续工作。

在部署架构中使用 Docker，体现了 Dify 对提供一个可扩展、可管理且高效的生产环境的承诺。这种配置支持轻松扩展、健壮的数据持久化和关注点分离——这些是现代可靠 Web 应用的关键要素。此外，Dify 的开源根基鼓励社区持续贡献，确保平台与技术发展和用户需求同步演进。

#### OpenAI GPT4-o 模型

截至（2024-05-13），执行 Dify 的 docker 更新与重启后，

```
...
create mode 100644 api/core/model_runtime/model_providers/openai/llm/gpt-4o-2024-05-13.yaml
create mode 100644 api/core/model_runtime/model_providers/openai/llm/gpt-4o.yaml
...
```

![Dify 中 OpenAI gpt4o 已就绪]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_OpenAI_gpt4o_is_here.jpg)

_图：最新的 GPT4-o 已可作为 OpenAI 模型之一使用。_

## 扩展 Dify 的能力边界——HTTP 请求与工具

Dify 呈现出一个直观的界面。一个醒目的“创建自定义工具（Create Custom Tool）”按钮，邀请你去打造独特的东西。它可以是任何东西，但我们不妨考虑一个用于查询会员福利保障范围的医疗保健 API 工具。这是一项具体的任务，却很好地展示了平台轻松满足细分需求的能力。

Dify 用户体验的精髓，浓缩在它处理工具描述与模式（Schema）的方式之中。例如，前文提到的医疗保健工具并非悬浮在数字虚空之中，而是扎根于一个明确的目标——帮助用户查询会员福利保障。遵循 OpenAPI-Swagger 规范的模式，用一种既标准化又易于理解的语言勾勒出 API 调用的结构。这意味着，即使是不熟悉 API 开发种种细节的人，也能把握这个工具的运作方式。

![通过 OpenAPI 规范创建自定义工具]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Create_Custom_Tool_via_OpenAPI_Spec.jpg)

_图：“创建自定义工具”界面专门展示了通过集成 API 来创建和编辑自定义工具的方式。_

端点配置进一步揭开了流程的神秘面纱，清晰地展示数据是如何接收和处理的。而服务器配置指向本地开发环境，则为用户提供了一个用于试验和测试的沙盒——这是任何正式上线之前的关键步骤。

### 连接本地 API 服务器的技巧

在本地开发环境中使用 Dify 时，挑战往往在于确保它能与宿主机上运行的服务通信。这对测试至关重要。这一过程从确定宿主机的 IP 地址开始，它充当运行在 Docker 容器内的 Dify 与本地服务之间的桥梁。执行 `ifconfig`，你可能会发现自己的 IP 地址形如 `192.168.50.197`。

有了这个信息，配置 HTTP 请求就变得简单直接。例如，要与某个服务交互，只需把请求指向本地开发服务器的端点 `http://192.168.50.197:8001/answer`。

同样，要获取对理解服务能力至关重要的 OpenAPI 规范，可以使用如下 URL：
- `http://192.168.50.197:8001/docs` 用于交互式 Swagger-UI。
- `http://192.168.50.197:8001/openapi.json` 用于获取 JSON 格式的规范。

对这种设置的一个优雅增强，是使用 `/etc/hosts` 文件。添加如下条目：
```
192.168.50.197 test.me.com
```

就可以简化对本地服务的访问。这不仅让 URL 更具可读性，还模拟了真实世界的 DNS 配置，为测试和开发提供了更贴近实际的环境。这种做法凸显了开发工具与服务之间无缝交互的重要性，确保测试既高效，又能反映真实的部署场景。

### 将 Dify 连接到本地 LLM

通过 [Ollama]((https://github.com/ollama/ollama)) 将 Dify 与本地 LLM 集成，为我们的模型利用 GPU 加速提供了一条精简的途径。Ollama 可通过简单的 CLI 或 REST API 访问，简化了应用与模型之间的交互。入门非常简单，只需从其网站[下载并安装 Ollama]((https://ollama.ai/download))。

对于使用 macOS 的用户，把 Ollama 与 Docker Desktop 搭配使用可以增强 GPU 加速能力。启动一个模型（例如 `llama3`）只需一条简单的命令。Ollama 会打理剩下的一切，包括在我们首次使用时下载模型。例如，`llama3-8b` 模型会被高效地压缩到 4GB 左右以便本地使用。

```
ollama run llama3
```

> 参考 Ollama 官方网站，它支持众多开源 LLM 模型。

[Dify 与 Ollama 的集成](https://docs.dify.ai/tutorials/model-configuration/ollama)十分顺畅，其设计可以把 Ollama 识别为模型提供商。在 macOS 上运行 Ollama 时，需要通过 `launchctl` 设置环境变量才能顺利运行。快速重启 Ollama 应用即可让这些设置生效。

```
launchctl setenv OLLAMA_HOST "0.0.0.0"
```

最后，在 Dify 中配置 Ollama 即完成全部设置，随时可以投入使用。这一集成不仅简化了流程，还通过 GPU 加速优化了本地模型的性能。

![Dify 模型提供商 Ollama Llama3 设置]({{ site.baseurl }}/images/dify-your-weekend-genai-magics/Dify_Model_Provider_Ollama_Llama3_Setup.jpg)

_图：把 Dify 的 Ollama 配置为模型提供商之一，其接口使用本地 IP 地址和端口。_

## 结语

当我们遇见 Dify 时，不只是偶然撞见又一个平台。我们是在深入一个经过专业设计的生态系统，它旨在支持我们与生成式 AI 技术同行的旅程的每一个阶段。从学习之初迸发的火花，到生产部署的最后阶段，Dify 犹如一座灯塔，指引用户穿越 AI 开发中时常浑浊的水域。

Dify 之美在于它的平易近人。无论我们是经验丰富的开发者，还是刚刚开始涉足 AI 的新手，Dify 都能各取所需。它让 AI 创作的过程民主化，使曾经看似令人生畏的任务变得亲切可及，甚至可以说充满乐趣。这不仅仅是拥有一件趁手的工具，更是拥抱一场冒险——一个用生成式 AI 的魔力把想法变为现实的探索周末。

## 参考文献

- [Dify.ai](https://dify.ai/) - Dify 是一个开源的 LLM 应用开发平台。它直观的界面融合了 AI 工作流、RAG 管道、智能体能力、模型管理、可观测性特性等，让你快速地从原型走向生产。
  - github 仓库：- langgenius/ [dify](https://github.com/langgenius/dify)
  - [Dify 社区文档](https://docs.dify.ai/)
  - [快速工具集成](https://docs.dify.ai/tutorials/quick-tool-integration)
  - [通过 API 进行开发](https://docs.dify.ai/user-guide/launching-dify-apps/developing-with-apis)
  - [通过 API 维护知识库](https://docs.dify.ai/features/datasets/maintain-dataset-via-api)
  - [Dify 与 Ollama 的集成](https://docs.dify.ai/tutorials/model-configuration/ollama)
  - [技术特性](https://docs.dify.ai/getting-started/readme/specifications-and-technical-features)

- Dify.ai, [Introducing Dify Workflow](https://dify.ai/blog/dify-ai-workflow), Dify Insights Blog, 8 Apr 2024. 
- Dify.ai, [Easy-to-Use LLMOps Platform for Visually Creating and Operating Your AI Native Applications](https://dify.ai/blog/open-source-llmops-platform-define-your-ai-native-apps), Dify Insights Blog, 10 May 2023.
- Dify.ai, [AI in Finance: Building an AI Investment Analysis Assistant with Dify](https://www.youtube.com/watch?v=Hm4kBQSkNaA), video, 18 Feb 2024.
- Xiao Yi, [Unleashing AI in Finance: Building an AI Investment Analysis Assistant with Dify](https://dify.ai/blog/build-your-own-investment-analysis-assistant-with-dify), Dify Insights, 26 Jan 2024.