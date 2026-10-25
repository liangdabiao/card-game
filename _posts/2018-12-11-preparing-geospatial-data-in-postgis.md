---
layout: post
title: 在 PostGIS 中准备地理空间数据
date: 2018-12-11 00:00:00.000000000 -00:00
permalink: preparing-geospatial-data-in-postgis
author: Benny Cheung
artwork:
  author: Style of Kandinsky’s Transverse Line transfer to a global projected map
tags:
  - Docker
  - PostGIS
  - Geospatial
  - OpenStreetMap
category: post
comments: true
image: images/preparing-geospatial-data-in-postgis/cover2x.jpg
images:
  cover: images/preparing-geospatial-data-in-postgis/cover.jpg
  header: images/preparing-geospatial-data-in-postgis/header.jpg
published: true
---

<!--excerpt.start-->
**空间**（Spatial）是传统数据库系统的流行扩展。
当数据带有某些空间属性时，例如街道地址或电话号码，
我们可以利用空间邻近性来分析和计算它们在空间中的关系。
我们通常是在为这些空间实体指派有限的几何表示与运算。
如果你有兴趣使用公共领域的空间数据来解决你的分析问题，
请继续阅读这篇地理空间数据的入门介绍，了解如何在 PostGIS 中加载和使用它们。
<!--excerpt.end-->

## 世界作为统一的一体？
让我们从一张奇特的地图投影说起，
[Dymaxion Map](https://en.wikipedia.org/wiki/Dymaxion_map)（戴马克松地图）是把世界地图投影到二十面体表面的投影方式，
它可以展开并压平成二维。这幅平面地图为了保持形状与尺寸而进行了大量中断。

![Dymaxion 地图投影]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Dymaxion_map_projection.jpg)

*图：Dymaxion 地图投影可以折叠成一个柏拉图立体——二十面体（图片来源：[Buckminster Fuller Institute](https://www.bfi.org/about-fuller/big-ideas/dymaxion-world/dymaxion-map)）*

使用这种投影，
它展示了几乎连续的一片大陆——涵盖地球全部大洲，而不是被海洋分割的大洲群。这一版本把地球描绘成"一座岛"，声称比其他地图投影方式更有助于凝聚人类。用这种投影支持分析还有一个有趣的结果：根据线粒体群体遗传学的[早期人类迁徙](https://en.wikipedia.org/wiki/Early_human_migrations)可以得到解释（见[图片](https://en.wikipedia.org/wiki/File:Map-of-human-migrations.jpg)）。
从这一投影视角看，迁徙模式几乎是从空间排布自然推导出来的。
我们或许无法立刻用 Dymaxion 地图解决人类的许多问题。
但积极的一面是，空间视角确实帮助了我们去看。

## 大纲
本文中，你将首先接触地理空间数据的概念。
一如既往，概念都配有实践实验作为补充。
使用 Docker 快速启动一个 PostGIS 实例，你可以把 OpenStreetMap（OSM）数据载入数据库。
随后，用 QGIS 连接这个 PostGIS 实例，就可以交互式地分析和可视化空间数据。
读到最后，你将对如何为未来的地理空间分析准备数据形成坚实的认识。

------------------
## 1. 什么是地理空间？
**地理空间**（Geospatial）的定义由两个紧密相关的概念组成：
*Geo* 即地理信息，*Spatial* 即对象在空间中排布的表示。
当一组空间坐标参照地球上的具体地点时，
我们称这些空间数据是*地理配准*的（georeferenced）；因此组合出了 geospatial 这个术语。
最著名的地理空间应用是 GIS（地理信息系统），
它被公认为一套软硬件系统，用于地理空间数据的输入、
存储、检索、制图、展示与分析。

地理空间数据以图层形式存在，每个图层包含两类信息：
图形与属性。前者也叫*几何数据*，后者叫*主题数据*。
图层中的信息可以以两种形式表示：栅格与矢量。

![地图图层示例]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Map_Layers.jpg)

*图：典型的地图图层示例——海岸线、边界与河流的组合*

###  1.1. 表示方式
栅格与矢量表示是两种不同的地理空间构思，用于为真实世界建模。
矢量数据是图形，通常由三种几何类型表示：点、线和面（多边形）。
为符合地理空间的要求，这些图形对象按其坐标位于地理配准空间中。
而在栅格数据中，图层（或空间）被视为一个网格，
每个单元格（也称像素）代表一个基本的信息元素。
栅格表示假设空间预先存在，对象被放入其中。
矢量与栅格表示之间的这种二分法，给我们留下了关于空间的两种不同
思想流派：在矢量表示中，
*空间因对象而存在*，没有对象就没有空间；
在栅格表示中，*空间是放置对象的直观观念*。

矢量数据由点、线和多边形组成。
在矢量数据集中，每个点表示空间中特定 (X,Y)（以及可选 Z）位置上的一个值。
矢量数据最适合表示离散要素：
例如，用点表示兴趣点（point-of-interests），用线表示道路，用多边形表示城市边界。

相比之下，栅格数据由像素组成：小而均匀的网格单元。

![地球某区域的栅格表示]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Raster_Pixels.jpg)

*图：某区域的栅格表示，显示每个单元格（像素）的颜色（红、绿、蓝）值。颜色值意味着什么需要解释。*

在栅格数据集中，每个像素都有一个值。表示等价数据的像素具有相同的值：
栅格非常适合表示大范围内的连续数据：例如高程数据或温度测量值。
栅格像素也可以用来表示颜色值：卫星影像就是这类数据的例子。
想象放大某个区域，你可以看到每个小方块都有一个独特的值；
这些像素拼在一起构成一幅图像。

###  1.2. 坐标系
上述几何概念同样适用于地理空间：
地球上的任何一点都可以用其纬度、经度和（可选）高程来描述。
用于描述地球表面上点的系统称为地理坐标系（Geographic Coordinate Systems，GCS）。

![地理坐标系图示]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Geographic_Coordinate_System.jpg)

*图：图示地球并非完美的椭球体。借助 WGS 84，全世界就如何在地球表面定位地理坐标达成了一致。*

GCS 使用一个数学定义的曲面——椭球体——来表示地球的形状。
基于该椭球体的复杂计算定义了可用于引用唯一点的坐标。
坐标系有很多种，有些比另一些更常用：WGS 84 是你最常见到的一个。

###  1.3. 地图投影
地图投影让我们把位置从 3D 曲面（如地球仪）转换到平面（如地图）。
通俗地理解球面投影，就像剥一个橙子，
然后试图把橙皮在桌上压平：它永远不会完全平整，
必须执行拉伸、切割或挤压橙皮之类的操作。
由于这些压平操作，地图投影必然会在某些方面扭曲它所绘制的要素。
可以根据扭曲地球表面的方式选择不同的投影。

![地图投影概念]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Map_Projection.jpg)

*图：地图投影的概念——试图把椭球状的地球表面压平到一张纸上。压平操作至少要在这些指标之一上产生扭曲：(1) 面积，(2) 方向 和/或 (3) 距离。*

为适应平面而被变换过的地理空间数据称为投影数据。
用于这一变换的投影是你数据文件独有的地理空间信息（元数据）的一部分。
在 Web 制图中，典型做法是使用 [EPSG 3857 投影](https://epsg.io/3857)，
也称为 Spherical Mercator 或 Web Mercator。
EPSG 3857 是一个投影坐标系，用于在 OpenStreetMap、Google Maps 等中渲染地图。

![Web Mercator Tissot]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/Web_Mercator_Tissot.jpg)

*图：Web Mercator (EPSG 3857) 投影，其 Tissot 指标线展示了面积扭曲：在地球仪上所有圆盘面积相等。（图片来源：Wikimedia Commons，作者 Stefan Kühn）*

##  2. 运行 Docker 化的 PostGIS
PostGIS 是 PostgreSQL 的空间扩展。
两个自由开源项目的组合符合许多行业标准（万分感谢！）。
PostgreSQL 支持许多较新的 ANSI SQL 特性。PostGIS 支持 Open
Geospatial Consortium（OGC）标准和新的 SQL Multimedia Spec（SQL/MM）
空间标准。学会 PostGIS 意味着你的知识对
其他商业与开源的地理空间数据库和制图工具同样有用。

使用 Docker 的一大优势是能够快速为软件系统构建软件环境。
为了快速启动一个 PostGIS 数据库，我们将使用预构建的 [PostGIS Docker 镜像 `kartoza/postgis`](https://hub.docker.com/r/kartoza/postgis/)。

* PostgreSQL 10.3
* PostGIS 2.4

以下各节总结了本文所需运行的关键步骤。更多信息，
可参考 Alex Urquhart 的优秀教程 [Setup PostGIS with Docker](https://alexurquhart.com/post/set-up-postgis-with-docker/)。

###  2.1. 持久化 Postgres 数据
在创建数据库之前，我们需要考虑数据库信息将如何存储在 Docker 中。
通常，我们在 Docker 中指定所谓的数据卷（volume），让数据库数据保存在容器之外。
有了外部持久卷，我们就可以在不丢失全部数据的情况下执行备份或升级数据库软件。

我们创建一个卷容器，用于在运行数据库进程的容器之外持久化 PostgreSQL 数据库文件：

```bash
docker volume create pg_data
```

###  2.2. 创建数据库容器
接下来我们用 `docker run` 创建 PostGIS 容器：

```bash
docker run --name=postgis -d \
    -e POSTGRES_USER=admin \
    -e POSTGRES_PASS=admin \
    -e POSTGRES_DBNAME=gis \
    -e ALLOW_IP_RANGE=0.0.0.0/0 \
    -p 5432:5432 \
    -v pg_data:/var/lib/postgresql \
    --restart=always \
    kartoza/postgis:10.0-2.4
```

让我们逐段拆解这条命令。更多运行该容器的示例可在 [Docker Hub](https://hub.docker.com/r/kartoza/postgis/) 找到。

* `docker run --name=postgis` 告诉 Docker 我们的新容器将命名为 `postgis`
* `-d` 在后台运行容器（detached 模式）
* `-e POSTGRES_USER=admin` `-e` 标志在容器内设置环境变量。这个变量用于配置 PostgreSQL 中的登录角色名，该角色将在数据库中拥有超级用户（admin）权限。你可以随意改名。
* `-e POSTGRES_PASS=admin` 设置一个环境变量，把登录角色的密码设为 `admin`。你可以随意设置。
* `-e POSTGRES_DBNAME=gis` 顾名思义，这个环境变量告诉容器在服务器上创建一个名为 `gis` 的新数据库。数据库创建后，PostGIS 扩展会在其上启用。
* `-e ALLOW_IP_RANGE=0.0.0.0/0` 告诉容器把 PostgreSQL 配置为接受任何人的连接。如果不设置，数据库将只接受来自 Docker 网络子网地址的连接。
* `-p 5432:5432` 把宿主 VM 的 5432 端口映射到容器的 5432 端口。这是必需的，因为数据库服务器默认监听 5432 端口。
* `-v pg_data:/var/lib/postgresql` 告诉容器文件系统把刚创建的 `pg_data` 卷挂载到 `/var/lib/postgresql` 路径。这意味着容器在该目录中保存或创建的任何数据都将保存到 `pg_data` 卷中。
* `--restart=always` 为容器创建重启策略。现在每次 Docker 虚拟机启动时，容器都会启动。如果不设置，每次 VM 启动后你就得手动 `docker start postgis` 启动容器。
* `kartoza/postgis:10.0-2.4` 告诉 Docker 从 Docker Hub 拉取 `kartoza/postgis` 仓库，使用 PostgreSQL 10.0 和 PostGIS 2.4 版本。其他可用版本可在 Docker Hub 上查看。

镜像下载完成后，用 `docker ps` 应该能看到容器已经启动：

```
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                    NAMES
50ab14716a91        kartoza/postgis:10.0-2.4   "/bin/sh -c /docker-…"   4 seconds ago       Up 3 seconds        0.0.0.0:5432->5432/tcp   postgis
```

如果想查看 `postgis` 容器的日志输出，可以使用 `docker logs`。

```bash
docker logs postgis
```

供参考，请注意 PostGIS 扩展安装在容器内的以下位置，

```
/usr/share/postgresql/10/contrib/postgis-2.4
```

万一需要执行 PostGIS 命令为新数据库启用空间功能（你需要超级用户权限）

```
createlang plpgsql <dbname>

psql -d <dbname> -f postgis.sql
psql -d <dbname> -f spatial_ref_sys.sql
```

要使用 `psql`，你有两个选择：(1) 从 localhost 使用 `psql`；(2) 在 PostGIS 容器内使用 `psql`。

###  2.3. (1) 从 localhost 执行 psql
`postgis` 容器启动后，如果主机上有 `psql` 客户端，你可以直接连接 `postgis`，
因为我们已把默认数据库端口 5432 暴露到了宿主机的 5432 端口。

```
psql -h localhost -U admin gis
Password for user admin:

psql (10.2, server 10.3 (Debian 10.3-1.pgdg90+1))
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

gis=#
```

###  2.4. (2) 在 PostGIS 容器内执行 psql
要进入 PostGIS 容器的数据库，在终端运行以下命令。
务必把 PASSWORD、DBNAME 和 USERNAME 参数替换为创建容器时使用的值。

```bash
docker exec -it postgis /bin/bash \
    -c "PGPASSWORD=admin psql -d gis -U admin -h localhost"
```

这条命令的作用是创建一个 `postgis` 容器实例，然后在容器内执行 `psql`。

连接成功后，将显示 `psql` 提示符。

```
psql (10.3 (Debian 10.3-1.pgdg90+1))
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

gis=#
```

##  3. 把 OSM 数据载入 PostGIS
[OpenStreetMap (OSM)](https://www.openstreetmap.org/) 是一个协作项目，旨在创建免费可编辑的全球地理空间数据。
OSM 众包数据以 Open Database License 提供，数据可以被自由地分享、修改并在数据库中使用。得益于这一免费数据源，许多社会与人道主义制图项目成为可能。
我们将在后续章节使用 OSM 数据。

###  3.1. 安装 Osm2pgsql
我们需要安装 `osm2pgsql` 程序来把 OSM 数据载入 PostGIS。

####  3.1.1. 从源码
先决条件列表见 osm2pgsql 的 README。
对老发行版需要注意的主要依赖是 C++11 编译器和 Boost 1.50 或更高版本。

####  3.1.2. Debian 或 Ubuntu

```
apt-get install osm2pgsql
```

####  3.1.3. Mac OS X
使用 Homebrew

```
brew install osm2pgsql
```

###  3.2. 载入多伦多 OSM 数据
OSM 数据可以从 [Geofabrik 下载服务器](http://download.geofabrik.de/)免费下载。
不过，我们的实验只覆盖多伦多地区，所以走一条更省事（体积小得多）的路线：只从 [OSM extracts for Toronto](https://download.bbbike.org/osm/bbbike/Toronto/) 下载 `Toronto.osm.pbf` 文件。

然后使用 `osm2pgsql`，我们可以把多伦多 OSM 数据载入 PostGIS
（注意：密码通过 `PGPASSWORD` 指定）。

> 有一篇有趣的文章：http://weait.com/content/build-your-own-openstreetmap-server 它说，"你必须使用 slim 模式"。

```
PGPASSWORD=admin osm2pgsql --slim -c -d gis --host localhost -U admin Toronto.osm.pbf
```

它会把 OSM 数据转换为 PostgreSQL 语句进行载入。
值得注意的是，我们从日志中观察到 OSM 数据使用的是 `3857 (Spherical Mercator)` 投影。

```
osm2pgsql version 0.96.0 (64 bit id space)

Using built-in tag processing pipeline
Using projection SRS 3857 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads

... skipped

Osm2pgsql took 67s overall
```

OSM 数据载入后，我们可以使用
[pgAdmin](https://www.pgadmin.org/)——PostgreSQL 或 PostGIS 最流行的管理工具。
例如，我们在这里用 `pgAdmin4` 查看 `gis` 数据库 `public` schema 的 `planet_osm_polygon` 表。

![用 pgAdmin4 查看 PostGIS 表数据]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS_pgAdmin4_SQL_ViewData.jpg)

*图：用 pgAdmin4 从 planet_osm_polygon 表查询。我们可以看到 planet_osm_polygon 是多种类型多边形的大集合。我们注意到 `amenity` 列可用于选择特定类型的多边形，例如 "parking"。*


##  4. 用 QGIS 展示 PostGIS 数据
[QGIS](http://qgis.org/) 是一款免费开源的跨平台桌面地理信息系统（GIS）应用，支持查看、编辑和分析地理空间数据。我们将用 QGIS 展示载入 PostGIS 的多伦多 OSM 数据。

我们应该能在 QGIS 的浏览器面板中添加到 PostGIS 的连接。右键点击 PostGIS 图标并点击"Create New Connection"，然后输入我们在第 2 节 `docker run` 命令中使用的数据库连接参数。
如果使用 Docker VM，"Host" 参数将是 Docker VM 的 IP（在终端窗口运行 `docker-machine ip`）。

![PostGIS QGIS 连接]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-connection.jpg)

*图：用 QGIS 创建新的 PostGIS 连接。我们使用对 PostGIS 容器实例执行 `docker run` 时的数据库创建设置。`admin` 密码在这里为演示目的特意显示。*

在下方的截屏中，可以看到已载入的多伦多地区 OSM 数据，并添加了 `planet_osm_polygon` 和 `planet_osm_road` 图层。

![PostGIS QGIS 多边形与道路]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-polygon-roads.jpg)

*图：只需右键点击 PostGIS 表即可添加为展示图层。添加图层后，你可以通过调整各种图形显示属性来美化图层，例如颜色、线宽、填充图案等。*

对 `planet_osm_polygon` 表做了一些检查之后，
为了减少展示过多多边形类型造成的杂乱，
我们可以通过 DB Manager 查询过滤、选择多边形类型的子集。

首先，我们选择所有非空的 `boundary` 多边形：

```
SELECT osm_id, boundary, name, way_area, way
    FROM planet_osm_polygon
    WHERE boundary is not NULL;
```

打开 DB Manager -> Query Layer 图标，执行选择，把结果加载到 `neighborhoods` 图层。

![PostGIS QGIS 查询停车位多边形]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-DBManager-Add-Query-Layer-neighborhoods-annotate.jpg)

*图：使用 DB Manager->Query Layer 对话框，用 SQL 语句从 planet_osm_polygon 表选择多边形子集。查询之后，你可以把结果加载为展示图层。在这个图示中，我们选择了非空的 boundary 多边形。*

我们可以看到 `neighborhoods` 图层的显示。

![PostGIS QGIS 显示社区多边形]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-polygon-neighborhoods.jpg)

继续更多选择，我们用下面的语句选择所有 `amenity='parking'` 多边形：

```
SELECT osm_id, amenity, way_area, way
    FROM planet_osm_polygon
    WHERE amenity='parking';
```

再次打开 DB Manager -> Query Layer 图标，执行选择，把结果加载到 `parking` 图层。

![PostGIS QGIS 查询停车位多边形]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-DBManager-Add-Query-Layer-parking.jpg)

*图：使用 DB Manager->Query Layer 对话框，用 SQL 语句从 planet_osm_polygon 表选择多边形子集。查询之后，你可以把结果加载为展示图层。在这个图示中，我们选择了停车位多边形。*

然后，我们可以看到 `parking` 图层显示在 `neighborhoods` 图层之上。

![PostGIS QGIS 显示社区多边形]({{ site.baseurl }}/images/preparing-geospatial-data-in-postgis/PostGIS-qgis-polygon-parking.jpg)

*图：把 planet_osm_polygon 的选择微调为只展示 `neighborhoods` 和 `parking` 之后，我们得到了多伦多下载区域停车区的清晰可视化。*

有了地理空间数据，我们就可以提出更深入的空间分析问题，例如：在"Toronto"的 `neighborhoods` 中，找出距名为 "King Street" 的 `planet_osm_road` 100 米以内的所有 `parking`。但分析话题的展开要留到下一篇文章了。

尽情享受这套完全开放、免费的地理空间技术栈吧！
