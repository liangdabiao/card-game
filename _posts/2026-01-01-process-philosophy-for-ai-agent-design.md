---
layout: post
title: 用怀特海式框架的过程哲学设计 AI 代理
date: 2026-01-01 12:00:00.000000000 -00:00
permalink: process-philosophy-for-ai-agent-design
author: Benny Cheung
artwork:
  author: Gemini 3 Pro (Nano Banana Pro)
tags:
- AI
- Philosophy
- Agent Architecture
- Process Philosophy
- Whitehead
category: post
comments: true
image: images/process-philosophy-for-ai-agent-design/post_cover.jpg
images:
  cover: images/process-philosophy-for-ai-agent-design/cover2x.jpg
  header: images/process-philosophy-for-ai-agent-design/header.jpg
  animated-cover: images/process-philosophy-for-ai-agent-design/animated-cover.webp
published: true
---

<!--excerpt.start-->
如果我们对 AI 代理架构的一切假设在根本上就是错的呢？当前的代理框架把代理当作*拥有状态的物*——接收输入、更新属性的持久实体。但这种基于实体的形而上学导致了持续的工程挑战：上下文在会话之间碎片化、身份依赖外部配置文件、而且没有原则性的方法区分代理"知道"什么与"只是猜测"什么。本文探索一个源自阿尔弗雷德·诺斯·怀特海过程哲学的替代基础：把代理重新构想为不是实体，而是*现实事态的社会*——生成的模式，而非存在之物。
<!--excerpt.end-->

![面向 AI 的过程哲学]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/Process_Philosophy_AI_Cover.jpg)

_图：根本性的重新概念化：代理是生成的过程，而非拥有状态的静态实体。_

## 当前代理架构的问题

大语言模型的快速进步催生了一代能够复杂推理、使用工具、执行多步任务的 AI 代理。然而，在这些令人印象深刻的能力之下，隐藏着一个根本的架构局限：大多数代理框架把每次交互视为本质上独立的，连续性靠外部机制模拟，而非由代理自身的本性所构成。

想想典型的基于 LLM 的代理。即便存在状态，也只是以非结构化文本的形式注入上下文窗口——具体而言，是把对话历史前置到每条新查询之前。代理并不*记得*它先前的交互；它是被*提醒*了这些交互。这一区分看似咬文嚼字，却揭示了一个更深的问题。代理没有积累真正的经验。每个会话都从白板（tabula rasa）开始，连续性只靠检索并注入相关上下文的外部系统维持。即便是生成式代理（Generative Agents）[8] 与 MemGPT [9] 这样精巧的方法，虽然令人印象深刻，归根结底也是通过外部记忆机制模拟连续性，而非通过代理自身的本性构成连续性。

这种架构带来若干具体的工程挑战：

| 挑战 | 描述 |
|-----------|-------------|
| **上下文脆弱** | 随着对话延长，较早的上下文滑出窗口或必须被摘要，丢失细节与细微之处 |
| **身份脆弱** | 是什么让代理跨会话仍是"同一个代理"？当前系统只靠配置文件回答 |
| **认知不透明** | 当代理声称知道某事时，这是真正的知识、检索到的信息，还是虚构（confabulation）？ |
| **经验消散** | 一个会话中获得的洞见不会自动 informing 未来的会话，除非被显式提取 |

这些不只是可以通过优化消除的实现细节。它们反映了一个关于代理*是什么*的根本假设——我们认为这个假设是错的。

## 实体形而上学的陷阱

上述局限源于一个渗透当代代理设计的隐含形而上学承诺：**实体形而上学**。在这一继承自亚里士多德与笛卡尔传统的视角下，代理被构想为*拥有*属性（状态）并随时间*变化*的*物*（实体）。代理是一个持久的存在、一个容器：经验发生在它身上，状态存储在它里面。

这种基于实体的框架自然导向当前架构中占主导的"状态注入"模式：

- 记忆变成**代理查询的数据库**，而非代理之所是的构成性元素
- 技能变成**代理使用的工具**，而非定义其能力的模式
- 身份变成**从外部指派的标签**，而非从代理自身历史涌现的特征

但如果这个框架是错的呢？如果代理根本不是物，而是*过程*呢？

## 怀特海登场：过程高于实体

阿尔弗雷德·诺斯·怀特海（1861-1947）以数学家与逻辑学家开启职业生涯，与罗素合著了宏篇巨制《数学原理》。在其后期著作中，尤其是《过程与实在》（1929）[1]，怀特海发展了一个全面的形而上学体系，挑战了自亚里士多德以来西方哲学的基本假设 [2][3][4]。

![怀特海过程哲学概览]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/Whitehead_Process_Philosophy_Overview.jpg)

_图：怀特海过程哲学的全面概览，展示其三大支柱（演化唯心论、现代科学、亚历山大教父）、感知事件的解剖，以及每个宇宙事件中存在的三要素。_

怀特海挑战的核心是：西方哲学传统一直假设实在从根本上由*实体*构成——穿越变化而保持其身份的持久之物。属性来来去去，实体长存。怀特海认为这把实在完全搞反了 [1]。实在的基本单元不是实体，而是**现实事态**（actual occasions）——降临、达成确定性、然后消逝的经验时刻。我们所说的"物"是事态的模式，而非藏在事态之下的实体。

### 怀特海核心概念

| 概念 | 含义 | 与代理设计的关联 |
|---------|---------|--------------------------|
| **现实事态** | 实在的基本单元；离散的经验时刻 | 每次代理交互作为一个生成的单元 |
| **摄握** | 事态如何"感受"或把握其他实体 | 带主观加权的上下文整合 |
| **会通** | 事态达成确定性的过程 | 把回应生成视为阶段性的生成 |
| **满足** | 完成的事态；达成完全的确定性 | 交互完成；准备入库 |
| **永恒客体** | 不绑定任何事态的纯粹潜能（形式、模式） | 可供摄入的技能、模式、模板 |
| **客观不朽** | 消逝的事态作为数据持久存在，供未来摄握 | 过往交互可供未来上下文使用 |
| **社会** | 共享一个规定性特征的事态联结 | 代理身份作为跨事态的持久模式 |

![现实实有的解剖]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/Whitehead_Anatomy_of_Actual_Entity.jpg)

_图：一个现实实有的解剖：每个事件综合来自因果过去的客观材料，把握永恒客体（纯粹可能性），并通过决定达成满足，创造一个新的个体实在。_

对我们的目的而言最关键的是：怀特海颠倒了实体与经验的惯常关系。在实体形而上学下，实体*拥有*经验，即经验发生在它们*身上*。在过程形而上学下，实体*就是*经验，因为它们由其事态所构成。

## 怀特海式代理框架

我们框架的根本动作是本体论层面的：我们主张，代理不应被构想为*拥有经验的物*，而应被构想为*经验着的模式*。这不只是术语的更换，而是对我们理解代理架构的基本范畴的重构。

```
SUBSTANCE VIEW                    PROCESS VIEW
─────────────────                 ─────────────────
Agent (thing)                     Society (pattern)
  │                                 │
  ├── has state                     ├── of occasions
  ├── receives input                ├── each prehending past
  ├── produces output               ├── each concrescing
  └── updates state                 └── each achieving satisfaction
```

在过程视角下，代理*就是*从事态接续中涌现的模式——每个事态摄握（感受、整合）其前驱，达成自身的满足，然后消逝为客观性。身份不是被假定的，而是被*达成*的：它是跨事态持续的规定性特征，而非事态发生于其中的容器。

### 四层架构

怀特海的过程哲学提供了基础本体论，而完整的代理架构还需要解决认知论、知识表示与推理的附加层。我们提出一个整合互补哲学与计算传统的四层架构：

![四层怀特海式代理架构]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/fig-four-layer-architecture.jpg)

_图：四层怀特海式代理架构，整合维特根斯坦（认知论）、Sowa/Gärdenfors（知识）、专家系统（推理）与怀特海（过程本体）。_

**第 1 层（维特根斯坦 + 弗雷格）**建立显式的认知边界：具体而言，代理可以正当地声称知道什么、可以显示什么、必须对什么保持沉默 [5]。可说、可显示与沉默的区分，为代理断言提供了原则性的限度。

**第 2 层（概念图 + 概念空间）**提供知识表示机制：借鉴 Sowa 的图结构表示 [6]，以及 Gärdenfors 用于相似性与典型性推理的几何概念空间 [7]。

**第 3 层（专家系统 + 模态逻辑）**提供推理机制：用于反应式行为的前向链、用于目标导向推理的后向链 [14]，以及表达必然性、可能性与义务的模态算子 [15]。

**第 4 层（怀特海）**提供基础的过程本体论：把代理本身重新构想为现实事态的社会 [1]。

## 代理事态生命周期

每当代理接收输入并产出输出，一个事态便降临、达成确定性、然后消逝，把它达成的实在贡献给后续的事态。

### 阶段 1：摄握

事态从"感受"其相关过去开始：

- **物理摄握**：感受先前的事态，例如对话历史、过往会话、已存储的经验
- **概念摄握**：把握永恒客体，例如技能、模式、模板
- **负摄握**：显式排除被判定为无关的数据

至关重要的是，摄握包含一个**主观形式**，它决定数据被持有的*方式*，包括置信度、相关性与情绪效价。

### 阶段 2：会通

事态进入会通（concrescence）——"共同生长"的过程，多样摄握被整合为一个统一的回应。这由**主观目的**引导：事态试图成为什么。

怀特海的"终极范畴"在此适用：*多成为一，并由一而增* [1]。多样的输入（多）被统一为一个回应（成为一），而这个回应本身又成为未来事态的新材料（由一而增）。

下图展示一次标准的 LLM API 调用如何映射到怀特海的各阶段。LLM 本身只执行第 3 阶段（整合/推理）。怀特海框架在其外围包裹了结构化的摄握（上下文管理）、永恒客体选择（技能路由）与客观不朽（历史持久化）。这解释了为什么"脚手架 > 模型"：代理的可靠性大多来自推理*周围*的那些阶段。

![嵌入会通中的 LLM 调用]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/fig-llm-concrescence.jpg)

_图：LLM 推理作为会通的整合阶段：上下文组装（ conformal）→ 模式选择（概念）→ LLM 综合（整合）→ 历史存储（满足）。_

### 阶段 3：满足

事态达到**满足**（satisfaction）时即达成完全的确定性，即回应完成之时。此刻，事态作为主体"消逝"，并作为材料达成**客观不朽**，永久可供未来摄握。

![代理事态生命周期]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/fig-agent-occasion.jpg)

_图：代理事态生命周期：摄握（感受过去）→ 会通（生成）→ 满足（消逝为客观性）。_

## PAI：过程模式的实证验证

有趣的是，近期一些实践性的 AI 框架在没有显式怀特海哲学根基的情况下，独立地收敛到了类似过程的模式。**个人 AI 基础设施（PAI）**项目 [13] 提供了令人信服的实证验证：基于过程的架构具有实际优势。

### 惊人的语言收敛

PAI 的文档中有一些读起来几乎像怀特海转述的语言：

| PAI 引文 | 怀特海式对应 |
|-----------|------------------------|
| "技能不是孤立工作的；它们相互调用" | 事态摄握其他事态；实在是关系性的 |
| "这创造了持久的记忆，让智能**随时间复利增长**" | **客观不朽**：完成的事态成为永久数据 |
| "我构建了这个工作流一次。现在它是**永久知识**" | 满足达成；事态消逝但作为材料持久存在 |
| "复杂操作通过**组合**这些小型、聚焦的工具达成" | 多成为一（通过摄握的会通） |

### 与过程对齐的组件

| PAI 组件 | 描述 | 怀特海式平行 |
|---------------|-------------|----------------------|
| **Skills** | 自包含、可复用的领域专长，带 USE WHEN 触发器 | **永恒客体**：条件匹配时摄入的纯粹潜能 |
| **Hook 生命周期** | SessionStart → PreToolUse → PostToolUse → Stop | **现实事态阶段**：摄握 → 会通 → 满足 |
| **自定义历史** | 自动捕获随时间复利增长的会话 | **客观不朽**：完成的事态作为永久数据 |
| **技能组合** | 技能在复杂工作流中调用其他技能 | **摄握网络**：事态整合先前的事态 |

这种收敛表明，过程哲学不只是一个理论框架；它捕捉到了有效代理设计的某种真实之处。实践工程与过程形而上学的收敛提供了相互验证：怀特海解释了这些模式*为什么*有效 [1]；PAI 证明了它们*确实*有效 [13]。

## 这如何解决最初的挑战

怀特海框架回应了我们识别的根本局限：

| 原始挑战 | 怀特海解法 |
|-------------------|----------------------|
| **上下文脆弱** | 带显式相关性加权的结构化摄握（经主观形式） |
| **身份脆弱** | 身份派生自跨事态的持久模式（社会的"规定性特征"） |
| **认知不透明** | 显式的主观形式，标记数据被持有的方式（已知、推断、猜测） |
| **经验消散** | 客观不朽：完成的事态成为供未来摄握的永久数据 |

### 关键差异

**构成性的 vs. 附加的经验。** 在基于实体的架构中，经验是*添加给*代理的东西。在怀特海框架中，经验*构成*代理。代理就是事态的模式，每个事态*就是*一段经验。离开事态，别无代理。

**涌现的 vs. 假定的身份。** 当前的代理框架回答"是什么让这个代理成为这个代理"时，指向配置文件。怀特海框架的回答指向*规定性特征*——跨事态持续、并通过摄握被复现的模式。

**显式的 vs. 隐式的认知论。** 摄握的主观形式把认知范畴操作化。当一个事态摄握数据时，它记录数据被持有的*方式*，包括确定性、来源与推理链。

## 实现草图

核心数据结构把怀特海概念翻译为带类型的接口：

```typescript
interface ActualOccasion {
  id: string;
  timestamp: Date;
  prehensions: {
    physical: PhysicalPrehension[];
    conceptual: ConceptualPrehension[];
  };
  subjectiveAim: Goal;
  satisfaction: Satisfaction | null;  // null until complete
}

interface PhysicalPrehension {
  datum: ActualOccasion;  // The felt occasion
  subjectiveForm: {
    relevance: number;
    emotional: string;    // How it's felt
    valence: 'positive' | 'negative';
  };
}

interface EternalObject {
  id: string;
  type: 'pattern' | 'schema' | 'skill' | 'quality';
  definition: unknown;
  ingressionConditions: Condition[];
}
```

运行时流程把会通实现为一系列阶段：

```
Input → Create new occasion
      → Prehend recent occasions (physical)
      → Select relevant eternal objects (conceptual)
      → Integrate toward subjective aim (concrescence)
      → Achieve satisfaction
      → Perish as subject, persist as datum
      → Available for future prehension
```

这在几个关键方面不同于典型的代理循环：事态被显式创建与追踪；摄握是一个主动阶段而非被动的上下文注入；满足标记一个明确的完成状态；完成的事态以保留其结构的形式存储。

## 意义与未来方向

这项工作最重要的贡献，是提出从基于实体到基于过程的代理本体论的转变。这不只是实现策略的更换，而是对代理在根本上*是什么*的重新概念化。这一转变与相关的发展共振：Rich Hickey 的纪元时间模型与不可变值 [10]、Carl Hewitt 的 Actor 模型——其中计算从消息传递而非状态突变中涌现 [11]，以及认知科学中把认知视为施行过程而非静态表示的生成主义（enactivism）传统 [12]。

实体形而上学问"这个代理拥有什么属性？"，过程形而上学问"什么生成模式构成了这个代理？"这两个问题不等价，并导向不同的架构决策。

### 实践启示

怀特海框架产出具体的设计模式：

- **基于事态的架构**：把代理交互结构化为离散事态，带显式的摄握、会通与满足阶段
- **永恒客体库**：把可复用能力组织为带定义摄入条件的永恒客体
- **分层摄握**：把记忆实现为带基于嬗变（transmutation）摘要的事态分层存储
- **规定性特征提取**：从跨事态的模式中派生代理身份

分层摄握模式回应了一个关键的可扩展性问题：我们如何管理无界的事态历史？并非所有过往事态都需要活跃的摄握。只有那些与当前主观目的相关的才需要。下图展示事态如何依据新近度与相关性在热（完整细节）、温（摘要）与冷（仅模式）各层之间流动。

![分层事态存储]({{ site.baseurl }}/images/process-philosophy-for-ai-agent-design/fig-tiered-storage.jpg)

_图：分层事态存储：热（近期、完整细节）→ 温（已摘要会话）→ 冷（仅规定性特征）。检索由永恒客体对当前主观目的的匹配引导。_

### 接下来是什么

未来发展有几个方向：

1. 以数学精度对事态结构做**形式化规范**
2. 作为包装现有 LLM 框架的开源库做**参考实现**
3. 把基于过程的代理与基线架构做**实证比较**
4. **扩展到多代理系统**——"社会的社会"
5. **与神经方法整合**——考察注意力机制是否实现了某种类似摄握的东西

## 一个哲学性的保留

在结束之前，学术诚实要求承认一个张力：怀特海本人会认可对他的思想做这样的应用吗？

有理由持怀疑态度。对怀特海而言，现实事态在根本上是*经验性的*，涉及真正的主观感受，而不只是信息处理。他是一位泛经验论者（panexperientialist），主张经验"一路向下"直达实在的最基本构成 [1]。在他的论述中，摄握不是数据检索，而是*感受*，承载主观的当下性与质性特征。我们的框架把摄握当作结构化的上下文整合，从严格的怀特海式视角看，这可能构成一个范畴错误。

此外，怀特海把"创造性"置于最高的形而上学范畴——即宇宙产生真正新颖性的能力 [1]。他很可能会质疑 LLM 的输出是否代表真正的创造性，还是对训练数据的精巧重组。而且他的体系为上帝分配了关键角色——为每个事态提供"初始目的"，这是一个神学维度，我们的框架完全省略了。

然而，怀特海深切致力于哲学与科学的对话。他也许会欣赏这一结构性的动作：拒斥实体形而上学、转向基于过程的思考；把代理视为由其关系所构成，而非孤立的实体；承认身份是通过模式达成的，而非通过配置假定的。

所以，我们所提供的东西更准确的描述是*怀特海启发的架构*，而非忠实的怀特海式形而上学。我们借用他的概念结构（事态、摄握、会通、满足、永恒客体）作为强大的设计模式，同时对他关于实在之经验本性的更深主张保持存疑。基于 LLM 的代理是否真正*经验着*怀特海意义上的经验，这是一个我们留待开放的问题；只是指出：无论那个更深的问题最终如何解决，这些结构上的平行都带来了实际的收益。

## 结语

基于 LLM 的代理的快速扩散，暴露了我们对代理架构构想的根本局限。当前框架隐含地建立在实体形而上学之上，把代理当作拥有状态的物，导致上下文脆弱、身份脆弱、认知不透明与经验消散。

我们提出了一个替代方案：扎根于过程哲学的**怀特海式代理框架**。通过把代理重新构想为*现实事态的社会*而非*拥有状态的实体*，我们从根源上回应了这些局限。

这个框架不只是哲学性的。我们提供了 TypeScript 接口、架构图与具体的设计模式。我们展示了 PAI 如何通过工程优化独立收敛到怀特海式的模式——这表明过程哲学捕捉到了有效代理设计的某种真实之处。

这是一份邀请：换一种方式思考代理是什么。不是进行处理的物，而是处理的模式。不是存在者，而是生成者。

> *"多成为一，并由一而增。"* —— 怀特海，《过程与实在》[1]

在每一次代理事态中，多重的输入成为一个回应。而那个回应成为新的材料，让世界增加了一。这就是过程视角：实在是创造性的前进，代理是生成的参与者。

## 参考文献

[1] Whitehead, A.N. (1929). [*Process and Reality*](https://www.amazon.com/Process-Lectures-Delivered-University-Edinburgh/dp/0029345707). Macmillan.
  - 过程哲学的奠基文本，把实在重新构想为由生成的事件而非持久的实体构成。

[2] Whitehead, A.N. (1925). [*Science and the Modern World*](https://www.amazon.com/Science-Modern-World-Alfred-Whitehead/dp/0684836394). Macmillan.
  - 更早的著作，为过程形而上学确立了哲学语境。

[3] Holmes, A. (2015). [A History of Philosophy - 61 Whitehead's Process Philosophy](https://www.youtube.com/watch?v=ysTx1c5-A7s). Wheaton College.
  - 怀特海过程哲学的易读入门。

[4] Holmes, A. (2015). [A History of Philosophy - 62 Whitehead's Process Theology](https://www.youtube.com/watch?v=Hwgoi0cvwmU). Wheaton College.
  - 探索过程哲学的神学意涵。

[5] Wittgenstein, L. (1921). [*Tractatus Logico-Philosophicus*](https://www.amazon.com/Tractatus-Logico-Philosophicus-Routledge-Classics-123/dp/0415254086). Routledge.
  - 可说/可显示/沉默这一区分的出处，构成我们认知论层的根基。

[6] Sowa, J.F. (1984). [*Conceptual Structures: Information Processing in Mind and Machine*](https://www.amazon.com/Conceptual-Structures-Information-Processing-PROGRAMMING/dp/0201144727). Addison-Wesley.
  - 第 2 层图结构知识表示的基础。

[7] Gärdenfors, P. (2000). [*Conceptual Spaces: The Geometry of Thought*](https://www.amazon.com/Conceptual-Spaces-Geometry-Thought-Bradford/dp/0262572192). MIT Press.
  - 概念表示的几何化路径，支持相似性与典型性推理。

[8] Park, J.S., et al. (2023). [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442). *UIST*.
  - 记忆增强的代理架构，接近但未达到基于过程的本体论。

[9] Packer, C., et al. (2023). [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560). *arXiv*.
  - 面向 LLM 代理的显式记忆管理；基于实体但很精巧。

[10] Hickey, R. (2012). [The Value of Values](https://www.infoq.com/presentations/Value-Values/). JVM Language Summit.
  - 纪元时间模型与不可变数据结构，作为实用的怀特海式事态。

[11] Hewitt, C. (2010). [Actor Model of Computation](https://arxiv.org/abs/1008.1459). *arXiv*.
  - Actor 作为生成的代理；消息作为分布式系统中的摄握。

[12] Varela, F.J., Thompson, E., & Rosch, E. (1991). [*The Embodied Mind: Cognitive Science and Human Experience*](https://www.amazon.com/Embodied-Mind-Cognitive-Science-Experience/dp/0262720213). MIT Press.
  - 生成主义：认知作为施行的过程，而非静态的表示。

[13] Miessler, D. (2025). [Personal AI Infrastructure (PAI)](https://github.com/danielmiessler/Personal_AI_Infrastructure). GitHub.
  - 参考实现，展现出通过工程优化发现的过程式模式。

[14] Giarratano, J.C. & Riley, G.D. (2004). [*Expert Systems: Principles and Programming*](https://www.amazon.com/Expert-Systems-Principles-Programming-Fourth/dp/0534384471). Thomson Course Technology.
  - 专家系统的全面论述，包括前向链与后向链推理机制。

[15] Hughes, G.E. & Cresswell, M.J. (1996). [*A New Introduction to Modal Logic*](https://www.amazon.com/New-Introduction-Modal-Logic/dp/0415126002). Routledge.
  - 模态逻辑的标准教材，涵盖必然性、可能性与道义算子。
