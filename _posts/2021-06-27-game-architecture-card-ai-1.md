---
layout: post
title: 卡牌游戏模型的游戏架构（第 1 部分）
date: 2021-06-27 12:00:00.000000000 -00:00
permalink: game-architecture-card-ai-1
author: Benny Cheung
artwork:
  author: Thomas Lehmann - Race for the Galaxy Cards
tags:
- Game Architecture
- Card Game
- Python
- AI
category: post
comments: true
image: images/game-architecture-card-ai-1/cover2x.jpg
images:
  cover: images/game-architecture-card-ai-1/cover.jpg
  header: images/game-architecture-card-ai-1/header.jpg
published: true
series: "Game Architecture"
series_order: 1
---

<!--excerpt.start-->
作为软件架构师，我们总是想知道一个软件系统是如何构建起来的。
同时，如果你还是一名玩家，你也会思索一款游戏是如何被设计和构建的；尤其是当你被卡牌游戏的 AI 打得"火冒三丈"的时候。
这种情绪反应被正向地推动，成为我个人历时一个月研究游戏架构以及如何构建游戏的原动力。
如此一来，我便可以好好研究那个把我击败的聪明 AI 了。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以将本文当作一场**播客讨论（Podcast Discussion）**来收听，它由 Google 实验性的 _NotebookLM_ 生成。两位 AI 主持人既风趣又富有洞见。

{{ site.baseurl }}/images/game-architecture-card-ai-1/Podcast_Card_Game_AI_Part_1.mp3
{% include open-embed.html %}
----

## 卡牌游戏——Race for the Galaxy（银河竞逐）
本文聚焦的卡牌游戏是 [Race for the Galaxy](https://boardgamegeek.com/boardgame/28143/race-galaxy)（银河竞逐，简称 RFTG）。为方便读者，我在此引用官方的游戏描述：

> 在卡牌游戏 Race for the Galaxy 中，玩家通过在面前打出代表世界（world）或科技与社会发展（development）的游戏卡牌来建立银河文明。某些世界允许玩家生产货物，之后可以将货物消费掉，在拥有相应科技时换取抽卡或胜利点数。这些奖励主要由不具生产能力的发展卡与世界卡提供，不过更高级的生产型世界同样会给予此类加成。

![Race for the Galaxy 卡牌游戏]({{ site.baseurl }}/images/game-architecture-card-ai-1/RaceForTheGalaxy.gif)

*图：Tom Lehmann 的 Race for the Galaxy 自 2007 年由 Rio Grande Games 发行以来，对桌游界产生了难以置信的影响，衍生出六个扩展包并支持最多 5 名玩家。这也促成了近期由 Theresa Duringer 与 Temple Gates Games 推出的 Race for the Galaxy 应用，登陆 [iOS](https://apps.apple.com/ca/app/race-for-the-galaxy/id1190675052)/[Andriod](https://play.google.com/store/apps/details?id=com.templegatesgames.RaceAndroid&hl=en_CA&gl=US)/[Steam](https://store.steampowered.com/app/579940/Race_for_the_Galaxy/) 并获得很高的评分。（图片来源：图中为该游戏的应用版本，玩家既可以在线对战，也可以与最顶尖的卡牌游戏 AI 之一较量。）*

继续引用游戏玩法说明：

> 在每一轮开始时，每位玩家秘密且同时地从七个角色中选择一个，这些角色与本轮推进所处的各个阶段相对应。通过选择角色，玩家为本轮激活了相应阶段，使每位玩家都有机会执行该阶段的行动。例如，如果有玩家选择了"定居"（settle）角色，那么每位玩家都有机会从手牌中定居一个星球。不过，选择该角色的玩家会获得一个仅适用于自己的加成。但加成也可能通过发展卡获得，所以你必须留意，其他玩家是否也在利用你所选择的角色。

### 游戏 AI 与源代码
我们很幸运，这款卡牌游戏的 AI 开发者 Keldon Jones 在他的[帖子](https://boardgamegeek.com/thread/438698/article/3878167?fbclid=IwAR1KXYI1li66vhPBPwzwNfr8Tvg2Giz5zN5eXpVaugtCQe8DLUbILjzBMUE#3878167)中描述了游戏 AI 的设计方式。尽管那是早在 2009 年，当时就已经在使用神经网络和强化学习来训练游戏 AI 了。游戏 AI 的[源代码以 C 语言编写](https://github.com/bnordli/rftg)，以 GNU General Public License version 2（GPLv2）许可发布。这是学习游戏及游戏 AI 如何开发的绝佳机会。在这个过程中，为了更好地理解，我用 Python 重写了这个游戏，这样我就能更方便地运行并可视化基于 AI 代码的实验。

### 文章大纲
本文从通用的游戏架构入手，识别一款游戏中的重要组件。该架构将为如何阅读游戏源代码提供一份蓝图。接着，我们将展示如何分析和设计 Keldon 的 RFTG C 代码，从而能够用面向对象的 Python 代码重写游戏组件。

在第 1 部分中，我们将通过描述一个游戏架构来打下基础。由于架构组件众多，本文将只聚焦 RFTG 的（1）游戏模型（Game Model）和（2）游戏资产（Game Assets）。一如既往地在理论与实践之间取得平衡，我们会搭建 Python 开发环境，以演示这款游戏面向对象转换的过程。这些是支撑游戏引擎所必需的基础工作。游戏 AI 则要留待后续文章了。

## 游戏架构
游戏是一个信息系统，它持续追踪*游戏宇宙*（game universe）中的各种状态，使得玩家（无论是人类还是 AI）都能够依据游戏规则，通过一系列合法行动与游戏交互。下面是一个令人难忘的、关于电脑游戏的精妙定义：

> 电脑游戏（Computer Game）是关注对象的模拟器。游戏引擎（Game Engine）只是一个拥有漂亮前端和明确规则的实时数据库。

![一窥游戏架构]({{ site.baseurl }}/images/game-architecture-card-ai-1/Game_Architecture_In_a_Nutshell.png)
*图：一窥游戏架构*

这是研究游戏架构的一个绝佳起点。该定义列出了电脑游戏引擎必须设计和实现的基础组件。这个定义只是为接下来的详细描述做铺垫。

### 游戏架构总览
尽管一款游戏的成功并非仅由架构决定——玩法才是关键，但拥有良好架构的理由再怎么强调也不为过。最重要的是，架构会厘清各个游戏组件的角色，这些角色必须经过协同决策与设计。缺乏架构上的清晰性会让游戏变得脆弱，进而损害游戏的构建与扩展。

![游戏架构总览]({{ site.baseurl }}/images/game-architecture-card-ai-1/Game_Architecture_Part1.png)
*图：游戏架构总览——组件按照其在系统中的功能角色分组。功能角色分别是 **（1）游戏故事与游戏资产**、**（2）游戏模型**、（3）游戏引擎、（4）游戏界面、（5）游戏 AI、（6）游戏物理（仅适用于基于物理的游戏），以及（7）硬件抽象。在研究任何游戏源代码时，这个架构都有助于对其功能角色进行分类*

游戏架构融合了以下几本优秀著作：
* 游戏架构 [[RollingsMorris04]](#RollingsMorris04)
* 游戏 AI [[MillingtonFung06]](#MillingtonFung06)
* 使用 Cocos2d 的游戏动画与图形框架 [[Englbert13]](#Englbert13)

如果你熟悉游戏开发，你会立刻认出并理解其中许多组件。也许你从未从架构的角度思考过它们。本文并不打算充当描述所有这些组件的教科书，而是会用粗线条的笔触，展示它们在游戏架构中的用法。

1. **游戏模型（Game Model）**——游戏的信息系统。对游戏状态的呈现构成了游戏模型。它展示了游戏对象之间的关系，以及可以改变游戏状态的各种操作。
2. **游戏故事（Game Story）**与**游戏资产（Game Asset）**——游戏的精髓。这组组件勾勒出游戏的背景故事，让玩家更加投入。游戏宇宙收集着玩家们的状态，以及约束游戏状态上合法操作的游戏规则。游戏资产是一个统称，涵盖游戏角色、地块、卡牌或代币的图形。游戏状态被投射到游戏资产上以供显示。
3. **游戏引擎（Game Engine）**——游戏的规则与渲染。游戏状态和操作被投射到显示器上，所有合法操作都会经过校验，并以动画形式呈现在屏幕上。
4. **游戏界面（Game Interface）**——游戏的管理系统。游戏偏好与设置是运行游戏不可或缺的一环。界面允许玩家选择游戏的可选元素，并依据游戏规则呈现所有合法操作、与玩家进行交互。
5. **游戏 AI（Game AI）**——游戏的大脑。以 AI 形式编码的自动化计算机代理，驱使这个自动化代理对操作做出智能选择。通常，AI 的编码目标是在遵守所有游戏规则的前提下赢得游戏。
6. **游戏物理（Game Physics）**（仅适用于基于物理的游戏）——游戏世界的物理规则。只有当游戏对象和动画必须遵守物理规律时才需要这个组件，例如重力会让物体落到地面，或者代理无法穿墙。该组件与游戏引擎紧密协作，按照物理规律产生可信的动画。
7. **硬件抽象（Hardware Abstraction）**——游戏的部署。游戏必须能够在具有各种图形能力的不同平台上运行。通过对硬件进行抽象，游戏只需编写一次代码，便能够在多种平台上运行，例如手机或 PC。

在本文（第 1 部分）中，我们将聚焦前两个组件：（1）*游戏模型*和（2）*游戏资产*。

## RFTG 游戏模型
按照上一节给出的游戏架构布局，理解一款游戏的首要任务是它的模型——游戏信息系统是如何组织的。

通过研究 `rftg.h` 中描述的主要数据结构——尽管它是用 C 语言编写的——我们对其进行分析、提取并重新设计，得到如下类图所示的数据模型。

![数据模型]({{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Game_Model.png)

由于枚举（enum）在设计中提供了丰富的游戏词汇，我们在下面的类图中将枚举与其使用类一并列出。

未被链接的 Phase[1-5] 枚举由 AI 使用，用于描述在该阶段中所采取的行动。

![带枚举的数据模型]({{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Data_Model_Enum.png)

这些枚举和对象都被转换为 Python 以便开发。
首要任务是把资产——卡牌设计与卡牌图像——加载到系统中。

## RFTG 游戏资产
卡牌游戏的资产通常来自两个主要来源：（1）卡牌设计（Card Designs）和（2）卡牌图像（Card Images）（RFTG 卡牌图像参见法律声明）。根据卡牌设计的复杂程度，玩家手中持有的卡牌或当前桌面上的卡牌将驱动游戏逻辑。游戏引擎会不断检查玩家回合中的合法行动。卡牌图像只是卡牌设计的图形化呈现。有吸引力或有意义的图形能够推动游戏沉浸式的叙事，让玩家直观地理解游戏状态。

> 法律声明：图像的版权归 Rio Grande Games 所有。Rio Grande Games 已在源代码中授予分发卡牌与目标图像的许可。但若无此声明，图像文件不得解包或再分发，也不得用于任何其他目的。

### 卡牌设计
卡牌设计在 `cards.txt` 文件中指定。我们必须能够加载并表达这些信息。
该文件格式的设计是每行一条命令，由单个字符代码驱动，如下所示：

| 代码 | 描述 | 说明 |
|------|-------------|---------|
| # | 注释 | 注释行
| N | 新卡牌 | 卡牌名称
| T | 卡牌类型 | 类型、费用和分值
| E | 扩展数量 | 该卡牌出现的扩展
| G | 货物类型 | 货物类型
| F | 标志 | （标志）列表
| P | 能力 | （阶段，能力名称）列表
| V | 胜利点加成 | （胜利点加成，加成类型，胜利点名称）列表

例如，卡牌设计中（类型 1）是世界，（类型 2）是发展。与普通卡牌相比，RFTG 的卡牌设计相对复杂：普通卡牌仅由数字和花色决定（花色从 # 到 A，例如红心），而每张 RFTG 卡牌都设计了众多属性，可适用于游戏中的多个阶段。

#### 类型 1（世界）示例

以类型 1（世界）卡牌为例，"New Vinland"（新文兰）的设计如下：
* `N` 代码表示 `name`（名称）为 "New Vinland"
* `T` 代码表示 `type`（类型）为 1（世界），`cost`（费用）为 2，`value (vp)`（胜利点分值）为 1
* `E` 代码表示 `expansion`（扩展）为 0，`repeat`（重复）为 1
* `G` 代码表示 `good type`（货物类型）为 NOVELTY
* `P` 代码表示适用于 `phase`（阶段）4 的能力：`consume`（消费）CONSUME_ANY 类型的货物，以 GET_2_CARD 获得 1 和 1
* `P` 代码表示适用于 `phase`（阶段）5 的能力：`produce`（生产）该世界的货物类型

<img src="{{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Card_Design_New_Vinland.png" width="500" />

#### 类型 2（发展）示例
以类型 2（发展）卡牌为例，"Replicant Robots"（复制机器人）的设计如下：
* `N` 代码表示 `name`（名称）为 "Replicant Robots"
* `T` 代码表示 `type`（类型）为 2（发展），`cost`（费用）为 4，`value (vp)`（胜利点分值）为 2
* `E` 代码表示 `expansion`（扩展）为 0，`repeat`（重复）为 2（即牌库中有 2 张）
* `P` 代码表示适用于 `phase`（阶段）3 的能力：通过 REDUCE 减少 2 点费用来 `settle`（定居）世界

<img src="{{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Card_Design_Replicant_Robots.png" width="500" />

这里并不是一篇涵盖 RFTG 卡牌所有设计可能性的完整教程，而只是用来说明 RFTG 卡牌设计的复杂性如何造就其策略性。相应地，游戏的复杂性也会让游戏 AI 的开发、训练和执行变得更加有趣。

## RFTG 的 Python 开发
有兴趣的读者可以在 [[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python) 中找到本文所描述的完整开发环境搭建说明、Python 源代码和 Jupyter notebook 实验。

### Jupyter Notebook 实验
开发实验记录在 Jupyter Notebook `rftg_cards.ipynb` 中，可以快速运行代码示例。
在 Visual Studio Code 中，请安装 Microsoft 的"Jupyter"扩展。在 VSCode 中激活 `rftg_cards.ipynb` 时，把 Python 内核切换为前面步骤中已设置好的 `rftg`。

![在 VSCode 中运行 Jupyter Notebook]({{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_VScode_Jupyter_Notebook-annotated.png)

### 枚举（Enum）
`rftg/enums.py` 实现了所有枚举定义，例如下面的 `CardType`：

```python
class CardType(Enum):
  WORLD        = 1
  DEVELOPMENT  = 2
```

对于更复杂的枚举，例如 `PhasePower`，则建模为元组值。

```python
class PhasePower(Enum):
  P1_DRAW               = (1,0)
  P1_KEEP               = (1,1)
  P1_DISCARD_ANY        = (1,2)
  P1_DISCARD_PRESTIGE   = (1,3)
  P1_ORB_MOVEMENT       = (1,4)
  P1_PER_REBEL_MILITARY = (1,5)
  ...
```

使用枚举有诸多好处，例如 IDE 支持、类型检查和序列化。

```python
from rftg.enums import PhasePower

# 获取枚举值
(phase, type) = PhasePower.P4_CONSUME_3_DIFF.value

# 按名称匹配枚举
code = PhasePower['P4_CONSUME_3_DIFF']
```

在 IDE 中使用时，智能提示（intellisense）会帮助你显示所有可能的枚举值。

![在 IDE 中使用枚举开发]({{ site.baseurl }}/images/game-architecture-card-ai-1/Develop_with_Enums.png)

### 类（Class）
`rftg/cards.py` 实现了所有的类定义，例如下面的 `Power` 类：

```python
from marshmallow import Schema, fields, post_load
from marshmallow_enum import EnumField

class Power:

  class Serializer(Schema):
    phase = fields.Integer()
    code = EnumField(PhasePower)
    value = fields.Integer()
    times = fields.Integer()

    @post_load
    def make_self(self, data, **kwargs):
      return Power(**data)

  serializer = Serializer()

  @staticmethod
  def from_json(json):
    return Power.serializer.load(json)

  def __init__(self, **kwargs):
    # 偷懒地接收所有关键字参数
    self.__dict__.update(kwargs)

  def __repr__(self):
    return '<Power {}/{}>'.format(self.phase, self.code)

  def __str__(self):
    return self.__class__.__name__ + ':' + str(vars(self))

  def to_json(self):
    return Power.serializer.dump(self)
```

该类使用 `marshmallow` 来支持序列化模式（schema），从而可以轻松地完成 JSON 的加载与导出。

例如，下面的代码演示了 `Power` 实例如何与 JSON 相互序列化。

```python
from rftg.cards import Power

power = Power(phase=1, code=PhasePower.P1_DISCARD_ANY, value=1, times=1)

json = power.to_json()
print(json)

power2 = Power.from_json(json)
print(power2)
```

输出看起来像这样：

```json
{
  'times': 1,
  'code': 'P1_DISCARD_ANY',
  'phase': 1,
  'value': 1,
  Power: {
    'phase': 1,
    'code': <PhasePower.P1_DISCARD_ANY: (1, 2)>, 'value': 1,
    'times': 1
  }
}
```

### 加载库（Library）
`Library` 类帮助解析卡牌设计文件 `cards.txt`，并创建所有卡牌的 `Design` 信息。由于我们已经把所有卡牌图像导出为单独的 PNG 文件，我们可以用 `Library` 加载所有卡牌的设计图像。

```python
from rftg.cards import Library

library = Library()
library.read_cards('cards.txt')
library.read_card_images('card_images')

len(library.designs)
```

卡牌设计的总数将为 280。

#### 检视卡牌设计
我们可以按照类的 `marshmallow` 模式规范，把卡牌 `Design` 序列化为 JSON 来进行检查。

```python
design = library.card_by_name("Old Earth")
design.to_json()
```

```json
{'index': 6,
 'name': 'Old Earth',
 'vp': 2,
 'expansion': {'index': 0, 'count': 1},
 'powers': [{'times': 0, 'code': 'P4_TRADE_ANY', 'value': 1, 'phase': 4},
  {'times': 0, 'code': 'P4_CONSUME_ANY', 'value': 0, 'phase': 4},
  {'times': 2, 'code': 'P4_GET_VP', 'value': 1, 'phase': 4}],
 'cost': 3,
 'good': 'NONE',
 'flags': ['START', 'START_BLUE'],
 'bonuses': [],
 'type': 'WORLD',
 'source': ['N:Old Earth',
  'T:1:3:2',
  'E@0:1',
  'F:START | START_BLUE',
  'P:4:TRADE_ANY:1:0',
  'P:4:CONSUME_ANY | GET_VP:1:2']}
```

#### 绘制卡牌设计
我们可以把原始的 `cards.txt` 源内容与卡牌图像一同显示，从而可视化一张卡牌设计。`CardDisplay` 是一个帮助绘图的工具类。`plot_single_card` 函数用于绘制单张卡牌，`save` 函数会把输出保存为 PNG 文件。

```python
import random
from rftg.display import Display, CardDisplay

display = Display('card_images', figsize=(16,8))
card_display = CardDisplay(library, display)
design = library.card_by_name("Old Earth")
text = '\n'.join(design.source)
fig, ax = card_display.plot_single_card(design.name, text=text)
card_display.save('RFTG_Card_Design_Old_Earth.png')
card_display.show()
```

输出图像如下：

<img src="{{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Card_Design_Old_Earth.png" width="500" />

### 牌库（Deck）
我们可以从存放卡牌设计的 `Library` 中收集一组 `Card`，组成一个 `Deck`（牌库）。要确定 RFTG 的某个具体牌库，我们需要指明应包含哪些扩展。首先，我们只取基础游戏的卡牌，即 `expansion=0`。

```python
from rftg.cards import Deck

deck = Deck(library)
deck.build_deck(0)
```

基础游戏的牌库将总共有 114 张卡牌（注意：某些卡牌设计会重复出现）

#### 绘制牌库卡牌
我们有一个便捷的 `CardDisplay.plot_cards` 函数，可以轻松绘制牌库中的卡牌。例如，下面的代码会从牌库中随机抽取 6 张卡牌。

```python
import random
from rftg.display import CardDisplay

samples = random.sample(deck.cards, 6)
fig, ax = card_display.plot_cards(samples, 6)
card_display.save('RFTG_Deck_Card_Samples.png')
card_display.show()
```

![牌库卡牌样本]({{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Deck_Card_Samples.png)

`CardDisplay.plot_cards` 函数非常灵活，如果卡牌更多，还可以显示多行。我们将用 `random.shuffle()` 洗牌，然后以 6 列绘制前 12 张卡牌（即共 2 行）。

```python
random.shuffle(deck.cards)
fig.ax = card_display.plot_cards(deck.cards[0:12], 6)
card_display.save('RFTG_Desk_Card_Samples_2_Rows.png')
card_display.show()
```

![更多牌库卡牌样本]({{ site.baseurl }}/images/game-architecture-card-ai-1/RFTG_Desk_Card_Samples_2_Rows.png)

### 玩家（Player）
`Player` 类记录游戏中一个参与者的行动状态。

```python
class Player:

  def __init__(self, **kwargs):
    self.name = kwargs.get('name', '')  # 玩家名称/颜色
    self.ai = kwargs.get('ai', False)   # 该玩家是否由 AI 扮演
    self.actions = []                   # 所选择的行动
    self.prev_actions = []              # 上一回合的行动
    self.phase_bonus_used = False       # 玩家已使用阶段加成
    self.start = 0                      # 玩家的起始世界
    self.placing = Location.DECK        # 在发展或定居阶段中所选的卡牌
    self.bonus_military = 0             # 本阶段迄今累积的军事加成
    self.bonus_reduce = 0               # 本阶段迄今累积的定居折扣
    self.end_discard = 0                # 回合结束时弃掉的卡牌数量
    self.vp = 0                         # 胜利点筹码
    self.end_vp = 0                     # 总胜利点（若游戏此时结束）
    self.winner = False                 # 玩家是否为获胜者
    self.drawn_round = 0                # 本轮（或上一轮）抽牌的数量
    self.skip_develop = False           # 玩家跳过了上一个发展阶段，尚未抽新牌
    self.skip_settle = False            # 玩家跳过了上一个定居阶段，尚未抽新牌
    self.low_hand = 0                   # 回合中的最小手牌数
    self.table_order = 0                # 出牌计数器
    self.phase_cards = 0                # 当前阶段获得的卡牌数
    self.phase_vp = 0                   # 当前阶段获得的胜利点
    self.choice_log = []                # 玩家选择的日志
    self.choice_history = []            # 玩家选择的历史

  def __repr__(self):
    return '<Player {}/{}>'.format(self.name, self.ai)

  def __str__(self):
    return self.__class__.__name__ + ':' + str(vars(self)))
```

### 游戏资源（GameResource）
`GameResource` 类保存一个游戏所需的全局资源，例如 `Library` 和 `CardDisplay`。

```python
class GameResource:

  def __init__(self, **kwargs):
    self.library = kwargs.get('library', None)
    self.display = kwargs.get('display', None)

  def __str__(self):
    return self.__class__.__name__ + ':' + str(vars(self))
```

### 游戏（Game）
最后同样重要的是，`Game` 类记录一个游戏的全局状态。

```python
class Game:

  def __init__(self, **kwargs):
    self.resource = kwargs.get('resource', None)
    self.session_id = kwargs.get('session_id', 0)
    self.simulation = kwargs.get('simulation', False) # 该游戏是一次模拟
    self.debug = kwargs.get('debug', False)           # 该游戏是否为调试游戏
    self.random_seed = 0                              # 当前随机种子
    self.start_seed = kwargs.get('start_seed', random.randint(0, 2**16)) # 指定起始种子以便回放
    self.players = kwargs.get('players', [])
    self.expanded = 0         # 使用的扩展数量
    self.promo = False        # 牌库中是否包含促销起始世界
    self.vp_pool = 0          # 奖池中剩余的胜利点数
    self.action_selected = [] # 本轮选择的行动
    self.cur_action = Phase.ACTION
    self.turn = 0
    self.round = 0
    self.game_over = False

    # 随机种子
    random.seed(self.start_seed)

    # 构建游戏专属的牌库
    self.deck = Deck(self.resource.library)
    self.deck.build_deck(self.expanded)

  def __repr__(self):
    return '<Game {}/{}>'.format(self.session_id, self.start_seed)

  def __str__(self):
    return self.__class__.__name__ + ':' + str(vars(self))
```

创建一个游戏会话的初始化步骤如下所示。

```python
from rftg.cards import Library, Deck, Card
from rftg.display import Display, CardDisplay
from rftg.game import GameResource, Game, Player

# 创建游戏资源
library = Library()
library.read_cards('cards.txt')
library.read_card_images('card_images')
library.load_actions('card_images')
print('Designs: {}'.format(len(library.designs)))

display = Display('card_images', figsize=(16,8))
card_display = CardDisplay(library, display)

# 构建游戏牌库
deck = Deck(library)
deck.build_deck(0)
print('Cards: {}'.format(len(deck.cards)))

# 创建玩家
player1 = Player(name="Blue", ai=False)
player2 = Player(name="Red", ai=False)
players = [player1, player2]

# 创建一个包含玩家的新游戏
rresource = GameResource(library=library, display=display)
game = Game(resource=resource, session_id='testing', deck=deck, players=players)
```

至此，RFTG 游戏信息模型就完成了！

## 结语
我们应当向游戏开发社区致敬。这是一条漫长的道路，即便只是覆盖一款设计精良的卡牌游戏的（1）游戏模型和（2）游戏资产，也是如此。在这个过程中，我们对卡牌游戏信息系统获得了关键的洞见，尽管我们关注的是一款特定的卡牌游戏 RFTG。对于热心的读者和经验丰富的开发者而言，这个卡牌游戏模型可以被泛化并应用到其他类型的卡牌游戏上。

![映射到扑克牌模型]({{ site.baseurl }}/images/game-architecture-card-ai-1/Map_to_Playing_Cards_Model.png)

*图：演示卡牌游戏模型如何映射到扑克牌模型。卡牌设计的可能性是无穷的。*

展望下一篇文章，我们将继续探索（3）游戏引擎和（4）游戏界面的开发。这些是未来开发、训练并可视化（5）游戏 AI 之前的必要前提。

* **>>** [卡牌游戏模型的游戏架构（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)
* [卡牌游戏行动的游戏架构（第 2 部分）]({{ site.baseurl }}/game-architecture-card-ai-2)
* [卡牌游戏 AI 的游戏架构（第 3 部分）]({{ site.baseurl }}/game-architecture-card-ai-3)

## 参考文献

### 游戏架构
* <a name="RollingsMorris04">[RollingsMorris04]</a> Andrew Rollings & Dave Morris, Game Architecture and Design: A New Edition（《游戏架构与设计：新版》）, 2004, New Rider Publishing, ISBN: 0-7357-1363-4
  * 这不是一本编程书，而是一本设计书。Andrew Rollings 和 Dave Morris 确实谈到了游戏架构，并用状态图和类层次结构草图剖析了一些顶级游戏，但这部分内容只占少数。书中大部分篇幅，是两位作者就更宏观的工程决策给出的有见地的观点。
* <a name="MillingtonFung06">[MillingtonFung06]</a> Ian Millington & John Funge, Artificial Intelligence for Games（《游戏人工智能》）, 2006, Elsevier, Morgan Kaufmann Pub., ISBN: 978-0-12-497782-2
  * 这是一本涵盖所有游戏 AI 实践、术语和专业知识的全面参考书。Ian Millington 把丰富的专业经验带入了提升游戏 AI 质量这一课题。他描述了大量来自真实游戏的例子，并通过详细的案例研究探索其背后的思想。
* <a name="Englbert13">[Englbert13]</a> Roger Engelbert, COCOS2dX by Example（《COCOS2dX 实战》）, 2013, Packt Pub., ISBN: 978-1-78-216734-1
  * 这是一本注重动手实践的书，教你利用久经考验的 Cocos2d 框架（Cocos2dX 是其 C++ 版本）构建自己的跨平台游戏。跟随六个教程完成六款截然不同的游戏，充分发挥 Cocos2d 框架的易用性与快速实现能力，从简单的想法逐步进阶到游戏开发中更高级的主题。
* <a name="Buschmann07">[Buschmann07]</a> Frank Buschmann & Kevlin Henney & Douglas C. Schmidt, Pattern-Oriented Software Architecture: On Patterns and Pattern Languages（《面向模式的软件架构：模式与模式语言》）, 2007, Wiley Pub., ISBN: 978-0-47-148648-0
  * 这是一本关于设计模式的高级软件架构书。它是唯一试图发展一门整合关键文献中各种模式的综合性语言的书，同时也是所有面向模式的软件架构（POSA）模式的参考手册。

### Race for the Galaxy（银河竞逐）
* <a name="Jones09">[[Jones09]](https://boardgamegeek.com/thread/438698/article/3878167?fbclid=IwAR1KXYI1li66vhPBPwzwNfr8Tvg2Giz5zN5eXpVaugtCQe8DLUbILjzBMUE#3878167)</a> Keldon Jones, Talk a bit about how the AI works（谈谈 AI 的工作原理）, 2009 年 9 月
  * 最新源代码 0.9.5，包含所有扩展：<https://github.com/bnordli/rftg>
* <a name="Tesauro95">[[Tesauro95]](https://bkgm.com/articles/tesauro/tdl.html)</a> Gerald Tesauro, Temporal Difference Learning and TD-Gammon（时间差分学习与 TD-Gammon）, Communications of the ACM, 1995 年 3 月 / 第 38 卷第 3 期
* <a name="TempleGates17">[[TemplateGates17]](https://www.templegatesgames.com/race-for-the-galaxy-ai/)</a> Race for the Galaxy AI, Temple Gates, 2017 年 12 月
  * Temple Gates 是应用版本的游戏开发商，其使用了 Keldon Jones 的 AI 引擎
* <a name="Cheung21">[[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python)</a> Benny Cheung, Game Architecture for Card Game Model (Part 1) - Jupyter Notebook, 2021 年 6 月