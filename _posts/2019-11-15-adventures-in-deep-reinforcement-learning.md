---
layout: post
title: 星际争霸 II 深度强化学习历险记
date: 2019-11-15 00:00:00.000000000 -00:00
permalink: adventures-in-deep-reinforcement-learning
author: Benny Cheung
artwork:
  author: AI
tags:
  - Reinforcement Learning
  - StarCraft II
  - AlphaStar
  - AI
category: post
comments: true
image: images/adventures-in-deep-reinforcement-learning/cover2x.jpg
images:
  cover: images/adventures-in-deep-reinforcement-learning/cover.jpg
  header: images/adventures-in-deep-reinforcement-learning/header.jpg
published: true
---

<!--excerpt.start-->
仅凭奖励、通过试错来学习的范式被称为**强化学习**（Reinforcement Learning，RL）。强化学习的精髓在于通过交互学习，模仿人类与环境互动的学习方式，其根源可追溯到行为主义心理学。正向奖励会强化带来奖励的行为。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/Podcast_Deep_Reinforcement_Learning_StarCraft_II.mp3
{% include open-embed.html %}
----

要定义强化学习问题，我们需要定义一个发生"感知-行动-学习"循环的环境。在这个环境中，代理（agent）观察给定的状态 t。代理按策略学习，在给定状态下采取可能有长期后果的行动，与环境交互。它进入时间步 t+1 的下一个状态，并更新策略。最终，代理从环境接收观察/状态以及作为反馈信号的奖励，并通过行动与环境交互。

强化学习问题可以形式化地描述为马尔可夫决策过程（Markov Decision Process，MDP）：它描述了强化学习的环境，即代理学习或运作的周遭与条件。马尔可夫过程是具有马尔可夫性质的状态序列——该性质断言：给定现在，未来与过去独立。最后一个状态的充分性意味着我们只需最后一个状态就能评估代理未来的选择。深度神经网络需要大量监督训练数据，而且对建模世界的变化不够灵活；另一方面，强化学习能够应对世界的变化，并最大化当前选择。

使用 PySC2 有助于理解强化学习的实践层面。与其从玩具示例开始，不如面对星际争霸 II 游戏的复杂性——它更贴近现实：AI 需要平衡资源、建造、探索、制定战略和战斗。为求胜而平衡多重目标并进行长期规划，让这款游戏的复杂度显得很真实。现有技术大多聚焦于单代理学习。其潜力在于扩展到多代理学习，应用协作博弈论（还记得电影《美丽心灵》吗？）

![用星际争霸 2 学习环境学习 DQN]({{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/StarCraft2_Q_Learning_Smart_AI_Agent_Training.jpg) 

*图：星示了星际争霸 II 学习环境的运行状态。左上角显示实际运行的星际争霸 II 游戏。SC2LE 捕获并报告来自游戏环境的观察。SC2LE 允许在右侧以视觉方式展示所有游戏观察。AI 代理可以获取这些观察并评估最优行动。*

游戏是强化学习研究的理想环境。实时战略（RTS）游戏上的强化学习问题，由于状态复杂、动作多样、时间跨度长，远比围棋上的问题困难。以下是我的实践研究笔记，记录了这一学习与动手的过程。本文旨在提供一份可循的简明实验路线图。每节先列出参考资料，随后说明可以尝试的内容。为方便读者，部分信息摘自原始资料，尤其是如何搭建与运行实验。一如既往，会用 Python 是这段历险的根本前提。

## PySC2 安装
* 参考：StarCraft II Learning Environment, <https://github.com/deepmind/pysc2>
* 参考：StarCraft II Client——用于与星际争霸 II 通信的协议定义 <https://github.com/Blizzard/s2client-proto>

PySC2 是 DeepMind 星际争霸 II 学习环境（SC2LE）的 Python 组件。它把暴雪娱乐的星际争霸 II 机器学习 API 暴露为 Python RL 环境。这是 DeepMind 与暴雪的合作，旨在把星际争霸 II 发展成 RL 研究的富环境。PySC2 为 RL 代理提供与星际争霸 II 交互的接口：获取观察、发送动作。

安装方式，

```
conda create -n pysc2 python=3.5 anaconda
conda activate pysc2

pip install pysc2==1.2
```

你可以运行一个代理来测试环境。UI 会展示代理的动作，有助于调试和可视化。

```
python -m pysc2.bin.agent --map Simple64
```

还有一个主要用于调试的人类代理接口，但它也可以用来玩游戏。UI 相当简单且不完整，但足以理解游戏的基本要素。此外，它在 Linux 上也能运行。

```
python -m pysc2.bin.play --map Simple64
```

运行代理或以人类身份游玩默认会保存一局录像（replay）。你可以运行以下命令观看录像：

```
python -m pysc2.bin.play --replay <path-to-replay>
```

这适用于任何录像，只要游戏能找到对应的地图。控制方式与玩游戏相同：`F4` 退出，`pgup/pgdn` 控制速度等。

你可以用 `--video` 标志把录像保存为视频

## PySC2 深度强化学习代理
* 参考：Oriol Vinyals et. al., StarCraft II: A New Challenge for Reinforcement Learning, Aug 2017. <https://arxiv.org/abs/1708.04782>
* 参考：StarCraft II / PySC2 深度强化学习代理（A2C） <https://github.com/simonmeister/pysc2-rl-agents>

该仓库为 pysc2 环境实现了 DeepMind 论文《StarCraft II: A New Challenge for Reinforcement Learning》所述的优势演员-评论家（Advantage Actor-Critic）代理基线。它使用 A3C 的同步变体（A2C）来高效地在 GPU 上训练，同时在其他方面尽可能贴近论文中描述的代理。

项目已确认的进展
* (/) A2C 代理
* (/) FullyConv 架构
* (/) 支持所有空间屏幕与小地图观察，以及非空间的玩家观察
* (/) 支持 DeepMind 论文描述的完整动作空间（独立预测所有参数）
* (/) 支持在所有小游戏上训练

遗憾的是，项目在达成以下目标之前停止了。
* (x) 报告所有小游戏的结果
* (x) LSTM 架构
* (x) 多 GPU 训练

### 快速安装指南

```
conda create -n pysc2 python=3.5 anaconda
conda activate pysc2

pip install --upgrade pip
pip install numpy
pip install tensorflow-gpu==1.4.0 --ignore-installed
pip install pysc2==1.2
```

安装星际争霸 II。在 Linux 上，使用 [3.16.1](http://blzdistsc2-a.akamaihd.net/Linux/SC2.3.16.1.zip)。

下载[小游戏](https://github.com/deepmind/pysc2/releases/download/v1.2/mini_games.zip)并解压到你的 StarcraftII/Maps/ 目录。

解压 zip 文件时，需要输入 `iagreetotheeula` 以接受 EULA。

### 训练与运行
还有几点要求需要注意。

* 它需要 cuda 8.0、cudnn 6.0（在 Linux 16.04 LTS、Titan-X 12 GB 上测试）
* 修改文件 `pysc2-rl-agents/rl/agents/a2c/agent.py` 第 202 行，把 `keepdims=True` 改为 `keep_dims=True`

测试：

```
python run.py my_experiment --map MoveToBeacon --envs 1 --vis
```

运行并训练（默认生成 32 个环境太多，减为 16 个）：

```
python run.py my_experiment --map MoveToBeacon --envs 16
```

不训练、只运行评估：

```
python run.py my_experiment --map MoveToBeacon --eval
```

训练或评估时可以用 `--vis` 标志可视化代理。
所有参数见 `run.py`。

摘要（summary）写入 `out/summary/<experiment_name>`，模型检查点写入 `out/models/<experiment_name>`。

在 MoveToBeacon 小游戏上训练约一小时、约 8K 回合之后，代理几乎能以最优方式追踪信标。（在 GPU TitanX Pascal (12GB) 上训练）

![在 MoveToBeacon 小游戏上训练后的 DQN 结果]({{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/DQN_MoveToBeacon_Minigame_actions.gif)

下面展示了分数随回合变化的曲线。

![DQN 在 MoveToBeacon 小游戏上训练的分数]({{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/DQN_MoveToBeacon_Minigame_train.jpg)

## 理解 PySC2 深度强化学习
* 参考：Gema Parreño, StarCraft II Learning environment full overview（7 篇博客系列）
    * **(III)** 理论：深度强化学习概览 <https://medium.com/@gema.parreno.piqueras/starcraft-ii-learning-environment-full-overview-iii-f7c2bc8cb4fb>
    * (IV) 理论：pySC2 环境 <https://medium.com/@gema.parreno.piqueras/starcraft-ii-learning-environment-full-overview-iv-2fb1a7c2667>
    * (V) 实验：pySC2 快速上手概览 <https://medium.com/@gema.parreno.piqueras/starcraft-ii-learning-environment-full-overview-v-34f5cb2a2220>
    * **(VI)** 实验：跳进机器学习代理 <https://medium.com/@gema.parreno.piqueras/starcraft-ii-learning-environment-full-overview-vi-ae3126aa6b35> 
    * (VII) 实验：运行与训练代理 <https://medium.com/@gema.parreno.piqueras/starcraft-ii-learning-environment-full-overview-vii-8b0775c1f3a7>
* 参考：Gema Parreño, Startcraft II Machine Learning research with DeepMind pysc2 python library .mini-games and agents. <https://github.com/SoyGema/Startcraft_pysc2_minigames>
* 参考：Gema Parreño, Startcraft Pysc2 Deepmind mini games creation <https://github.com/SoyGema/Startcraft_pysc2_minigames/tree/master/docs>

### 小游戏任务描述
小游戏是受控环境，可能有助于挖掘星际争霸 II 的游戏特性。通用的星际争霸 II 学习系统可能是一项令人生畏的任务。因此，把这些任务拆分为小任务以推进研究，是一个合乎逻辑的选择。小游戏聚焦星际争霸 II 对战的不同元素。

为了孤立地研究游戏的各个元素，并为玩完整游戏提供更细粒度的渐进步骤，DeepMind 构建了若干小游戏。它们是小型地图上的聚焦场景，目的在于以清晰的奖励结构测试一部分动作和/或游戏机制。与完整游戏只有胜/负/平的奖励不同，小游戏的奖励结构可以奖励特定行为（在相应的 .SC2Map 文件中定义）。

### 代理
关于脚本代理（scripted agent），有一个包含若干实现的 python 文件。scripted_agent.py 聚焦于 HallucinIce 地图，会施放幻象执政官（Archon Hallucination）。此外还有一个类，把所有幻象动作放进列表，代理在这些动作中随机选择。

随着新版 PySC2 发布，HallucinIce 小游戏也提供了 Q-Learning 和 DQN 代理

### 如何在你的环境中运行小游戏
下载或克隆该仓库，或点击这里下载全部小游戏

把 .SC2 文件放到 /Applications/StarCraft II/Maps/mini_games/ ——有时 Maps 文件夹可能不存在。如果不存在，请先创建——

打开 `pysc2/maps/mini_games.py`，把下列小游戏名称加入 mini-games 数组

```
mini_games = [  ## This mini-games names should alredy been in your list
    "BuildMarines",  # 900s
    "CollectMineralsAndGas",  # 420s
    "CollectMineralShards",  # 120s
    "DefeatRoaches",  # 120s
    "DefeatZerglingsAndBanelings",  # 120s
    "FindAndDefeatZerglings",  # 180s
    "MoveToBeacon",  # 120s   ##Now you add this few lines 
    "SentryDefense", # 120s
    "ForceField", # 30s
    "HallucinIce", # 30s
    "FlowerFields", # 60s
    "TheRightPath", # 300s
    "RedWaves", # 180s
    "BlueMoon", # 60s
    "MicroPrism", # 45s
]
```

我们可以把 `Starcraft_pysc2_minigames/Agents/scripted_agent.py` 的示例复制为 `pysc2/agents/scripted_agent_test.py`。随后，我们就可以用新的小游戏测试这些脚本示例机器人代理。你应该会看到类似如下的画面。

![小游戏 HallucinIce]({{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/mini_game_RL_HallucinationArchon.gif)

*图：摘自 Gema Parreño 的博客《StarCraft II Learning environment》——运行幻象执政官脚本代理*

### 安装
我们需要从 `requirements.txt` 安装所需的包；不过，要按如下方式注释掉部分包：
* 从我们定制过的源安装 `pysc2`
* 从源码安装 `keras-contrib`，从这里下载 <https://github.com/keras-team/keras-contrib>

```
# PySC2==2.0
numpy==1.14.0
Keras==2.2.2
Keras-Applications==1.0.4
# keras-contrib==2.0.8
Keras-Preprocessing==1.0.2
keras-rl==0.4.2
pandas==0.22.0
```

从源码安装 `keras-contrib`，

```
cd keras-contrib
python setup.py install
```

完成前述 `mini_games.py` 修改后，你可以从源码手动安装 pysc2

```
cd pysc2
python setup.py install
```

你可以用新增的小游戏测试安装，

```
python -m pysc2.bin.agent --map HallucinIce 
```

你可以用 `scripted_agent_test.py` 测试新的小游戏机器人代理，

```
python -m pysc2.bin.agent --map HallucinIce --agent pysc2.agents.scripted_agent_test.HallucinationArchon
```

## AlphaStar，下一个层级？
* 参考：DeepMind's AlphaStar Team, AlphaStar: Mastering the Real-Time Strategy Game StarCraft II, Jan 2019. <https://deepmind.com/blog/article/alphastar-mastering-real-time-strategy-game-starcraft-ii>
* 参考：Dave Gershgorn, DeepMind's StarCraft Bot Has a 191-Year Head Start on Humanity, Oct 2019. <https://onezero.medium.com/deepminds-starcraft-bot-has-a-191-year-head-start-on-humanity-4590032b23ab>

当学习者们还在聚焦用 SC2LE 理解较小规模的小游戏强化学习时，DeepMind AlphaStar 团队已成功扩大规模，训练出击败顶级职业星际争霸 II 玩家的 AI。在 2018 年 12 月举行的一系列测试赛中，AlphaStar 以 5-0 干净利落地击败了 Team Liquid 的 Grzegorz "MaNa" Komincz——世界最强的职业星际争霸选手之一。

观看《AlphaStar: The inside story》
{% include youtubePlayer.html id="UuhECwm31dM" %}


AlphaStar 使用了一种新颖的多代理学习算法。神经网络首先通过监督学习，从暴雪公开的匿名人类对局[录像数据](https://github.com/Blizzard/s2client-proto/tree/master/samples/replay-api)中训练。这让 AlphaStar 能够通过模仿，学习玩家在天梯上使用的基本微观与宏观策略。

![代理训练方法]({{ site.baseurl }}/images/adventures-in-deep-reinforcement-learning/SCII-BlogPost-Fig03.width-700.jpg)

*图：摘自 AlphaStar 团队的博客文章，展示多代理强化学习过程是如何构建的。*

随后，这些初始策略被用来播种一个多代理强化学习过程。他们创建了一个持续进行的联赛，联赛中的代理——竞争者（AI）——彼此对战，如同人类通过在[星际争霸天梯](https://starcraft2.com/en-us/ladder/grandmaster/1)上对战来体验游戏。新的竞争者通过从现有竞争者分叉而动态加入联赛；每个代理从与其他竞争者的对局中学习。这种新的训练形式把[基于种群](https://deepmind.com/blog/capture-the-flag/)和[多代理](https://arxiv.org/pdf/1711.00832.pdf)强化学习的思想又推进了一步：它创造了一个持续探索星际争霸对战巨大策略空间的过程，同时确保每个竞争者既能很好地应对最强策略，又不会忘记如何击败较早的策略。

看起来，使用深度强化学习的新一代 AI 前景可期！

## 更多参考文献
* David Silver, Reinforcement Learning, a series of 10 youtube video lectures <https://www.youtube.com/watch?v=2pWv7GOvuf0&list=PLzuuYNsE1EZAXYR4FJ75jcJseBmo4KQ9->
  * 如果你刚接触 RL，想理解强化学习的数学与哲学背景，这个系列非常宝贵。
* Richard Sutton and Andrew Barto, Reinforcement Learning: An Introduction, MIT Press, 2017, ISBN:9780262193986
  * 本书清晰而简明地讲述了强化学习的关键思想与算法，讨论范围从该领域智识根基的历史到最新的进展与应用。
* Rowel Atienza, Advanced Deep Learning with Keras, Packt Publishing, 2018, ISBN:978788629416
  * 第 9 章：深度强化学习
