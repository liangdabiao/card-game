---
layout: post
title: SingularityNET AI 服务集成
date: 2018-02-18 01:00:00.000000000 -05:00
permalink: singularitynet-ai-service-integration
author: Benny Cheung
artwork:
  author: Style of Kandinsky transfer to an AI head image
tags:
  - AI
  - Blockchain
  - SingularityNET
  - Machine Learning
category: post
comments: true
image: images/singularitynet-ai-service-integration/cover2x.jpg
images:
  cover: images/singularitynet-ai-service-integration/cover.jpg
  header: images/singularitynet-ai-service-integration/header.jpg
published: true
---

<!--excerpt.start-->
随着 **AI** 与 **Blockchain** 技术的出现及其对业务的指数级影响，新近发布的开源项目 **SingularityNET** <https://singularitynet.io/> 确实具有革命性：它把两种技术结合成一个去中心化的 AI 服务协作市场，并由区块链智能合约背书。在 SingularityNET 平台中，AI 的红利成为造福所有人的全球公共基础设施；任何人都可以访问 AI 技术，或成为其发展的利益相关者。任何人都可以向 SingularityNET 添加 AI/机器学习服务供网络使用，并换取网络支付代币。
<!--excerpt.end-->

目前，唯一的技术信息来自他们的白皮书（2017 年 12 月 19 日）<https://public.singularitynet.io/whitepaper.pdf>，它让我们得以窥见技术细节。在挣扎了几天之后，我发现了一条通过 hack SingularityNET 源代码来消化其技术信息的路径。本文总结如何实现一个 AI 服务提供方与 SingularityNET 服务封装 API 的集成。示例 AI 服务使用 Tensorflow 实现的 MNIST 图像分类。

![SingularityNET 去中心化 AI+区块链架构]({{ site.baseurl }}/images/singularitynet-ai-service-integration/SingularityNET_decentralized_AI.jpg)

*图：SingularityNET 高层系统架构，说明平台如何支持 AI 代理之间的交互，并使用区块链智能合约记录交易。*

本文将走完实验 SingularityNET AI 代理集成的各个步骤：创建虚拟环境、检出源码、运行 MNIST Tensorflow 代理示例、讲解服务适配器开发，以及展示通过配置完成的服务集成。

显然，还有更多 SingularityNET 主题，必须留待其他文章探讨。

> <div style="color:red">SingularityNET 正在推进另一个 alpha 版本。本文自 2018-05-03 起已过时；不过其中的信息对于理解协作式 AI 代理的功能视图与实现视图仍然有用。关于最新的 SingularityNET 开发，读者可参考 Wiki：<https://github.com/singnet/wiki/wiki>。</div>

### 创建虚拟环境
虚拟环境让分离不同项目变得容易，避免各组件之间因依赖与版本要求不同而产生的问题。在终端客户端输入以下命令，其中 `envname` 是你想给环境起的名字，把 `x.x` 替换为你想使用的 Python 版本。

```
conda create -n envname python=x.x anaconda
```

从现在起，我们使用 `singnet` 作为环境名。

```
conda create -n singnet python=3.6 anaconda
```

要激活或切换到你的虚拟环境，只需输入 `singnet`——即你创建环境时起的名字。

```
source activate singnet
```

### SingularityNET 源代码
最前沿的 SingularityNET 代码位于 <https://github.com/singnet/singnet>

把代码检出（checkout）到本地文件系统，

```
git clone https://github.com/singnet/singnet.git
```

检出之后，本文假设代码位于 `singnet` 目录。

### 安装 SingularityNET 代理依赖

```
cd singnet/agent
pip install -r requirements.txt
```

安装列表很长。

```
(singnet) [bcheung@Benny-Cheung:agent] pip install -r requirements.txt
Collecting aiohttp (from -r requirements.txt (line 1))
  Downloading aiohttp-3.0.1-cp36-cp36m-macosx_10_11_x86_64.whl (371kB)
    100% |████████████████████████████████| 378kB 1.0MB/s
...
Successfully installed Jinja2-2.10 MarkupSafe-1.0 PyYAML-3.12 Pygments-2.2.0 aiohttp-3.0.1 aiohttp-cors-0.6.0 aiohttp-jinja2-0.16.0 alabaster-0.7.10 argh-0.26.2 async-timeout-2.0.0 attrs-17.4.0 babel-2.5.3 bleach-1.5.0 bson-0.5.2 cchardet-2.1.1 chardet-3.0.4 commonmark-0.5.4 coverage-4.5.1 coveralls-1.2.0 cytoolz-0.9.0 docopt-0.6.2 docutils-0.14 eth-abi-0.5.0 eth-keys-0.1.0b4 eth-tester-0.1.0b11 eth-utils-0.8.0 feedparser-5.2.1 fire-0.1.2 funcsigs-1.0.2 future-0.16.0 html5lib-0.9999999 hvac-0.4.0 idna-2.6 idna-ssl-1.0.0 imagesize-1.0.0 isodate-0.6.0 jsonrpcclient-2.5.2 jsonrpcserver-3.5.3 jsonschema-2.6.0 livereload-2.5.1 lru-dict-1.1.6 markdown-2.6.11 mock-2.0.0 multidict-4.1.0 numpy-1.14.0 packaging-16.8 pathtools-0.1.2 pbr-3.1.1 pluggy-0.6.0 port-for-0.3.1 protobuf-3.5.1 py-1.5.2 pyaml-17.12.1 pyparsing-2.2.0 pysha3-1.0.2 pytest-3.4.0 pytest-cov-2.5.1 pytz-2018.3 rdflib-4.2.2 recommonmark-0.4.0 requests-2.18.4 rlp-0.6.0 semantic-version-2.6.0 six-1.11.0 snowballstemmer-1.2.1 sphinx-1.7.0 sphinx-autobuild-0.7.1 sphinx-rtd-theme-0.2.4 sphinxcontrib-websupport-1.0.1 tensorflow-1.3.0 tensorflow-tensorboard-0.1.8 toolz-0.9.0 tornado-4.5.3 urllib3-1.22 uvloop-0.9.1 watchdog-0.8.3 web3-3.16.5 werkzeug-0.14.1 yarl-1.1.1
```

一切依赖都顺利装好、没有磕绊，生活看起来很美好——这在折腾（hacking）经历中可是件稀罕事。

### Docker 前置要求
根据 SingularityNET 官方网站，SingularityNET 可运行于 Mac OS X，或任何装有 Python 3 以及
Docker（或 Docker for Mac）的 Linux。他们的核心开发者日常在 Mac OS X Sierra、Linux Mint Mate 18.2 和 Linux Ubuntu 16.04 LTS 等系统上开发。

Docker 和 Docker Compose 被大量使用，因此你必须安装较新版本的 Docker，才能利用大部分自动化能力，并把自己与"依赖地狱"隔离开——直接把复杂依赖的软件装进宿主开发操作系统时常常会遭遇这种地狱。

当前的开发演示从一个 `dev` docker 容器运行，可以使用辅助工具 shell 脚本从你喜欢的 bash 终端启动：`singnet/tools.sh`。

### MNIST 代理运行示例
启动 Dev 服务器，演示多个代理的交互。

```
tools.sh dev
```

执行会显示如下内容：

```
(singnet) [bcheung@Benny-Cheung:singnet] ./tools.sh dev
...
... it will take a few minutes to pull docker images for the first run
...
======== Running on http://0.0.0.0:8000 ========
(Press CTRL+C to quit)
```

你可以查看 SingularityNET 的 docker 容器正在运行，如 Docker 的 Kitematic UI 所示。

![SingularityNET Docker 运行中]({{ site.baseurl }}/images/singularitynet-ai-service-integration/SingularityNET_Dev_Docker_Running.jpg)

*图：展示 dev 配置启动了多个 docker 运行容器。它们的启动由 docker compose 协调*

你可以在浏览器打开 `http://localhost:8000` 进行交互。

#### Chrome 开发者工具检查
要获得更深入的洞见，我们可以用 Chrome 开发者工具检查网络流量。

你需要开着 Network 标签页并按类型 'All' 过滤，然后重新加载页面。这会展示一个正在建立的网络调用连接。你可以点击该连接，查看与服务器之间来回发送的流量。

![SingularityNET Chrome 开发者检查工具]({{ site.baseurl }}/images/singularitynet-ai-service-integration/Singularity_Chrome_Dev_Network_View.jpg)

*图：Chrome 开发者工具展示请求数据。*  

在 UI 上点击 `[Preform]` 按钮后，MNIST 请求连同图像数据一起发出。MNIST 图像数据是一个 784（28x28）维的像素向量。预测响应既可以在 UI 上查看，也可以在 Chrome 网络检查工具中查看。

![SingularityNET Chrome 开发者预测响应]({{ site.baseurl }}/images/singularitynet-ai-service-integration/Singularity_Chrome_Dev_Network_View_Prediction.jpg)

*图：Chrome 开发者工具展示响应结果。*  

预测结果是 `7`，置信度为完美的 `1`，即它确信看到了数字七。

### 服务适配器开发

#### ServiceAdapterABC
服务适配器抽象基类上有三个相当常用的方法：`__init__` 用于初始化成员变量，`post_load_initialize` 用于确保所有依赖服务都已加载，最后的 `perform` 用于执行 AI 业务逻辑、根据请求返回结果。

![SingularityNET Chrome 开发者预测响应]({{ site.baseurl }}/images/singularitynet-ai-service-integration/Singularity_Dev_ServiceAdapterABC.jpg)

我们来看服务适配器的成员变量。在这里列出所有依赖服务很重要，务必记住。如果需求得到满足，变量 `requirements_met` 将被设为 `True`。unavailable 标志定义特定服务当前是否在线。系统其余部分利用这个 `available` 标志来自动管理事务。

源代码位置：`singnet/agent/sn_agent/service_adapter.py`

```python
class ServiceAdapterABC(ABC):
    """
    This is the service adapter base, all other service adapters are based on it.
    """

    type_name = "Base"

    def __init__(self, app, service: Service, required_services: List[Service] = None) -> None:
        self.app = app
        self.service = service
        self.required_services = required_services
        self.required_service_adapters = []
        self.requirements_met = False
        self.available = False
```

`post_load_initialize` 会递归调用所有所需服务的 post 节点进行初始化。如果一个服务依赖于另外三四个服务，它们会被一起加载、一起初始化。

```python
    def post_load_initialize(self, service_manager: ServiceManager):
        """
        This will hunt out all the agents required to fulfill the required ontology ids

        We should periodically call this if it is false - an agent might come alive that can support this
        :return:
        """
        if not self.required_services is None:
            for required_service in self.required_services:
                service_adapter = service_manager.get_service_adapter_for_id(required_service.node_id)
                self.required_service_adapters.append(service_adapter)
        self.requirements_met = self.has_all_requirements()

        logger.info('Service Adapter: %s initialized. Requirements met: %s', self.type_name, self.requirements_met)
        # print('Service Adapter: %s initialized. Requirements met: %s' % (self.type_name, self.requirements_met))

    def has_all_requirements(self):
        """
        Check to see if our all required services are available
        :return:
        """
        for required_service_adapter in self.required_service_adapters:
            if not required_service_adapter.has_all_requirements():
                return False
        return True
```

还有 `start` 和 `stop` 函数，用于让服务上线和下线。

```python
    def start(self):
        """
        If init sets up all the connections, start is here to ensure that the worker is actually alive and can process
        :return:
        """
        self.available = True

    def stop(self):
        """
        This will take the worker offline but does not need to be re-initialized
        :return:
        """
        self.available = False
```

`can_perform` 函数用于节省网络开销：在发出一个会涉及区块链合约、并可能产生一些 gas 费用的请求之前，你可以直接询问代理能否执行。

```python
    def can_perform(self) -> bool:
        """
        This is a boolean flag indicating if this worker can do whatever work it says it can.

        An answer of no can be because it is offline, or perhaps it is too busy.
        :return:
        """
        return self.requirements_met and self.available and self.all_required_agents_can_perform()

    def all_required_agents_can_perform(self):

        if self.required_services is None:
            return True

        for required_service_adapter in self.required_service_adapters:
            if not required_service_adapter.can_perform():
                return False
        return True
```

`perform` 函数是大部分工作完成的地方。让我们深入 MNIST 分类器看看细节。

```python
    @abstractmethod
    def perform(self, job: JobDescriptor):
        """
        This is where the work gets done, the worker will block here until the work itself is done
        :param args:
        :param kwargs:
        :return:
        """
        pass
```

#### TensorFlowMNIST

![SingularityNET Chrome 开发者预测响应]({{ site.baseurl }}/images/singularitynet-ai-service-integration/Singularity_Dev_ServiceAdapterTensorFlowMNIST.jpg)

MNIST 服务适配器其实什么也不做，只是在初始化时检查分类器 ID。在 `post_load_initilize` 中，我们完成分类器的大部分训练与模型加载工作。

源代码位置：`singnet/agent/adapters/tensorflow/mnist/__init__.py`

```python
class TensorflowMNIST(ServiceAdapterABC):
    type_name = "TensorflowMNIST"

    def __init__(self, app, service: Service, required_services: List[Service] = None):
        super().__init__(app, service, required_services)
        if not service.node_id == MNIST_CLASSIFIER_ID:
            raise RuntimeError("TensorflowMNIST cannot perform service %s", service.node_id)
```

`post_load_initilize` 的细节不再赘述。我们需要放入对输入图像和分类器图本身的引用。还有几处检查路径是否存在以及加载模型的逻辑。

```python
def post_load_initialize(self, service_manager: ServiceManager):

    # Train the model or load a pretrained model from the cache.

    # There are member variables for the placeholders:
    #     self.input_images - images to be classified
    #     self.keep_prob - controls the dropout during training
    #
    # As well as member variables we need to have during perform:
    #     self.classifier_graph - the tensorflow graph for the image classifier
    #     self.model_path - the location of the cached model

    # Import data
    data_directory = os.path.join(AGENT_DIRECTORY, "input_data")
    mnist_data = input_data.read_data_sets(data_directory, one_hot=True)

    # Create the model - first initialize to default
    tf.reset_default_graph()

    # Define a placeholder for the images
    self.input_images = tf.placeholder(tf.float32, [None, 784])

    # Define loss and optimizer
    input_labels = tf.placeholder(tf.float32, [None, 10])

    # Build the graph for the deep neural network classifier
    self.classifier_graph, self.keep_prob = build_classifier_graph(self.input_images)

    with tf.name_scope('loss'):
        cross_entropy = tf.nn.softmax_cross_entropy_with_logits(labels=input_labels, logits=self.classifier_graph)
    cross_entropy = tf.reduce_mean(cross_entropy)

    with tf.name_scope('adam_optimizer'):
        train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

    with tf.name_scope('accuracy'):
        correct_prediction = tf.equal(tf.argmax(self.classifier_graph, 1), tf.argmax(input_labels, 1))
        correct_prediction = tf.cast(correct_prediction, tf.float32)
    accuracy = tf.reduce_mean(correct_prediction)

    # Initialize the variables (i.e. assign their default value)
    initializer = tf.global_variables_initializer()

    # 'Saver' op to save and restore all the variables

    self.model_path = os.path.join(AGENT_DIRECTORY, "model_data", "model.ckpt")
    saver = tf.train.Saver()

    # Create our long-running Tensorflow session
    self.session = tf.Session()

    logger.debug("Checking for pre-trained model in {0}".format(self.model_path))
    if os.path.exists(self.model_path + ".index"):
        logger.debug("Restoring from pre-trained model")

        # Initialize variables
        self.session.run(initializer)

        # Restore model weights from previously saved model
        saver.restore(self.session, self.model_path)

        if CHECK_ACCURACY:
            accuracy = accuracy.eval(feed_dict={
                self.input_images: mnist_data.test.images,
                input_labels: mnist_data.test.labels,
                self.keep_prob: 1.0})
            logger.debug("test accuracy {0}".format(accuracy))

    else:
        logger.debug("No checkpoint - training model from scratch")
        self.session.run(initializer)

        # Train the model
        for i in range(20000):
            batch = mnist_data.train.next_batch(50)
            if i % 100 == 0:
                train_accuracy = accuracy.eval(feed_dict={
                    self.input_images: batch[0], input_labels: batch[1], self.keep_prob: 1.0})
                logger.debug('step {0}, training accuracy {1}'.format(i, train_accuracy))
            train_step.run(feed_dict={
                self.input_images: batch[0],
                input_labels: batch[1],
                self.keep_prob: 0.5})

        # Save model weights to disk
        save_path = saver.save(self.session, self.model_path)
        logger.debug("Model saved in file: {0}".format(save_path))

        if CHECK_ACCURACY:
            accuracy = accuracy.eval(feed_dict={
                self.input_images: mnist_data.test.images,
                input_labels: mnist_data.test.labels,
                self.keep_prob: 1.0})
            logger.debug('Test accuracy {0}'.format(accuracy))
```

主要工作在 `perform` 中完成。我们启动一个会话来恢复模型。然后逐一遍历 job 中的每个 job item。我们取出输入，把它们从请求提供的数据中提取出来。对类型做些合理性检查之后，运行实际的分类器。最后，我们取出置信度，让预测与置信度值关联起来，随结果一起发送。

```python
def perform(self, job: JobDescriptor):

        # Process the items in the job. A single job may include a request to classify
        # many different images. Each item, in turn, may be an array of images.
        results = []
        for job_item in job:

            # Make sure the input type is one we can handle...
            input_type = job_item['input_type']
            if input_type != 'attached':
                logger.error("BAD input dict %s", str(job_item))
                raise RuntimeError("TensorflowMNIST - job item 'input_type' must be 'attached'.")

            # Get the images to classify, while making sure our job item dict is of the appropriate format.
            input_data = job_item['input_data']
            if input_data is None:
                raise RuntimeError("TensorflowMNIST - job item 'input_data' must be defined.")
            images_to_classify = input_data.get('images')
            if images_to_classify is None:
                raise RuntimeError("TensorflowMNIST - job item 'input_data' missing 'images'")

            # Get the predication and confidence for each image in this job item
            prediction = tf.argmax(self.classifier_graph, 1)
            confidence = tf.nn.softmax(self.classifier_graph)
            predictions = prediction.eval(session=self.session,
                                          feed_dict={self.input_images: images_to_classify, self.keep_prob: 1.0})
            confidences = confidence.eval(session=self.session,
                                          feed_dict={self.input_images: images_to_classify, self.keep_prob: 1.0})
            prediction_confidences = []
            for index in range(0, len(images_to_classify)):
                prediction_confidence = confidences[index][predictions[index]]
                prediction_confidences.append(float(prediction_confidence))

            logger.debug("Predictions: {0}".format(predictions))
            logger.debug("Confidences: {0}".format(prediction_confidences))

            # Add the job results to our combined results array for all job items.
            single_job_result = {
                'predictions': predictions.tolist(),
                'confidences': prediction_confidences,
            }
            results.append(single_job_result)

        return results
```

### 服务配置
在 `singnet/agent/dev_config.yml` 文件中，我们必须用唯一标识符 `deadbeef-aaaa-bbbb-cccc-111111111101` 定义 TensorflowMNIST 服务，并指定要加载的模块 `adatper.tensorflow.mnist.TensorflowMNIST` 类。

```yaml
services:

- service: deadbeef-aaaa-bbbb-cccc-000000000101
  module: examples.simple_adapter.SimpleAdapter

- service: deadbeef-aaaa-bbbb-cccc-111111111101
  module: adapters.tensorflow.mnist.TensorflowMNIST

...
````

但愿这个代理集成示例点燃了你对 SingularityNET 平台的好奇心，为终极的 SingularityNET AI + 区块链开发做好准备。
