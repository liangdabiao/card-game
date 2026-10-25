---
layout: post
title: 用 Pharo 学习 Smalltalk
date: 2016-04-18 12:00:00.000000000 -05:00
permalink: using-pharo-to-learn-smalltalk
author: Benny Cheung
tags:
  - Smalltalk
  - Pharo
  - RaspberryPi
category: post
comments: true
image: images/using-pharo-to-learn-smalltalk/cover2x.jpg
images:
  cover: images/using-pharo-to-learn-smalltalk/cover.jpg
  header: images/using-pharo-to-learn-smalltalk/header.jpg
published: true
---

<!--excerpt.start-->Pharo 是编程语言与环境 Smalltalk 的一个开源实现。Pharo 源自 Squeak 的一个分支，而 Squeak 是由 Smalltalk-80 团队创建的开源 Smalltalk 环境。本文探讨如何使用 Pharo 学习 Smalltalk，包括 Pharo 独特的包管理工具，以及在树莓派上运行 Pharo。
<!--excerpt.end-->

Inria 是生产 Pharo 以及许多相关软件分析工具的公司。你可以在这里找到工具列表：

* [Inria 软件列表](http://rmod.inria.fr/web/software)

## Pharo 视频教程
以下是一些推荐的 Pharo 学习教程：

* [学习 Pharo](http://www.jarober.com/blog/st4u.ssp)
* [学习更多 Pharo](https://www.youtube.com/playlist?list=PLqbtQ7OkSta0ULYAd7Qdxof851ybh-_m_)

## Pharo 图书
Pharo 团队编写了一本优秀的免费图书，可以从这里下载：

* [《Deep into Pharo》一书](http://deepintopharo.com)

## 演讲
1. Marcus Denker 对 Smalltalk 和 Pharo 环境做了一次很好的介绍：
[Pharo Objects at Your Fingertip](https://www.youtube.com/watch?v=xhPlUaXpCU4) 及
[演讲幻灯片](http://www.slideshare.net/MarcusDenker/pharo-objects-at-your-fingertips)

1. Tudor Girba - Pharo: Playing with Live Object
（其中展示了 AtomMorph 演示）
* [Playing with Live Object](https://vimeo.com/97315968)

1. Laurent Laffont - Manipulating Objects
（其中展示了 AtomMorph 演示；该视频偏技术细节）
* [Manipulating Live Objects](https://vimeo.com/34810463)

Pharo 是面向对象语言圈里酷酷的新面孔。它受 Smalltalk 启发，是动态的，自带一个以对象为中心的活编程环境（live programming environment）。而且，它非常小巧。但最重要的是，它通过挑战几乎所有"约定俗成"的东西，让严肃的编程变得有趣。例如，设想这样一个环境：你可以扩展 Object、修改编译器、定制对象检查器（inspector），甚至构建你自己的领域专用调试器——而且做这一切时甚至不必停机。

## Monticello
大量软件包可以在以下网站找到：

* [Smalltalk Hub 共享包仓库](http://smalltalkhub.com)

找到你喜欢的包之后，可以使用 *Monticello Browser* 添加该包。

![Pharo Monticello 添加仓库示例]({{ site.baseurl }}/images/using-pharo-to-learn-smalltalk/Pharo_Monticello_Add_Repo_Examples_annotated_sm.png)

例如，可以这样添加仓库：

{% highlight smalltalk %}
MCHttpRepository
	location: 'http://smalltalkhub.com/mc/PharoExtras/MorphExamplesAndDemos/main'
	user: ''
	password: ''
{% endhighlight %}

### BouncingAtomsMorph 演示

这里重复一下使用 AtomMorph 演示的教程步骤。对于初学 Smalltalk 的新手来说，
这个 AtomMorph 演示展示了 Smalltalk 动态特性及其开发环境的众多惊人能力。

![Pharo AtomMorph 演示]({{ site.baseurl }}/images/using-pharo-to-learn-smalltalk/Pharo_AtomsMorph_Example_annotated_sm.png)

- BouncingAtomsMorph new openInWorld
- meta-click>open explorer
- 选中若干子 morph>AtomMorph 并 inspect 它
- self color: Color red
- self velocity: 0@0
- 把红色的 AtomMorph 拖出来
- 把它变大
- 复制它
- inspect 它
- self velocity: 2@3
- 把它拖回 BouncingAtomsMorph
- 点击红色菜单手柄并嵌入它
- self browse the AtomMorph
- browse velocity:
- browse velocity: 的发送者 -> browse #bounceIn:
- 修改为弹跳后发出蜂鸣：

{% highlight smalltalk %}
bounced ifTrue: [self velocity: vx @ vy. Beeper beep ].
{% endhighlight %}

- 太吵了，所以增加对红色的测试

{% highlight smalltalk %}
bounced ifTrue: [self velocity: vx @ vy.
	self color = Color red ifTrue: [Beeper beep] ].
{% endhighlight %}

替代方案：

- 创建一个颜色不同且会蜂鸣的 AtomMorph 子类
- 定义 BeepingAtomMorph

{% highlight smalltalk %}
bounceIn: aRect
	| bounced |
	bounced := super bounceIn: aRect.
	bounced ifTrue: [ Beeper beep ].
	^ bounced

defaultColor
	^ Color red
{% endhighlight %}

- BeepingAtomMorph new openInWorld
- 实例化并嵌入它
- 查找 bounceIn: 的发送者
- 查看 BouncingAtomMorph>>step 中对 AtomMorph 类的测试
- 我们可以按如下方式修改 BouncingAtomMorph>>step 或 BeepingAtomMorph：

{% highlight smalltalk %}
isMemberOf: aClass
	^AtomMorph == aClass
{% endhighlight %}

显示所有 Morph 实例

{% highlight smalltalk %}
BouncingAtomsMorph allInstances.
{% endhighlight %}

移除所有 Morph
检查 BoundingAtomsMorph 对象并执行，

{% highlight smalltalk %}
self removeAllMorphs
{% endhighlight %}

## Metacello
Metacello 是 Monticello（Smalltalk 中使用的版本管理系统）的包管理系统。《Deep into Pharo》书中有一章专门介绍 Metacello，深入讲解了这一系统。另一方面，当我刚开始使用 Metacello 时，我需要的是更简单直接的内容，也就是本文所描述的这些。

在 Pharo 中，Metacello 以 *Configuration Browser* 的形式呈现，你可以用它轻松安装更多软件包。

![Pharo Metacello 浏览器]({{ site.baseurl }}/images/using-pharo-to-learn-smalltalk/Pharo_Metacello_Browser_sm.png)

### 安装软件包

|---
| 软件包  | 说明 |
|:--------:|-------------|
| Roassal2 | Roassal 使用简短而富有表现力的 Smalltalk 表达式对对象进行图形化渲染。它提供了大量交互手段以获得更好的用户体验。绘画、涂刷、互连、缩放、拖放，这些都会让你与任意对象模型更加亲近。文档见 [Roassal2 文档](http://agilevisualization.com)
| NeoJSON | JSON（JavaScript Object Notation）是一种流行的数据交换格式。针对这一简单格式已有多种实现。NeoJSON 是该格式一个更灵活、更高效的读写器。文档见 [NeoJSON 论文](https://github.com/svenvc/docs/blob/master/neo/neo-json-paper.md)
| NeoCSV | CSV（逗号分隔值）以及更一般的其他分隔符取值格式（如 TSV）可能是最常见的数据交换格式。针对这一简单格式已有多种实现。NeoCSV 是该格式一个更灵活、更高效的读写器。文档见 [NeoCSV 论文](https://github.com/svenvc/docs/blob/master/neo/neo-csv-paper.md)

## 在树莓派上运行 Pharo

对于树莓派爱好者来说，我们当然可以在树莓派上运行 Pharo。最简单的方式是下载预编译成果：
[预编译的树莓派 Pharo 镜像](https://ci.inria.fr/pharo-contribution/view/ARM/job/RaspberryPi-Cross-Compilation-FastBltBit/lastSuccessfulBuild/artifact/vmSources/results.tar.gz)

如果你是硬核 Pi 玩家，想自己动手，可以探索这个链接：
[手工编译 Pharo](https://pharoweekly.wordpress.com/2014/02/19/raspberrypi/)

拿到编译好的 Pharo 镜像后，复制到树莓派上，比如说放到名为 *Pi_Pharo* 的文件夹。
下图展示了通过 VNC viewer 在树莓派上运行 Pharo 的画面。

{% highlight shell %}
% cd Pi_Pharo
% ./PharoS Pharo.4.0.image
{% endhighlight %}

![运行在树莓派上的 Pharo 4]({{ site.baseurl }}/images/using-pharo-to-learn-smalltalk/Pharo_on_Raspi2_sm.png)

今天就写到这里，祝你学习 Smalltalk 愉快！
