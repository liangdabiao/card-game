---
layout: post
title: 在 Replicate 上用 FLUX 模型做个人嵌入与微调
date: 2024-12-27 12:00:00.000000000 -00:00
permalink: personal-fine-tuning-with-flux
author: Benny Cheung
artwork:
  author: FLUX.1
tags:
- Generative AI
- Stable Diffusion
- Image Generation
category: post
comments: true
image: images/personal-fine-tuning-with-flux/post_cover.jpg
images:
  cover: images/personal-fine-tuning-with-flux/cover2x.jpg
  header: images/personal-fine-tuning-with-flux/header.jpg
published: true 
---

<!--excerpt.start-->
图像生成在过去六个月取得了巨大进展。作为一个深入投入探索 [Stable Diffusion]({{ site.baseurl }}/stable-diffusion-training-for-embeddings) 和 [Dreambooth]({{ site.baseurl }}/dreambooth-training-for-personal-embedding) 等模型创造潜力的人，我发现尖端 FLUX 模型的能力时兴奋不已。这一突破性创新，加上 Replicate 易用的云端平台，改变了创建个性化嵌入的流程。
<!--excerpt.end-->

![用 Flux 生成的个人嵌入图像]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/bennycheung_imagination_post.jpg)

_图：用 FLUX.1 模型把个人嵌入的全部创意想象投射到多元宇宙。可能性无穷无尽。_

不再受困于繁琐的本地安装，这套更新的方法解锁了前所未有的简单与易得。无论你是创意专业人士、AI 爱好者，还是仅仅好奇如何把自己想象的多元宇宙版本变为现实，本指南将带你走完旅程的每一步。

文末我们还会讲进阶话题：多 LoRA 生成——组合多个微调模型以获得无与伦比的创造力——以及构建微服务来简化图像生成。这些章节将赋能你扩展创意工作流，把前沿工具集成到你自己的应用中。潜入其中，发现 FLUX 模型与 Replicate 的力量如何重新定义我们生成和微调个人嵌入的方式。

----
> 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/personal-fine-tuning-with-flux/Podcast_Personal_Embedding_with_Flux.mp3
{% include open-embed.html %}

-----
## 为什么选择 FLUX 模型？

[FLUX.1](https://blackforestlabs.ai/ultra-home/) 是 Black Forest Labs 今年夏天发布的文生图模型家族。FLUX.1 模型为开源图像模型树立了新标准：它们可以生成逼真的手、清晰的文字，甚至那个出奇困难的任务——有趣表情包。

### FLUX.1 的变体

FLUX.1 系列包含三个主要版本，各自面向特定用例：

- **FLUX.1 pro**：面向商业用途的高级闭源模型。它提供卓越性能，并通过官方 API 支持企业级定制。
    
- **FLUX.1 dev:** 由 FLUX.1 pro 派生的开源非商业版本，提供高质量图像和稳健的提示词遵循能力，同时更加高效。（_本文将聚焦使用 FLUX.1 dev 进行个人嵌入与微调。_）
    
- **FLUX.1 schnell:** 开源的轻量模型，针对速度与最小内存占用优化，适合本地开发与个人项目。

## 起步：用 Replicate 微调 FLUX 模型

为确保神经网络得到正确训练，必须提供足够数量的图像，涵盖你在各种造型、姿势与背景下的样子。如果你只给 AI 你摆一种姿势或穿一套衣服的照片，它就只能生成与该输入匹配的图像。给 AI 一组多样的图像去学习，才能确保更广泛的选择与图像。

### 准备训练数据

开始微调之前，你需要一组代表你想教给模型的概念的图像。这些图像应足够多样，覆盖概念的不同侧面。例如，如果针对特定人物微调，要包含各种场景、姿势和光照下的图像。

以下是指南：

- 为获得最佳效果，使用 12-20 张图像
- 尽可能使用大图
- 使用 JPEG 或 PNG 格式
- 可选：为每张图像创建同名的对应 .txt 文件，内含说明文字（caption）

图像（及可选的说明）备齐后，打包为一个 zip 文件。

### 示例——为微调准备个人图像

基于 Jame Cunliffe 的[视频教程](https://www.youtube.com/watch?v=P1dfwViVOIU&ab_channel=JAMESCUNLIFFE)，以下是我实践成功的推荐搭配：
- 3 张不同角度的全身照
- 5 张不同视角的半身照
- 12 张展现各种面部表情的特写

用 [BIRME](https://www.birme.net/?target_width=512&target_height=512) 或 Photoshop 之类的工具把图像调整为 512x512 像素。调整后，把文件压缩为 zip 压缩包。

![Stable_Diffusion_00_SelfPortrait_Capture20_02]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/Stable_Diffusion_00_SelfPortrait_Capture20_02.png)

图像（及可选说明）备齐后，打包成一个 zip 文件，例如我们在下例中使用 `bennycheung-portrait.zip`。

-----
## 用 Replicate 微调

[Replicate.ai](https://replicate.com/) 是一个强大的平台，简化机器学习模型在云端的部署、执行与微调。它提供由各研究者与开发者开发的大量预训练模型，让用户能够处理图像生成、自然语言处理等任务及其他专门的 AI 应用。

### 搭建环境

Replicate 提供友好的用户界面，但开发者可能更偏向以编程方式与平台交互。这种方式的最大优势是可以自动化生成流程，无需依赖可能繁琐的用户界面。

话虽如此，如果你偏好亲自动手的交互体验，Replicate 的 Web UI 始终是可以手动执行生成任务的选择。

### 安装

```
pip install replicate
```

设置 `REPLICATE_API_TOKEN` 环境变量

```
export REPLICATE_API_TOKEN=<paste-your-token-here>
```

也可以在 API 调用时指定 `token` 参数。

### 创建你的模型

创建一个新模型，作为微调权重的归宿。流程完成后，训练好的模型就存放在这里。

```python
import replicate

model = replicate.models.create(
    owner="bennycheung", # change to your owner name
    name="flux-dev-bennycheung-lora", # change to your LoRA model name
    visibility="private", # or "public" if you prefer
    hardware="gpu-t4", # Replicate will override this for fine-tuned models
    description="A fine-tuned FLUX.1 bennycheung model"
)

print(f"Model created: {model.name}")
print(f"Model URL: https://replicate.com/{model.owner}/{model.name}")
```

### 用 LoRA 训练做微调

使用 [Ostris 的 AI Toolkit](https://replicate.com/ostris/flux-dev-lora-trainer/train) 经 LoRA（低秩适应）微调 FLUX.1 [dev]。一次 1,000 步的典型训练耗时 15-20 分钟，花费不到 2 美元。

> 别忘了把 `bennycheung-portrait.zip` 改成你自己的 zip 文件名。
> destination 应是你的模型名 `{owner}/{model}`。

```python
import replicate

training = replicate.trainings.create(
    version="ostris/flux-dev-lora-trainer:4ffd32160efd92e956d39c5338a9b8fbafca58e03f791f6d8011f3e20e8ea6fa",
    input={
        "input_images": open("bennycheung-portrait.zip", "rb"),
        "steps": 1000,
        "trigger_word": "bennycheung",
    },
    destination=f"bennycheung/flux-dev-bennycheung-lora"
)

print(f"Training started: {training.status}")
print(f"Training URL: https://replicate.com/p/{training.id}")
```

注意，此时为模型选哪种硬件并不重要，因为 Replicate 会把所有 FLUX.1 微调都路由到 H100。

**微调的最佳实践**

1. **数据集质量：** 使用 12-18 张高分辨率图像（最好 1024x1024）。
2. **风格一致性：** 避免同一主体的年龄、发型等变化。
3. **触发词：** 选择清晰、唯一的标识符（例如 "CustomSubject"）。
4. **参数调优：** 调整 LoRA rank 与推理步数来打磨模型表现。

------
## 用 Replicate 生成图像

微调完成后，你可以用编程方式生成图像：
 
```python
import replicate
 
name = "bennycheung_SciFi"
prompt = """
bennycheung as Gordon Freeman in Half Life, wearing the hev suit,
ultra realistic, intricate, elegant, highly detailed, digital painting,
artstation, smooth, sharp focus, illustration, in the style of greg rutkowski
"""

# change the following to generate multiple images
# batch_size control how many images are generated per call, range from 1 to 4
# n control how many iterations
# Total number of generated images = n * batch_size
#
start_idx = 1
batch_size = 1
n = 1

for k in range(0, n):
    print(f"Generating {batch_size} images for {name} {k}...")

    output = replicate.run(
        "bennycheung/flux-dev-bennycheung-lora",
        input={
            "model": "dev",
            "prompt": prompt,
            "prompt_strength": 1.0,
            "lora_scale": 0.9,
            "num_outputs": batch_size,
            "aspect_ratio": "2:3",
            "output_format": "png",
            "guidance_scale": 3.5,
            "output_quality": 80,
            "num_inference_steps": 28,
            "disable_safety_checker": True,
        }
    )
    
    # If the model returns multiple outputs
    for idx, file_output in enumerate(output):
        idx = idx + start_idx
        with open(f'{name}_{idx:03d}.png', 'wb') as f:
            f.write(file_output.read())
    
    start_idx += batch_size
```

FLUX 模型生成了超逼真的图像，确实像极了身为 [Gordon Freeman](https://en.wikipedia.org/wiki/Gordon_Freeman)（《半衰期》游戏主角）的我。

![bennycheung_SciFi_006]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/bennycheung_SciFi_006.png)

依据给定图像调整图像生成（即 `img2img`）
> `image` 参数指定一个本地文件，例如代码中所示的 `bennycheung_HalfLife_protrait_04.png`。

```python
output = replicate.run(
    "bennycheung/flux-dev-bennycheung-lora",
    input={
        "image": open("bennycheung_HalfLife_protrait_04.png", "rb"),
        "model": "dev",
        "prompt": "bennycheung as Gordon Freeman in Half Life, wearing the hev suit , ultra realistic, intricate, elegant, highly detailed, digital painting, artstation, smooth, sharp focus, illustration, in the style of greg rutkowski",
        "go_fast": False,
        "lora_scale": 1,
        "megapixels": "1",
        "num_outputs": 1,
        "aspect_ratio": "2:3",
        "output_format": "png",
        "guidance_scale": 3,
        "output_quality": 80,
        "prompt_strength": 0.8,
        "extra_lora_scale": 1,
        "num_inference_steps": 28
    }
)
```

以下是使用 `img2img` 生成的示例图像。变化依据第一张图像。

![bennycheung_HalfLife_protrait_04_Tweak_it]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/bennycheung_HalfLife_protrait_04_Tweak_it.jpg)

-----
## 使用多个 LoRA 微调模型

把微调模型与 FLUX 组合的另一个例子：使用 Freeman 的蒸馏服（[roadmaus/stillsuit](https://replicate.com/roadmaus/stillsuit) 模型）搭配我的私人外形（`bennycheung/flux-dev-bennycheung-lora` 模型）。目标？在厄拉科斯星球上做时尚宣言！

![bennycheung_Dune_post_01]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/bennycheung_Dune_post_01.jpg)

以下代码演示了 Replicate 调用：通过设置 `extra_lora` 与 `extra_lora_scale` 参数，在生成中同时使用这些模型。

```python
# Need REPLICATE_API_TOKEN environment variable.
#
import replicate
 
name = "bennycheung_Dune"
prompt = """
The image portrays "bennycheung" as a character in the 2021 film "Dune,"
wearing the iconic ST1LLSU1T and nose piece designed for survival in the harsh
desert environment of the planet Arrakis. This photograph captures the
character in a position amidst a rocky setting. The ST1LLSU1T is a marvel of
practical design and functionality, engineered to conserve and filter the
wearer's bodily moisture. The suit is comprised of numerous intricate
components, such as hydration tubes running along the body, filters
strategically placed at crucial collection points, and a snug, form-fitting
material that appears both robust and flexible to allow for mobility and
endurance in the brutal desert climate.

The character's suit is primarily dark in color, varying shades of grey
and black which helps it blend seamlessly into
the desert rock background. The materials of the ST1LLSU1T likely combine
advanced synthetic fabrics and possibly some organic elements, designed to
maximize both efficiency in moisture recovery and comfort. The texture of the
suit exhibits a rugged and utilitarian aesthetic, evident in the reinforced
patches and ribbed sections on the arms and legs, which suggest areas requiring
extra protection and durability. There is noticeable wear, indicative of the
character's extensive use and adaptation to the extreme conditions of Arrakis.

Wrapped around the character's neck and shoulders is a light, sand-colored
scarf, adding layers to the attire that offer protection from the sun and sand.
The scarf's soft, woven texture contrasts with the more structured and
technical design of the ST1LLSU1T. The lighting in the scene is muted, natural
daylight filtering into what seems to be a sheltered segment of the desert,
casting subtle shadows and highlights on the suit. This lighting accentuates
the detailed construction of the ST1LLSU1T, highlighting its rugged surface and
the practical placement of its components. The character's facial expression
and body language convey a sense of alertness and determination, reflected in
the intense gaze of their striking blue eyes. This eye color is heavily
pronounced against the neutral tones of the surrounding environment and the
suit, suggesting a significant narrative element in the movie "Dune" related to
the use of the spice known as Melange, which turns the eyes blue. Anatomically,
the character appears to be young, with a lean and athletic build, emphasizing
the suitability and necessity of the ST1LLSU1T's design for such a physique.

The gloves worn by the character are integrated into the suit, with flexible
joints and padded areas, likely aids for handling tools and weapons in the
harsh terrain. As a medium, the photograph is highly detailed, with a sharp
focus on the character and the ST1LLSU1T, allowing viewers to appreciate the
intricacies of the suit's design. The composition places the character slightly
off-center, with the surrounding rock formations framing them, underscoring the
isolation and the challenging environment of Arrakis. The image encapsulates
the essence of survival and resilience central to the film's theme, with the
ST1LLSU1T standing out as a critical element in this depiction. The person
depicted in the image remains unidentified.
"""

# change the following to generate multiple images
# batch_size control how many images are generated per call, range from 1 to 4
# n control how many iterations
# Total number of generated images = n * batch_size
#
start_idx = 1
batch_size = 1
n = 4

for k in range(0, n):
    print(f"Generating {batch_size} images for {name} {k}...")

    output = replicate.run(
        "bennycheung/flux-dev-bennycheung-lora",
        input={
            "model": "dev",
            "extra_lora": "roadmaus/stillsuit",
            "extra_lora_scale": 0.8,
            "prompt": prompt,
            "prompt_strength": 0.8,
            "lora_scale": 0.95,
            "num_outputs": batch_size,
            "aspect_ratio": "2:3",
            "output_format": "png",
            "guidance_scale": 3.5,
            "output_quality": 80,
            "num_inference_steps": 50,
            "disable_safety_checker": True,
        }
    )
    
    # If the model returns multiple outputs
    for idx, file_output in enumerate(output):
        idx = idx + start_idx
        with open(f'{name}_{idx:03d}.png', 'wb') as f:
            f.write(file_output.read())
    
    start_idx += batch_size
```

-----
## Replicate 图像生成器服务

如果你正在探索如何利用 Replicate 把商业想法变为现实，可能性是广阔的。无论你在构建依赖 AI 驱动图像生成、定制嵌入或个性化内容创作的应用，Replicate 都提供了健壮的基础。

对旨在交付实际应用的企业而言，可以采用微服务架构，与 Replicate API 服务无缝集成。该系统通过纳入数据管理、图像生成追踪与护栏（guardrails）等特性增强功能，带来更高的可靠性与可控性。

![图像生成 RESTful API 服务]({{ site.baseurl }}/images/personal-fine-tuning-with-flux/Image_Generation_RESTful_API_Service.jpg)

_图：基于云的架构，利用 Kubernetes 集群管理 API 请求、数据管理与图像存储，同时把 GPU 密集的图像生成卸载给 Replicate API 服务。_

这套实现是一个可部署于任何平台的云端系统，围绕 Kubernetes 集群构建，管理用于数据管理、图像存储与护栏执行的微服务。系统包含面向客户端交互的 API Gateway 和处理请求、与图像生成器服务器通信的 REST API。图像生成器服务器验证输入、施加护栏，并把请求发往外部的 Replicate API 服务——后者负责 GPU 密集的图像生成。生成的图像与元数据存储在云存储和 Postgres 数据库中，支持高效的图像检索与活动追踪。通过把计算密集型任务卸载给 Replicate API，整个系统轻量、省钱且易于维护。

### 用 FastAPI 构建微服务

我们可以把生成流水线集成进 FastAPI REST API：

关键考量：
- **可选参数**：ModelInput 类包含带默认值的可选参数，与 API 文档中的规定一致。
- **动态输入字典**：input_dict 动态创建，只包含非 None 的参数，确保只把实际提供的参数发送给 Replicate API。
- **媒体类型处理**：media_type 根据请求中指定的 output_format 动态设置，让端点能够以期望的格式返回图像。

```python
from fastapi import APIRouter, HTTPException
from fastapi.responses import StreamingResponse, FileResponse, Response
from pydantic import BaseModel
from typing import Optional

from dotenv import load_dotenv
from io import BytesIO

import os
import requests
import replicate

load_dotenv(dotenv_path=".env")

router = APIRouter()

model_name_lookup = {
    "default": "black-forest-labs/flux-schnell",
    "bennycheung": "bennycheung/flux-dev-bennycheung-lora"
}

class ModelInput(BaseModel):
    prompt: str
    image: Optional[str] = None
    mask: Optional[str] = None
    aspect_ratio: Optional[str] = "1:1"
    height: Optional[int] = None
    width: Optional[int] = None
    prompt_strength: Optional[float] = 0.8
    model: Optional[str] = "dev"
    num_outputs: Optional[int] = 1
    num_inference_steps: Optional[int] = 28
    guidance_scale: Optional[float] = 3.0
    seed: Optional[int] = None
    output_format: Optional[str] = "webp"
    output_quality: Optional[int] = 80
    disable_safety_checker: Optional[bool] = False
    go_fast: Optional[bool] = False
    megapixels: Optional[str] = "1"
    lora_scale: Optional[float] = 1.0
    extra_lora: Optional[str] = None
    extra_lora_scale: Optional[float] = 1.0


@router.post("/image/{model_name}")
async def generate_image(input_data: ModelInput, model_name: str):
    try:
        # Use the provided model_name or default to a specific model version
        model_name = model_name_lookup.get(model_name, model_name_lookup["default"])
        
        input_dict = input_data.dict()
        input_dict = {k: v for k, v in input_dict.items() if v is not None}

        output = replicate.run(
            model_name,
            input=input_dict,
        )

        # Fetch the generated image
        image_url = output[0]
        image_response = requests.get(image_url)
        image_response.raise_for_status()

        # Create an in-memory buffer
        image_buffer = BytesIO(image_response.content)
        image_buffer.seek(0)

        # Determine the appropriate media type
        media_type = f"image/{input_data.output_format}"

        # Stream the image back to the client
        return StreamingResponse(image_buffer, media_type=media_type)

    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

这段代码导入必要的 FastAPI 与第三方库，例如定义输入模型的 pydantic、发起 HTTP 请求的 requests，以及运行机器学习模型的 replicate。它还用 dotenv 从 `.env` 文件加载环境变量。`ModelInput` 类是一个 Pydantic 模型，定义图像生成流程期望的输入参数。

`@router.post("/image/{model_name}")` 装饰器定义生成图像的端点。`generate_image` 函数以一个 `ModelInput` 实例为输入，执行以下步骤：

1. 定义用于图像生成的生成式模型 `model_name`。
2. 把输入模型实例转换为字典，过滤掉值为 None 的可选参数。
3. 调用 `replicate.run` 函数，以提供的输入参数执行机器学习模型。
4. 从模型输出中取得生成图像的 URL。
5. 向图像 URL 发起 HTTP GET 请求获取图像内容。
6. 创建内存缓冲区（`BytesIO`）存储图像内容。
7. 根据 output_format 参数确定图像的适当媒体类型。
8. 返回携带图像内容与相应媒体类型的 `StreamingResponse`。

如果图像生成过程中发生任何异常，将抛出状态码 500（内部服务器错误）的 `HTTPException`，异常消息包含在响应详情中。

------
## 结语

FLUX 模型与 Replicate 云平台先进能力的结合，是 AI 驱动图像生成领域的变革性工具。这一组合简化了创建个性化嵌入与微调模型的流程，为广泛的应用提供了效率与易得性。

例如，营销专业人士可以用这些工具为活动设计吸睛的视觉素材，让品牌以最小的努力在拥挤的市场中脱颖而出。同样，艺术家与设计师可以解锁新的创意领域，创作突破传统媒体边界的独特而精细的艺术作品。开发者则可以把这些先进模型集成到自己的 AI 应用中，通过纳入高度定制、极具视觉冲击力的素材来提升用户体验。

此外，FLUX 模型的适应性使它成为探索新颖用例的宝贵资产。它能够生成带有用户特定特征的高质量、精细图像，确保可以满足跨行业的个性化需求。潜在应用超越传统范畴：从建筑可视化和教育内容，到为游戏或虚拟现实创建沉浸式环境，可能性没有边界。

本文概述了有效驾驭这些工具的关键步骤，为希望利用图像生成前沿进展的用户提供了坚实的研究基础。遵循此处详述的最佳实践，用户可以释放 FLUX 模型的全部潜力，把它的能力定制到自己的具体需求上，在各自的领域推动创新。AI 驱动的视觉创意的未来已经到来，FLUX 模型就是你通往无限可能世界的门户。

------
## 参考文献

- Yjg, [What is the Replicate & How to use it?](https://medium.com/@yjg30737/what-is-the-replicate-how-to-use-it-get-the-api-key-proper-model-name-etc-10a2cb983ceb), Medium, 25 Aug 2024.
  - Replicate.ai 是一个简化在云端部署、运行和微调机器学习模型流程的平台。它提供由不同开发者与研究者创建的多种预训练模型，可用于解决图像生成、自然语言处理乃至专门 AI 任务。

- deepfates, [Fine-tune FLUX.1 with your own images](https://replicate.com/blog/fine-tune-flux), Replicate Blog, 14 Aug 2024.

- Dark_infinity, [Understanding Prompting and Captioning for LoRAs](https://civitai.com/articles/8487/understanding-prompting-and-captioning-for-loras), civitai Blog, 30 Oct 2024.
  - LoRA 非常适合微调 Stable Diffusion 模型：在不从零重训的情况下加入专门知识。不过，要真正发挥其潜力，关键是理解 LoRA 如何以及何时影响图像生成。本指南探讨 LoRA 行为的关键侧面，包括它们与潜在噪声空间的交互、为什么它们在某些场景下可能显得不生效，以及用 caption 训练如何影响它们对提示词的响应。

- [Replicate HTTP API documentation](https://replicate.com/docs/reference/http)
  - 基本 Replicate 提供全部 API 管理工具。
  - 在 [api.replicate.com/openapi.json](https://api.replicate.com/openapi.json) 下载 schema

- Replicate Documentation, [black-forest-labs](https://replicate.com/black-forest-labs) / [flux-schnell API Schema](https://replicate.com/black-forest-labs/flux-schnell/api/schema#input-schema)
