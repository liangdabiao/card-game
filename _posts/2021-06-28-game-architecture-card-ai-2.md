---
layout: post
title: 卡牌游戏行动的游戏架构（第 2 部分）
date: 2021-06-28 12:00:00.000000000 -00:00
permalink: game-architecture-card-ai-2
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
image: images/game-architecture-card-ai-2/cover2x.jpg
images:
  cover: images/game-architecture-card-ai-2/cover.jpg
  header: images/game-architecture-card-ai-2/header.jpg
published: true
series: "Game Architecture"
series_order: 2
---

<!--excerpt.start-->
承接上一篇[卡牌游戏模型的游戏架构（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)，我们定义了一套游戏架构，作为研究《银河竞逐》（Race for the Galaxy）卡牌游戏的参照。本文聚焦组件 (3) **游戏引擎**与 (4) **游戏界面**。
<!--excerpt.end-->

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/game-architecture-card-ai-2/Podcast_Card_Game_AI_Part_2.mp3
{% include open-embed.html %}
----

![游戏架构总览]({{ site.baseurl }}/images/game-architecture-card-ai-2/Game_Architecture_Part2.png)

*图：游戏架构总览——组件按其在系统中的功能角色分组。功能角色为 (1) 游戏故事与游戏资产，(2) 游戏模型，**(3) 游戏引擎**，**(4) 游戏界面**，(5) 游戏 AI，(6) 游戏物理（仅物理游戏需要），以及 (7) 硬件抽象。研究任何游戏的源码时，这一架构都有助于归类其功能角色*

(3) **游戏引擎** —— 游戏的规则与渲染。游戏状态与操作被投影到显示器上。所有合法操作都会被检查并以动画形式呈现在屏幕上。

(4) **游戏界面** —— 游戏的管理系统。游戏偏好与设置是运行游戏的组成部分。界面允许玩家选择游戏的可选元素。所有合法操作都依据游戏规则呈现并与玩家交互。

在第 2 部分中，我们将继续以[银河竞逐](https://boardgamegeek.com/boardgame/28143/race-galaxy)（RFTG）为例描述游戏架构。一如既往地在理论与实践之间取得平衡，我们搭建 Python 开发环境来演示面向对象的游戏引擎。

## RFTG Python 开发
有兴趣的读者可以从 [[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python) 找到完整的开发环境搭建说明、Python 源码以及本文描述的 Jupyter notebook 实验。

### Jupyter Notebook 实验（第 2 部分）
第 2 部分的开发实验记录在 Jupyter Notebook `rftg_game.ipynb` 中，可以快速运行代码示例。
在 Visual Studio code 中安装 Microsoft 的"Jupyter"扩展。在 VScode 中打开 `rftg_game.ipynb` 后，把 Python kernel 切换为 `rftg`——它已按代码 `README.md` 的说明配置好。

![运行 VSCode Jupyter Notebook]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_VScode_Jupyter_Notebook-annotated.png)

## RFTG 游戏引擎
游戏引擎定义了玩家必须遵守的一组规则。它还借助追踪玩家与游戏进度的游戏信息系统来维护游戏状态。从游戏数据模型中抽取出来，下面的 UML 图突出了代表游戏引擎组件的那些类。

`Game` 是一个遵循确定规则集、由一组玩家与游戏模型交互的会话。在最高层面，一个游戏是一叠 `Deck` 卡牌、一组 `Player` 以及关联 `GameResource` 的组合。资源包括作为 `Library` 的卡牌设计库和作为 `CardDisplay` 的卡牌展示。

* Game - `Game` 类记录游戏的全局状态。
* Deck - `Deck` 类表示游戏中的一叠卡牌。
* Player - `Player` 类记录游戏中一个行动者（actor）的动作状态。
* GameResource - `GameResource` 类记录游戏所需的全局资源，例如 `Library` 和 `CardDisplay`。

![数据模型]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Game_Model.png)

玩家是行动者，可以是人类或计算机（AI）。每位玩家必须就游戏流程与规则做出决策。`Decision` 是抽象类，可以实现为交互式的 `UIDecision`——人类在游戏动作时被调用，或实现为自动化的 `AIDecision`——计算机被调用行动。

### 创建游戏
创建一个游戏会话，初始化步骤如下：
* 创建 `Library` 并载入卡牌设计与图像
* 创建 `CardDisplay` 以提供便利的卡牌图像展示
* 创建引用 `Library` 的一叠 `Deck` 卡牌
* 创建一组 `Player`，可以是人类或计算机 AI
* 创建 `GameResource`，由 `Library` 和 `Display` 组成，将来可扩展更多游戏资源，例如网络通信与数据存储。
* 最后，`Game` 由 `GameResource`、`Deck` 和 `Player` 列表组成。

稍后我们将讨论 `Player` 如何把游戏控制委托给 `Decision`。

```python
from rftg.cards import Library, Deck, Card
from rftg.display import Display, CardDisplay
from rftg.game import GameResource, Game, Player

# create the game resources
library = Library()
library.read_cards('cards.txt')
library.read_card_images('card_images')
library.load_actions('card_images')
print('Designs: {}'.format(len(library.designs)))

display = Display('card_images', figsize=(16,8))
card_display = CardDisplay(library, display)

# build the game deck
deck = Deck(library)
deck.build_deck(0)
print('Cards: {}'.format(len(deck.cards)))

# create the players
player1 = Player(name="Blue", ai=False)
player2 = Player(name="Red", ai=False)
players = [player1, player2]

# create a new game with players
rresource = GameResource(library=library, display=display)
game = Game(resource=resource, session_id='testing', deck=deck, players=players)
```

`Game` 还可以接受 `start_seed` 参数为随机数生成器播种。默认情况下，如果未指定，它会随机生成一个 `start_seed`。否则，它将使用给定的 `start_seed`，从而可以重放游戏的随机序列。

例如，使用 `start_seed=280966`

```python
game = Game(resource=resource, session_id='testing', deck=deck, players=players,
            start_seed=280966)
```

### 牌库操作
游戏模型构建完成后，我们从游戏的牌库操作开始，例如挑选起始星球和从牌库随机抽卡。

#### 挑选起始星球
RFTG 基础游戏的起始星球 `Card` 带有 `CardFlag.START` 标志。`get_cards_with_flag` 可以搜索卡牌找到起始星球卡。从牌库中识别出这些卡之后，我们用 `pick_cards` 把它们取出（即从牌库移除）。`card.location` 将从 `Location.DECK` 变为 `Location.HAND`。

```python
start_cards = game.deck.get_cards_with_flags(CardFlag.START)
game.deck.pick_cards(start_cards)
print(start_cards)
print(game.deck.remain_cards())

game.resource.display.plot_cards(start_cards, len(start_cards))
game.resource.display.show()
```

```
[<Card 6/Old Earth/Location.HAND/-1>, <Card 7/Epsilon Eridani/Location.HAND/-1>,
<Card 8/Alpha Centauri/Location.HAND/-1>, <Card 9/New Sparta/Location.HAND/-1>,
<Card 10/Earth's Lost Colony/Location.HAND/-1>]
90
```

![从牌库挑选起始星球]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Game_Deck_Start_World.png)

#### 随机抽 6 张牌
剩余的卡牌用 `shuffle_cards` 洗牌。随后，我们用 `draw_cards` 从牌库抽 6 张。`card.location` 将从 `Location.DECK` 变为 `Location.HAND`。抽出的牌分配给一位玩家，例如 player_index=0。

```python
game.deck.shuffle_cards()
cards = game.deck.draw_cards(6)
game.deck.set_cards_player(cards, player_index=0)
print(cards)
print(game.deck.remain_cards())

game.resource.display.plot_cards(cards, len(cards))
game.resource.display.show()
```

```
[<Card 76/Merchant Guild/Location.HAND/0>, <Card 18/Rebel Miners/Location.HAND/0>,
<Card 24/Contact Specialist/Location.HAND/0>, <Card 33/Rebel Outpost/Location.HAND/0>,
<Card 12/Public Works/Location.HAND/0>, <Card 44/Investment Credits/Location.HAND/0>]
84
```

![从牌库随机抽 6 张牌]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Game_Deck_Draw_6_Cards.png)

### 决策
我们必须把游戏状态呈现给行动者——无论人类还是 AI——以做出决策。`Decision` 是一个抽象超类，定义了全部决策操作：`UIDecision` 为人类行动者实现，`AIDecision` 为计算机 AI 行动者实现。

```python
from rftg.decision import *
from rftg.enums import *

decision = UIDecision(game)
```

决策对象必须用 `Game` 模型初始化。我们从几个决策操作的示例开始。由于我们使用的是 `UIDecision` 实现，游戏会展示卡牌等待人类输入。

#### 决定起始星球
游戏必须向行动者（例如玩家 0）呈现起始星球卡列表，以选择起始星球。这里仅作演示。更现实的做法是把起始星球随机分配给玩家。

```python
decision.make_choice(0, Choice.START, cards=start_cards)
```

`UIDecision make_choice` 会把 `Choice.START` 路由为交互式询问起始星球卡。行动者从卡牌列表中选择一个数字来选定星球，游戏将记录玩家选定的起始星球。

![玩家决策：选择起始星球]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Decision_01_Choose_Start_World.png)

例如，玩家 0 选择了 "4"，对应 "Earth's Lost Colony"。我们调用游戏显示，把玩家的 `Location.ACTIVE` 绘制到玩家的 tableau。此外，我们还绘制玩家 `Location.HAND` 的手牌。

```python
game.plot_player_cards(0, Location.ACTIVE)
game.plot_player_cards(0, Location.HAND)
```

![玩家决策后：已选起始星球与手牌]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Decision_02_Show_Start_World_and_On_Hand.png)

#### 决定行动
行动者的下一个重要决策是选择行动。游戏必须展示行动列表（例如对玩家 0）以供选择。

```python
decision.make_choice(0, Choice.ACTION)
```

`UIDecision make_choice` 会把 `Choice.ACTION` 路由为交互式询问行动。行动者必须选择一个行动，游戏将记录玩家选定的行动。

![玩家决策：选择一个行动]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Decision_03_Show_Actions.png)

例如，玩家 0 选择了行动 "2"，对应 `Action.DEVELOP`。然后我们绘制玩家当前的行动。

```python
game.plot_player_actions(0)
```

![显示玩家对所选行动的决策]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Decision_04_Chosen_Actions.png)

#### 弃牌
由于卡牌在游戏中充当主要货币，玩家必须决定弃掉若干卡牌作为各种用途的支付。

```python
cards = game.get_player_cards(0, location=Location.HAND)
decision.make_choice(0, Choice.DISCARD, cards=cards, discard=2)
```

例如，玩家选择弃牌 "1,4"，其中 "," 分隔多张卡牌的选择。弃牌行动后，玩家的 `Location.ACTIVE` 和 `Location.HAND` 卡牌如图所示。

![显示玩家的弃牌决策]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Decision_05_Discard_Action.png)

## 开始游戏
最后，游戏函数已经足够，可以展示一个完整示例：构建游戏模型并启动一局双人游戏。

```python
from rftg.enums import *
from rftg.cards import Library
from rftg.display import CardDisplay
from rftg.game import GameResource, Game, Player

library = Library()
card_display = CardDisplay(library)
resource = GameResource(library=library, display=card_display)

player1 = Player(name="Blue", ai=False)
player2 = Player(name="Red", ai=False)
players = [player1, player2]
game = Game(resource=resource, session_id='testing', expanded=0, players=players)
```

游戏以洗好的牌库开始。为每位玩家随机分配一个起始星球，并从牌库抽 6 张随机卡牌。

```python
game.deck.shuffle_cards()
start_worlds = game.deck.get_random_start_world_cards()
for player_index in range(len(game.players)):
  player = players[player_index]
  print('Player:{} Hand'.format(player.name))

  # assign player start world
  player.start = start_worlds[player_index].index
  game.deck.set_cards_player([ start_worlds[player_index] ], player_index)
  game.deck.set_cards_location([ start_worlds[player_index] ], Location.ACTIVE)

  # draw player hand
  game.deck.draw_cards(number=6, player_index=player_index)
  game.plot_player_cards(player_index=player_index, location=Location.ACTIVE)
  game.plot_player_cards(player_index=player_index, location=Location.HAND)
```

玩家 "Blue" 的 `Location.ACTIVE` 与 `Location.HAND` 卡牌的绘制。

![游戏演示玩家 Blue 的手牌]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Game_Demo_Blue_Hand.png)

玩家 "Red" 的 `Location.ACTIVE` 与 `Location.HAND` 卡牌的绘制。

![游戏演示玩家 Red 的手牌]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Game_Demo_Red_Hand.png)

## 游戏界面
到目前为止，我们用 `matplotlib` 展示游戏图形并提供用户交互。Jupyter notebook 的方式纯粹出于实验目的，离令人满意的游戏界面还很远。本节中，我们将开始用更好的游戏框架改造游戏界面。

[cocos2d](#Cocos2d) 是一个构建 2D 游戏、演示及其他图形/交互式应用的框架。
它可在 Windows、OS X 和 Linux 上运行，面向用 Python 语言编写的应用。

cocos2d 构建在 [pyglet](#pyglet) 之上。pyglet 是 Python 的跨平台窗口与多媒体库，面向游戏开发及其他视觉丰富的应用。它支持窗口、用户界面事件处理、游戏手柄与摇杆、OpenGL 图形、加载图像和视频、播放声音与音乐。pyglet 可在 Windows、OS X 和 Linux 上运行。

### 安装 Cocos2d
下载并安装 `cocos2d` 非常简单，适用于任何平台。

```
python setup.py install
```

> 有一个 bug：[Mac 上 layer 尺寸是窗口尺寸的四分之一](https://github.com/los-cocos/cocos/issues/303)

#### Mac 上的补丁
应用以下补丁修复 Mac 视网膜（retina）显示问题，之后就能正常工作了！

```
diff --git a/cocos/director.py b/cocos/director.py
index 516fb8b..34aad7e 100644
--- a/cocos/director.py
+++ b/cocos/director.py
@@ -595,6 +595,12 @@ def scaled_resize_window(self, width, height):
         """
         # physical view size
         pw, ph = width, height
+        # Are we on HiDPI display?
+        viewportw, viewporth = self.window.get_viewport_size()
+        if viewportw > pw:
+            # yes
+            pw = viewportw
+            ph = viewporth
         # virtual (desired) view size
         vw, vh = self.get_window_size()
         # desired aspect ratio
@@ -642,6 +648,12 @@ def unscaled_resize_window(self, width, height):
         """
         self._usable_width = width
         self._usable_height = height
+        # Are we on HiDPI display?
+        viewportw, viewporth = self.window.get_viewport_size()
+        if viewportw > width:
+            # yes
+            self._usable_width = viewportw
+            self._usable_height = viewporth
 
         if self._resize_no_events:
             # setting viewport geometry, not handling an event
```

### 读取并展示卡牌

#### 以 pyglet 图像格式读取卡牌图像
在 `Library` 中，我们将以 pyglet 图像格式读取卡牌图像，使其可以直接用作纹理。

```python
  def read_card_image(self, card_path, card_index):
    card_name = self.card_image_name(card_index)
    card_filename = '{}/{}.png'.format(card_path, card_name)
    # card_image = mpimg.imread(card_filename)
    card_image = pyglet.resource.image(card_filename)
    return card_image
```

> 注意，如果想以 PIL 图像导入，可以用以下代码把它转换为 pyglet 图像。

```python
temp_image = PIL.Image.open("card_images/card006.png")
raw_image = temp_image.tobytes()  # tostring is deprecated, must use tobytes()
image = pyglet.image.ImageData(temp_image.width, temp_image.height,
                              'RGB', raw_image, pitch=-temp_image.width * 3)
```

### 载入卡牌库并定义游戏
我们需要载入卡牌库并准备游戏牌库。

```python
from rftg.enums import *
from rftg.cards import Library, Deck, Card
from rftg.game import GameResource, Game, Player

library = Library()
library.setup()
print('Designs: {}'.format(len(library.designs)))

resource = GameResource(library=library)
player1 = Player(name="Blue", ai=False)
player2 = Player(name="Red", ai=False)
players = [player1, player2]
game = Game(resource=resource, session_id='testing', expanded=0, players=players)
```

### 展示卡牌精灵与动画
要使用 `cocos2d`，以下类是构建的最低要求：
* `Sprite` 表示一张可以在玩家区域上做动画的卡牌图像
* `Layer` 容纳 `Sprite`
* `Scene` 描述一组 `Layer`
* `Director` 绘制窗口并运行 `Scene`

```python
from pyglet.gl import *
from pyglet.window import key

from cocos.actions import *
from cocos.director import director
from cocos.layer import Layer
from cocos.scene import Scene
from cocos.sprite import Sprite
```

我们将定义 `CardsContainer`——cocos `Layer` 的子类——来收集一组 `CardLayer`。
cocos `Layer` 可以递归地包含其他 `Layer` 类。`CardsContain` 负责卡牌的布局与交互。

```python
class CardsContainer(Layer):
  is_event_handler = True   #: enable pyglet's events

  def __init__(self, library:Library, pos, pickable=True):
    super(CardsContainer, self).__init__()

    self.library = library
    self.pos = pos
    self.pickable = pickable

  def add_cards(self, cards):
    dx = 800 / (len(cards) + 1)

    index = 1
    for card in cards:
      pos = (
        int(self.pos[0] + (dx * index)),
        self.pos[1]
        )
      index += 1
      cardNode = CardLayer(library, card, pos)
      self.add(cardNode)

  def on_mouse_press(self, x, y, buttons, modifiers):
    """This function is called when any mouse button is pressed

    (x, y) are the physical coordinates of the mouse
    'buttons' is a bitwise or of pyglet.window.mouse constants LEFT, MIDDLE, RIGHT
    'modifiers' is a bitwise or of pyglet.window.key modifier constants
       (values like 'SHIFT', 'OPTION', 'ALT')
    """
    if self.pickable:
      cardNodes = self.get_children()
      for cardNode in cardNodes:
        cardNode.is_picked(x, y)
```

`CardLayer` 也是 cocos `Layer` 的子类。它负责持有卡牌图像及其显示状态。例如，卡牌是否已被选中。

```python
class CardLayer(Layer):

  is_event_handler = False #: enable pyglet's events

  def __init__(self, library:Library, card:Card, pos):
    super(CardLayer, self).__init__()

    self.card_index = card.index
    self.image = library.card_by_id(card.index).image
    self.image.anchor_x = self.image.width // 2
    self.image.anchor_y = self.image.height // 2
    self.pos = pos
    self.sprite = Sprite(self.image, scale=0.25)
    self.add(self.sprite)
    self.selected = False
    self.bbox = self.get_bbox()

  def get_bbox(self):
    lx = self.pos[0] - (self.sprite.width / 2.0)
    hx = self.pos[0] + (self.sprite.width / 2.0)
    ly = self.pos[1] - (self.sprite.height / 2.0)
    hy = self.pos[1] + (self.sprite.height / 2.0)
    return (lx, ly, hx, hy)

  def is_picked(self, x, y):
    (lx, ly, hx, hy) = self.bbox
    if (lx <= x) and (x <= hx) and (ly <= y) and (y <= hy):
      self.do_selected(selected=True)
    else:
      self.do_selected(selected=False)

  def do_selected(self, selected=False):
    if self.selected:
      if not selected:
        self.sprite.do(MoveBy((0, -PICK_OFFSET), ANIMATE_DURATION))
    else:
      if selected:
        self.sprite.do(MoveBy((0, PICK_OFFSET), ANIMATE_DURATION))
    self.selected = selected
    self.bbox = self.get_bbox()

  def on_enter(self):
    super(CardLayer, self).on_enter()

    self.sprite.position = WINDOW_WIDTH/2, WINDOW_HEIGHT-100
    self.sprite.do(Delay(START_DELAY) + MoveTo((self.pos), ANIMATE_DURATION))
```

`cocos2d` 的 `Director` 负责创建并运行动画。所有卡牌 `Sprite` 被定义并插入一个 `Scene`。`Director` 将取得 `Scene` 并运行其中的节点。

```python
director.init(width=WINDOW_WIDTH, height=WINDOW_HEIGHT, resizable=True, caption='Race for the Galaxy - Cocos')
playerActive = CardsContainer(library, (0, 300), pickable=False)
playerActive.add_cards(game.get_player_cards(player_index, location=Location.ACTIVE))
playerHand = CardsContainer(library, (0, 100), pickable=True)
playerHand.add_cards(game.get_player_cards(player_index, location=Location.HAND))
scene = Scene(
  playerActive,
  playerHand
)
director.run(scene)
```

用 cocos2d 框架之后，游戏界面好看多了。

![银河竞逐 Python]({{ site.baseurl }}/images/game-architecture-card-ai-2/RFTG_Cocos_Python_12fps.gif)
*图：展示这款卡牌游戏以 Python 设计并重写。游戏 UI 与动画用 Cocos2d 的 Python 版实现。（版权说明：图像版权归 Rio Grande Games 所有）*

## 结语
我们探索了 (3) 游戏引擎与 (4) 游戏界面的开发。游戏状态被呈现给行动者——无论人类还是 AI——以做出决策。本部分展示了面向人类行动者的 `UIDecision`，以演示抽象 `Decision` 类的用法。AI 代理将在下一篇文章（第 3 部分）驱动 `AIDecision` 实现。在结尾，我们把游戏界面转换到 cocos2d 的 Python 版。这款卡牌游戏实验开始变得有趣得多。

* [卡牌游戏模型的游戏架构（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)
* **>>** [卡牌游戏行动的游戏架构（第 2 部分）]({{ site.baseurl }}/game-architecture-card-ai-2)
* [卡牌游戏 AI 的游戏架构（第 3 部分）]({{ site.baseurl }}/game-architecture-card-ai-3)

## 参考文献

### 银河竞逐
* <a name="Jones09">[[Jones09]](https://boardgamegeek.com/thread/438698/article/3878167?fbclid=IwAR1KXYI1li66vhPBPwzwNfr8Tvg2Giz5zN5eXpVaugtCQe8DLUbILjzBMUE#3878167)</a> Keldon Jones, Talk a bit about how the AI works, Sep 2009
  * 最新源码 0.9.5，含全部扩展 <https://github.com/bnordli/rftg>
* <a name="Tesauro95">[[Tesauro95]](https://bkgm.com/articles/tesauro/tdl.html)</a> Gerald Tesauro, Temporal Difference Learning and TD-Gammon, Communications of the ACM, March 1995 / Vol. 38, No. 3
* <a name="TempleGates17">[[TemplateGates17]](https://www.templegatesgames.com/race-for-the-galaxy-ai/)</a> Race for the Galaxy AI, Temple Gates, Dec 2017
  * Temple Gates 是使用 Keldon Jones 的 AI 引擎的 App 版游戏开发商
* <a name="Cheung21">[[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python)</a> Benny Cheung, Game Architecture for Card Game Action (Part 2) - Jupyter Notebook, Jun 2021

### 界面开发
* <a name="Cocos2d">[[Cocos2d]](http://python.cocos2d.org/doc/index.html#:~:text=cocos2d%20is%20a%20framework%20for,written%20in%20the%20python%20language)</a> cocos2d 0.6.5 Documentation
  * 基本概念 <http://python.cocos2d.org/doc/programming_guide/basic_concepts.html>
  * 源码 <https://github.com/los-cocos/cocos
* <a name="pyglet">[[pyglet]](https://pyglet.readthedocs.io/en/latest/)</a> pyglet Documentation
  * cocos2d 使用 pyglet 事件框架处理事件，<https://pyglet.readthedocs.io/en/latest/programming_guide/events.html>
  * pyglet.window 事件 <https://pyglet.readthedocs.io/en/latest/modules/window.html>
