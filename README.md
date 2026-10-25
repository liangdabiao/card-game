# 如何搭建 Jekyll 博客

## 编辑 Jekyll 配置
Jekyll 的 `_config.yml` 定义了站点的所有配置键；请将这些键的值改成你自己的相关信息

## 编辑链接页面
这组使用 `pages` 布局的页面，会以链接形式显示在侧边栏导航中。默认包括：
  1. About（关于）- about.md 介绍你自己和你的站点
  2. Research（研究）- research.md 列出研究条目

## 撰写博客页面
把你的博客页面放入 `_posts` 目录，文件名必须遵循 `YYYY-MM-DD-your-post-title.md` 的格式。在文章内，你至少需要以下元数据，布局应使用 `post`。

例如，

```
---
layout: post
title: YOLO for Real-Time Food Detection
date: 2018-06-07 01:00:00.000000000 -05:00
permalink: yolo-for-real-time-food-detection
author: Benny Cheung
artwork:
  author: Style of Kandinsky's TransverseLine transfer to food images
tags:
  - Deep Neural Network
  - Object Detection
  - YOLO
category: post
comments: true
image: images/yolo-for-real-time-food-detection/cover2x.jpg
images:
  cover: images/yolo-for-real-time-food-detection/cover.jpg
  header: images/yolo-for-real-time-food-detection/header.jpg
published: true
---
```

`image.cover` 元数据键用于文章的预览图（假定尺寸为 400x300）。`image.header` 元数据键用于文章的页头图（假定尺寸为 800x160）。你可以在 `_posts` 目录中找到示例。对于社交媒体站点，定义 `image` 后，社交媒体帖子中引用本站链接时会显示该图片。


### 文章摘要
标签 `<!--excerpt.start-->` 和 `<!--excerpt.end-->` 标记文章中摘要的开始和结束，摘要将显示在首页的文章摘要区块中。

例如，

```
<!--excerpt.start-->
The obsession of recognizing snacks and foods has been a fun theme ...
<!--excerpt.end-->
```

### 图片位置
按照本站的组织约定，文章中引用的所有图片都应放在 `images` 目录中，引用时以 `{{ site.baseurl }}/images/` 作为前缀。

## 本地测试
你可以在本地安装 Jekyll，并通过运行以下命令在本地提供页面服务（`--baseurl` 将配置的站点 URL 覆盖为空，使所有链接都能正确指向 localhost，从而可以在本地运行），

```
  jekyll server --baseurl ""
```

现在，你可以访问 `localhost:4000` 查看你的博客页面。

> alias jekyllserve='bundle exec jekyll serve --livereload'
