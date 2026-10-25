---
layout: post
title: FHIR 服务器搭建与运行
date: 2018-04-15 01:00:00.000000000 -05:00
permalink: fhir-server-up-and-running/
author: Benny Cheung
artwork:
  author: Style of Kandinsky transfer to a blockchain image
tags:
  - FHIR
  - Electronic Health Record
  - Medical
  - Big Data
  - Blockchain
category: post
comments: true
image: images/fhir-server-up-and-running/cover2x.jpg
images:
  cover: images/fhir-server-up-and-running/cover.jpg
  header: images/fhir-server-up-and-running/header.jpg
published: true
series: "Blockchain Healthcare"
series_order: 1
---

<!--excerpt.start-->
区块链是一个让患者电子健康档案（Electronic Health Record）既可访问又安全的热门话题，人们畅谈着患者的终极梦想：完整掌握自己的病史、用药清单、化验结果、医生记录等。但这里有一个根本问题——*数据从哪里来？* 即使医疗机构正在勤勤恳恳地把患者数据数字化，这些数据在其数据库边界之外依然不可访问。
<!--excerpt.end-->

有许多尝试在弥合这道大数据鸿沟。有兴趣的读者可以学习 Coursera 的 *Health Informatics on FHIR* 课程 <https://www.coursera.org/learn/fhir>，了解这段历史、这个问题，以及 FHIR 如何让区块链共享医疗记录的梦想保持生机。本文面向区块链的信仰者——正如[克尔凯郭尔](https://en.wikipedia.org/wiki/S%C3%B8ren_Kierkegaard)的名言"信仰之跃"（A leap of faith）——搭建一台可用于研究与开发的、功能完备的 FHIR 服务器。

## FHIR 加持！
**FHIR** 是一个平台规范，它定义了一组可在整个医疗流程、所有司法辖区和众多不同场景中通用的能力。本文将引导你完成 **FHIR 服务器**的*安装、配置与测试*。有了 FHIR 服务器，医疗机构的数据就可以通过标准化 RESTful API 和 JSON 格式资源轻松访问。

此外，为限定本文的讨论范围，这里不讨论安全性话题。

### 快速链接
参考技术栈链接：
* FHIR 官方网站：<http://hl7.org/fhir>
* Hapi FHIR——Java 开源 FHIR API：<http://hapifhir.io/>
* SMART on FHIR——连接 FHIR 服务器的开放应用标准：<http://docs.smarthealthit.org/>

本文使用的工具套件及其链接：
* STU3 沙箱数据：<http://docs.smarthealthit.org/data/stu3-sandbox-data.html>
* STU3 示例患者生成器：<https://github.com/smart-on-fhir/sample-patients-stu3>
* 为 FHIR Bundle 添加标签及资源上传器：<https://github.com/smart-on-fhir/tag-uploader>
* SMART on FHIR 患者浏览器：<https://github.com/smart-on-fhir/patient-browser>

### 步骤概览
以下步骤将引导你完成服务器安装、数据生成和 UI 测试。这些步骤已在 Mac（OS X El Capitan）上验证可行。

## 编译 Hapi FHIR 软件包
去下载 Hapi FHIR Server <http://hapifhir.io/download.html>。撰写本文时，最新的 **DSTU3**（Draft Standard for Trial Use 3，试验用标准草案第 3 版）已经稳定。
我们将基于这个最新发布标准来构建服务器和数据。

HAPI 主要使用 Apache Maven 构建。即使你使用 IDE，也应该先执行一次命令行构建，再尝试在 IDE 中让一切正常工作。

执行以下命令进行构建：

```shell
mvn install
```

注意，完整构建耗时较长，因为要执行所有单元测试。最后你应该会看到类似下面的画面：
![Hapi FHIR 构建完成]({{ site.baseurl }}/images/fhir-server-up-and-running/Hapi_FHIR_mvn_build_completed.jpg)

## 配置 Hapi FHIR JPA 服务器
Hapi FHIR Server 编译完成后，我们就可以配置位于 `hapi-fhir-jpasserver-example` 的 JPA 服务器示例。

我们将把根 URL 上下文改成一个更短的名字 `hapi-fhir`（即让 URL 更短）。

```shell
vi pom.xml
```

在 Jetty 服务器插件配置一节中，把 `contextPath` XML 值改为 `/hapi-fhir`。

```xml
<!-- The following is not required for the application to build, but allows you to test it by issuing "mvn jetty:run" from the command line. -->
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <configuration>
                <webApp>
                    <contextPath>/hapi-fhir</contextPath>
                    <allowDuplicateFragmentNames>true</allowDuplicateFragmentNames>
                </webApp>
            </configuration>
        </plugin>
    </plugins>
</pluginManagement>
```

## 启动 Hapi FHIR JPA 服务器
编辑完 `pom.xml` 后，就可以用内嵌的 Jetty 网页服务器启动 Hapi FHIR Server 了，

```shell
mvn jetty:run
```

Hapi FHIR Server 示例 UI 位于：

```
http://localhost:8080/hapi-fhir
```

![HAPI FHIR 服务器 UI]({{ site.baseurl }}/images/fhir-server-up-and-running/HAPI_FHIR_Server.jpg)

Hapi FHIR 的 RESTful API 可通过这个根 URL 访问：

```
http://localhost:8080/hapi-fhir/baseDstu3
```

我们可以用 Hapi FHIR Server 的 UI 进行测试，例如检索特定患者 ID `smart-1032702`，如图所示，

![HAPI FHIR 服务器 UI 读取]({{ site.baseurl }}/images/fhir-server-up-and-running/HAPI_FHIR_Server_API.jpg)

不过等一下，我们需要先填充一些示例数据。下一节将生成一些虚构（但贴近真实）的数据集。

## 生成示例数据集
跑一台没有健康记录的服务器毫无意义。为了给服务器填充一些逼真的虚构数据，我们将使用 STU3 示例患者生成器 <https://github.com/smart-on-fhir/sample-patients-stu3>

这个工具的主要用途是生成 JSON 文件形式的 FHIR STU3 transaction bundle。生成之后，这些 bundle 可以通过 FHIR 服务器的 API 插入任何兼容的 FHIR 服务器。

该生成器需要 `Python 2.7` 才能运行。

从 'bin' 目录运行以下命令可获得一般性帮助信息，

```shell
cd sample-patients-stud3/
python generate.py --help
```

在 `../out` 目录中生成测试数据文件：

```shell
python generate.py --write-fhir ../out --id-prefix "smart"
```

该数据集中所有示例数据的 ID 都会带上 `smart` 前缀。

## 上传示例数据集到服务器
我们将使用 Tag Uploader <https://github.com/smart-on-fhir/tag-uploader>

```shell
cd tag-updloader/
npm install
```

上一步生成的示例数据位于 `out` 目录。我们可以把这个目录移到 `tag-uploader` 之下。由于 Javascript 文件 `index.js` 是主入口，我们可以把它重命名为 `tag-uploader.js` 以便后续执行。

为指定目录下（深度遍历）找到的每个 JSON bundle 添加标签。

```shell
node tag-uploader -d {SOURCE_DIR} -t "Some tag" -w
```

上传指定目录下（深度遍历）找到的 JSON bundle：

```shell
node tag-uploader -d {SOURCE_DIR} -t "Some tag" -S {SOME_FHIR_SERVER}
```

在我们的例子中，`{SOURCE_DIR}` 是 `out` 目录，`{SOME_FHIR_SERVER}` 是 `http://localhost:8080/hapi-fhir/baseDstu3`

```shell
node tag-uploader -d out -t "smart" -w
node tag-uploader -d out -t "smart" -S http://localhost:8080/hapi-fhir/baseDstu3 
```

上传全部示例数据需要几分钟。标签 `smart` 可以用来区分不同的数据集。

## 用 Patient-Browser 可视化示例数据
我们将使用 `SMART on FHIR` 的患者浏览器 <https://github.com/smart-on-fhir/patient-browser>

```shell
cd patient-browser/
```

配置位于 `build` 目录，将使用默认配置文件 `stu3-open-sandbox.json5`。我们可以编辑 `url`，让它指向我们的 Hapi FHIR Server。

```
...
server: {
    // "DSTU-2" or "STU-3"
    type: "STU-3",

    // Api URL
    url: "http://localhost:8080/hapi-fhir/baseDstu3",
...
```

`patient-browser` 之所以默认使用 `stu3-open-sandbox` 配置，是因为 `src/App/index.js` 中定义了 `DEFAULT_CONFIG` 常量。

```javascript
const DEFAULT_CONFIG = "stu3-open-sandbox";
```

配置修改完成后，运行以下命令启动服务器

```shell
npm install
```

经过全部编译和 webpack 打包后，项目运行于

```
http://localhost:9001
```

你可以看到已加载的示例患者列表。
![患者浏览器——患者列表]({{ site.baseurl }}/images/fhir-server-up-and-running/SMART_FHIR_Patient_Browser.jpg)

点击某个患者，即可浏览该患者的详细信息。
![患者浏览器——患者详情]({{ site.baseurl }}/images/fhir-server-up-and-running/SMART_FHIR_Patient_Browser_Patient_Details.jpg)

就写到这里，Happy Hapi FHIRing！
