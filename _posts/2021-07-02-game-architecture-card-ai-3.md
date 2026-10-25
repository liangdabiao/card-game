---
layout: post
title: 卡牌游戏 AI 的游戏架构（第 3 部分）
date: 2021-07-03 12:00:00.000000000 -00:00
permalink: game-architecture-card-ai-3
author: Benny Cheung
artwork:
  author: Board Game Arena, Race for the Galaxy Board Game
tags:
- Game Architecture
- Card Game
- Python
- AI
category: post
comments: true
image: images/game-architecture-card-ai-3/cover2x.jpg
images:
  cover: images/game-architecture-card-ai-3/cover.jpg
  header: images/game-architecture-card-ai-3/header.jpg
published: true
series: "Game Architecture"
series_order: 3
---

<!--excerpt.start-->
"卡牌游戏的游戏架构"系列的最后一篇将聚焦了不起的《银河竞逐》（Race for the Galaxy）AI。尽管 Keldon Jones 早在 2009 年就发布了他的 RFTG AI 源码 [[Jones09]](#Jones09)，它已经在使用神经网络与强化学习来训练游戏 AI——远早于 DeepMind 的 AlphaGo 用成功吸引全世界目光于强化学习之前。
<!--excerpt.end-->

卡牌游戏架构的核心是让人类玩家保持投入、为游戏提供无尽重玩价值的 AI。我们必须赋予 AI 代理（无论单个还是一组玩家）依据游戏规则与逻辑做出智能决策的能力。AI 行动将是影响未来游戏状态的走法。决策的奖励很简单："赢得"目标游戏。

----
> [2024/10/04] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/game-architecture-card-ai-3/Podcast_Card_Game_AI_Part_3.mp3
{% include open-embed.html %}
----

![游戏架构总览]({{ site.baseurl }}/images/game-architecture-card-ai-3/Game_Architecture_Part3.png)
*图：游戏架构总览——组件按其在系统中的功能角色分组。功能角色为 (1) 游戏故事与游戏资产，(2) 游戏模型，(3) 游戏引擎，(4) 游戏界面，**(5) 游戏 AI**，(6) 游戏物理（仅物理游戏需要），以及 (7) 硬件抽象。研究任何游戏的源码时，这一架构都有助于归类其功能角色*

要训练一个强化学习（RL）代理学会聪明地玩卡牌游戏，
需要搭建一个完整的游戏环境，捕捉全部机制与规则，让代理能像真正的人类玩家那样与游戏交互。最难的部分是开发一个能与自己聪明对弈的游戏。带着这一思路，在卡牌游戏中使用 RL 的研究日益流行，例如卡牌游戏 Easy21 [[Amrouche20]](#Amrouche20)、UNO [[Pfann21]](#Pfann21)。RLCard 工具包 [[Daochen19]](#Daochen19) 更进一步，为卡牌游戏研究提供了通用的 RL 框架。本文将继续聚焦《银河竞逐》卡牌游戏。

不过，RFTG 卡牌游戏的规则复杂度很难用良定义的游戏状态表示来表达，不像 Easy21 或 UNO 那么简单。我们必须提升抽象层级来描述 RFTG AI。我们希望以下大纲能引导读者走上正确的方向。

## RFTG Python 开发
有兴趣的读者可以从 [[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python) 找到完整的开发环境搭建说明、Python 源码以及本文描述的 Jupyter notebook 实验。

### Jupyter Notebook 实验（第 3 部分）
第 3 部分的开发实验记录在 Jupyter Notebook `rftg_ai.ipynb` 中，可以快速运行代码示例。
在 Visual Studio code 中安装 Microsoft 的"Jupyter"扩展。在 VScode 中打开 `rftg_ai.ipynb` 后，把 Python kernel 切换为 `rftg`——它已按代码 `README.md` 的说明配置好。

![运行 VSCode Jupyter Notebook]({{ site.baseurl }}/images/game-architecture-card-ai-3/RFTG_VScode_Jupyter_Notebook-annotated.png)

### AI Notebook 的安装
运行这个 AI notebook `rftg_ai.ipynb` 需要以下 Python 模块。

```
pip install keras==2.4.3
pip install tensorflow==2.5.0
pip install pydot
```

此外，我们需要定义 `.keras/keras.json` 使用 tensorflow 后端。

```json
 {
  "image_data_format": "channels_last",
  "epsilon": 1e-07,
  "floatx": "float32",
  "backend": "tensorflow"
}
```

## 时序差分——强化学习
汇总 [Keldon 的帖子](#Jones09)与 [Temple Gate 的博客文章](#TempleGates17)提供的信息，我们了解到 RFTG AI 沿用 [Tesauro 的 TD-Gammon](#Tesauro95) 思想，使用时序差分（Temporal Difference，TD）神经网络。是的，神经网络与强化学习技术自 90 年代起就已应用于游戏。

强化学习最大的吸引力之一是：它不需要预定义模型（即 model-free），也无需人类输入来生成训练数据。神经网络通过反复与自己对弈来学习，例如，RFTG AI 经过 30,000 局模拟对局的迭代训练，找出神经网络节点的权重。
这一学习算法的创新思想在于：每一轮之后更新神经网络的权重，以缩小它对之前各轮棋局位置的评估与它对当前轮棋局位置评估之间的差值——故名"时序差分学习"。

在网络训练期间，AI 在每一轮检视一组可能的合法走法及其全部可能回应（经由最优策略），把每个产生的游戏状态喂入其评估函数，并选择能让玩家取得最高分的行动。TD 的创新在于它用强化学习增量地学习其评估函数的方式。

每一轮之后，学习算法按以下规则更新神经网络中的每个权重：

$$
w_{t+1} - w_t = \alpha(Y_{t+1} - Y_t)\sum_{k=1}^{t}\lambda^{t-k} \nabla_w Y_k
$$

其中：

* $w_{t+1} - w_t$ 是相对上一轮的值要改变的权重数量。
* $Y_{t+1} - Y_t$ 是当前轮与上一轮棋局评估之间的差值。
* $\alpha$ 是"学习率"参数。
* $\lambda$ 是一个参数，决定当前棋局评估的差值应向先前的估计反馈多少。$\lambda = 0$ 使程序只修正上一轮的估计；$\lambda = 1$ 使程序试图修正之前所有轮次的估计；0 与 1 之间的 $\lambda$ 值规定了较早估计的重要性随时间"衰减"的不同速率。
* $\nabla_w Y_k$ 是神经网络输出相对权重的梯度：即改变权重对输出有多大影响。

关于更多强化学习内容，读者可以参阅 Tambet Matiisen 的出色教程 [[Matiisen15]](#Matiisen15)、David Silver 的 DeepMind RL 讲座 [[Silver17]](Silver17)，以及 Sutton 的经典 RL 教材 [[Sutton17]](Sutton17)。

### 游戏状态
游戏状态可以表示决策者可获得的任何有助于描述当前局势的信息。这可以是玩家手牌的类型、对手持有卡牌的数量，或关于已打出卡牌的信息。毫无疑问，RFTG 是一款复杂的卡牌游戏，因为它有一套精细的规则与行动阶段。

通过阅读输入节点名的列表，我们可以理解游戏状态是如何喂入网络的。游戏状态可以是，

* 游戏是否结束
* VP 池大小
* 最大场上卡牌数
* 手牌类型
* 手牌数量
* 玩家货物数量
* 对手货物数量
* 等等

清单可以从双人游戏的 *700 多个*一直到五人游戏的 *1,800 多个*（使用 expansion=6）。

### 神经网络
这些输入并非只喂入一个神经网络。根据 Keldon 的代码，他有十二个独特的神经网络模型，每个针对不同的扩展与玩家人数训练。如果我们跑一局双人游戏，AI 使用的网络与三人游戏不同。对每个游戏模型，有两种神经网络在起作用，各有其主函数。

例如，
* Eval 网络——expansion 0（基础游戏）2 人游戏的 `eval` 权重文件 `rftg.eval.0.2.net`
* Role 网络——expansion 0（基础游戏）2 人游戏的 `role` 权重文件 `rftg.role.0.2.net`

两种网络权重文件格式相同；内容随以下变化，
* 输入层、隐藏层、输出层大小
  * 例如 `rftg.eval.0.2.net`：(704, 50, 2)
  * 例如 `rftg.role.0.2.net`：(605, 50, 7)
* 训练迭代次数
* 输入节点名列表
* 各层节点的权重

下面的 Python 类 `Network` 是网络权重加载器的实现。

```python
import numpy as np

# RFTG Network Loader
class Network:

	def __init__(self):
		pass

	def network_name(self, network, expansion, players, advanced=False):
		network_name = "rftg.%s.%d.%d%s.net" %(network, expansion, players, ("a" if advanced else ""))
		return network_name

	def load_net(self, network, expansion, players, advanced=False):
		self.network = network
		self.expansion = expansion
		self.players = players
		self.advanced = advanced
		self.network_name = self.network_name(network, expansion, players, advanced)

		filename = 'network/{}'.format(self.network_name)
		fp = open(filename)
		# read network layers size
		(input, hidden, output) = fp.readline().strip().split(' ')
		print(input, hidden, output)
		# read number of training iterations
		training = fp.readline().strip()

		self.num_input = int(input)
		self.num_hidden = int(hidden)
		self.num_output = int(output)

		# read input names
		self.input_names = []
		for i in range(0, self.num_input):
			name = fp.readline().strip()
			self.input_names.append(name)

		self.input = np.zeros(shape=(self.num_input, 1))
		print('Input Layer: ', self.input.shape)

		# read hidden nodes
		self.hidden = np.ndarray(shape=(self.num_input, self.num_hidden))
		for r in range(0, self.num_hidden):
			for c in range(0, self.num_input):
				self.hidden[c, r] = float(fp.readline().strip())
		print('Hidden Layer: ', self.hidden.shape)

		# read output nodes
		self.output = np.ndarray(shape=(self.num_hidden, self.num_output))
		for r in range(0, self.num_output):
			for c in range(0, self.num_hidden):
				self.output[c, r] = float(fp.readline().strip())
		print('Output Layer: ', self.output.shape)

		# all done
		fp.close()
```

例如，读取 expansion 0、2 人、非 advanced 游戏的 `role` 网络。网络文件为 `rftg.role.0.2.net`。

```python
network = Network()
network.load_net('role', 0, 2)
```

网络文件信息被打印出来。

```
605 50 7
Input Layer:  (605, 1)
Hidden Layer:  (605, 50)
Output Layer:  (50, 7)
```

#### Eval 网络
`eval` 网络的输出可以被理解为每位玩家的获胜概率。每一行对应从该玩家视角计算 eval 网络，因为有些信息只有特定玩家知道，例如手牌。例如，下面的 `eval` 网络面向 2 人游戏，即输出层为每位玩家的获胜分数提供 2 个输出。

![RFTG Eval 神经网络]({{ site.baseurl }}/images/game-architecture-card-ai-3/RFTG_Eval_NN.png)
*图：展示全连接的 `eval` 网络层。（注：为减少线条，未画出全部连接。）输入层节点是设计好的游戏状态，输出层节点是每位玩家的获胜分数（本例中 2 名玩家 2 个节点）。*

#### Role 网络
还有一个重要决策是在每轮开始时选择哪个行动。这一决策的处理稍有不同。首先，AI 预测每个对手可能做什么。这是用第二个 `role` 网络完成的。这个网络与 eval 网络非常相似，区别是它为玩家可能做出的 7 类行动选择分别提供一个输出。

![RFTG Role 神经网络]({{ site.baseurl }}/images/game-architecture-card-ai-3/RFTG_Role_NN.png)

*图：展示全连接的 `role` 网络层。（注：为减少线条，未画出全部连接。）输入层节点是设计好的游戏状态，输出层节点是 7 类行动选择。*

### 决策
使用网络很简单：把全部所需的游戏状态收集起来作为网络输入。
两个网络为游戏状态打分之后，分数代表玩家获胜的概率与行动选择的角色，由 AI 处理并为一个行动做出决策。

![RFTG AI 决策]({{ site.baseurl }}/images/game-architecture-card-ai-3/RFTG_Game_Model_Decision.png)

*图：抽象超类 `Decisions` 的函数被 `AIDecisions` 函数的实现所覆写*

正如我们已经考察过的，`Decision` 是一个抽象超类，定义了全部决策操作：`UIDecision` 为人类行动者实现，`AIDecision` 为计算机 AI 行动者实现。AI 代理决定打出的卡牌，代表它正在采取的行动。

做出选择的逻辑还有很多，我们不去深挖细节。AI 的 `make_choice` 函数 `choice_actions` 会把 `Choice` 枚举映射到相应的行动函数。

```python
  def make_choice(self, who, type, **kwargs):
    print('{} make_choice'.format(self.__class__.__name__))
    choice_action = {
      Choice.ACTION:  self.choose_action,
      Choice.START:   self.choose_start,
      Choice.DISCARD: self.choose_discard,
      Choice.PLACE:   self.choose_place,
      Choice.PAYMENT: self.choose_payment,
      Choice.SETTLE:  self.choose_settle,
      Choice.TRADE:   self.choose_trade,
      Choice.CONSUME: self.choose_consume,
      Choice.CONSUME_HAND: self.choose_consume_hand,
      Choice.GOOD:    self.choose_good,
      Choice.LUCKY:   self.choose_lucky,
      Choice.WINDFALL: self.choose_windfall,
      Choice.PRODUCE: self.choose_produce,
    }
```

## 用 Keras 构建深度神经网络
本节中，我们接着使用 [Keras: the Python Deep Learning API](https://keras.io/) 重写 RFTG 的神经网络。虽然感觉有点大材小用，但目的是演示如何用这个强大的工具包获得更大的网络灵活性；随后，我们有望扩展网络能力，例如使用 LSTM（长短期记忆）网络。学习 Keras，我们推荐 Keras 作者 Francois Chollet 的书 [[Chollet18]](#Chollet18)。

### 定义 Keras 层
这个 Keras 层只接受第一维等于网络输入大小（例如 605）的 2D 张量作为输入（axis 0，即 batch 维，未指定，因此接受任意值）。这一层将返回一个张量，其第一维被变换为 50。因此这一层只能连接到期望 50 维向量输入的下游层。

第二层没有接收输入形状参数——它自动把输入形状推断为前一层的输出形状。

最后一层使用 `softmax` 激活。这意味着网络将在 7 个不同输出类别上输出概率分布——对每个输入样本，网络产生一个 7 维输出向量，其中 output[i] 是样本属于类别 i（即 role 选择）的概率。7 个分数之和为 1。

```python
from keras import models
from keras import layers

model = models.Sequential(name=network.network_name)
# input layer implicit in the input_shape data
# hidden layer
model.add(layers.Dense(50, activation='relu', input_shape=(network.num_input,), name='hidden'))
# output layer
model.add(layers.Dense(7, activation='softmax', name='output') )
```

### 可视化模型
可以通过调用模型的 summary() 函数创建摘要，它返回一个字符串，进而可以打印。

```python
model.summary()
```

```
Model: "rftg.role.0.2.net"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
hidden (Dense)               (None, 50)                30300     
_________________________________________________________________
output (Dense)               (None, 7)                 357       
=================================================================
Total params: 30,657
Trainable params: 30,657
Non-trainable params: 0
_________________________________________________________________
```

摘要有助于理解简单模型，但对具有多输入或多输出的模型可能令人困惑。Keras 还提供了绘制网络神经网络图的函数，让更复杂的模型更易理解。

Keras 的 `plot_model()` 函数会为你的网络创建图示。

```python
from keras.utils.vis_utils import plot_model

plot_filename='{}.png'.format(network.network_name)
plot_model(model, to_file=plot_filename, show_shapes=True, show_layer_names=True)
```

![绘制 Keras DNN 网络]({{ site.baseurl }}/images/game-architecture-card-ai-3/rftg.role.0.2.net.png)


### 把网络权重载入各层
我们已经加载了 RFTG `role` 网络权重。定义好 Keras 模型之后，我们可以把权重分配到模型的相应层：
* 加载的 network.hidden -> hidden_layer
* 加载的 network.output -> output_layer

```python
# find the DNN layer by name
hidden_layer = model.get_layer('hidden')
output_layer = model.get_layer('output')

# set the loaded network weights into the DNN layers
hidden_layer.set_weights([network.hidden, np.zeros(shape=hidden_layer.output_shape[1])])
output_layer.set_weights([network.output, np.zeros(shape=output_layer.output_shape[1])])
```

我们可以检查隐藏层的权重。

```python
hidden_layer.get_weights()

[array([[ 0.02684849, -0.07043163,  0.17830057, ...,  0.08084569,
         -0.09834311,  0.3189879 ],
        [ 0.03219581,  0.06759464, -0.08408735, ...,  0.08180485,
          0.12012478,  0.03991435],
        [-0.07090639, -0.01249493,  0.05813789, ...,  0.04127461,
          0.0998257 , -0.01798295],
        ...,
        [-0.8491952 , -0.30811298, -0.12133851, ...,  0.01339607,
         -0.10589286,  0.00837689],
        [ 1.2423377 , -0.0644151 ,  0.24686392, ...,  0.06034593,
          0.04830525,  0.00200542],
        [-0.5312858 ,  0.17594491,  0.3015494 , ..., -0.10620899,
         -0.03211459, -0.07428143]], dtype=float32),
 array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
        0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
        0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.],
       dtype=float32)]
```

### 用神经网络做预测（打分）
网络权重载入 Keras 模型之后，我们可以验证：模型实例的 `predict()` 方法返回全部 7 个角色上的概率分布。

生成一组随机游戏状态作为测试输入数据。

```python
random_input = np.random.randint(2, size=hidden_layer.input_shape[1])
x_input = np.array([random_input])
```

我们的输入向量将是 605 维的（0 或 1）。

```
array([[0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 1, 1, 0,
        0, 0, 1, 0, 1, 1, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 0, 1, 0,
        ...
        1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 0, 0, 1, 0, 0, 1, 0, 1, 1, 0, 1, 0,
        1, 1, 1, 0, 0, 1, 1, 0, 0, 0, 0]])
```

我们其实不知道这个随机游戏状态向量意味着什么。随机游戏状态多半毫无意义。出于兴趣，我们可以打印网络输入名来核对。这些输入名加载自 RFTG 的网络文件。

```python
for i in range(0, len(random_input)):
  if random_input[i] == 1:
    print('{}'.format(network.input_names[i]))
```

这只是演示如何用 Keras 模型对输出预测（即 `role` 选择）打分。输出预测（即角色）如下，

| 角色索引 | 角色名 |
|-------|------|
| 0 | Explore +5
| 1 | Explore +1,+1
| 2 | Develop
| 3 | Settle
| 4 | Consume-Trade
| 5 | Consume-x2
| 6 | Produce

```python
predictions = model.predict(x_input)
```

```python
predictions

array([[0.15077034, 0.22324738, 0.38111785, 0.03640797, 0.09331292,
        0.07307366, 0.04206994]], dtype=float32)
```

对于这些预测，
* 输出形状为 7
* 总概率之和应为 1
* 最高分是选择索引 2（即 `Develop`，分数约 0.38）。

```python
predictions[0].shape

(7,)
```

```python
np.sum(predictions[0])

1.0
```

```python
np.argmax(predictions[0])

2
```

## 结语
在这最后一篇文章中，我们探索了 RFTG 卡牌游戏中的 (5) 游戏 AI。我们看到了强化学习如何帮助训练一个能自己跟自己对弈的神经网络。神经网络训练完成后，游戏 AI 可以智能地决策，为人类玩家提供无穷的挑战。

此外，我们演示了如何把网络转换为 Keras、重定义 RFTG 各层并对一些随机游戏状态做出预测。对于有经验的 AI 开发者，时序差分、强化学习与深度神经网络技术可以应用于训练其他类型的卡牌游戏。希望这个"卡牌游戏的游戏架构"系列有助于启发更好的卡牌游戏设计，产出更强大的卡牌游戏 AI！

* [卡牌游戏模型的游戏架构（第 1 部分）]({{ site.baseurl }}/game-architecture-card-ai-1)
* [卡牌游戏行动的游戏架构（第 2 部分）]({{ site.baseurl }}/game-architecture-card-ai-2)
* **>>** [卡牌游戏 AI 的游戏架构（第 3 部分）]({{ site.baseurl }}/game-architecture-card-ai-3)

## 参考文献

### 游戏 AI
* <a name="MillingtonFung06">[MillingtonFung06]</a> Ian Millington & John Funge, Artificial Intelligence for Games, 2006, Elsevier, Morgan Kaufmann Pub., ISBN: 978-0-12-497782-2
  * 这是一本覆盖全部游戏 AI 实践、术语与专知的综合参考。Ian Millington 把丰富的职业经验带入改进游戏 AI 质量的问题。他描述了大量来自真实游戏的例子，并通过详细的案例研究探讨背后的思想。

### 银河竞逐 AI
* <a name="Jones09">[[Jones09]](https://boardgamegeek.com/thread/438698/article/3878167?fbclid=IwAR1KXYI1li66vhPBPwzwNfr8Tvg2Giz5zN5eXpVaugtCQe8DLUbILjzBMUE#3878167)</a> Keldon Jones, Talk a bit about how the AI works, Sep 2009
  * 最新源码 0.9.5，含全部扩展 <https://github.com/bnordli/rftg>
* <a name="Tesauro95">[[Tesauro95]](https://bkgm.com/articles/tesauro/tdl.html)</a> Gerald Tesauro, Temporal Difference Learning and TD-Gammon, Communications of the ACM, March 1995 / Vol. 38, No. 3
* <a name="TempleGates17">[[TemplateGates17]](https://www.templegatesgames.com/race-for-the-galaxy-ai/)</a> Race for the Galaxy AI, Temple Gates, Dec 2017
  * Temple Gates 是使用 Keldon Jones 的 AI 引擎的 App 版游戏开发商
* <a name="Cheung21">[[Cheung21]](https://github.com/bennycheung/RaceGalaxyAI-Python)</a> Benny Cheung, Game Architecture for Card Game AI (Part 3) - Jupyter Notebook, Jun 2021

### 强化学习
* <a name="Matiisen15">[[Matiisen15]](https://neuro.cs.ut.ee/demystifying-deep-reinforcement-learning/)</a> Tambet Matiisen, Demystifying Deep Reinforcement Learning, Dec 2015
* <a name="Silver17">[[Silver17]](https://www.youtube.com/watch?v=2pWv7GOvuf0&list=PLzuuYNsE1EZAXYR4FJ75jcJseBmo4KQ9-)</a> David Silver, Reinforcement Learning, a series of 10 youtube video lectures
  * 如果你刚接触 RL，想理解强化学习的数学与哲学背景，这个系列非常宝贵。
* <a name="Sutton17">[Sutton17]</a> Richard Sutton and Andrew Barto, Reinforcement Learning: An Introduction, MIT Press, 2017, ISBN:9780262193986
  * 本书清晰而简明地讲述了强化学习的关键思想与算法，讨论范围从该领域智识根基的历史到最新的进展与应用。
* <a name="Atienza18">[Atienza18]</a> Rowel Atienza, Advanced Deep Learning with Keras, Packt Publishing, 2018, ISBN:978788629416
  * 第 9 章：深度强化学习


### 面向卡牌游戏 AI 的强化学习
* <a name="Amrouche20">[[Amrouche20]](https://towardsdatascience.com/playing-cards-with-reinforcement-learning-1-3-c2dbabcf1df0)</a> Matyas Amrouche, Playing cards with Reinforcement Learning, May 2020
  * Matyas 展示了如何为卡牌游戏 Easy21 思考并实现 RL。
  * Matyas 的 Jupyter notebook 信息量很大，是有用的参考 <https://github.com/Matyyas/Easy21>
* <a name="Pfann21">[[Pfann21]](https://towardsdatascience.com/tackling-uno-card-game-with-reinforcement-learning-fad2fc19355c)</a> Bernhard Pfann, Tackling the UNO Card Game with Reinforcement Learning, Jan 2021
  * Bernhard 的 Jupyter notebook 与 UNO 游戏模型 <https://github.com/bernhard-pfann/uno-card-game_rl>
* <a name="Daochen19">[[Daochen19]](https://arxiv.org/abs/1910.04376)</a> Daochen Zha, et. al., RLCard: A Toolkit for Reinforcement Learning in Card Games, Oct 2019, AAAI-20 Workshop on Reinforcement Learning in Games
  * RLCard 是面向卡牌游戏强化学习（RL）的工具包。它支持多种卡牌环境，接口易用，可实现各种强化学习与搜索算法。RLCard 的目标是连接强化学习与不完全信息游戏。 
  * 源码：<https://github.com/datamllab/rlcard>

### 用 Python 做深度学习
* <a name="Chollet18">[Chollet18]</a> Francois Chollet, Deep Learning with Python, Manning Publications, 2018, ISBN: 9781617294433
