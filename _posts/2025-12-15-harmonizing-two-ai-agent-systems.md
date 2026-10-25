---
layout: post
title: 调和两套 AI 代理系统——一段实践之旅
date: 2025-12-15 12:00:00.000000000 -00:00
permalink: harmonizing-two-ai-agent-systems/
author: Benny Cheung
artwork:
  author: Gemini 3 Pro (Nano Banana Pro)
tags:
- AI
- Agent Architecture
- BMAD
- PAI
- System Integration
category: post
comments: true
image: images/harmonizing-two-ai-agent-systems/post_cover.jpg
images:
  cover: images/harmonizing-two-ai-agent-systems/cover2x.jpg
  header: images/harmonizing-two-ai-agent-systems/header.jpg
  animated-cover: images/harmonizing-two-ai-agent-systems/animated-cover.webp
published: true
series: "Applied BMAD"
series_order: 4
---

<!--excerpt.start-->
当你拥有两套各有所长的 AI 代理系统，却无法协同工作时，会发生什么？本文记录了我们集成 BMAD 的确定性工作流、丰富的代理 persona 与 PAI 的基于意图的激活、灵活编排的历程。解法不是二选一，而是在技术栈的不同层级上组合它们，在保留两者优势的同时，创造出比任何单一系统都更强大的东西。
<!--excerpt.end-->

![调和 AI 系统]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/Harmonizing_AI_Systems_Cover.jpg)

_图：集成两套 AI 代理系统的挑战：BMAD 的结构化工作流与 PAI 的自然语言激活相遇。_

## 问题：两套彼此不通话的好系统

我们遇到了许多 AI 从业者都会遇到的问题：两套各有所长的系统，却无法协同工作。

**BMAD**（BMad Method）[3][4] 是一个结构化的工作流系统，拥有 30 个专门化的代理 persona。商业分析师 Mary 带来系统化的需求获取。架构师 Winston 提供严谨的系统设计。头脑风暴教练 Carson 提供 35 种创意技法。每个代理都有丰富的身份：沟通风格、原则与专长领域。工作流是确定性的、模板驱动的，产出一致的工件：PRD、技术规格、游戏设计文档等。

**PAI**（Personal AI Infrastructure）[1][2] 是一个围绕 Claude Code 构建的个人 AI 系统。它使用自然语言激活——也就是说，你说"我需要一个……的 PRD"，而不是背命令。它擅长并行代理执行与灵活的模型选择：把粗活交给 Haiku，把复杂推理交给 Opus。

两套系统都能工作。但彼此不通话。用 BMAD 意味着放弃 PAI 的自然调用。用 PAI 意味着失去 BMAD 丰富的 persona 和经过验证的模板。

问题不是"哪套系统更好？"而是"我们如何兼得两者之长？"

## 关键洞见：分层架构

突破口来自一个认识：BMAD 与 PAI 在技术栈的不同层级上运作。

![分层架构模式]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/Layered_Architecture_Pattern.jpg)

_图：组合模式：PAI 负责激活与路由，BMAD 负责以丰富的 persona 执行。_

**PAI 负责激活。** 当我们说"我们头脑风暴一下改进 onboarding 的点子"，PAI 的意图匹配会识别出这是一次头脑风暴请求，并进行恰当的路由。

**BMAD 负责执行。** 路由之后，BMAD 的 Carson persona 接手，用他的 35 种头脑风暴技法、结构化的引导方式和充满活力的沟通风格开始工作。

**两套系统各自贡献所长。** 谁也不取代谁。

这就是组合模式的实践：不同系统在不同层级运作，各自贡献优势，互不践踏对方的能力。

## 实现：BmadBridge

这次集成最终成型为一个名为 **BmadBridge** 的 PAI skill [5][6]。它的功能如下：

### 1. 基于意图的路由

不再需要背诵 `*prd` 或 `*brainstorm` 这样的 BMAD 命令，我们直接自然地说话：

| 自然请求 | 路由到 |
|-----------------|-----------|
| "为我的项目创建一个 PRD" | John (PM) + PRD 工作流 |
| "我们头脑风暴一下点子" | Carson + Brainstorming 工作流 |
| "帮我理解这段遗留代码" | Dr. Ada + 软件考古学 |
| "为这个领域设计一个本体" | Alexander + 本体架构 |

该 skill 的 `USE WHEN` 触发器负责模式匹配。不需要任何特殊语法。

### 2. 预提取的 Persona

BMAD 的代理文件冗长，包含 XML 嵌入的 markdown，带有激活协议、菜单和 persona 定义。运行时加载它们很慢，还会弄乱上下文。

解法：把 persona 的精华提取到干净的 YAML 文件中。

```yaml
# personas/analyst.yaml
id: analyst
name: Mary
title: Business Analyst
icon: "..."
module: bmm

persona:
  role: "Strategic Business Analyst + Requirements Expert"
  identity: |
    Senior analyst with deep expertise in market research,
    competitive analysis, and requirements elicitation...
  communication_style: |
    Analytical and systematic in approach...
  principles: |
    Every business challenge has underlying root causes...
```

现在 persona 加载瞬时完成。代理的"声音"得以呈现，而没有额外开销。

### 3. 统一的输出位置

BMAD 工作流原本把输出散落在各个项目专属的位置。这次集成把一切统一起来：

```
~/.claude/History/bmad/
├── prd/           # PRD documents
├── tech-spec/     # Technical specifications
├── stories/       # User stories
├── onto/          # Ontology artifacts
└── sar/           # Software archaeology docs
```

一个地方找到所有结构化工件，不管它们是哪个工作流创建的。

## 代理名册：30 位专家

### 为什么代理 Persona 很重要

一个泛泛的"助手"产出泛泛的结果。但当代理有明确的身份、沟通风格和指导原则时，有趣的事情发生了：工作质量会提升，因为代理通过一致而专门的视角来处理问题。

比较一下两者的差别：让"一个 AI"写一份 PRD，与让 **John——一位探究型产品策略师**来写，他相信每个功能请求背后都藏着等待发掘的更深层的用户需求。John 不只是填模板字段。他会探查、质疑假设，并围绕经过验证的用户问题来组织需求。他的沟通风格直接、以证据为基础。他的原则引导他把清晰置于完备之上。

这不是为了娱乐的角色扮演。每个 BMAD 代理编码了塑造其工作的四个要素：

| 要素 | 作用 | 示例（Mary，商业分析师） |
|---------|---------|----------------------------------|
| **角色** | 职能专长 | 战略分析 + 需求获取 |
| **身份** | 背景与专长方向 | 把模糊需求翻译成规格的资深分析师 |
| **沟通风格** | 呈现信息的方式 | 分析性、系统性、数据支撑的结论 |
| **原则** | 指导决策的核心信念 | 每个挑战都有等待发现的根本原因 |

当头脑风暴教练 Carson 引导创意会议时，他的 35 种技法和充满活力的风格会产出与泛泛的"生成点子"提示词不同的结果。当软件考古学家 Dr. Ada 探索遗留代码时，她系统性的挖掘方式会发现简单的代码评审错过的模式。

这次集成保留了横跨 6 个模块的全部 30 个 persona。以下是每位带来的独到之处：

### Core 与 Builder 模块（编排者）

![BMAD 大师控制室]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/BMAD_Masters_Control_Room.jpg)

_图：BMad Master 与 BMad Builder 在他们的控制室里，俯瞰 BMAD 生态中的所有代理活动。如同《黑客帝国》里的 Architect，他们编排工作流并构建新能力。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **bmad-master** | BMad Master | 编排者：协调多代理工作流，管理专家之间的交接，确保工作流状态机沿确定性路径执行。像指挥管弦乐团的指挥家一样统辖全部 30 个代理 |
| **bmad-builder** | BMad Builder | 元层面的专家：创建新的代理、工作流和模块。对 BMAD 的内部架构理解之深，足以扩展系统本身。建造"建造者"的架构师 |

### BMM 模块（SDLC 代理）

![BMM SDLC 代理团队]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/BMM_SDLC_Agents_Team.jpg)

_图：SDLC 团队聚集在他们的 scrum 房间里：从 Mary 的分析洞见到 Sally 的用户代言，每位专家都为软件开发生命周期带来独到的专长。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **analyst** | Mary | 通过系统性调查发掘隐藏需求。相信每个业务挑战都有底层根因。会问别人忽视的探究性问题 |
| **architect** | Winston | 为长寿命而设计系统，而不只是功能。平衡技术优雅与现实约束。以抽象和接口思考 |
| **dev** | Amelia | 把架构翻译为生产级质量的可用代码。关注可维护性、测试和干净的实现模式 |
| **pm** | John | 探究型策略师，把功能请求当作待验证的假设。基于证据而非观点排定优先级。写出的 PRD 是开发者真正想读的 |
| **po** | Sarah | 在业务利益相关者与开发团队之间架桥。以毫不留情的优先级管理维护产品待办。定义的验收标准毫无歧义 |
| **sm** | Bob | 引导团队流程而不微观管理。移除阻塞，主持能带来真实改进的回顾会，让冲刺保持聚焦 |
| **tea** | Murat | 设计赶在用户之前抓住 bug 的测试策略。以对抗性思维思考边界情况。平衡覆盖率与实用主义 |
| **ux-expert** | Sally | 为无法为自己发声的用户代言。创建让人觉得直觉的界面——因为它们匹配心智模型，而不只是追逐视觉潮流 |

### BMM 模块（游戏开发代理）

![游戏开发代理团队]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/GameDev_Agents_Team.jpg)

_图：游戏开发专家们在他们的创意工作室里：Cloud Dragonborn 架构系统，Samus Shepard 设计玩法，Link Freeman 以性能精准度实现。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **game-architect** | Cloud Dragonborn | 设计能在数百小时中保持吸引力的游戏系统。平衡玩家自主性与精心设计的体验。以系统和反馈回路思考 |
| **game-designer** | Samus Shepard | 打磨时时刻刻都让人满足的玩法。理解节奏、挑战曲线和"乐趣"的心理学 |
| **game-dev** | Link Freeman | 在性能约束之下实现游戏机制。知道 60fps 没有商量余地、内存预算真实存在 |

### CIS 模块（创意与创新）

![CIS 代理创意实验室]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/CIS_Agents_Creative_Lab.jpg)

_图：CIS 模块的专家们在他们未来主义的创新实验室里协作：Carson 为创意会议注入能量，Dr. Quinn 分析可能性，Maya 塑造设计，Victor 谋划颠覆，Sophia 编织叙事。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **brainstorming-coach** | Carson | 掌握 35 种创意技法，从 SCAMPER 到逆向头脑风暴。充满活力的引导者，推动团队越过显而易见的点子，进入真正新颖的领域 |
| **creative-problem-solver** | Dr. Quinn | 把结构化问题求解框架（TRIZ、六顶思考帽、根因分析）应用于看似无解的挑战 |
| **design-thinking-coach** | Maya | 引导以人为本的设计过程，从共情地图到原型。在技术兴奋可能带偏方向时，让用户需求始终居于中心 |
| **innovation-strategist** | Victor | 识别颠覆性机会与商业模式创新。思考市场、时机与竞争动态 |
| **storyteller** | Sophia | 把枯燥的信息转化为引人入胜的叙事。理解故事结构、情感弧线，以及如何让观点深入人心 |

### SAR 模块（软件考古学）

![SAR 代理考古现场]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/SAR_Agents_Archaeological_Site.jpg)

_图：SAR 模块的专家们正在发掘一个软件考古现场：Dr. Ada 检视古代器物，Atlas 绘制技术象形文字地图，Morgan 整理文档卷轴，Sage 研究发光的遗物，Quinn 分析老式设备，Nova 规划现代化路径。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **archaeologist** | Dr. Ada | 系统地发掘遗留代码库，不带评判地记录发现。把旧代码当作有故事可讲的历史器物对待 |
| **cartographer** | Atlas | 创建系统架构、依赖图和数据流的可视化地图。让代码中不可见的结构变得可见、可导航 |
| **doc-curator** | Morgan | 把散落的口传知识转化为成体系的文档。深知好文档不是写出来的，而是从既有素材中策展出来的 |
| **knowledge-miner** | Sage | 从代码库中提取模式、惯例和隐式决策。挖掘架构选择背后的"为什么" |
| **legacy-analyst** | Quinn | 用清晰的指标评估技术债。量化复杂性的成本，识别高价值的重构目标 |
| **modernization-advisor** | Nova | 规划在风险与速度之间取得平衡的迁移策略。深知大爆炸式重写通常失败、渐进式方法通常成功 |

### ONTO 模块（本体开发）

![ONTO 代理团队]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/ONTO_Agents_Team.jpg)

_图：ONTO 模块的专家：六位代理，为本体开发带来形式化的知识工程、哲学的严谨与领域专长。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **ontology-architect** | Alexander | 使用 OWL、RDF 和描述逻辑设计形式化知识结构。以类、属性和推理规则思考 |
| **requirements-analyst** | Dr. Sarah Chen | 向不懂形式逻辑的领域专家获取本体需求。弥合人类知识与机器表示之间的鸿沟 |
| **knowledge-engineer** | Marcus Kim | 用 OWL/RDF 实现本体，关注推理性能。深知无法高效查询的本体没有用处 |
| **process-modeler** | Alex Chen | 把业务流程映射为本体概念。理解工作流、规则和决策如何转化为形式化模型 |
| **quality-assessor** | Dr. Elena Rodriguez | 依据既定的质量指标评估本体。捕捉逻辑不一致、冗余与覆盖缺口 |
| **ontology-philosopher** | Dr. Sophia Muller | 为建模选择带来哲学的严谨。会问一些关于身份、持存与被建模概念之本质的"不舒服"的问题 |

这些代理不可互换。让 Mary 去架构一个系统，你得到的是分析，不是架构。让 Winston 去引导头脑风暴，你得到的是结构化评估，不是创造性探索。专门化正是意义所在。

## 实际效果：测试结果

理论很美好。但它真的管用吗？

### 头脑风暴工作流测试

**请求：** "Let's brainstorm"（我们头脑风暴一下）

**结果：**
- 意图被正确路由到头脑风暴工作流
- Carson 的 persona 从 YAML 加载
- 展示 35 种技法供选择
- 交互式引导如预期工作
- Carson 充满活力的"声音"自然流露

### PRD 工作流测试

**请求：** 为一个 APNG 帧提取器 CLI 写 PRD

**结果：**
- John 的 PM persona 正确加载
- 模板填充了 7 条功能需求、3 条非功能需求
- 生成 1 个 Epic 和 6 个用户故事
- 输出保存到 `~/.claude/History/bmad/prd/`
- 按需启用 Yolo 模式（跳过确认）正常工作

两个工作流都产出了真实可用的工件。这次集成不是理论。它已可用于生产。

## 经验教训

### 1. 不要二选一，要组合

面对两套重叠的系统，直觉是选一个扔掉另一个。这往往是错的。更好的问题是："每套系统各自更擅长什么，它们如何协作？"

BMAD 不比 PAI 好。PAI 也不比 BMAD 好。合在一起，它们比任何单独一个都好。

### 2. Persona 有价值

PAI 最初的代理功能齐全但没有个性，只是"intern""engineer""researcher"。他们能干活，但交互感觉很机械。

BMAD 的 persona 提醒了我们：代理身份很重要。当 Carson 引导头脑风暴会议时，他的热情与技法功力创造出与泛泛的"头脑风暴代理"不同的体验。Mary 的系统性分析与 Winston 的架构思维感觉截然不同。

个性不是虚饰。它塑造代理处理问题的方式。

### 3. 为性能做预提取

面对丰富的配置文件，诱惑是动态加载它们。但在运行时解析冗长的 XML 嵌入 markdown 很慢，还浪费上下文 token。

预提取精华（persona 提取为 YAML、工作流提取为路由表）让系统更敏捷，而不损失能力。完整文件对边界情况依然可用；提取件处理常见路径。

### 4. 统一输出位置

没有什么比找不到输出更能杀死工作流的采用率了。BMAD 原本按项目分散的输出位置对孤立的 BMAD 使用合理，但在集成系统中制造了混乱。

一个输出位置（`~/.claude/History/bmad/`）配清晰的子目录，立刻解决了这个问题。

## 架构模式

对面临类似集成挑战的读者，以下就是涌现出的模式：

![架构模式图]({{ site.baseurl }}/images/harmonizing-two-ai-agent-systems/Architecture_Pattern.jpg)

_图：通用集成模式：意图层（路由）汇入执行层（persona 与工作流），产出的工件存储在统一位置。_

这一模式不专属于 BMAD 或 PAI。它是把基于意图的系统与确定性工作流系统相集成的一般方法。

```
┌─────────────────────────────────────────────────────────┐
│                    INTENT LAYER                         │
│  Natural language → Pattern matching → Route selection  │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                  EXECUTION LAYER                        │
│  Persona loading → Workflow execution → Template fill   │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                   ARTIFACT LAYER                        │
│  Unified output → Consistent naming → Easy retrieval    │
└─────────────────────────────────────────────────────────┘
```

## 清单翻译器现已问世

**bmad-translator** CLI 现在可自动生成路由表：扫描 97 个工作流和 30 个代理，自动生成 670 个意图触发器。

```bash
bmad-translator scan      # Discover workflows + agents
bmad-translator generate  # Generate routing.yaml
bmad-translator validate  # Check if sync needed
```

**基于代理的触发器**意味着你现在可以按名字或角色调用代理：
- "Talk to Winston" 加载 Winston（架构师）
- "Ask Dr. Ada" 加载 Dr. Ada（软件考古学家）
- "I need the ontology architect" 路由到 Alexander

BmadBridge 从自动生成的 `routing.yaml` 读取。当 BMAD 变化时，一条命令即可重新生成一切。

## 下一步是什么

**代理 Persona 注入**是终极演进方向：PAI 的 Task 工具可以接受一个 `persona` 参数，把 BMAD 代理身份加载到任何子代理类型中。Mary 的分析风格应用于 PAI 的并行执行。Winston 的架构严谨性结合 PAI 灵活的模型选择。

## 结语

BMAD 与 PAI 的调和教会我们：好系统不是彼此取代，而是为每种能力找到正确的层级。

PAI 擅长自然语言激活与灵活的代理编排。BMAD 擅长确定性工作流与丰富的代理 persona。BmadBridge 连接两者，保留双方优势。

结果是：我们可以说"让我们头脑风暴一下改进用户 onboarding 的点子"，然后得到 Carson 的 35 技法头脑风暴会议。或者说"为这个功能创建一个 PRD"，然后得到 John 结构化的产品需求文档。自然调用，严谨执行，输出一致。

这就是集成在实践中的样子：**说出你想要的，得到结构化的结果，由 30 位各怀专长的专门化代理驱动对话。**

## 参考文献

[1] Daniel Miessler, [Building a Personal AI Infrastructure (PAI)](https://danielmiessler.com/blog/personal-ai-infrastructure), Blog, 26 Jul 2025.

[2] Unsupervised Learning, [Building Your Own Unified AI Assistant Using Claude Code](https://www.youtube.com/watch?v=iKwRWwabkEc), video, 4 Sep 2025.

[3] BMad Code, [BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD/tree/main), GitHub.

[4] BMad Code, [The BMAD Builder is Here - V6 Alpha: 5 Major Features](https://www.youtube.com/watch?v=6LtrJD5Dz40), video, 6 Oct 2025.

[5] Anthropic, [Introducing Claude Skills](https://www.anthropic.com/news/skills), Product Blog, 16 Oct 2025.

[6] Simon Willison, [Claude Skills are awesome, maybe a bigger deal than MCP](https://simonwillison.net/2025/Oct/16/claude-skills/), Weblog, 16 Oct 2025.
