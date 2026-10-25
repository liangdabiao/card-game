---
layout: post
title: 在 Windows 10 上挖以太坊
date: 2017-09-17 01:00:00.000000000 -05:00
permalink: ethereum-mining-on-windows-10/
author: Benny Cheung
artwork:
  author: Clara Cheung
  url: https://www.instagram.com/pk_clara/
tags:
  - Mining
  - Ethereum
  - Crypto Currency
  - GPU
category: post
comments: true
image: images/ethereum-mining-on-windows-10/cover2x.jpg
images:
  cover: images/ethereum-mining-on-windows-10/cover.jpg
  header: images/ethereum-mining-on-windows-10/header.jpg
published: true
---

<!--excerpt.start-->
专用 GPU 的价值早已超越游戏需求，事实证明它也能满足深度学习研究的专业需要。
事实证明，现代显卡非常擅长达成虚拟现实所需的帧率。最令我们惊讶的是，GPU 挖加密货币（如以太坊）再次变得有利可图，我们可以从现有的硬件配置中获利。本文将快速讲解挖矿的基础知识，并指导如何在配备 GPU 的 Windows 10 机器上搭建以太坊挖矿。
<!--excerpt.end-->

## 比特币挖矿简史
几年前，读完化名作者"中本聪"（Satoshi Nakamoto）关于[《Bitcoin: A Peer-to-Peer Electronic Cash System》](https://bitcoin.org/bitcoin.pdf)的论文——它为开放网络描述了一种分布式信任方案——其中概念与解法带来的计算之美激发了我的兴趣，让我想尝试比特币挖矿。2013 年比特币价值约 120 美元，相比其发明后短短几年内从零价值起步的增长，令人印象深刻。

为了降低这次冒险的风险，和当时所有人一样，我们从用游戏 GPU 达到有利可图的算力（下文解释）开始。
盈利能力用"产出的币价值相对支付的电费"来衡量。如果币值高于电费成本，你就是在一个有利可图的挖矿生意中（不含初始设备成本）。
比特币最初的哈希算法（SHA256）执行复杂但固定的操作序列，可以被 ASIC（专用集成电路）硬件取代。ASIC 的计算速度和能效很快就扼杀了 GPU 挖矿的盈利能力。为了留在比特币挖矿的游戏里，Adafruit 给出了组装[树莓派比特币矿机](https://learn.adafruit.com/piminer-raspberry-pi-bitcoin-miner/initial-setup-and-assembly)的教程：它以极低功耗达到 2 GH/s 的哈希率（见下图），可以让挖矿多存活一段时间。

![以 2 GH/s 运行的树莓派矿机]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Bitcoin_RaspberryPi_Mining_Rig.jpg)

*图：以 2 GH/s 运行的树莓派矿机，树莓派接上了 Adafruit 16x2 LCD + Keypad 套件以显示挖矿统计。那一排 ASICMiner Block Erupter USB 棒，每根具备 300 MH/s 算力。每个 Block Erupter 功耗 5W，让这套矿机在 2013 年非常有利可图。但到 2017 年它已毫无用处。*

就比特币挖矿的历史而言，ASIC 专用硬件的崛起，伴随着[比特币网络难度](https://bitcoinwisdom.com/bitcoin/difficulty)的指数级增长，基本上摧毁了所有基于 GPU 的挖矿盈利能力。

## 以太坊挖矿
直到最近，[以太坊](https://www.ethereum.org/)——被认为是 [Bitcoin 2.0](https://bitconnect.co/bitcoin-news/89/is-ethereum-bitcoin-20)——专门把它的 [Ethash 设计为内存困难算法](https://www.vijaypradeep.com/blog/2017-04-28-ethereums-memory-hardness-explained/)，让基于 ASIC 的方案不切实际；随后，Ethash 让基于 GPU 的挖矿重新有利可图。此外，进一步激励矿工的是，以太坊交易价格经历了显著增长。以太坊不仅展示了它的盈利能力，也展示了它作为未来流行加密货币的交易流动性。

作为一名以太坊矿工和研究者，出于挖矿目的，有几个基本概念必须澄清。如果你不关心计算之美，可以直接跳到"挖矿之路"一节。

### 用区块链实现分布式信任
关键的加密货币技术是区块链。区块链把所有交易事实"锁定"进空间与时间之中，使得在开放分布式网络上篡改几乎不可能。"空间"是内容，"时间"是内容的序列。设想一个**区块**（Block）的内容是资金转移的事实，该区块的哈希值将提供一个唯一的签名（或摘要），用以判断区块内容是否可信。如果区块内容以任何方式被篡改，哈希值就会不同，于是所有人都知道该内容不可信。由于区块链已分布在整个开放网络中，其他未被篡改的区块链副本可以继续维护真相。被篡改的区块链只会被无视。

![通过展示哈希值彼此链接来解释区块链安全]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Explain_Blockchain_Security.jpg)

*图：这幅图解释了区块链的安全性：当前区块含有前一个区块的哈希值。如果前一个区块的内容被修改，哈希值就会不同。*

此外，不可能通过逆向工程从内容伪造出相同的哈希值，仿佛把内容修改到足以伪造哈希值。这正是[密码学哈希函数](https://en.wikipedia.org/wiki/Cryptographic_hash_function)的精妙之处：哪怕只改动一个字符，哈希值都会剧烈变化，让你根本无法控制变化的方向。

### 挖矿作为价值的工作量证明
对于加密货币（如比特币或以太坊）的区块链而言，区块的内容将是币的转移。设想下面的区块记录着币的交易，我们总可以追溯一枚币的来源。例如，$$A$$ 发出的币可以通过区块链的连接按时间回溯，其中 $$A$$ 币的资金总量来自 $$B \rightarrow A$$ 和 $$C \rightarrow A$$ 两笔交易。然而，我们注意到 $$C$$ 的原始资金似乎凭空而来。

![把加密货币的币生成解释为挖矿的价值]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Explain_Mining_Coin_Generation.jpg)

*图：这幅图说明所有收发的币都可以沿区块链追溯。唯独 $$C$$ 资金的来源似乎是凭空产生的——那正是挖矿的奖励。*

$$C$$ 的原始资金就是挖矿奖励。矿工以电力消耗的形式提供算力，为一个区块产出哈希值（签名）。寻找正确哈希值的努力也被称为[工作量证明](https://en.bitcoin.it/wiki/Proof_of_work)（Proof of Work）。最先成功找到正确哈希值的矿工将获得一定数量的币作为奖励。这正是"挖矿"的概念：用解决哈希值难题来换取币，同时让区块链在空间与时间中前进。

许多流行观点质疑比特币或以太坊的价值。以电力消耗形式体现的算力，就是币的价值所在。目前，产出 1 枚比特币大约耗费 1000 美元的电力，比特币的内在价值就储存于此。而当你收到一张印好的 100 美元钞票时，除了"一张纸做的承诺"，它没有任何内在价值。100 美元钞票没有任何储备来支撑其价值（美元早已不再储存等值黄金）。但比特币或以太坊的价值，是由其产出的难度储存起来的。

### 用难度保证时效性
剩下的问题是网络难度的意义。设想一个 1 到 10 的猜数游戏，猜对的难度是 1/10 的概率。我们说等价地：1 次猜测等于 1 次哈希，难度为 10。平均而言，玩家需要猜 5 次才能命中正确数字。然而，如果这个猜数游戏在短时间内发生了太多次猜测，正确的猜测可能出现过快。猜数游戏的设计者（整个网络）会自动调整难度——从 10 调到 100——来减缓正确猜测出现过快的速度。这就是用网络难度来控制正确猜测出现速度的本质。

![解释哈希难度]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Explain_Hash_Difficulty.jpg)

*图：这幅图解释了必须操纵哈希函数 $$f()$$ 的输入，才能生成满足指定难度的哈希结果。难度是一种易于比较的模式，在本例中，哈希结果要小于 100。*

在任何情况下，矿工都**不**应该修改交易内容来操纵哈希结果；矿工只能修改 "nonce" 字段来生成哈希结果，并检查哈希结果是否满足目标要求。每改动一次 "nonce"，重新计算哈希结果并与目标要求比较，就恰好是**一次**哈希。以太坊哈希速度目前以 MH/s（每秒 1,000,000 次哈希）为单位、用 GPU 来衡量。例如，Nvidia GTX 1070 不超频可以达到 25 MH/s（超频后提速 20%，达到 30 MH/s）。

不同的加密货币有不同的哈希算法和要求，但哈希的本质相同。如果一名矿工第一个算出满足全网要求的哈希结果，那么这名获胜的矿工将获得币的奖励！这正是挖矿的目标。显然，只用很小的算力去赢过整个网络是非常困难的；这就是为什么可以应用[矿池](https://en.wikipedia.org/wiki/Mining_pool)（Mining Pool）的概念来合力取胜。

## 挖矿之路
挖矿的前提是拥有一台装有现代 GPU 的机器。对于典型的游戏玩家或深度学习研究者来说，你很可能已经有一台可以部署以太坊挖矿的 Windows 10 机器。

![显示 Claymore 矿机算力与 MSI Afterburner GPU 设置]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/DX4_Ehtererum_1070_Overclock.jpg)

*图：完成以下全部设置后，Claymore 以太坊矿机的运行画面。GPU 可以为你的钱包卖力工作。*

以下是 10 个步骤的快速过览。

**(1) 在 Windows 10 上设置好 GPU**

**(2) 设置以太坊钱包**：[Jaxx](https://jaxx.io) 是一款可在任何平台运行的多用途加密货币钱包。矿工需要拿到以太坊钱包的公开地址；该公开地址用于存放挖矿奖励。

![Jaxx 以太坊钱包的公开地址]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Jaxx_Ethereum_Wallet_public_address.jpg)

**(3) 以太坊挖矿软件**：下载并安装 [Claymore-Ethereum-Miner](https://github.com/nanopool/Claymore-Dual-Miner/releases)，它与 Nvidia GPU 配合效果最佳。请遵循 Nanopool 帮助页面的 [Mining Clients - Claymore - Windows](https://eth.nanopool.org/help) 说明，接下来几步会重复其中的重要信息。

本文使用的是 Windows 10 版的 `Claymore's Dual Ethereum AMD GPU Miner v9.8 (Windows/Linux)`。

**(4) 选择要加入的矿池**：Claymore 以太坊矿机默认使用 [Nanopool](https://nanopool.org/)，它有出色的 API 和 Web UI。矿工无需在 Nanopool 注册。一旦某个以太坊公开地址通过 Nanopool 收到奖励，账户就会以该公开地址自动建立。

**(5) 安装超频工具**：[MSI Afterburner](https://www.msi.com/page/afterburner) 用作超频设置与监控。
下面展示 Nvidia GTX 1070 GPU 的一组优化超频设置：产出 30 MH/s、功耗 100 瓦、温度 61°C、风扇转速 40%。

![Nvidia GTX 1070 超频设置]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/MSI_Afterburner_overclock_settings.jpg)

由于 `Ethash`（以太坊哈希函数）本质上受 GPU 显存限制，只需超频显存即可，无需提高 GPU 核心频率。这可以降低 GPU 的功耗与温度，同时保持出色的算力！

**(6) 设置并运行矿机脚本**：Claymore 矿机自带许多批处理脚本用于挖矿。为简单起见，从纯以太坊挖矿脚本开始，按照说明修改为你自己的以太坊钱包公开地址。

以下是 `start_only_eth.bat` 文件修改为矿工信息后的样子：

```dos
setx GPU_FORCE_64BIT_PTR 0
setx GPU_MAX_HEAP_SIZE 100
setx GPU_USE_SYNC_OBJECTS 1
setx GPU_MAX_ALLOC_PERCENT 100
setx GPU_SINGLE_ALLOC_PERCENT 100

EthDcrMiner64.exe -epool eth-us-east1.nanopool.org:9999 -ewal 0xf1cfc50567acf36c55ba75540b7f1c3f1aaa9869/mining_name/my_email@gmail.com -epsw x -mode 1 -ftime 10
```

它使用美东海岸地址 `eth-us-east1.nanopool.org:9999` 的 Nanopool 作为矿池。一般经验法则是：选择离你地理位置最近的矿池地址。**重要**：别忘了把 `0xf1cfc50567acf36c55ba75540b7f1c3f1aaa986` 改成你自己的以太坊钱包公开地址；否则，你就是在给我的钱包捐款 `:)`。
下一个 `/` 之后，给矿机起个名字。再下一个 `/` 是可选的，你可以提供邮箱用于接收通知，并获得调整 Nanopool 支付计划的权限，也就是说，提供一个邮箱是件好事。

**(7) 监控挖矿运行**：你可以观察以 MH/s（每秒 1,000,000 次哈希）计量的算力。下面的屏幕输出展示了典型的 Claymore 矿机日志报告。

![Claymore 矿机输出窗口读数]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Claymore_Miner_readings.jpg)

从图示输出可以看到，以太坊挖矿日志带有 `ETH:` 前缀。我们可以看到矿机正在从我们在矿机脚本中设置的 `eth-us-east1.nanopool.org:9999` 拉取任务，这确认了我们已加入 Nanopool 挖矿。图中可见的 `GPU0`——这台 PC 上唯一的 GTX 1070——正以 30 MH/s 的速度哈希。当矿池走运并获胜时，会报告 `SHARE FOUND`。共享的计算贡献将相对整个矿池对获胜区块的贡献而获得奖励！

**(8) 保持凉爽并交电费**：我们需要懂得如何用以太坊计算器估算收益，保持挖矿有利可图。这里有一个计算器：输入算力、功耗和每千瓦时电费即可估算利润 [https://www.cryptocompare.com/mining/calculator/eth](https://www.cryptocompare.com/mining/calculator/eth)

例如，GTX 1070 算力 30 MH/s，功耗 110 瓦，平均电价 0.12 美元/千瓦时。

![以太坊计算器示例]({{ site.baseurl }}/images/ethereum-mining-on-windows-10/Ethereum_Calculator_Sample_20170917.jpg)

*图：`2017 年 9 月 17 日`的以太坊利润计算截屏，当时 1 ETH 交易价 263.89 美元*

估算的美元收益受当前以太坊交易价格影响。用已挖到的 ETH 数量来衡量更容易。

**(9) 重复 (6)-(8)**：访问 Nanopool 并搜索以太坊公开地址。它会显示挖矿统计数据。

**(10) 扩大挖矿规模**：如果你的挖矿胃口变大，那么是时候学习如何组建[以太坊矿架（Mining Rig）](https://www.youtube.com/watch?v=mYgd33zaNc8)了。这期视频介绍如何搭建一套 6 卡 GTX 1070 GPU、算力 180 MH/s 的矿架，值得一看。关于如何组装这套专用硬件，值得再写一篇文章。

{% include youtubePlayer.html id="mYgd33zaNc8" %}


祝以太坊挖矿愉快！

### 捐赠

ETH: 0xf1cfc50567acf36c55ba75540b7f1c3f1aaa986

BTC: 1ogLdBTrLBLxPb27JAiMVUieXxxQ428Jg

SIA: 720bfb6473361e94376634ebbbc64ebc7932f4f4b3e5f2f6556e1140216fae440990d9dc9105
