---
layout: post
title: "在 Claude Code 中听你的 AI 代理干活"
date: 2026-01-25 12:00:00.000000000 -00:00
permalink: hear-your-ai-agents-work
author: Benny Cheung
artwork:
  author: Gemini 3 Pro
tags:
- AI
- Voice
- Text-to-Speech
- Claude Code
- Multi-Agent Systems
- Developer Experience
category: post
comments: true
image: images/hear-your-ai-agents-work/post_cover.jpg
images:
  cover: images/hear-your-ai-agents-work/cover2x.jpg
  header: images/hear-your-ai-agents-work/header.jpg
  animated-cover: images/hear-your-ai-agents-work/animated-cover.webp
published: true
---

<!--excerpt.start-->
当五个 AI 代理并行运行时，如果不盯着五个终端，你怎么知道发生了什么？答案出人意料地"低技术"：你用听的。通过为 Claude Code 添加语音通知，我们把无声的终端输出变成了环境化的、听觉感知的体验。本文展示如何用 hooks、一个本地服务器和 ElevenLabs API，为任何 Claude Code 代理构建语音通知系统。无论你运行的是 BMAD 工作流代理、自定义研究代理，还是你自己的多代理系统，你都能听到"研究完成"是一个声音，而"构建成功"是另一个声音。
<!--excerpt.end-->

----

![从终端散发出的声波]({{ site.baseurl }}/images/hear-your-ai-agents-work/Voice_System_Cover.png)

*图：AI 交互的未来，可能比我们预想的更"人"。有时最好的界面是声音。*

## 无声代理的问题

现代 AI 开发给了我们一件了不起的东西：把复杂任务委托给多个并行工作的 AI 代理的能力 [4]。Claude Code 的 Task 工具让我们可以孵化专门化的代理：翻文档的研究员、写代码的工程师、设计系统的架构师。BMAD [6] 这样的代理框架又增添了更多专门化 persona：拆解需求的分析师 Mary、设计方案的架构师 Winston。

但有个问题。所有这些代理都在无声地工作。

想象这个场景：你让 Claude Code 研究 3 家公司、分析一个代码库的安全问题、起草一份技术规格——全部并行。三个代理启动并开工。你切换去做别的任务、查邮件、泡咖啡。等你回来，你完全不知道哪些代理完成了、哪些还在工作、有没有谁遇到了问题。你唯一的选择是逐个检查终端、通读输出、拼凑当前状态。

![安全第一：沉默的危险]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_02.png)

*图：无声代理的盲区：混乱与失联，对比平静的听觉感知。*


不带语音通知地运行并行 AI 代理，就像让五个人在五个房间里干活，除了反复挨个查看，没有任何办法知道他们何时完成。

改变一切的洞见很简单：我们有耳朵。为什么不用？

## 解法：环境化的语音通知

![升级：环境音频感知]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_03.png)

*图：把 Claude Code 从纯视觉界面变为听觉感知。不同代理不同声音，实现被动监控。*

我们构建的解法把 Claude Code 从纯视觉界面变成环境化的、听觉感知的体验 [3]。当代理完成任务时，你会"听到"——字面意义上的。不同代理用不同声音说话，所以不用看屏幕你就知道谁完成了。

研究员用 Domi 分析性的音色宣布："Found 5 papers on AI reasoning techniques."

工程师用 Bella 平稳的声音报告："Refactored authentication module successfully."

架构师用 Antoni 战略性的节奏总结："Designed microservices architecture with 4 services."

这适用于任何 Claude Code 代理系统：
- **内置代理**：researcher、engineer、architect（经 Task 工具）
- **BMAD 代理** [6]：Mary、Winston 及其他工作流 persona
- **自定义代理**：任何遵循简单约定的、你自建的代理

让我们看看如何构建这个系统。

## 系统架构总览

语音通知系统由四个组件协同组成：

| 组件 | 角色 | 技术 |
|-----------|------|------------|
| **COMPLETED 行约定** | 代理用 12 个词总结工作 | 提示词工程 |
| **Hook 系统** | 检测代理完成，提取消息 | Claude Code hooks（TypeScript） |
| **语音服务器** | 接收通知，调用 TTS API | 本地 HTTP 服务器（Bun） |
| **ElevenLabs API** | 把文本转换为自然语音 | 外部 TTS 服务 |

![PAI 语音系统架构]({{ site.baseurl }}/images/hear-your-ai-agents-work/pai-voice-system-flowchart.png)

*图：受 PAI 框架启发的四组件架构 [3]：代理产出 COMPLETED 行，hooks 检测并解析它们 [1]，语音服务器调用 ElevenLabs [2]，音频通过系统扬声器播放。*

关键设计决策是为每种代理类型使用不同的声音。这带来：
- **不用看的识别**：仅凭耳朵就知道谁完成了
- **认知卸载**：你专注于别处时，大脑被动处理音频
- **并行感知**：多个完成不会互相冲撞；每个都有独特的声音

## COMPLETED 行约定

整个系统由一个极其简单的约定驱动：代理在其响应中包含一行 `COMPLETED:`，总结它完成了什么。

```markdown
COMPLETED: Successfully analyzed the codebase and found 3 security issues.
```

这一行会被朗读出来，通过你的扬声器。

![代码：COMPLETED 行约定]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_05.png)

*图：COMPLETED 行约定：最多 12 个词、聚焦结果、朗读起来自然的语言。*

### COMPLETED 行的规则

| 规则 | 原因 |
|------|--------|
| 最多 12 个词 | 保持语音输出简短自然 |
| 过去时 | 描述已完成之事 |
| 聚焦结果 | "Found 3 issues" 而非 "Searched for issues" |
| 无技术行话 | 朗读时必须自然 |

### 加入你的代理提示词

任何代理系统都可以采用这个约定。只需在代理的指令中加入：

```markdown
Always end your response with a COMPLETED line summarizing what you accomplished:
COMPLETED: [12 words max describing the outcome]
```

这适用于 Claude Code Task 工具代理、BMAD 工作流代理 [6]、自定义的 skill 代理，或任何能遵循指令的 Claude Code 子代理。

### 为什么用约定，而不用结构化数据？

我们选择简单的文本约定而非结构化 JSON，理由如下：

- **简单**：无需复杂解析
- **健壮**：正则提取可靠
- **人类可读**：在日志与语音中同样有效
- **通用**：任何代理都能遵循这个模式

比较一下差别：
- **好**："Found 5 papers on AI reasoning techniques."
- **坏**："Task research_papers completed with status SUCCESS and result_count 5."

第一种朗读起来自然。第二种听起来像机器人。既然这段文字要被读出来，自然语言胜出。

## Hook 系统

Claude Code hooks 是响应事件自动运行的脚本 [1]。把它们想象成触发器："当 X 发生时，自动做 Y。"

![触发器：Claude Code Hooks]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_06.png)

*图：Hooks 在代理完成时触发：读取输出、提取 COMPLETED 行、识别代理类型、发往语音服务器。*

### 相关的 Hook 事件

| Hook | 触发条件 | 用途 |
|------|---------|---------|
| `stop` | 主代理完成 | 解析主响应 |
| `subagent_stop` | 子代理完成 | 解析研究员、工程师、BMAD 代理等 |

Hooks 位于 `~/.claude/hooks/`，必须在 `~/.claude/settings.json` 中配置 [1]：

```json
{
  "hooks": {
    "stop": [
      {
        "type": "command",
        "command": "bun run ~/.claude/hooks/stop-hook.ts"
      }
    ],
    "subagent_stop": [
      {
        "type": "command",
        "command": "bun run ~/.claude/hooks/subagent-stop-hook.ts"
      }
    ]
  }
}
```

### Hooks 做了什么

subagent-stop-hook 处理大部分语音通知。逻辑流程如下：

```typescript
// Pseudocode for subagent-stop-hook.ts
1. Read agent output from stdin
2. Extract COMPLETED line using regex: /COMPLETED:\s*(.+)/
3. Identify agent type from context (researcher, engineer, mary, winston, etc.)
4. Look up voice ID for that agent type in voices.json
5. POST to http://localhost:8888/notify
```

### 代理类型检测

Hook 从 Task 工具的 `subagent_type` 参数或输出中的模式识别代理类型。你可以为自定义代理扩展它：

```typescript
// Example: Detecting BMAD agents
if (output.includes('Mary') || subagentType === 'mary') {
  agentType = 'mary';
} else if (output.includes('Winston') || subagentType === 'winston') {
  agentType = 'winston';
} else if (subagentType === 'researcher') {
  agentType = 'researcher';
}
```

检测逻辑可以简单也可以精巧，取决于你的代理系统的需要。

## 语音配置

![选择你的阵容：声音映射]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_08.png)

*图：为代理角色分配独特声音，实现即时的听觉识别。配置在 voices.json 中管理。*

每种代理类型映射到一个独特的声音，配置在 `~/.claude/agent-voices/voices.json`：

```json
{
  "voices": {
    "main": {
      "voice_id": "21m00Tcm4TlvDq8ikWAM",
      "voice_name": "Rachel"
    },
    "researcher": {
      "voice_id": "AZnzlk1XvdvUeBnXmlld",
      "voice_name": "Domi"
    },
    "engineer": {
      "voice_id": "EXAVITQu4vr4xnSDxMaL",
      "voice_name": "Bella"
    }
  }
}
```

### 默认声音映射

AgentVoices 仓库 [5] 包含常见代理类型的默认映射：

| 代理 | 声音 | 特点 |
|-------|-------|-----------------|
| **Main** | Rachel | 温暖、专业 |
| **Researcher** | Domi | 清晰、分析性 |
| **Engineer** | Bella | 平稳、技术感 |
| **Architect** | Antoni | 自信、战略性 |
| **Designer** | Elli | 创意、表现力 |
| **Writer** | Josh | 健谈、流畅 |
| **Pentester** | Arnold | 低沉、技术感 |

**BMAD 代理（自定义示例）：**

| 代理 | 声音 | 特点 |
|-------|-------|-----------------|
| **Mary (Analyst)** | Domi | 清晰、分析性 |
| **Winston (Architect)** | Antoni | 自信、战略性 |
| **Party Mode** | Rachel | 温暖、协调者 |

### 声音选择哲学

选择的声音应当：
- **独特**到足以凭耳朵识别代理类型
- 匹配 **persona**（分析师配分析性的声音、设计师配创意的声音）
- 对内容类型有合适的**节奏**
- 长时间使用不让耳朵疲劳

使用几天后，你就能只凭声音认出代理——就像不用看就知道隔壁房间哪位同事在说话。

## 语音服务器

语音服务器是一个本地 HTTP 服务，连接 hooks 与 ElevenLabs TTS API [2]。

![中继：本地语音服务器]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_07.png)

*图：基于 Bun 构建，追求速度。连接 hooks 与云 API，管理队列与限流，支持隔离测试。*

### 为什么需要本地服务器？

- **解耦**：hooks 保持简单；服务器处理复杂度
- **状态管理**：可以排队通知、管理限流
- **配置**：语音设置集中在一处
- **可测试性**：可以脱离 hooks 独立测试语音

### 仓库结构

[AgentVoices](https://github.com/bennycheung/AgentVoices) 仓库 [5] 的组织如下：

```
AgentVoices/
├── server.ts              # Voice notification server
├── voices.json            # Voice configuration
├── hooks/
│   ├── stop-hook.ts       # Main agent completion handler
│   └── subagent-stop-hook.ts  # Sub-agent completion handler
├── scripts/
│   ├── install.sh         # Setup installation
│   ├── start.sh           # Server startup
│   ├── stop.sh            # Server shutdown
│   ├── restart.sh         # Server restart
│   ├── status.sh          # Status check
│   └── uninstall.sh       # Removal script
├── examples/              # Usage examples
└── README.md
```

安装器把服务器部署到 `~/.claude/agent-voices/`，hooks 复制到 `~/.claude/hooks/`。日志写入 `~/Library/Logs/agent-voices.log`。

### API 端点

服务器暴露两个端点：

**POST /notify** —— 触发语音通知：

```bash
curl -X POST http://localhost:8888/notify \
  -H "Content-Type: application/json" \
  -d '{"message": "Task completed", "voice_id": "21m00Tcm4TlvDq8ikWAM"}'
```

参数：
- `message`（必填）：要朗读的文本
- `voice_id`（可选）：ElevenLabs voice ID（省略则用默认）
- `voice_enabled`（可选）：设为 `false` 跳过语音输出
- `title`（可选）：用于日志的通知标题

**GET /health** —— 检查服务器状态：

```bash
curl http://localhost:8888/health
```

服务器收到 notify 请求后，调用 ElevenLabs，接收音频流，并通过你的扬声器播放。

### 作为 macOS 服务运行

服务器通过 macOS LaunchAgent 在登录时自动启动：
- 无需手动启动
- 崩溃后自动重启
- 记录日志便于调试

这意味着只要你登录着，语音通知就始终可用。

## ElevenLabs 集成

ElevenLabs 提供文本转语音引擎，让声音听起来自然而非机械 [2]。

### 为什么选 ElevenLabs？

- **质量**：接近真人的语音合成
- **声音多样**：数百种声音可选
- **API 简单**：文本转语音只需一个端点
- **定价合理**：轻度使用有免费层

### API 调用结构

ElevenLabs 文本转语音 API [2] 使用简单的端点结构：

```typescript
const response = await fetch(
  `https://api.elevenlabs.io/v1/text-to-speech/${voiceId}`,
  {
    method: 'POST',
    headers: {
      'xi-api-key': apiKey,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      text: message,
      voice_settings: { stability: 0.5, similarity_boost: 0.75 }
    })
  }
);
// Returns audio stream (MP3)
```

服务器接收 MP3 音频流并立即播放。

### 成本管理

| 层级 | 字符/月 | 费用 |
|------|------------------|------|
| Free | 10,000 | $0 |
| Starter | 30,000 | $5/mo |
| Creator | 100,000 | $22/mo |

**用量估算：**
- 平均 COMPLETED 消息：约 50 字符
- 每天 200 条通知 = 每天 10,000 字符
- 免费层大约够重度使用一天

**控制成本的技巧：**
- 消息保持简短（12 词规则有帮助）
- 日常或批量任务禁用语音
- 只对重要通知使用语音

## 组装起来

让我们用完整的例子过一遍系统在实践中的运作。

### 示例：Claude Code 研究员代理

![战斗场景：研究员]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_10.png)

*图：研究员场景：下达命令，用户离开，完成时以声音宣告。无需盯着屏幕即获知信息。*

```
1. USER: "Research the latest AI papers on reasoning"

2. Claude Code spawns Task tool with subagent_type="researcher"

3. RESEARCHER AGENT: Works, then responds:
   "Found 5 relevant papers on chain-of-thought reasoning..."
   COMPLETED: Found 5 papers on AI reasoning techniques.

4. subagent-stop-hook.ts fires:
   - Detects: subagent_type = "researcher"
   - Extracts: "Found 5 papers on AI reasoning techniques."
   - Looks up: Domi voice ID

5. USER HEARS (in Domi's voice):
   "Found 5 papers on AI reasoning techniques."
```

### 示例：BMAD 工作流代理

```
1. USER: "Use Mary to analyze the requirements"

2. BmadBridge skill spawns Mary analyst agent

3. MARY AGENT: Analyzes, then responds:
   "Requirements analysis complete with 12 user stories..."
   COMPLETED: Analyzed requirements and created 12 user stories.

4. subagent-stop-hook.ts fires:
   - Detects: agent type = "mary" (from output patterns)
   - Extracts: "Analyzed requirements and created 12 user stories."
   - Looks up: Domi voice ID (analytical voice for analyst)

5. USER HEARS (in Domi's voice):
   "Analyzed requirements and created 12 user stories."
```

### 示例：并行代理

同时启动多个代理时，语音系统从容处理：

![终极组合：并行代理]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_11.png)

*图：三个研究员代理并行工作，分别在 8 秒、10 秒和 12 秒宣告完成，提供实时的进度听觉感知。*

每次完成都在发生时宣告，让你不用盯着终端就实时掌握进度。

## 搭建指南

想构建你自己的？从这里开始。

### 前置要求

- macOS（在 11+ 上测试）
- 已安装 Claude Code
- Bun 运行时
- ElevenLabs 账户（免费层即可）

### 快速开始

![启动序列：搭建指南]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_12.png)

*图：一个安装脚本搞定依赖、hook 放置和服务创建。前置要求：macOS、Claude Code、Bun、ElevenLabs API key。*


```bash
# 1. Install Bun (if not already installed)
curl -fsSL https://bun.sh/install | bash

# 2. Get ElevenLabs API key from elevenlabs.io
# Add to your environment
echo "ELEVENLABS_API_KEY=your_key_here" >> ~/.env

# 3. Clone the reference implementation [5]
git clone https://github.com/bennycheung/AgentVoices.git
cd AgentVoices

# 4. Run the installer
./scripts/install.sh
```

安装器会：
- 安装依赖
- 部署服务器到 `~/.claude/agent-voices/`
- 复制 hooks 到 `~/.claude/hooks/`
- 注册为 macOS 服务（登录时自动启动）
- 启动语音服务器

**验证安装：**

```bash
# Check server status
./scripts/status.sh

# Test voice notification
curl -X POST http://localhost:8888/notify \
  -H "Content-Type: application/json" \
  -d '{"message":"Voice system ready"}'
```

你应该会通过扬声器听到 "Voice system ready"。

### 添加你的自定义代理

![扩充队伍：自定义代理]({{ site.baseurl }}/images/hear-your-ai-agents-work/Hear_Your_AI_Agents_Work_frame_13.png)

*图：添加自定义代理的三步：在 voices.json 中注册、更新 hook 检测逻辑、在提示词中加入 COMPLETED 约定。*

1. 在 `~/.claude/agent-voices/voices.json` 中添加代理类型和你选定的 voice ID
2. 更新 `~/.claude/hooks/subagent-stop-hook.ts` 中的 hook 逻辑以检测你的代理类型
3. 在代理的提示词中加入 COMPLETED 约定

### 服务器管理

```bash
./scripts/status.sh    # Check if running
./scripts/start.sh     # Start server
./scripts/stop.sh      # Stop server
./scripts/restart.sh   # Restart server
./scripts/uninstall.sh # Remove installation
```

### 故障排查速查表

| 症状 | 检查 | 修复 |
|---------|-------|-----|
| 没有声音 | 服务器在运行吗？ | `./scripts/status.sh` |
| 401 错误 | API key 有效吗？ | `grep ELEVENLABS_API_KEY ~/.env` |
| 端口被占 | 8888 上有东西？ | `lsof -ti:8888 \| xargs kill -9` |
| 查看日志 | 检查错误 | `tail -f ~/Library/Logs/agent-voices.log` |
| 缺少 Bun | 运行时装了吗？ | `which bun` |

## 扩展与变体

基础系统可以向多个方向扩展。

### 替代 TTS 服务

| 服务 | 优点 | 缺点 |
|---------|------|------|
| ElevenLabs | 质量最好 | 规模化后成本高 |
| OpenAI TTS | API 简单 | 声音较少 |
| macOS `say` | 免费、离线 | 质量机械 |
| Bark | 开源 | 需要 GPU |

如果成本是顾虑，可以用 macOS 内置的 `say` 命令处理日常通知，把 ElevenLabs 留给重要的通知。

### 完成通知之外

同一架构支持其他通知类型：
- **错误告警**：失败时用不同的声音/语调
- **进度更新**："Starting phase 2 of 5"
- **交互式提示**："Should I continue with the refactor?"
- **系统状态**："Memory usage high" 警告

### 跨平台考量

- **Linux**：用 `paplay` 或 `aplay` 代替 macOS 音频
- **Windows**：用 PowerShell 音频 API
- **远程服务器**：通过 WebSocket 把音频串流到本地机器

## 结语

我们着手解决一个简单的问题：如何在不盯着多个终端的情况下掌握多个 AI 代理的状态？答案出人意料地"低技术"。

语音通知把 Claude Code 从纯视觉界面变成环境化的、听觉感知的体验。架构刻意保持简单：
1. 一个文本约定（COMPLETED 行）
2. 检测并解析的 hook 脚本
3. 一个桥接 TTS 的本地服务器
4. 一个生成自然语音的 API

每种代理类型不同的声音意味着你可以只凭耳朵识别谁完成了。研究员说话的方式与工程师不同，两者又都与架构师不同。几天之后，这就成了第二天性。你听到 Domi 的声音，就知道研究做完了，无需从手头的工作中抬头。

这个元洞见值得一提：我们构建了精密的多代理 AI 系统 [4]，然后发现我们需要的竟是最古老的通告机制——有人喊一声"我搞定了！"AI 交互的未来，可能比我们预想的更"人"。

试着给一个代理完成事件加上语音。当你第一次在泡咖啡时听到 AI 助手宣告"Task complete"，你会纳闷为什么不是每个 AI 系统都这么做。

----

## 参考文献

[1] Anthropic. [*Claude Code Hooks Documentation*](https://docs.anthropic.com/en/docs/claude-code/hooks). Anthropic Docs, 2025.
  - 配置 Claude Code hooks 的官方文档

[2] ElevenLabs. [*Text to Speech API Documentation*](https://docs.elevenlabs.io/). ElevenLabs Docs, 2025.
  - 语音合成的 API 参考

[3] Daniel Miessler. [*Building a Personal AI Infrastructure (PAI)*](https://danielmiessler.com/blog/personal-ai-infrastructure). danielmiessler.com, Jul 2025.
  - 启发了这一语音系统方案的 PAI 架构

[4] Benny Cheung. [*Process Philosophy for AI Agent Design*]({{ site.baseurl }}/process-philosophy-for-ai-agent-design). Benny's Mind Hack, Jan 2026.
  - 相关阅读：面向多代理系统的怀特海式框架

[5] Benny Cheung. [*AgentVoices*](https://github.com/bennycheung/AgentVoices). GitHub, Jan 2026.
  - 参考实现：面向 Claude Code 代理的语音通知系统

[6] BMad Code Organization. [*BMAD Method*](https://github.com/bmad-code-org/BMAD-METHOD/tree/main). GitHub, 2025.
  - Agentic Agile Driven Development 的基础，即敏捷 AI 驱动开发的突破性方法（Breakthrough Method of Agile AI-Driven Development）
  - 另见：[*BMAD Method 用户指南*](https://github.com/bmad-code-org/BMAD-METHOD/blob/main/docs/user-guide.md)
