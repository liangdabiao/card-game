---
layout: post
title: 用树莓派监视零食
date: 2016-04-28 12:00:00.000000000 -05:00
permalink: snack-watching-with-raspberry-pi
author: Benny Cheung
tags:
  - SnackWatcher
  - RaspberryPi
  - Python
category: post
comments: true
image: images/snack-watching-with-raspberry-pi/cover2x.jpg
images:
  cover: images/snack-watching-with-raspberry-pi/cover.jpg
  header: images/snack-watching-with-raspberry-pi/header.jpg
published: true
series: "Food Detection"
series_order: 1
---

<!--excerpt.start-->
项目起初是 [Jonah Group](http://www.jonahgroup.com) 的一个趣味项目，
**Snack Watcher** 的设计目标是看管公司的"零食桌"。一旦"零食桌"上出现了新的
"零食"，它就会向聊天频道、电子邮件或消息推送报告事件："零食来了！"，同时张贴一张
图片，并尝试对观察到的零食进行分类。它既支持用于交互式零食查看的网站，也支持用于
程序化零食查询的 RESTful API。
<!--excerpt.end-->

[Snack Watcher Github 仓库](https://github.com/jonahgroup/SnackWatcher)

* 摄像头连接后对准"零食桌"

![Snack Watcher Setup 2]({{ site.baseurl }}/images/snack-watching-with-raspberry-pi/Snack_Watcher_Setup_02.jpg)

* `snack-web` 捕获的图像样本，带有斑点状态（绿色表示新增，红色表示移除）和斑点分类结果（从分类器训练结果来看，它们看起来像"包装食品"）

![Snack Web Image Capture]({{ site.baseurl }}/images/snack-watching-with-raspberry-pi/Snack_Watcher_Capture.png)

## snack-web
`snack-web` 是一个展示零食监视结果的 Web 应用，其设计目标是能在树莓派 2 或 3 上
配置并运行。`snack-web` 既可以手动驱动（通过网页），也可以程序化驱动（通过 RESTful API）
来拍照，并把快照推送到 `static/images` 目录。此外，RESTful API 也可被程序化地用于
监视并返回图像。这个 API 是与外部系统集成的关键特性，提供了报告零食状态的各种实用工具。

### snack-web 首页
下图展示了 `snack-web` 的首页，首页菜单项如下：

- **Links（链接）**：显示最近 N 张零食捕获图像及其处理阶段
- **Calibrate（校准）**：拍摄一张背景图像，用于校准背景颜色
- **Snap（抓拍）**：立即从摄像头抓拍一张零食图像
- **Teach（教学）**：（需要高级配置）目前仍在密集开发中，教学模块设计用于交互式地分类零食，以便后续训练。它需要分类器配置才能工作。

![Snack Web Front Page]({{ site.baseurl }}/images/snack-watching-with-raspberry-pi/snack_web_link_page_sm.png)

对于每一次零食图像捕获，系统都会收集一组处理过的图像用于调试。用户可以借此了解零食是如何被识别出来的。系统检测到的每个斑点都会被存储起来，用于展示和训练。颜色编码的斑点含义为：*绿色*表示新增斑点，*黄色*表示静止斑点，*红色*表示被移除的斑点。点击每张图像栏，即可显示更大的图像以便细致查看。

![Snack Web Item Display]({{ site.baseurl }}/images/snack-watching-with-raspberry-pi/snack_web_item_display_sm.png)

## RESTful API
<http://snack-web:8000/api>
（把 `snack-web` 替换为你的主机地址。）

图像和操作也可以通过 RESTful API 访问。可用的 URI 资源见下表。

*表：snack-web RESTful API*

| API | HTTP | 说明 |
|-----|------|-------------|
| /snacks/ | GET | 返回所有图像，数量可能非常庞大
| /snacks/snap | GET | 拍摄快照并返回最新图像。该调用会拍摄一张快照并返回处理后的图像。
| /snacks/id/{id} | GET | 返回图像 {id}。该调用按数据库 id 获取图像；若未找到，则返回 null。
| /snacks/state/{class_state} | GET | 获取所有匹配给定 class_state 的斑点。该调用返回按 c1ass_state 过滤后的斑点列表。
| /snacks/state | PUT | 按 _id 更新斑点状态信息。该调用接受一组 (id, c1ass, c1ass_state) 对象，并更新数据库中相应的斑点。
| /snacks/class/names | GET | 获取类别名列表。该调用返回斑点可被分类的所有类别名。
| /snacks/last | GET | 获取最后一张图像。该调用按 date_created 降序返回最新图像；若不存在，则返回 null。
| /snacks/last/{int:n} | GET | 获取最后 n 张图像。该调用按 date_created 降序返回最新的 n 张图像列表。
| /snacks/last/summary | GET | 获取最新摘要。该调用返回最新处理图像的摘要，包括新增、重复和被移除的斑点；若不存在任何图像，则返回 null。

### 快照命名约定

拍摄相机快照时，图像会按照快照的日期时间写入一个以创建时间命名的文件夹：

```
snack-{year}_{month}_{day}-{hour}_{minute}_{second}
```

例如 `snack-2015_06_17-13_14_58` 表示创建于日期 `2015-06-17`、时间 `13:14:58`。


图像返回的 JSON 结果示例：用 `curl` 命令请求最后一张图像。

```
curl http://snack-web:8000/api/snacks/last
```

图像列表示例：用 `curl` 命令请求所有零食图像。

```
curl http://snack-web:8000/api/snacks/
```
