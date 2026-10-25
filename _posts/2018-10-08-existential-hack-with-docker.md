---
layout: post
title: 用 Docker 玩微服务的存在主义危机
date: 2018-10-08 00:00:00.000000000 -00:00
permalink: existential-crisis-with-microservices-using-docker/
author: Benny Cheung
artwork:
  author: Style of blocks transfer to Klee's Gradation Red
tags:
  - Microservices
  - Docker
  - Python
  - Flask
category: post
comments: true
image: images/existential-crisis-with-microservices-using-docker/cover2x.jpg
images:
  cover: images/existential-crisis-with-microservices-using-docker/cover.jpg
  header: images/existential-crisis-with-microservices-using-docker/header.jpg
published: true
---

<!--excerpt.start-->
作为软件架构师，我们毫不怀疑微服务架构和 Docker 部署都有助于为系统设计带来
灵活性与可扩展性。然而，由于分布式服务与虚拟机的存在，系统身份的存在变得模糊，
由此引出一个问题：这种新形式设计意图的身份究竟由什么构成。
本文指出设计意图之一是面向数据服务的快速开发。
在理解 Docker 部署的基础之后，本文提供了一个 [Flask/PostgreSQL/Docker 起步框架](https://github.com/bennycheung/Flask-DB-Docker)，
可用于快速构建任何带数据库后端的面向数据服务。
放在更大的背景图景中，本文为读者未来构建更宏大的系统架构组合做好准备。
<!--excerpt.end-->

![个人身份]({{ site.baseurl }}/images/existential-crisis-with-microservices-using-docker/Personal_Identity.jpg)

*图：个人身份是 (1) 一具身体，(2) 一个大脑，还是 (3) 一个无形的灵魂？对比软件系统的 (1) 机器，(2) 软件，(3) 意图，来引出系统的存在之问。*

在本文中，你将学到：

* 从[关于系统存在的哲学沉思](#philosophy)开始
* 接着[理解 Docker](#docker)
* 然后，构建[用 Docker 快速搭建微服务](#rapid)
    * 学习如何用 [Docker 编排](#orchestration)构建微服务栈
    * 学习如何用 Docker 构建[持久卷——纯数据容器模式](#persistence)
    * 学习如何创建 [Postgres DB 镜像](#db)
    * 学习如何创建 [API starter-api 镜像](#api)
    * 学习如何使用 API 镜像把[样本（医疗程序代码）导入数据库](#data)
* 最后，以[下一步](#next)作结

------------------
## 关于系统存在的哲学沉思
"这个系统是我的孩子"，是软件系统创造者常用来指称其发明的类比。
这种情感依恋在于：创造者的设计与创造精神都显现在系统之中。
设想软件是系统的大脑，身体则是运行软件的物理机器。
系统的灵魂——创造者的欲望与意图编码于其中——通过功能风格展现出来。
第一次运行 Docker 容器时，它那虚拟而短暂的存在，
奇怪地唤起了关于"存在"的哲学趣味。

让我们运行 Docker <https://www.docker.com/> 命令，

```
$ docker run ubuntu echo “hello world”
```

输出显示在终端上，

```
hello world
```

负责说 "hello world" 的那台机器经历了，

* Docker 从 Docker registry 拉取最新的 `ubuntu` 操作系统镜像
* Docker 启动一个 `ubuntu` 镜像的实例作为运行中的容器——虚拟机
* Docker 执行 bash shell 命令 `echo "hello world"`
* Docker 把终端输出 "hello world" 显示到宿主机
* Docker 退出虚拟机并终止

最令人思绪万千的是：这台虚拟机只为执行命令而存在了短短一瞬，
随后就消失回虚无之中。机器的身体与软件的大脑都是暂时的，
只有 `echo` 程序的灵魂留存下来，向我们展示它说过 "hello world" 的痕迹。
于是有了存在之问——你的软件系统真的存在于这个分布式、虚拟化的世界吗？
还是只有软件系统的灵魂存在？

让我们先弄清 Docker 能提供什么，然后我们就有能力回答关于这种快速开发新设计形式的问题了。

----------------------
## 理解 Docker
本节我们定义基本的 Docker 术语，并学习 Docker 镜像究竟如何工作。

* *Docker 镜像* -
Docker 镜像就像一张黄金模板。一个镜像由操作系统（Ubuntu、centos 等）
及其上安装的应用组成。这些镜像被称为基础镜像（base image）。Docker 基础镜像是 Docker 容器的构建基石，容器由它创建。
镜像可以使用 Docker 内置工具从零构建。你也可以把其他用户创建的 Docker 镜像（来自
Docker 公共仓库 Docker hub）作为你容器的基础镜像。

* *Docker registry* - 
Docker registry 是存放 Docker 镜像的仓库。它可以是公共的或私有的。由 Docker 维护的公共
registry 称为 Docker hub。用户可以从 Docker registry 上传和
下载镜像。公共 registry 收藏了大量官方和用户创建的镜像。要创建容器，你既可以使用其他用户创建的公共镜像，也可以上传自己的镜像到公共或私有 registry 再使用。

* *Docker 容器* -
容器更像是一个目录加上应用程序的执行环境。它
创建于 Docker 镜像之上，完全隔离。每个容器拥有自己的
用户空间、网络和安全设置。容器持有运行应用程序所需的全部
文件与配置。容器可以被创建、运行、启动、迁移和删除。

![Docker 术语]({{ site.baseurl }}/images/existential-crisis-with-microservices-using-docker/Docker_Terminologies.jpg)

*图：全部 Docker 术语的图示，以及它们在 Docker 环境中如何协同工作。*

每个 Docker 镜像是多个层的组合。这种分层方式为创建 Docker 镜像提供了
绝佳的抽象。这些层使用统一文件系统（AUFS）合并为
单一单元。AUFS 把每一层存储为一个普通目录，文件带有 AUFS 元数据。这确保了所有文件和目录对特定层是唯一的。AUFS 通过合并镜像的所有层来创建一个挂载点。对镜像的任何更改都会写入最顶层。

![Docker 镜像层示例]({{ site.baseurl }}/images/existential-crisis-with-microservices-using-docker/Docker_Image_Layers.jpg)

*图：Docker 镜像各层的图示，顶层依赖底层逐层构建。*

这就是 Docker 容器非常轻量的原因。
例如，当你更新一个现有应用时，Docker 要么在现有镜像上创建一个新层，要么更新现有层。新创建的层会引用它的前一层。Docker 不会像虚拟机那样为应用重建整个镜像。当你把更新后的新镜像推送到 registry 时，它不会重新分发整个镜像，而是只更新现有基础镜像之上的层。Docker registry 用于存储 Docker 镜像。镜像可以从 registry 推送和拉取，用于启动容器。Docker registry 有两类。

* *公共 registry* -（Docker hub <https://hub.docker.com/>）包含官方 Docker 镜像和用户创建的
镜像。
* *私有 registry* - 用户创建的、用于在数据中心存储私有 Docker 镜像的 registry。你也可以在 Docker hub 中创建私有 registry。

registry 中的所有镜像都可以用 Docker 客户端搜索。镜像可以
拉取到 Docker 主机上，用于创建容器和新镜像。

正如我们在前面的 "hello world" 示例中演示的，容器是应用程序的执行环境。它包含
全部操作系统文件、用户添加的文件和元数据。从根本上说，容器从镜像启动，
因此镜像上安装和配置的应用程序将在由它创建的容器上运行。由于镜像处于*只读*格式，
容器启动时，会在镜像之上为容器添加一个*读写*层，供应用程序运行并做出更改。
Docker 客户端从 Docker 二进制或 REST API 接收运行
容器的命令。

成功执行 docker run 命令后，对比之前的 "hello world" 示例，
这里更精确地描述 Docker 在后台会执行的动作。

1. Docker 检查命令中指定的 Docker 镜像是否已存在于
宿主机本地。如果本地存在，Docker 将使用该镜像创建容器。如果
不存在，它会（根据
Docker 主机配置）从公共或私有 registry 把镜像下载到主机。
2. 本地的或拉取的镜像将用于创建新容器。
镜像就绪后，Docker 会在镜像之上创建一个读写文件系统。
4. 然后 Docker 为容器创建必要的网络接口，
以便与宿主机交互。
5. Docker 从地址池中检查可用 IP 地址，并分配一个给
容器。
6. 然后它执行命令中指定的命令，例如 /bin/bash shell。
7. 最后它记录全部输入/输出与错误，供用户了解容器的
状态。

请记住，容器不是经典虚拟机，而"只是"共享同一个 Linux 内核命名空间的一组进程。

既然 Docker 为我们提供了具备完全隔离执行环境的快速容器（虚拟机），
我们就能用这种部署技术快速构建带数据库后端的面向数据微服务。

----------------------
## 用 Docker 快速搭建微服务

快速起步框架请从 <https://github.com/bennycheung/Flask-DB-Docker> 检出，
它演示了如何用 Docker 部署一个带数据库后端的面向数据微服务 API。

起步框架使用的技术组合为，
* 把 PostgreSQL <https://www.postgresql.org/> Docker 化，运行在 Docker 容器内
* 把用 Python/Flask <http://flask.pocoo.org/> 编写的 API 服务 Docker 化，使之能与 PostgreSQL 通信
* 用 docker-compose <https://docs.docker.com/compose/> 编排 Docker 化的容器，并在部署的服务与运行 PostgreSQL 的容器之间建立链接

有了这个起步框架，我们可以轻松搭建出贴近现实的微服务架构。
我推荐阅读 Martin Fowler 和 James Lewis 关于微服务架构的白皮书。
他们的白皮书对[微服务架构](http://martinfowler.com/articles/microservices.html)背后的动因有深入得多的阐述。


### Docker 编排
我们需要把 Docker 在主机上跑起来，参见 <https://www.docker.com/get-started> 了解你的主机安装方法。

下面是部署图，它使用 Docker 容器部署起步框架的全部服务。

![起步微服务部署]({{ site.baseurl }}/images/existential-crisis-with-microservices-using-docker/Starter_Microservices_Deployment.jpg)

*图：微服务栈的 Docker 化部署，由数据库后端协同编排。*

按照以下步骤构建全部所需的 docker 镜像并启动应用栈，
* 运行 `buildImages.sh` 构建全部所需的 docker 镜像
* 运行 `startStack.sh` 启动应用栈
* 如果要关闭应用栈，运行 `stopStack.sh` 会挂起应用栈
* 如果要移除应用栈，运行 `teardownStack.sh` 会删除全部应用容器
* 如果要移除全部 docker 镜像，运行 `cleanupImages.sh` 以不留痕迹

启动一个全新的应用栈，

```
./buildImages.sh
./startStack.sh
```

关闭并清空应用栈，

```
./stopStack.sh
./teardownStack.sh
./cleanupImages.sh
```

要达到任何中间应用状态或保留已构建的 docker 镜像，
请相应地调整运行的脚本。

对于想了解起步框架如何构建的好奇读者，
以下各节提供详细说明：如何构建 Docker 镜像，
以及如何创建起步微服务及其依赖。

----------------
### 用 Docker 做持久卷——纯数据容器模式
数据容器的主要目的是保留 `/var/lib/postgresql/data` 持久卷。
由于同一主机上可能运行多个数据容器，让 docker 识别镜像唯一性的方法
是在镜像中放入一个唯一的 `README.md` 文件。

-- Dockerfile --

```dockerfile
FROM busybox
VOLUME /var/lib/postgresql/data

RUN mkdir /app
WORKDIR /app

ADD README.md /app/

CMD /bin/sh
```

创建卷镜像：

```
cd busybox
docker build -t starter/postgres_datastore .
```

启动卷镜像容器：

```
docker run -i -t --name starter-postgres_data starter/postgres_datastore
```

由于 `starter-postgres_data` 容器很可能永远不需要更新，即便需要，我们也可以轻松地按需迁移数据，所以我们从本质上规避了丢失容器数据的问题，同时保持了良好的可移植性。

现在我们可以创建任意多的 `starter/postgres_datastore` 实例，并使用任意多的 `starter-postgres_data` 风格容器的卷（前提是命名唯一或使用容器 ID）。这比我们自己挂载文件夹更容易脚本化，因为我们让 docker 完成了繁重的工作。

特别酷的一点是：这些纯数据容器甚至不需要处于运行状态，只需要存在即可。

--------------------------
### 创建 Postgres DB 镜像
这是创建 Postgres 10.2 docker 镜像。与数据容器镜像类似，
我们也把一个唯一的 `README.md` 复制进镜像，让 docker 识别其唯一性。
镜像中加入 `vim-tiny`，允许我们在需要时进入容器做些编辑。

-- Dockerfile --

```dockerfile
FROM postgres:10.2

RUN apt-get update && apt-get install -y vim-tiny

RUN mkdir /app
WORKDIR /app

ADD README.md /app/

EXPOSE 5432
```

创建 DB 镜像：

```bash
cd db
docker build -t starter/postgres .
```

启动 DB 镜像容器：

```bash
docker run -d \
    -e POSTGRES_USER=admin \
    -e POSTGRES_PASSWORD=admin \
    -e POSTGRES_DB=starter \
    --volumes-from starter-postgres_data \
    --name starter-postgres \
    -p 5432:5432 starter/postgres
```

尝试连接 starter-postgres 容器，

```bash
psql -h localhost -U admin starter
```

-----------------------
### 创建 API starter-api 镜像
你可以在 [Starter API 文档](https://github.com/bennycheung/Flask-DB-Docker/blob/master/starter-api/README.md)中阅读起步 API 微服务。
这是一个 Python/Flask RESTful API 实现，连接 Postgres 后端，
演示对持久数据的完整 CRUD 操作。我们不自动启动 Python 应用（见注释掉的 `ENTRYPOINT` 与 `CMD`），而是从命令行启动。

-- Dockerfile --

```dockerfile
FROM python:3.5

RUN mkdir /app
WORKDIR /app

ADD requirements.txt /app/
RUN pip install -r requirements.txt

ADD . /app

# If you want to start the Python application automatically, uncomment the following
# EXPOSE 5000
# ENTRYPOINT ["python", "manage.py", "runserver"]
# CMD ["-t", "0.0.0.0", "-p", "5000"]
```

创建 API 镜像：

```
cd starter-api
docker build -t starter/starter-api .
```

启动 Starter API 容器：

```bash
docker run -d -t \
    -v `pwd`/logs:/app/logs \
    --name starter-api \
    -p 5000:5000 starter/starter-api \
    python manage.py runserver -t 0.0.0.0 -p 5000
```


### 把样本（医疗程序代码）导入数据库
导入样本医疗程序代码时，我们想启动一个 `starter/starter-api` 实例。不过，这个实例需要与当前运行的网络处于同一个 Docker 网络。
启动栈时，会创建一个名为 `flask-db-docker_default` 的网络。
可以运行 `docker network ls` 确认。

    $ docker network ls

    NETWORK ID          NAME                      DRIVER              SCOPE
    e337657076ec        bridge                    bridge              local
    88cc0cfe1cc7        composer_default          bridge              local
    945ac8820956        flask-db-docker_default   bridge              local
    7749efcaa3dc        host                      host                local
    0c4c4f14a186        none                      null                local

我们应该会看到 `starter-postgres` 和 `starter-api` 都属于 `flask-db-docker_default` 网络，

    $ docker network inspect flask-db-docker_default

    ...
    "Containers": {
        "b43291da217f705ea3d1676fb757b1dafc7c8fdf18f2df32d441e248a6ab8b31": {
            "Name": "starter-postgres",
            "EndpointID": "d01bdf7da8de378b51928a7e7e2a55fab6995865ea5d1c7793e9923c9409f287",
            "MacAddress": "02:42:ac:1d:00:03",
            "IPv4Address": "172.29.0.3/16",
            "IPv6Address": ""
        },
        "e05d5fba146fc4f1c018df41f1cae6bf4fe3d54395572cd73b2c3a3c9bbf859b": {
            "Name": "starter-api",
            "EndpointID": "19e27585ae2005d6310bc6ec8dfbfe8a405400cb4196bfd90206a1fbd9f5700c",
            "MacAddress": "02:42:ac:1d:00:02",
            "IPv4Address": "172.29.0.2/16",
            "IPv6Address": ""
        }
    }

然后我们启动一个 `starter-api` 容器实例，加入 `flask-db-docker_default` 网络，
以便访问同一个 `starter-postgres` 数据库连接。
我们启动一个交互式 shell `-ti` /bin/sh。容器运行后会通过 `--rm` 删除容器参数自动
销毁。

```bash
docker run --rm -ti --network flask-db-docker_default starter/starter-api /bin/sh
```

进入容器后，我们可以通过在 `starter-api/startup` 文件夹中运行以下命令来执行初始的 `starter` 数据库创建。

```bash
python manage.py resetdb
```

我们创建用户 `admin`，密码 `admin`（当然这不安全）。

数据库创建好后，我们可以从 `tex-api/startup` 文件夹插入全部医疗程序代码来使用这个数据库。

```bash
docker run --rm -t \
    --network flask-db-docker_default \
    -v `pwd`/starter-api/startup:/app/startup \
    starter/postgres /bin/bash -c \
    "PGPASSWORD=admin psql -h starter-postgres -U admin starter < /app/startup/starter_pxcodes.sql"
```

我们可以从已加载的 `starter-api` 获取医疗程序数据。

```bash
http --auth admin:admin GET http://localhost:5000/api/v1.0/pxcodes/
```

你应该会看到 100 条医疗程序代码的分页 JSON 响应。

```
HTTP/1.0 200 OK
Content-Length: 15973
Content-Type: application/json
Date: Thu, 27 Sep 2018 22:18:00 GMT
ETag: "678ca404a51794f10b8747ea1eeb4959"
Server: Werkzeug/0.9.6 Python/3.5.6

{
    "items": [
        {
            "procedure": "EXTRACRANIAL PROCEDURES W/O CC/MCC",
            "pxcode": "039",
            "url": "http://localhost:5000/api/v1.0/pxcodes/039"
        },
        {
            "procedure": "DEGENERATIVE NERVOUS SYSTEM DISORDERS W/O MCC",
            "pxcode": "057",
            "url": "http://localhost:5000/api/v1.0/pxcodes/057"
        },
        ... skipped ...
    ],
    "meta": {
        "first": "http://localhost:5000/api/v1.0/pxcodes/?per_page=100&page=1",
        "last": "http://localhost:5000/api/v1.0/pxcodes/?per_page=100&page=1",
        "next": null,
        "page": 1,
        "pages": 1,
        "per_page": 100,
        "prev": null,
        "total": 100
    }
}

```

## 下一步
即便机器的身体与软件的大脑只能以分布式、虚拟化的形式存在，系统身份
已被承认为一种新的设计形式——快速微服务架构风格。
我们用一个由数据库后端支持的面向数据 API 演练了这种新设计风格，
它可以被快速构建与部署。有了展示的起步框架 <https://github.com/bennycheung/Flask-DB-Docker>，
我们已准备好用微服务风格的组合来构建更大的软件系统。
