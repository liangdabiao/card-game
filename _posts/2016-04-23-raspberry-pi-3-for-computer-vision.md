---
layout: post
title: 用树莓派 3 做计算机视觉
date: 2016-04-23 12:00:00.000000000 -05:00
permalink: raspberry-pi-3-for-computer-vision/
author: Benny Cheung
artwork:
  author: David Cheung
  url: https://cryotube.deviantart.com
tags:
  - AI
  - Computer Vision
  - RaspberryPi
  - OpenCV
  - SimpleCV
  - Python
category: post
comments: true
image: images/raspberry-pi-3-for-computer-vision/cover2x.jpg
images:
  cover: images/raspberry-pi-3-for-computer-vision/cover.jpg
  header: images/raspberry-pi-3-for-computer-vision/header.jpg
published: true
---

<!--excerpt.start-->
有了树莓派 3，开发计算机视觉项目不再困难，也不再昂贵。计算机视觉是一种图像处理与识别方法，特别适合应用到树莓派上。你可以为物联网设备装上计算机视觉组件：看家护院、监测冰箱里的啤酒、照看孩子。一旦完成初始搭建，可能性就是无穷的！

本文总结如何设置树莓派 3、如何安装实用的计算机视觉库 OpenCV 和 SimpleCV、如何安装机器学习框架 Orange。有了这套软件工具，再加上树莓派 3 自带的 Wifi、蓝牙和可选的 OpenGL，你的视觉项目就可以动工了。
<!--excerpt.end-->

![Raspberry Pi 3 Computer Vision]({{ site.baseurl }}/images/raspberry-pi-3-for-computer-vision/Pi3_Vision_Setup.jpg)

## 安装树莓派 3
你需要[下载 Raspbian Jessie](https://www.raspberrypi.org/downloads/) 磁盘镜像。

下载 Raspbian 磁盘镜像后，在 Mac 上可以使用 [Pi Filler](http://ivanx.com/raspberrypi/) 来制作 SD 卡镜像。

首次启动后，运行 `sudo raspi-config` 执行以下操作：

- 扩展根文件系统以使用整个 SD 卡（第一个选项）
- 启用 SSH（高级选项）
- 编辑 `/etc/hostname`，设置一个实用的主机名，例如 `my-pi`
- 编辑 `/etc/hosts`，把 127.0.0.1 指向该主机名，例如 `my-pi`
- 重启：`sudo shutdown -r now`

由于要写入 SD 卡，我们需要以 root 权限运行安装脚本。
在某些发行版中，可以在命令前加 `sudo`；而在另一些发行版中，则需要 su 到 root。
具体做法请查阅你的操作系统文档。

但对于标准 Debian 安装而言，运行：

```
sudo apt-get update
sudo apt-get upgrade
```

### 网络设置
如果你想配置 Wifi 和蓝牙，MakeUseOf 的这篇[《如何升级到树莓派 3》](http://www.makeuseof.com/tag/upgrade-raspberry-pi-3/)指南是非常宝贵的资源。

### VNC 服务器
如果你想设置对树莓派的远程桌面访问，下面是一份出色的指南：
[如何用 Mac 自带工具控制树莓派（VNC 连接）](http://quaintproject.wordpress.com/2013/03/03/how-to-control-your-raspberry-using-mac-on-board-tools/)

## 安装 OpenCV 和 SimpleCV
OpenCV 是一个 C++ 编程函数库，主要面向实时计算机视觉。SimpleCV 在其之上提供了一个" batteries included（自带全套电池）"式的封装，例如与 OCR 引擎 Tesseract 或著名的 `Orange` 机器学习框架的集成。

参考这篇[安装笔记](http://simplecv.readthedocs.org/en/latest/HOWTO-Install%20on%20RaspberryPi.html)，它描述了一种超简单、超快速的方法，让树莓派装上带 SimpleCV 模块的 OpenCV，避免了其他博客中描述的许多痛苦步骤。

只需运行以下命令即可安装 OpenCV 的必要依赖：

```
sudo apt-get install python-setuptools
sudo apt-get install python-pip
sudo apt-get install ipython python-opencv python-scipy python-numpy python-pygame
```

所有 OpenCV 依赖安装完成后，我们可以继续安装 SimpleCV——一个构建在 OpenCV 之上的封装 API，它让计算机视觉变得真正简单。
从 github 下载 SimpleCV 并从源码安装。

```
sudo pip install https://github.com/sightmachine/SimpleCV/zipball/master
sudo pip install svgwrite
```

等待这些命令运行一段时间（会花一些时间，去喝杯饮料吧），SimpleCV 就应该全部就绪了。把兼容的摄像头连接到板子的 USB 口，然后打开 `simplecv` shell。

```
raspberry@pi:~$ simplecv

SimpleCV:1> c = Camera()

SimpleCV:2> image = c.getImage()

SimpleCV:3> image.save('test.jpg')
SimpleCV:3: 1

SimpleCV:4> exit
```

查看结果图像 `test.jpg`，确认捕获正常。
至此你已确认树莓派正在运行 SimpleCV，并能与你的 USB 摄像头协同工作。

## 安装 Orange——机器学习工具
Orange <http://orange.biolab.si> 是一款基于组件的数据挖掘软件，包含一系列数据可视化、探索、预处理和建模技术。我们将在后续文章中进一步讨论 Orange。

*受限于树莓派的性能，我们在 Pi 上编译 Orange 框架时遇到了很大困难。目前你可能需要在 Linux PC（Ubuntu 12.04）上运行项目的机器学习组件。*

你可以从 [Orange Github 源码仓库](https://github.com/biolab/orange)克隆

```
git clone https://github.com/biolab/orange.git
```

要构建并安装 Orange，可以使用 orange 根目录下的 setup.py
（需要 GCC、Python 和 numpy 开发头文件）。如果你按照前面的步骤安装了
OpenCV 和 SimpleCV，那么你的树莓派上已经具备所需的全部软件包。

使用方法：解压每夜构建的源码并运行：
（警告：编译将耗时数小时！）

```
python setup.py build
sudo python setup.py install
```

这还会安装 orange-canvas 脚本，让你可以从命令行启动 Orange Canvas。

若要在本地安装 orange，运行：

```
python setup.py install --user
```

这会把 orange 安装到 /home/<username>/.local/lib/pythonX.Y/site-packages/orange/。

## 启发灵感的计算机视觉项目

### 水果分类器
一个启发人心的图像识别项目，用于对水果分类。

<http://jmgomez.me/a-fruit-image-classifier-with-python-and-simplecv/>

### 乐高分类器
它使用 SimpleCV 库和 SVM 机器学习来分类乐高积木零件。

<https://www.youtube.com/watch?v=yupShUY5WS4>

### 人脸相机追踪
使用 OpenCV 的 Haar 类人脸检测，我们可以连接一台两轴相机来追踪人脸运动。

<https://www.youtube.com/watch?v=GD8ArIM-5BQ>

### 计算机视觉与松鼠大军
这场出色的 PyCon 演讲向我们展示了如何用 Python 和 OpenCV 把自家后院"军事化"：用水枪吓走那些偷吃鸟食的松鼠。

<https://www.youtube.com/watch?v=QPgqfnKG_T4>
