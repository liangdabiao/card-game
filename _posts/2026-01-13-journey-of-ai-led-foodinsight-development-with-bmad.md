---
layout: post
title: 用 BMAD 走完 AI 主导的 FoodInsight 开发之旅
date: 2026-01-13 12:00:00.000000000 -05:00
permalink: journey-of-ai-led-foodinsight-development-with-bmad
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- AI
- Software Development
- BMAD
- Edge Computing
- Food Detection
- Claude Code
category: post
comments: true
image: images/journey-of-ai-led-foodinsight-development-with-bmad/post_cover.jpg
images:
  cover: images/journey-of-ai-led-foodinsight-development-with-bmad/cover2x.jpg
  header: images/journey-of-ai-led-foodinsight-development-with-bmad/header.jpg
  animated-cover: images/journey-of-ai-led-foodinsight-development-with-bmad/animated-cover.webp
published: true
series: "FoodInsight"
series_order: 3
---

<!--excerpt.start-->
如果一个软件项目里最高产的工程师自己一行代码都没写过，会怎样？FoodInsight 证明这并非假设。3 天内交付 29 个故事、98 个故事点、约 5000 行代码，而人类监督只花了约 20 小时——这套边缘 AI 食物监测系统展示了当 AI 代理驱动整个开发生命周期时，一切皆有可能。
<!--excerpt.end-->

![FoodInsight 主视觉]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/FoodInsight_Hero.jpg)

_图：边缘 AI 食物检测：一台树莓派摄像头监视餐桌，用 YOLO11 实时检测日式菜肴。每道菜都有边界框和置信度分数，全部在本地处理。_

本文记录了 FoodInsight——一套完全通过 AI 主导的软件开发构建的边缘 AI 食物监测系统——的完整开发历程。从需求收集到实现，AI 代理编排了整个生命周期，展示了人机协作工程的新范式。

## 问题

办公室茶水间和自助餐厅有一个共同的烦恼：没人知道有哪些零食，除非走过去看。人工盘点既繁琐又很快被放弃。现有的智能方案依赖云服务，摄像头画面离开本地会引发隐私顾虑。

挑战：构建一套食物监测系统，**智能**到能自动检测物品，**私密**到所有图像都留在本地，**简单**到能在廉价的边缘硬件上运行。

## 愿景

FoodInsight 是一套隐私优先、本地优先的食物库存监测系统，在树莓派边缘设备上使用 YOLO11 目标检测。系统实时检测食物、追踪库存变化，并提供面向消费者的 PWA 查询可用性。

**关键设计原则：**
- **边缘优先**：全部推理都在树莓派本地完成
- **隐私优先**：只有元数据（计数，而非图像）离开设备
- **本地优先**：SQLite 数据库，无需云依赖

![FoodInsight 架构]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/FoodInsight_Architecture_Diagram_Pro.jpg)

_图：本地优先架构：相机帧流经边缘设备上的运动检测和 YOLO11n 推理。只有库存增量（计数，而非图像）推送到本地 SQLite 后端。PWA 轮询更新。_

但用 AI 代理构建这么复杂的系统，光靠提示词是不够的。没有结构，我们会重蹈"黑箱"问题的覆辙——正是它让企业团队对 AI 生成的代码心存戒备。它需要一套结构化的方法论。于是 BMAD 登场。

## 为什么是 BMAD？抽象-控制的取舍

投资 AI 辅助开发的企业领导者常常发现，承诺的生产力收益代价高昂：治理、可追溯性与架构完整性的丧失。无结构的、提示词驱动的 AI 会制造难以维护、审计和扩展的"黑箱"代码库。

![从提示者到架构师]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/BMAD_Prompter_vs_Architect_Pro.jpg)

_图：范式转变：对单体 AI 的临时提示产生脆弱的、脱离上下文的结果。BMAD 把开发者从"提示者"提升为"智能系统的架构师"，围绕问题领域编排专门化的 AI 代理。_

### 抽象陷阱

随着编程抽象的层级不断提高——从汇编到高级语言再到自然语言提示词——开发者获得了速度，却面临相应的**控制力丧失**。有了 AI 生成的代码，这种取舍显著加深。

![抽象与控制的层级]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/Level_of_Abstraction_and_Control_Pro.jpg)

_图：更高的抽象加速开发，却牺牲控制力。AI 工具放大了速度，代价是精确性与可问责性。_

其哲学根基，正如 Harrison Ainsworth 的 "Tractatus Computo Philosophicus" 所言："软件是一种逻辑构造，其正确性最终必须交由人类判断" [1]。当前的 AI 工具虽然强大，但随着抽象增加会造成明显的控制力丧失，让开发者难以保证精确性、追踪意图、放心地修改 AI 生成的方案。

### BMAD：从混沌到清晰

**BMAD 方法**（Breakthrough Method for Agile AI-Driven Development）提供了重建控制的关键框架 [2]。它利用专门化的 AI 代理，每个代理承载特定角色：分析师、产品经理、架构师、Scrum 主管、产品负责人、开发者和 QA。

真正的变革在于为这一代理式流程叠加健壮的**基于 Git 的版本控制**。从 PRD 到架构再到细粒度故事，每个工件都被当作版本化资产：

| 收益 | 描述 |
|---------|-------------|
| **可追溯性** | 来自人类或 AI 的每次变更都被追踪，消除黑箱效应 |
| **协作评审** | 故事经过严格的 PR 评审，人类检查 AI 生成的内容 |
| **单一事实来源** | 所有工件版本化并集中管理；整个团队基于一致的蓝图运作 |
| **生产力提升** | 自动化创建高保真工件，加速开发周期 |

![BMAD 工件控制流]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/BMAD_Artifact_Control_Flow_Pro.jpg)

_图：控制回路：人类把控制清单（PRD、架构文档、故事规格）提交到 Git。AI 代理生成代码时读取这些约束，代码随后经过自动化验证。人类撰写的工件支配 AI 的行为。_

FoodInsight 是对这一理论框架的实践验证。以下各节记录 BMAD 代理如何驱动完整的开发生命周期。

## 开发过程中的 AI 代理

方法论确立后，究竟是谁构建了 FoodInsight？不是写代码的人类开发者，而是一支由专门化 AI 代理组成的团队，每个代理都有独特的专长与个性。开发利用了 **BMAD** 工作流系统，这些拥有独特 persona 的代理驱动了开发的每个阶段 [2]。

![BMM SDLC 代理团队]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/BMM_SDLC_Agents_Team.jpg)

_图：SDLC 团队聚集在他们的 scrum 房间里：从 Mary 的分析洞见到 Sally 的用户代言，每位专家都为软件开发生命周期带来独到的专长。_

| 代理 | 名字 | 独到之处 |
|-------|------|------------------------|
| **analyst** | Mary | 通过系统性调查发掘隐藏需求 |
| **architect** | Winston | 为长寿命设计系统，在优雅与约束之间取得平衡 |
| **dev** | Amelia | 把架构翻译为生产级代码 |
| **pm** | John | 探究型策略师，把功能请求当作待验证的假设 |
| **po** | Sarah | 以毫不留情的优先级在利益相关者与开发团队之间架桥 |
| **sm** | Bob | 引导团队流程，移除阻塞，让冲刺保持聚焦 |
| **ux-expert** | Sally | 用匹配心智模型的界面为用户代言 |

### 阶段 1：需求与分析

**分析师 Mary** 主导了初始的需求收集：

| 交付物 | 代理 | 产出 |
|-------------|-------|--------|
| 产品需求文档 | Mary (Analyst) | `FoodInsight PRD.md` |
| 用户故事与验收标准 | Mary + Sarah (PO) | Epic 拆分 |
| 市场分析 | Mary | 竞争格局 |

PRD 捕捉了：
- 问题陈述（人工库存盘点的低效）
- 目标用户（办公室管理员、自助餐厅员工）
- 核心功能（检测、追踪、告警）
- 成功指标（准确性、延迟、用户采用率）

### 阶段 2：架构与设计

**架构师 Winston** 设计了技术基础：

| 交付物 | 代理 | 产出 |
|-------------|-------|--------|
| 解决方案架构 | Winston (Architect) | `FoodInsight Solution Architecture.md` |
| 系统组件 | Winston | 检测流水线、API 设计 |
| 技术选型 | Winston | YOLO11n、FastAPI、Vue 3、SQLite |

**UX 设计师 Sally** 创建了用户体验：

| 交付物 | 代理 | 产出 |
|-------------|-------|--------|
| UX 规格 | Sally (UX Expert) | `FoodInsight UX Specification.md` |
| 设计系统 | Sally | "Playful & Vibrant" 主题 |
| 组件库 | Sally | 卡片、徽章、动画 |

### 阶段 3：Epic 拆分

**产品经理 John** 与 **产品负责人 Sarah** 把工作拆分为 6 个 epic：

| Epic | 故事 | 点数 | 焦点 |
|------|---------|--------|-------|
| Epic 1 | 6 | 21 | 边缘检测系统 |
| Epic 2 | 5 | 18 | 后端与消费端应用 |
| Epic 3 | 4 | 13 | UX 基础 |
| Epic 4 | 4 | 13 | UX 惊喜（动画） |
| Epic 5 | 4 | 11 | UX 打磨（无障碍） |
| Epic 6 | 6 | 22 | 边缘管理端 UX 改版 |

**合计：29 个故事，98 个故事点**

## 用并行工程师代理实现

这里是 AI 主导开发真正有趣的地方。我们不是让一个代理按顺序处理故事，而是分析依赖关系，同时放出多个代理。结果如何？三天的串行工作被压缩为一天。

实现阶段使用 Claude Code 的**并行代理架构**来最大化效率 [3]。多个工程师代理同时在相互独立的故事上工作。

### 依赖图分析

在启动并行工作之前，系统分析了故事之间的依赖：

```
Epic 6 Dependency Graph:
E6-S1 (Design Tokens) ──┬── E6-S2 (Status Cards)
                        ├── E6-S3 (Camera Preview)
                        └── E6-S4 (Dark Mode)
                              │
E6-S2 + E6-S3 ──────────────── E6-S5 (Responsive Layout)
                              │
E6-S5 ─────────────────────── E6-S6 (Accessibility)
```

### 并行执行策略

```
Wave 1: E6-S1 (Design Tokens) - Foundation
Wave 2: E6-S2, E6-S3, E6-S4 - Parallel (3 agents)
Wave 3: E6-S5 (Responsive) - Depends on Wave 2
Wave 4: E6-S6 (Accessibility) - Final integration
```

这种方式通过并发执行相互独立的故事，缩短了总实现时间。

## 迭代打磨循环

人类反馈驱动快速迭代：

### 示例：状态卡片优化

**初始实现：**
- 全高状态卡片配大号字体
- 垂直空间占用过多

**用户反馈：**"状态卡片太高了，占了太多纵向空间"

**AI 响应：**
```css
/* Before */
.status-card { padding: var(--space-lg); }
.status-card__value { font-size: 2.5rem; }

/* After */
.status-card { padding: var(--space-sm) var(--space-md); }
.status-card__value { font-size: 1.5rem; }
```

### 示例：布局重排

**用户反馈：**"把设备信息放到摄像头预览上面"

**AI 响应：**重构 HTML 布局，把静态内容移到动态摄像头区块之上，改善了信息层级。

### 示例：Emoji 增强

**用户反馈：**"总是显示爆米花 emoji 太无聊了"

**AI 响应：**把 emoji 映射从 12 项扩展到 60 多种食物，并使用大小写不敏感的查找。最让我们惊讶的是速度：一句抱怨在几分钟内就变成了 60 多项的查找表。没有开单，没有写规格，只有即时响应：

```typescript
const emojiMap: Record<string, string> = {
  // Snacks
  chips: '🥔', donut: '🍩', cookie: '🍪',
  // Drinks
  bottle: '🍶', water: '💧', soda: '🥤',
  // Fruits
  apple: '🍎', banana: '🍌', orange: '🍊',
  // ... 60+ mappings
  default: '🍿'
}
```

注意这些例子中的共同模式：人类反馈聚焦于*改什么*，从不*怎么改*。AI 处理实现细节，让人类得以专注于用户体验与设计意图。

## 技术实现亮点

### 边缘检测流水线

![检测流水线]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/FoodInsight_Detection_Pipeline_Pro.jpg)

_图：相机帧以 30 FPS 流经运动检测，运动检测为昂贵的 YOLO11n 推理设置闸门。只有检测到运动的帧才进入目标检测，随后 ByteTrack 负责持续追踪，最后进入库存状态管理。_

### 管理门户（Flask + HTMX）

- 基于 HTMX 轮询的实时状态卡片
- 实时 MJPEG 相机预览
- ROI 配置 UI
- 基于 CSS 自定义属性的深色模式支持

![边缘管理仪表盘]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/FoodInsight_Admin_Dashboard_Full.jpg)

_图：边缘管理仪表盘：实时检测视图，显示带 YOLO11 边界框的相机画面、紧凑的状态卡片、设备信息、持久化的库存计数和最近事件日志。_

### 消费端 PWA（Vue 3 + Vite）

- Pinia 状态管理
- 实时库存轮询
- 动画状态徽章
- "补货"庆祝动效

![消费端 PWA]({{ site.baseurl }}/images/journey-of-ai-led-foodinsight-development-with-bmad/FoodInsight_Test_App_01.jpg)

_图：消费端 PWA 展示可用库存：实时显示检测到的物品及其数量、状态徽章和食物专属 emoji 图标。绿色边框表示有货。_

### 设计系统

用 CSS 自定义属性实现一致的主题：

```css
:root {
  --snack-purple: #8b5cf6;
  --fresh-green: #10b981;
  --warning-orange: #f59e0b;
  --space-sm: 8px;
  --space-md: 16px;
  --radius-lg: 16px;
}
```

## 文档与知识共享

AI 代理驱动的不仅是代码，还有成体系的文档。它们把知识捕获当作 SDLC 的有机组成部分。

### 项目文档

| 文档 | 用途 |
|----------|---------|
| README.md | 项目概览、快速上手、API 参考 |
| Server README | FastAPI 后端文档 |
| App README | Vue 3 PWA 文档 |
| Epic 故事 | 详细的实现规格 |

### 截图集成

捕获并集成了运行中组件的真实截图：
- 管理仪表盘（检测叠加层、状态卡片）
- ROI 配置 UI
- 消费端 PWA（有货与缺货状态）

### 技术博文

撰写了两篇配套博文，与更广泛的社区分享这段旅程：

1. **[From YOLO v2 to YOLO11: Food Detection Evolution]({{ site.baseurl }}/from-yolo-v2-to-yolo11-food-detection-evolution)** [4]
   - 训练定制食物检测模型
   - 数据集准备（UECFOOD100）
   - 各 YOLO 版本性能对比

2. **[FoodInsight: Edge AI Food Monitoring]({{ site.baseurl }}/foodinsight-edge-ai-food-monitoring)** [5]
   - 完整系统架构
   - 隐私优先的设计原则
   - 实现指南

## 关键收获

### 效果好的地方

1. **BMAD 代理 persona**：独特的代理角色（分析师、架构师、UX）产出了聚焦、高质量的工件
2. **并行执行**：依赖感知的并行化显著缩短了实现时间
3. **迭代反馈**：快速的人类反馈循环支持快速打磨
4. **文档先行**：PRD 与架构文档为实现提供了清晰指导

### 挑战与对策

| 挑战 | 对策 |
|-----------|----------|
| 代理上下文限制 | 以清晰的摘要做结构化交接 |
| 代理之间的一致性 | 设计 token 与风格指南 |
| 集成测试 | 故事中定义端到端测试场景 |

### 人机协作模式

```
Human: High-level intent ("make cards thinner")
   ↓
AI: Technical translation (reduce padding, font-size)
   ↓
Human: Visual validation (review changes)
   ↓
AI: Commit and document
```

## 项目成果

### 指标

| 指标 | AI 主导 | 传统估算 |
|--------|--------|---------------------|
| 故事总数 | 29 | 29 |
| 故事点 | 98 | 98 |
| 组件 | 3（边缘、后端、PWA） | 3 |
| 代码行数 | ~5000+ | ~5000+ |
| 开发周期 | ~3 天 | ~6-8 周 |
| 人类工时 | ~20 小时 | ~400+ 小时 |
| AI 代理会话 | 50+ | N/A |

人类工时 20 对 400+ 的悬殊对比，正是核心价值主张。人类从写代码转向指挥 AI 代理并验证产出。

### 产出的工件

- 产品需求文档
- 解决方案架构
- UX 规格
- 6 个 Epic 规格、29 个故事
- 完整实现（Python、TypeScript、CSS）
- 文档与 README 文件
- 2 篇技术博文

## 复制这一方法

对想采用 AI 主导开发的团队：

1. **从 BMAD 代理起步**：写任何代码之前，先用 Mary（分析师）做需求。PRD 是人类意图与 AI 执行之间的契约。

2. **一切皆版本化**：PRD、架构、故事和代码全部进入 Git。这创造了让 AI 生成的工件变得可信的审计轨迹。

3. **先映射依赖**：在放出并行代理之前，弄清哪些故事依赖其他故事。没有依赖意识的并行执行会制造集成噩梦。

4. **保持人类在环**：视觉校验能抓住 AI 漏掉的问题。模式是：AI 提议，人类评审，AI 打磨。

5. **快速迭代**：小的反馈循环（"把卡片改薄一点"）胜过大的前期规格。只要方向清晰、聚焦，AI 极擅长快速迭代。

6. **边走边写文档**：把文档当作 SDLC 的一部分，而不是事后补记。AI 代理可以随代码一起生成 README、API 文档和博文。

## 如果我们没用 BMAD 会怎样？

没有结构化的方法论会发生什么？我们大概会有一个能跑的原型，但没有架构文档、各组件代码风格不一致，以及那个可怕的"它能跑但没人知道为什么"的问题。

PRD 与架构文档成了我们的安全网。每一行 AI 生成的代码都能追溯到经过人类验证的需求。当我们需要理解某个组件为何如此设计时，答案在 Git 里，而不是在某人的脑子里。

没有 BMAD 的代理 persona，我们也会失去专门化的聚焦。Mary 的分析师视角抓住了"把这个功能写出来"这类泛泛提示词会错过的需求。Winston 的架构严谨性防止了快速原型常见的债务累积。这些 persona 不只是角色扮演；它们是把结构化思维应用到每个阶段。

## 结语

FoodInsight 证明：只要编排得当，AI 代理可以有效驱动整个软件开发生命周期。BMAD 工作流系统提供了结构，并行工程师代理实现了高效开发。人类监督对验证与创意方向依然不可或缺，但分析、设计与编码的重活被成功地委托给了 AI。

FoodInsight 的历程指向一种新的分工：**AI 代理处理软件开发的机械性环节**——包括分析、设计、编码与文档，**而人类提供创意方向、质量判断与领域专长。**

> **"这不是 AI 取代开发者；而是 AI 放大了开发者能达成的成就。"**

这个项目为未来的 AI 主导开发计划提供了模板：展示专门化的代理 persona、清晰的文档与迭代反馈循环如何结合，产出生产级质量的软件。

---

## 参考文献

[1] Harrison Ainsworth. [*Tractatus Computo Philosophicus*](https://www.hxa.name/articles/content/tractatus-computo-philosophicus_hxa7241_2010.html). HXA7241, 2010.
  - 关于"软件是需要人类判断的逻辑构造"的哲学框架

[2] BMad Code. [*BMAD Method*](https://github.com/bmad-code-org/BMAD-METHOD/tree/main). GitHub, 2025.
  - 敏捷 AI 驱动开发的突破性方法框架

[3] Anthropic. [*Claude Code*](https://claude.ai/claude-code). Anthropic, 2025.
  - 具备并行代理能力的 CLI AI 编码助手

[4] Benny Cheung. [*From YOLO v2 to YOLO11: Food Detection Evolution*]({{ site.baseurl }}/from-yolo-v2-to-yolo11-food-detection-evolution). Benny's Mind Hack, 2026.
  - 用各版本 YOLO 训练定制食物检测模型

[5] Benny Cheung. [*FoodInsight: Edge AI Food Monitoring*]({{ site.baseurl }}/foodinsight-edge-ai-food-monitoring). Benny's Mind Hack, 2026.
  - 完整系统架构与实现指南

[6] Benny Cheung. [*Applied BMAD: Reclaiming Control in AI Development*]({{ site.baseurl }}/bmad-reclaiming-control-in-ai-dev). Benny's Mind Hack, 2025.
  - BMAD 方法的实践应用

[7] Benny Cheung. [*Harmonizing Two AI Agent Systems: A Practical Journey*]({{ site.baseurl }}/harmonizing-two-ai-agent-systems). Benny's Mind Hack, 2025.
  - 将 BMAD 与个人 AI 基础设施集成

[8] Daniel Miessler. [*Building a Personal AI Infrastructure (PAI)*](https://danielmiessler.com/blog/personal-ai-infrastructure). Daniel Miessler Blog, 2025.
  - 个人 AI 基础设施的基础概念

[9] FoodInsight Project. [*GitHub Repository*](https://github.com/bennycheung/FoodInsight). GitHub, 2026.
  - FoodInsight 系统完整源码
