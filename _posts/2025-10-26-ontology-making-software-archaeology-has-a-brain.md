---
layout: post
title: 本体论——让软件考古学拥有可查询的大脑
date: 2025-11-01 12:00:00.000000000 -00:00
permalink: ontology-the-queryable-brain-of-software-archaeology/
author: Benny Cheung
artwork:
  author: Nano Banana
tags:
- Ontology
- Software Archaeology
- AI Engineering
- BMAD
category: post
comments: true
image: images/ontology-the-queryable-brain-of-software-archaeology/post_cover.jpg
images:
  cover: images/ontology-the-queryable-brain-of-software-archaeology/cover2x.jpg
  header: images/ontology-the-queryable-brain-of-software-archaeology/header.jpg
  animated-cover: images/ontology-the-queryable-brain-of-software-archaeology/animated-cover.webp
published: false
---

<!--excerpt.start-->
每个资深开发者都熟悉那种感觉。你接手一套复杂、业务攸关的系统，而最初的架构师早已离开。系统有多套数据库、一张微服务之网，唯一的文档只是几页过时的 wiki。你的工作是添加新功能或修复关键缺陷，但首先，你必须回答一个看似简单的问题："如果我改了这里，什么会坏？"
<!--excerpt.end-->


## 遗留系统的噩梦

这不只是存在了十年的单体系统才有的问题。设想"软件考古学工具平台"（Software Archaeology Tools Platform）——一套只有六个月历史的生产系统。它已经有 8 个微服务、3 个不同的数据库（Neo4j、PostgreSQL 和 Redis）、47 个 API 端点，以及 12 张归属不明的数据库表。雪上加霜的是，设想团队已经失去了最初三名开发者中的两位——66% 的流失率——关键的机构知识也随之而去。新团队只能拼凑数据是如何在这个复杂系统中流动的。

有一种强大的方法可以穿透这种复杂性。通过为系统创建一个形式化的语义模型——**软件考古学本体**（Software Archaeology Ontology，本文简称 _SAR ontology_）——你可以构建一个"可查询的大脑"，保存一张持久、准确、可发现的架构与数据流地图。本文拆解把这一方法应用于生产系统后最具影响力的五点收获。

可以把本文看作软件考古学的_缺失拼图_：用 BMAD-V6 SAR 模块理解遗留系统，用 BMAD-V6 ONTO 模块记录并分析已经完成的工作。随后，这些知识可以用来以强有力的证据和对影响的信心去现代化遗留系统。

![SAR_Ontology_Dev_Cycle]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/SAR_Ontology_Dev_Cycle.png)
_图："宇航员指枪"梗图被重新演绎，用来说明软件考古学工具平台的自指之力。这一分层分析确立了软件考古学本体作为自信地进行遗留系统现代化的"可查询大脑"，拼完了软件考古学的完整拼图。_


### 1. 你真的可以直接向代码库提问

传统上，理解数据血缘是一场痛苦的、手工的追猎。它是一次考古挖掘：翻遍陌生的代码、过时的 wiki，以及最后一位碰过这些代码的资深工程师模糊的记忆。这是猜谜，不是工程。

本体驱动的方法粉碎了这一范式。你得到的不再是侦探式工作，而是确定性的答案。通过对系统的组件（如服务、API、数据库、表）及其之间的关系（如 dependsOn、calls、produces、writesTo）建模，你创建了一张知识图谱。有了这张图，你就可以用 SPARQL 这样的查询语言提出精确、复杂的问题，并得到确定的答案。

![SAR_Ontology_01_Software_Entity]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/SAR_Ontology_01_Software_Entity.png)

_图：展示 SAR 本体中建模软件实体（Software Entity）层级的部分。该本体可以加载到流行的 Protégé 平台进行维护与可视化。我们也可以用 Python（使用 rdflib 或 owlready2）以编程方式管理和查询本体。_

例如，你不再需要手动猜测，而是可以直接问：

- "哪些组件拥有最多的依赖（耦合热点）？"
- "哪些 API 端点写入哪些数据库表？"
- "当用户与 React UI 仪表盘交互时，数据存储在哪里？"
- "哪些技术债条目的修复优先级最高？"

至关重要的是，这些问题的答案不是硬编码的。它们是通过追踪本体中已形式化建模的关系实时发现的。这让系统理解从一门艺术变成一门科学。这个模型的美妙之处在于：哪怕答案不完整，也是有价值的。一个返回空结果的查询并不意味着工具失败；它意味着你发现了知识模型中一个需要填补的空白。

**关键洞见：** 数据血缘追踪的质量取决于 SAR 本体中系统关系建模的完整程度。结果中的空白指示本体需要增强的地方。

#### 从本体到知识图谱的原理

**本体**（ontology）充当蓝图或形式化语义模型（即 schema），规定某个领域内的结构与关系，定义诸如 `Database` 这样的概念，以及它如何通过 `sar:dependsOn` 等属性与 `Service` 关联。相比之下，**知识图谱**（knowledge graph）是持有具体的现实世界事实的实际数据仓库——在本例中即完整的软件考古学平台数据集——它表示为派生自本体 schema 并受其约束的实例三元组。

![SAR_Ontology_Explained_relation_to_KG]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/SAR_Ontology_Explained_relation_to_KG.png)

_图：知识图谱是机器可读的数据结构，它把本体中建模的物理世界与数字世界的知识表示出来。_

因此，知识图谱是_构建在_本体之上的，把抽象的结构定义（schema 三元组）与具体数据（实例三元组）结合起来。这种结合让复杂的操作任务成为可能，例如对知识图谱执行 SPARQL 查询以自动追踪跨组件的完整数据血缘——而由此得到的血缘追踪的准确性，直接取决于系统关系最初在 SAR 本体中**建模**的完整程度。

![SAR_Ontology_Instance_Example]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/SAR_Ontology_Instance_Example.png)

_图：例如，来自 BMAD-V6 系统探索的真实实例数据，展示了识别出的真实组件、技术债条目和架构模式。_

### 2. 团队人员流动不必成为知识黑洞

团队人员流动是关键的业务风险。当经验丰富的开发者离开时，他们带走了大量未成文的关于系统架构、设计理由和运维怪癖的知识。正如这个真实系统所示，短短三个月内失去最初三名开发者中的两位，造成了巨大的知识真空。

本体充当一个持久、集中的知识库，缓解这一风险。它捕捉通常在团队变动中丢失的关键"机构知识"，将其形式化为可查询的模型。服务之间的关系、各数据库表的用途、数据在工作流中的流动，不再只存在于开发者的头脑中；它们成为知识图谱中显式的事实。

它对新人入职（onboarding）的影响是巨大的。对我们团队而言，让新开发者具备生产力所需的时间，从没有本体时估计的数周，缩短到有本体时的数天。这是因为新团队成员可以独立探索系统、自己回答问题，从而大幅缩短了上手时间，也降低了团队流失带来的风险。无论今天团队里是谁，系统都保持可理解、可维护。

### 3. 影响分析成为科学，而非猜测

这一问题因我们讨论的团队人员流动而放大。当机构知识不复存在，对复杂系统的任何修改都变成一场高风险的猜谜游戏。"如果我修改这个 PostgreSQL 表结构，哪些服务会坏掉？"没有完整的地图，开发者退化到 `grep`-and-pray（边搜边祈祷），指望文本搜索能挖出每一个关键依赖。这不是策略；这是生产事故的配方。

本体把影响分析从猜测转变为确定性过程。在修改 PostgreSQL schema 的场景下，只需执行一条 SPARQL 查询，就能找出依赖该数据库的每一个服务、API 端点和组件。查询返回一份确定的受影响组件清单，消除一切含糊。

![ONTO_Agents_Collective_Actions_01]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/ONTO_Agents_Collective_Actions_01.png)

这份清单成为确定性计划的基础。你甚至可以通过给每个组件分配平均耗时来生成具体的工时估算，把一个含糊的任务转化为"2.5 人天"这样的可量化指标。这一能力把一场持续数天、令人焦虑的猜谜活动，变成一小时的精确、数据驱动的规划。

### 4. 业务价值不是"软"的——它巨大且可度量

我们都曾为"写文档"争取时间而苦恼，这项任务常因其"软"价值而被轻视。本体驱动的方法彻底重构了这场讨论，交出的投资回报一点也不"软"。

就我们的系统而言，一个示例 ROI 计算显示：本体每年交付的总价值是数万美元的劳动力节省，在初始投入仅 20 小时建模加每年 10 小时维护的情况下，产生约 10 倍的生产力潜在 ROI。这一价值来自常见开发活动中可量化的时间节省、降低的入职成本，以及避免高昂生产事故。

对特定、重复性任务的估算时间节省是这一价值中最引人注目的部分：

| 活动                       | 没有本体     | 有本体     |
| ------------------------------ | ---------------- | ------------- |
| 理解数据流       | 2-3 周        | 30 分钟    |
| 找出某数据库表的所有使用方 | 1 周           | 5 分钟     |
| 规划数据库迁移    | 2 周          | 4 小时       |

关键洞见很简单：系统越复杂，为它建模本体的 ROI 就越高。过去需要数周人工努力的事，现在一条查询几分钟即可完成。

### 5. 你的系统文档终于可以"活"起来

传统文档的根本缺陷——无论是 wiki 里的一张图还是仓库里的一个 Markdown 文件——在于它是静态的（即"死"的）。它几乎在写下的那一刻就开始过时。开发进展飞快，文档鲜能跟上，很快变成误导性或错误信息的来源。

本体方法创造的是一份_活文档_。它建立了"使用驱动维护"的良性循环。当工程师运行一个查询来寻找数据血缘路径而查询没有返回结果时，这并不是工具的失败。它是一个清晰的信号：系统已经变化，模型需要更新。缺失的关系是一个需要填补的显式空白。

![ONTO_Agents_Collective_Actions_12]({{ site.baseurl }}/images/ontology-the-queryable-brain-of-software-archaeology/ONTO_Agents_Collective_Actions_12.png)

这意味着，把本体用于日常任务——执行影响分析或给新开发者做入职引导——这一行为本身就在验证并维护其准确性。模型成为一个能够随系统成长的演进资产：从高层架构起步，随需逐步添加更多细节，例如细粒度的列级数据血缘。

### 结语：你的系统在说话，你在听吗？

这不是一场理论演习；它是驯服我们所构建系统的一门实用、高 ROI 的学科。把系统知识当作一等公民的、可查询的资产来对待，我们就重新掌控了自己的架构。软件考古学平台证明：这对真实的生产系统是一条切实可行的路径。

遗留系统的噩梦并非不可避免；它是一个数据问题。通过给我们的系统一个可查询的大脑，我们把含糊的噩梦变成与我们自己架构的对话。关于你的遗留系统的数据流，你的团队今天有哪个关键问题没人能自信地回答？

## 参考文献

- [BMad Method](https://github.com/bmad-code-org/BMAD-METHOD/tree/main) - Agentic Agile Driven Development 的基础，即 Breakthrough Method of Agile AI-Driven Development（敏捷 AI 驱动开发的突破性方法）
  - [BMad Method 用户指南](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/docs/user-guide.

- Benny Cheung, [BMAD V6 Intellectual Ecosystem for Understanding Legacy Software]({{ site.baseurl }}/bmad-v6-intellectual-ecosystem), Benny's Mind Hack, 17 Oct 2025.
    - 提出 BMAD-V6 框架是对 AI 思考方式的深刻转变：把 AI 从被动的查询工具，变为完整智识生态系统中用于系统性问题求解的集成组件。这一哲学——包括定义问题的严格初始纪律以及人类与 AI 的协作——被具体应用于解构、理解和解决遗留软件的复杂性这一挑战，把混乱无形的问题转化为有边界、可应对的挑战。

- Benny Cheung, [Unlocking the Secrets of Tabletop Games Ontology]({{ site.baseurl }}/unlocking-secrets-of-tabletop-games-ontology), Benny's Mind Hack, 24 Feb 2025.
    - 探索为桌游构建本体，以此作为为一个复杂、含糊的领域创建一致、形式化、可发现表示的方法。这一努力凸显了构建这样的本体如何超越单纯的数据组织，创造出一个可用于查询、推理并从系统架构与逻辑中生成新洞见的语义模型。

- Natalya F. Noy and Deborah L. McGuinness, [Ontology Development 101: A Guide to Creating Your First Ontology](https://protege.stanford.edu/publications/ontology_development/ontology101.pdf), Stanford University.

- [Protégé](https://protege.stanford.edu/) - 一款免费开源的本体编辑器与框架，用于构建智能系统，具有面向简单与复杂应用的插件架构。

## 附录——SAR 本体

该本体为软件考古学（SAR）领域提供了形式化的语义模型，支持对遗留软件系统知识的系统性捕获、集成与分析。该本体在软件考古学活动中统一术语，支持关于系统关系的自动推理，并促进棕地（brownfield）开发过程中的知识传递。

### 什么是 SAR 本体？

**SAR 本体**是一个形式化的、机器可读的知识模型，它定义了：

- **概念**（类）：存在什么事物？（例如 LegacySystem、Component、Agent）
- **属性**（关系）：事物如何关联？（例如 dependsOn、executes、contains）
- **实例**（个体）：具体例子？（例如 PaymentService、Dr. Ada 考古学家）
- **公理**（规则）：适用什么约束？（例如一个 Component 只能属于一个 Module）

在高层上，SAR 本体的构造如下结构：

```
BFO (Basic Formal Ontology)
├── Continuant (Persistent Entities)
│   ├── Software Entities
│   │   ├── LegacySystem (kind)
│   │   │   ├── Monolith
│   │   │   ├── DistributedSystem
│   │   │   └── HybridSystem
│   │   ├── Component (kind)
│   │   │   ├── Module, Service, Library, Database
│   │   └── CodeArtifact (kind)
│   │       ├── SourceFile, Class, Function
│   ├── Agents (roles)
│   │   └── 6 agent types (Archaeologist, Cartographer, etc.)
│   ├── Quality & Patterns
│   │   ├── TechnicalDebt (mode)
│   │   ├── ArchitecturePattern (kind)
│   │   └── CodeSmell (mode)
│   └── Documentation
│       └── README, APIDoc, ArchitectureDoc, Runbook
└── Occurrent (Processes/Events)
    ├── ArchaeologyPhase (phase)
    │   └── 5 phases (Exploration → Modernization)
    └── Workflow (kind)
        └── 5 workflow categories
```

SAR 本体为软件考古学创建了一个语义模型： 
- 96 个类 - 领域概念（系统、代理）
- 23 个属性 - 关系（dependsOn、executes）
- 1,076 个三元组 - 丰富的知识图谱 
- 26 个能力问题（Competency Questions） - 经过验证的用例

相较于传统方法，使用 SAR 本体有许多优势：

| 传统方法        | 本体方法                   |
| --------------------------- | ----------------------------------- |
| 知识存在于人们的头脑中 | **形式化的知识图谱**      |
| 文档散落各处     | **单一语义模型**           |
| 手工追踪依赖   | **自动化 SPARQL 查询**        |
| 口口相传的知识丢失       | **持久、可查询的知识** |
| 各自为政的理解        | **共享词汇**               |
