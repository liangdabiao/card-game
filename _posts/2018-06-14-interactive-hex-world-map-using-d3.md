---
layout: post
title: 用 D3 制作交互式六边形世界地图
date: 2018-06-14 00:00:00.000000000 -05:00
permalink: interactive-hex-world-map-using-d3/
author: Benny Cheung
artwork:
  author: Style of Kandinsky's TransverseLine transfer to World Hex Grid
tags:
  - D3
  - Hexagon
  - Tessellation
  - Mapping
category: post
comments: true
image: images/interactive-hex-world-map-using-d3/cover2x.jpg
images:
  cover: images/interactive-hex-world-map-using-d3/cover.jpg
  header: images/interactive-hex-world-map-using-d3/header.jpg
published: true
---

<!--excerpt.start-->
大自然是如何启发我们认识最优几何的？
蜜蜂以精密工程建造蜂巢：一排排棱柱形巢室，横截面是完美的六边形。
如果你在水面上吹出一层泡泡，它们会立即重排成三面墙式的接点，彼此之间大致呈 120 度的等角。你永远不会看到一堆方形泡泡。
对这条自然规律——自组织系统中自动优化出的、近似六边形的结构——有一个精彩的解释，详见 [Why Nature Prefers Hexagons (2016)](http://nautil.us/issue/35/boundaries/why-nature-prefers-hexagons)。
<!--excerpt.end-->

![六边形的自然]({{ site.baseurl }}/images/interactive-hex-world-map-using-d3/Hexagonal_Nature_700.jpg)

我至今记得阅读[《The Computational Beauty of Nature》](https://www.amazon.com/Computational-Beauty-Nature-Explorations-Adaptation-ebook/dp/B00MHAV10O)时的愉悦。
书中的主题教会了我重要的一课：开发可以分为职业生活的即时所需，以及丰盈人生的精神食粮。**空间镶嵌（Spatial Tessellation）**属于我计算探险的精神食粮一侧，
因为空间表示与推理是我持续进行的研究课题。
我感受着空间计算的纯粹乐趣，并试图让我的理解更贴近自然。

本文将沿着以下探究路径展开，

* 理解六边形坐标系
* 创建覆盖全球的六边形网格
* 把世界地形聚合到六边形网格
* 把六边形网格投影到地球仪上
* 用 D3 实现用于可视化的六边形网格与地球仪

你会在[本页看到一个可运行的交互式演示](#demo)；随后你可以自行检出[演示代码](#code)来玩。

## 用六边形地图瓦片覆盖地球
从镶嵌的实用角度讲，许多策略游戏都有充分理由选择六边形瓦片作为游戏棋盘。
一个主要优点是：任何瓦片中心与其所有相邻瓦片中心的距离都相等。
就策略而言，这种等距特性保证了策略移动的一致性（见下图右图，移动到所有相连格子都是 1 步（绿色箭头））。
与之相比，在矩形网格中，
对角线移动是不公平的：它只需 1 步，而等价的正交移动需要 2 步
（见下图左侧矩形网格，对角线移动（红色箭头）等价于 2 次正交移动（绿色箭头））。

![矩形与六边形镶嵌对比]({{ site.baseurl }}/images/interactive-hex-world-map-using-d3/Rectangular_vs_Hexagonal_Tessellation_600.jpg)

我曾好奇是否有人考虑过把六边形瓦片系统
与传统地理系统（经度/纬度）结合起来。我觉得用六边形瓦片覆盖地球仪，
并能把地理坐标映射到某个瓦片上，会是一件有趣的事。这属于[离散全球网格系统的前景](https://www.geoconnexion.com/uploads/publication_pdfs/int-v13i7-OGC-the-promise-of-discrete-global.pdf)这一议题，超出了本文的 speculate 范围。

### 六边形坐标系
要理解*六边形坐标系*及其数学，
我首先想到的优秀网站是 Amit 的游戏编程资料，以及他关于[六边形网格](http://www.redblobgames.com/grids/hexagons/)的交互式讲解合集。
本文将直接沿用 Amit 阐述的结果。

与我们从中学就熟悉的*笛卡尔坐标系*不同——后者只有 1 种网格布局——
*六边形坐标系*有 4 种网格布局！对于本文的世界地图，
我选择使用 "odd-r" 水平布局，即奇数行向右推移。

![odd-r 水平六边形网格布局]({{ site.baseurl }}/images/interactive-hex-world-map-using-d3/Hex_odd-r_grid_layout.png)

> 你可以自由选择其他六边形布局类型，只是数学稍有不同。后文会看到，`hexlib.js` 实现应该支持全部 4 种布局类型。

### 六边形分箱（Hexagon Binning）
D3 自带一个[六边形分箱包](https://github.com/d3/d3-hexbin)，可用于把数据聚合成更粗的表示再展示。它不是渲染数万个点的散点图，
而是把这些点分箱进几百个六边形来呈现数据。这里选用 `hexbin` 对世界地形高度图（那张较小的黑白图片）的平均高度做颜色编码。

![世界地形高度图映射到 HexBin]({{ site.baseurl }}/images/interactive-hex-world-map-using-d3/world_height_hex_overlay_700.jpg)

对落入同一个六边形分箱的所有地形像素，计算平均像素值并映射为颜色，
梯度从红色（低海拔）到蓝色（高海拔）。

## 六边形网格生成与投影
为了支持交互性，三个 Javascript 文件协同把屏幕坐标映射为对应的六边形网格坐标。同时，六边形点的位置会被发送到旋转的地球仪上展示。

```html
<script src="js/hexmap/hexlib.js"></script>
<script src="js/hexmap/hexlib_ui.js"></script>
<script src="js/hexmap/hexglobe.js"></script>
```

遗憾的是，把 2D 六边形圆柱投影映射到 3D 球面地图并不完美（*无论怎么做，投影都会失真*），如你所见，靠近南北极处六边形格子的面积发生了变形。也许有兴趣的读者可以改进这段代码？

完整的演示源码可以从配套的 [D3HexMap github 仓库](https://github.com/bennycheung/D3HexMap)检出。
演示之后的章节会有更多解释。

### 交互式可视化演示 {: #demo }
试着把光标移到六边形网格上，可以看到地球仪旋转到以高亮的六边形格子（红点）为中心。

<link rel="stylesheet" href="css/hexmap/hexlib.css">
<script src="js/d3/d3.v3.min.js"></script>
<script src="js/d3/topojson.v1.min.js"></script>
<script src="js/d3/d3.hexbin.js"></script>


<div id="globecontainer">
</div>

<div class="float-container">
  <div style="width:280px;display:inline-block;margin:5px">
    <svg id="grid-offset-odd-r" width="700" height="365"></svg>
  </div>
</div>

<div id="hexbin"></div>

<script src="js/hexmap/hexlib.js"></script>
<script src="js/hexmap/hexlib_ui.js"></script>
<script src="js/hexmap/hexglobe.js"></script>

<script>
  var grid_odd_r = makeHexOddR("#grid-offset-odd-r", 8, 700, 365);
  makeHexBinMap(grid_odd_r, "#hexbin", 8, "images/interactive-hex-world-map-using-d3/world_height_700.png", 700, 365);
</script>

<script src="js/hexmap/globe.js"></script>

<script src="js/jquery/jquery-1.7.min.js"></script>
<script src="js/jquery/jquery.event.drag-2.2.js"></script>


## 实现细节

### hexlib.js
六边形表示与数学函数实现在 `hexlib.js` 中，
它本质上就是 Amit 的 Hex Grid 实现，细节见 <https://www.redblobgames.com/grids/hexagons/implementation.html>

### hexlib_ui.js
六边形网格的交互性由 `hexlib_ui.js` 提供，
它是从 <http://www.redblobgames.com/grids/hexagons.html> 的六边形讲解页面中仔细抽取并清理出来的，
让代码具有最少的依赖，从而可以轻松地在其他页面复用。

`makeGridDiagram(svg, cubes)` 将：

1. 转换一组六边形 cube 对象，创建节点列表 = {cube: Cube 对象, key: 字符串, node: 包含 polygon 的 <g> 的 d3 selection}
2. 创建 diagram = 图根 <g> 的 d3 selection
3. 创建 polygons = 每个瓦片 <g> 内六边形的 d3 selection
4. 定义 update = function(scale, orientation)，在任何朝向变化（包括初始化）时调用
5. 设置 onLayout = 回调函数，在每次 update 之前调用（用于分配新的 cube 坐标）；update 时会立即调用
6. 设置 onUpdate = 回调函数，在每次 update 之后调用；延迟调用，且仅当没有新的 update 发生时

六边形网格图随后由 `hexglobe.js` 的 `makeHexOddR()` 函数调用 `addHexCoordinates()`；
这就是检测鼠标移动交互性的机制。
如果鼠标悬停在六边形网格上，
被选中的 (行,列) 格子会加上高亮 class，并调用地球仪的 `updateGlobe()` 函数传入所选格子的 (lon, lat)，用于地球仪旋转与视角定位。

### hexglobe.js
最后，在前面各例程的支持下，
`hexglobe.js` 添加了一组便捷函数，用于构建可视化所需的交互式六边形网格与地球仪。这里简要描述如下，

* `makeHexOddR(svg, hexsize, width, height)`
  * 创建一个 "odd-r" 六边形网格，给定六边形像素尺寸以及参考地形图的宽度、高度。
* `screenToLonLat(grid_odd_r, screenPoint)`
  * 把图像上的屏幕坐标变换为 (lon, lat) 点
* `hexToLonLatHex(grid_odd_r, hex)`
  * 把六边形网格 (q, r) 坐标变换为一张由 (lon, lat) 点列表构成的多边形
* `hexWorldGrid(grid_odd_r)`
  * 为六边形世界网格创建 topojson 线要素
* `makeHexBinMap(grid_odd_r, hexbin_id, hexsize, image_url, width, height)`
  * 定义 d3.hexbin
  * 定义颜色梯度映射
  * 读取地形图像并对每个像素值做 hexbin()
    * 对每个六边形，求像素值的平均
  * 对每个六边形，按其梯度范围把平均像素值映射为颜色
  * 用所有着色的六边形多边形构建世界六边形 "odd-r" 网格

地球仪由 `world-110m.json` topojson 文件构建，然后六边形网格被投影到地球仪表面。代码设置了一个定时器来检查地球仪投影是否已更新；一旦更新，地球仪就会旋转并重绘到以给定 (lon, lat) 为中心。

### 组装起来 {: #code }
主可视化页面 `index.html` 可以从[这里](https://github.com/bennycheung/D3HexMap/index.html)检出。

html 代码如下，

```html
<h1>Interactive Hex Global Visualization</h1>
<p>
Try to move your cursor over the hex grid to view the globl rotated to center (the red dot) on the highlighted hex.
</p>

<link rel="stylesheet" href="css/hexmap/hexlib.css">
<script src="js/d3/d3.v3.min.js"></script>
<script src="js/d3/topojson.v1.min.js"></script>
<script src="js/d3/d3.hexbin.js"></script>


<div id="globecontainer">
</div>

<div class="float-container">
  <div style="width:280px;display:inline-block;margin:5px">
    <svg id="grid-offset-odd-r" width="800" height="417"></svg>
  </div>
</div>

<div id="hexbin"></div>

<script src="js/hexmap/hexlib.js"></script>
<script src="js/hexmap/hexlib_ui.js"></script>
<script src="js/hexmap/hexglobe.js"></script>

<script>
  var grid_odd_r = makeHexOddR("#grid-offset-odd-r", 10, 800, 417);
  makeHexBinMap(grid_odd_r, "#hexbin", 10, "images/world_height.png", 800, 417);
</script>

<script src="js/hexmap/globe.js"></script>

<script src="js/jquery/jquery-1.7.min.js"></script>
<script src="js/jquery/jquery.event.drag-2.2.js"></script>
```

> 欢迎从中获得启发，并告诉我对这些代码的任何进一步修改与应用！
祝你享受这段六边形之旅。

## 其他有趣的六边形网格布局
另外，可以看看 [vraid/earthgen](https://github.com/vraid/earthgen-old)，
它使用六边形（外加少量五边形）生成模拟的地球表面，并附有源码（见 `planet/grid/create_grid.cpp`）。
