---
layout: post
title: "FoodInsight：本地优先架构的边缘 AI 食物监测"
date: 2026-01-12 00:00:00.000000000 -05:00
permalink: foodinsight-edge-ai-food-monitoring
author: Benny Cheung
artwork:
  author: Gemini 3 Pro (Nano Banana Pro)
tags:
  - Edge AI
  - Computer Vision
  - YOLO
  - Food Detection
  - Local-First Architecture
category: post
comments: true
image: images/foodinsight-edge-ai-food-monitoring/cover2x.jpg
images:
  cover: images/foodinsight-edge-ai-food-monitoring/cover2x.jpg
  header: images/foodinsight-edge-ai-food-monitoring/header.jpg
  animated-cover: images/foodinsight-edge-ai-food-monitoring/animated-cover.webp
published: true
series: "FoodInsight"
series_order: 2
---

<!--excerpt.start-->
桌上有什么？这是个简单的问题，但要可靠地回答它，你得亲自走到厨房。FoodInsight 通过将边缘 AI 与本地优先（local-first）架构相结合改变了这一点：把一台 Raspberry Pi 和摄像头变成智能食物监测系统。不依赖云端，没有任何图像离开你的网络，只有对可用食物的实时洞察——由 YOLO11 和 UEC FOOD 100 数据集驱动。
<!--excerpt.end-->

![FoodInsight 主图]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_Hero.png)

_图. FoodInsight 实际运行效果：一台 Raspberry Pi 摄像头监控餐桌，使用 YOLO11 实时检测日式菜肴。每道菜都会获得一个边界框和置信度分数，从米饭（0.94）到天妇罗（0.92），全部在边缘硬件上本地处理。_

在上一篇文章[《You Only Look Once：食物检测的 8 年演进》]({{ site.baseurl }}/from-yolo-v2-to-yolo11-food-detection-evolution)中，我们回顾了从 YOLO v2 到 YOLO11 的非凡历程，并训练了一个在 UEC FOOD 100 数据集上达到 0.79 mAP50 的食物检测模型 [1]。该模型能识别 100 类日式料理，从米饭、味噌汤到天妇罗和寿司。那篇文章的结尾留下了一个设想：如果把这个模型部署到桌面之外，去监测真实餐桌上的真实食物，会怎样？

本文回答了这个问题。我们把上一篇文章中训练的 YOLO11 模型原封不动地部署为 **FoodInsight** [2]——一个完整的边缘 AI 系统：它监测桌上的食物，追踪哪些食物出现或消失，并通过一个简单的手机应用展示当前库存。整套系统的硬件成本不到 140 美元，完全离线运行，所有处理都在本地完成。没有云订阅，没有任何数据离开你的网络，没有隐私顾虑。

![FoodInsight 系统总览]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_System_Overview.png)

_图. FoodInsight 把一台带摄像头的 Raspberry Pi 变成智能食物监测站。边缘 AI 使用 YOLO11 检测食物，用 ByteTrack 追踪库存变化，并通过本地 PWA 提供实时状态。_

---

## 可见性问题

每个共用厨房都在上演同样的故事：有人做好了饭，菜肴摆上餐桌或料理台，其他人却不知道那里有什么。他们要么反复去查看，要么四处打听，要么干脆错过这顿饭。结果就是饭菜凉了、功夫白费，以及那种不知道有什么可吃的挥之不去的不确定感。

这种模式在许多场景中反复出现：

- **家庭厨房**：家人不知道晚饭好了没有，也不知道桌上有哪些菜
- **聚餐**：客人看不到桌子另一头的菜，也不知道还剩哪些选择
- **办公室午餐间**：员工专门走到厨房，只为看看外送的午餐到了没有
- **食堂与自助餐**：就餐者和工作人员往往要等菜品没了才知道需要补货

解决方案看似显而易见：拍张照片分享出去。但照片是静态的，需要有人去拍，几分钟内就会过时，也不会告诉你什么时候发生了变化。

如果监测能够自动、持续且保护隐私地进行呢？

---

## FoodInsight 登场

FoodInsight 就是我们在可见性问题上的答案。它是一个三组件系统，在尊重隐私、完全离线运行的前提下实时监测食物。

### 核心组件

| 组件 | 技术 | 用途 |
|-----------|------------|---------|
| **边缘检测** | Raspberry Pi + YOLO11n [3] | 检测并追踪食物 |
| **本地后端** | FastAPI + SQLite | 存储库存，提供 API |
| **消费端 PWA** | Vue 3 + Vite | 展示当前食物供应情况 |

设计哲学是"本地优先"（local-first）[4]。所有数据都留在你的网络内。边缘设备直接处理视频帧，从不存储或传输图像。只有库存数量会传到后端，而这也只发生在你的本地网络之内。

![FoodInsight 架构]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_Architecture_Diagram.png)

_图. FoodInsight 的本地优先架构：摄像头帧在边缘设备上流经运动检测和 YOLO11n 推理。只有库存增量（数量而非图像）会推送到本地 SQLite 后端。PWA 每 30 秒轮询一次更新。_

### 为什么选择本地优先？

把 FoodInsight 做成本地优先系统并非随意之举，它源自几条约束和原则：

**隐私始于设计（Privacy by Design）**：在共享空间部署食物监测摄像头会引发切实的隐私顾虑。FoodInsight 通过全本地处理、从不存储图像，消除了那种令用户不适的"被监控"感。

**零云端成本**：云后端会带来持续开销，无论是计算、存储还是 API 调用。而本地 SQLite 数据库除了初始硬件之外分文不花。

**可靠性**：断网时本地系统不会瘫痪。对于查看食物供应这样平凡的需求，可靠性比功能更重要。

**简单**：云基础设施需要配置、凭证、计费和持续维护，而本地 SQLite 只需要一个文件。

---

## 边缘端：检测发生的地方

FoodInsight 的核心是运行在 Raspberry Pi 上的边缘检测服务。计算机视觉就是在这里把摄像头帧转化为可操作的库存数据。

### 硬件需求

| 档位 | 硬件 | 成本 | 性能 |
|------|----------|------|-------------|
| **经济型** | Raspberry Pi 4 + Camera Module 3 | 约 110 美元 | 1-3 FPS |
| **标准型** | Raspberry Pi 5 + Camera Module 3 | 约 139 美元 | 4-6 FPS |
| **开发型** | Mac/Linux + 网络摄像头 | - | 45+ FPS |

为什么只用 CPU 而不加 AI 加速器？Hailo-8L 可以把性能提升到 15-25 FPS，但会让硬件成本增加 70 美元。对于监测一张相对静态的餐桌，4-6 FPS 绰绰有余。运动检测进一步降低了平均处理负载，CPU 大部分时间处于空闲。省下的钱换来的是简单：不需要加速器驱动，没有散热管理的复杂性，只有直截了当的 CPU 推理。

开发档位在部署前于台式机或笔记本上运行同样的代码进行测试。这在开发过程中被证明极其宝贵：我们在 M4 Pro Mac 上达到了约 45 FPS，可以快速迭代而无须等待较慢的 Raspberry Pi。

### 检测流水线

每一帧摄像头画面都流经一条精心设计的流水线：

![检测流水线]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_Detection_Pipeline.png)

_图. FoodInsight 检测流水线：30 FPS 的摄像头帧先流经运动检测，由它把守昂贵的 YOLO11n 推理入口。只有检测到运动的帧才会以 4-6 FPS 进入目标检测，接着交给 ByteTrack 做持续跟踪，最后由库存状态管理器把增量推送到后端。_

**运动检测**节省 CPU 周期。摄像头以 30 FPS 采集，但只有当画面发生变化时我们才运行昂贵的 YOLO 推理。一个阈值为 0.008 的简单帧差分算法即可检测运动。没有运动就意味着没有推理，从而大幅降低 CPU 负载。

**YOLO11n 推理**以 NCNN 格式 [5] 运行，专为 ARM CPU 优化。YOLO11n 中的"n"代表"nano"，即最小、最快的变体。在 Raspberry Pi 5 上它可以达到 4-6 FPS，足以监测餐桌这类相对静态的场景。

**ByteTrack** [6] 维护目标在帧间的持续身份。如果没有跟踪，每一帧都会报告"新"检测。ByteTrack 为目标分配持久的 ID，让系统能够判断物品何时被拿走（ID 消失）或何时被放入（出现新 ID）。

**库存状态管理器**衔接检测与后端 API。它维护每个食物类别的本地计数，把每帧的检测结果与之前的状态比较并计算增量。当计数变化时，它批量打包更新（每秒最多一次 API 调用），只把差异推送到后端。这既减少了网络流量，也确保后端始终反映餐桌的当前状态。

### 部署 UEC FOOD 100 模型

FoodInsight 部署的正是我们上一篇文章中训练的那个模型。这个 YOLO11n 模型以 NCNN 格式只有 22MB，小到可以轻松装进 Raspberry Pi，同时识别 UEC FOOD 100 数据集 [1] 中的全部 100 个类别，其中包括各式各样的日式菜肴：

```python
# config/settings.py - UEC FOOD 100 类别
FOOD_CLASSES = [
    "rice", "eels_on_rice", "pilaf", "chicken_and_egg_on_rice",
    "pork_cutlet_on_rice", "beef_curry", "sushi", "chicken_rice",
    "fried_rice", "tempura_bowl", "bibimbap", "toast", "croissant",
    "roll_bread", "raisin_bread", "chip_butty", "hamburger", "pizza",
    "sandwiches", "udon_noodle", "tempura_udon", "soba_noodle",
    "ramen_noodle", "beef_noodle", "tensin_noodle", "fried_noodle",
    "spaghetti", "Japanese_tofu_and_vegetable_chowder", "pork_miso_soup",
    "chinese_soup", "beef_bowl", "kinpira_style_sauteed_burdock",
    "croquette", "grilled_eggplant", "sauteed_vegetables", "chinjaorosu",
    "stew", "oden", "omelet", "ganmodoki", "jiaozi", "stir_fried_pork",
    # ... 全部 100 个类别
]
```

与从行人到红绿灯什么都检测的通用 COCO 预训练模型不同，我们的模型只专注于食物。这种专门化，加上可配置的类别过滤器，大幅减少了场景中非食物物体造成的误检。

### 通过 ROI 遮罩保护隐私

摄像头帧并非每个部分都需要处理。FoodInsight 允许管理员定义一个感兴趣区域（Region of Interest, ROI），把检测限制在特定范围内：

![ROI 配置]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_ROI_Configuration.png)

_图. 管理门户通过简单的拖拽界面配置 ROI。只有定义的区域会被用于食物检测，ROI 之外的区域被遮罩，永远不会被分析或存储。_

这有多重目的：

1. **隐私**：排除行人可能经过的区域
2. **准确性**：减少背景物体带来的误检
3. **性能**：处理的像素更少，推理更快

管理门户运行在 Flask + HTMX [7] 上，而不是扩展 FastAPI。为什么要用另一个框架？在资源受限的边缘设备上 Flask 更轻量，HTMX 无需重型 JavaScript 就能提供动态 UI，而且 MJPEG 视频流在 Flask 中实现起来更简单。该门户只能在本地网络内访问，提供带检测叠加层的实时视频流、展示当前库存，并允许修改配置。

---

## 后端：本地数据存储

后端承担两项职责：接收来自边缘设备的库存更新，并为消费端应用提供 API。它用 FastAPI [8] 和 SQLite [9] 构建，体现了本地优先的理念。

### SQLite：完美的本地数据库

为什么用 SQLite 而不是 Firebase 或其他云数据库？对于单设备原型来说，云同步徒增复杂而无收益。SQLite 以零云端成本、零配置和完全离线运行提供了我们所需的一切。代价是没有多设备同步和异地备份，但对一个厨房原型而言，这两点都无关紧要。

SQLite 带来了：

- **零配置**：无需运行服务器，无需管理凭证
- **单文件**：整个数据库就在 `data/foodinsight.db` 中
- **符合 ACID**：即使写入中途断电也依然可靠
- **快速**：对如此简单的数据模型，查询耗时仅为亚毫秒级

数据库模式记录了库存状态和变更历史：

```sql
-- 当前库存状态
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    item_name TEXT NOT NULL UNIQUE,
    display_name TEXT,
    count INTEGER NOT NULL DEFAULT 0,
    confidence REAL DEFAULT 0.0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 检测事件日志
CREATE TABLE events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    event_type TEXT NOT NULL,  -- 'FOOD_TAKEN' | 'FOOD_ADDED'
    item_name TEXT NOT NULL,
    count_before INTEGER NOT NULL,
    count_after INTEGER NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### API 设计

API 遵循一个简单的模式：公开端点用于读取，需认证的端点用于管理。

| 端点 | 方法 | 认证 | 用途 |
|----------|--------|------|---------|
| `/inventory` | GET | 无 | 当前食物库存 |
| `/inventory/update` | POST | 无 | 边缘设备推送更新 |
| `/inventory/events` | GET | 无 | 近期变更历史 |
| `/admin/status` | GET | Basic | 系统状态 |
| `/admin/config` | PUT | Basic | 更新配置 |

公开的库存端点有意不要求认证。本地网络中的任何人都可以查看有哪些食物。这消除了使用摩擦，也符合"一目了然"的设计目标。

### 响应格式

一个典型的库存响应反映了 UEC FOOD 100 的类别：

```json
{
  "device_id": "kitchen-001",
  "location": "Dining Table",
  "last_updated": "2026-01-12T14:30:00",
  "items": [
    {"item_name": "rice", "display_name": "Rice", "count": 3, "confidence": 0.94},
    {"item_name": "pork_miso_soup", "display_name": "Pork Miso Soup", "count": 2, "confidence": 0.89},
    {"item_name": "grilled_salmon", "display_name": "Grilled Salmon", "count": 1, "confidence": 0.91},
    {"item_name": "tempura", "display_name": "Tempura", "count": 0, "confidence": 0.0},
    {"item_name": "sushi", "display_name": "Sushi", "count": 4, "confidence": 0.87}
  ]
}
```

数量为 0 的条目仍保留在响应中，这样应用可以显示"已吃完"，而不是把条目完全隐藏。这让就餐者即使错过了，也知道曾经供应过什么。

## 消费端体验：PWA

最后一块拼图是用户真正看到的东西：一个渐进式 Web 应用（Progressive Web App, PWA），可在任何设备上展示当前的食物供应情况。

### 设计原则

| 原则 | 实现 |
|-----------|----------------|
| **一目了然** | 大卡片、清晰的计数、醒目的库存状态 |
| **零门槛** | 无需登录、无需注册，打开即看 |
| **移动优先** | 触控友好，适配任意屏幕尺寸 |
| **快速** | 2 秒内加载完成，积极利用缓存 |
| **可安装** | 添加到主屏幕，获得接近原生应用的体验 |

### 技术选型

PWA 使用 Vue 3 [10]（Composition API）、Pinia 做状态管理、Tailwind CSS 做样式，Vite [11] 负责构建与开发。整个打包产物经 gzip 压缩后约 33KB。

```typescript
// stores/inventory.ts
export const useInventoryStore = defineStore('inventory', {
  state: () => ({
    items: [] as FoodItem[],
    lastUpdated: null as Date | null,
    loading: false
  }),

  actions: {
    async fetchInventory() {
      this.loading = true
      try {
        const response = await fetch('/api/inventory')
        const data = await response.json()
        this.items = data.items.map(item => ({
          ...item,
          inStock: item.count > 0
        }))
        this.lastUpdated = new Date(data.last_updated)
      } finally {
        this.loading = false
      }
    }
  }
})
```

### 自动刷新

PWA 每 30 秒轮询一次更新。这个间隔在数据新鲜度和移动设备续航之间做了权衡。界面上可见的"最后更新"时间戳让用户知道数据有多新。

为什么用轮询而不是 WebSocket 实时更新？轮询的实现和调试更简单，与 CDN 缓存配合良好，也不需要维护持久连接。代价是用户最多可能看到 30 秒前的旧数据，但对于查看食物供应来说，这个延迟可以接受。如果延迟真的成为问题，实时 WebSocket 更新是未来阶段的自然扩展。

![FoodInsight PWA]({{ site.baseurl }}/images/foodinsight-edge-ai-food-monitoring/FoodInsight_PWA_Screenshot.png)

_图. FoodInsight PWA 以卡片网格的形式展示当前餐食状态，涵盖 UEC FOOD 100 的类别。绿色表示有货并显示数量，灰色表示已吃完。该界面在手机、平板和桌面上都能使用。_

---

## 部署：从开发到厨房

FoodInsight 运行在三种环境中：在 Mac/Linux 桌面上开发，在目标 Raspberry Pi 上测试，在实际厨房或茶水间中投产。

### 开发模式

为了快速迭代，我们把所有东西都跑在开发机上：

```bash
# 终端 1：实时摄像头的边缘检测
python run_dev.py --config dev_config.json

# 终端 2：后端 API
cd server && uvicorn app.main:app --port 8000

# 终端 3：PWA 开发服务器
cd app && bun run dev
```

`dev_config.json` 文件指定了便于开发的设置，指向用 UEC FOOD 100 训练的模型：

```json
{
  "machine_id": "dev-mac",
  "model_path": "models/yolo11n_food100.ncnn",
  "camera_index": 0,
  "admin_port": 8080,
  "api_url": "http://localhost:8000",
  "allowed_classes": ["rice", "pork_miso_soup", "sushi", "ramen_noodle",
                      "tempura", "grilled_salmon", "beef_curry", "udon_noodle"],
  "motion_threshold": 0.008
}
```

`model_path` 指向上一篇文章训练后导出的 NCNN 模型。`allowed_classes` 可以过滤为你预期会出现的特定类别，也可以留空以检测全部 100 个食物类别。

### 生产部署

在 Raspberry Pi 上，一切都以 systemd 服务的方式运行：

```bash
# 安装边缘检测服务
sudo cp systemd/foodinsight-edge.service /etc/systemd/system/
sudo systemctl enable foodinsight-edge
sudo systemctl start foodinsight-edge

# 安装后端服务
sudo cp systemd/foodinsight-api.service /etc/systemd/system/
sudo systemctl enable foodinsight-api
sudo systemctl start foodinsight-api
```

这些服务会在开机和故障后自动重启。日志流入 journald 供调试：

```bash
# 监控边缘检测日志
journalctl -u foodinsight-edge -f

# 监控 API 日志
journalctl -u foodinsight-api -f
```

### PWA 托管选项

PWA 有两种托管方式：

**本地托管**把一切都保留在本地：

```bash
cd app && bun run build
python -m http.server 3000 -d dist
```

**云托管**通过 Cloudflare Pages 实现从任意位置访问，而后端仍留在本地：

```bash
cd app && bun run build
wrangler pages deploy dist --project-name=foodinsight
```

采用云托管时，PWA 从 CDN 加载，但 API 调用仍然指向你的本地后端。用户需要 VPN 或本地网络访问才能看到真实数据。

---

## 性能与结果

部署之后，FoodInsight 达到了 PRD [2] 中定义的全部目标：

| 目标 | 指标 | 实际达成 |
|------|--------|----------|
| 模型精度（mAP50） | ≥75% | 79%（来自训练） |
| 检测延迟 | 每帧 <500ms | 170-250ms |
| API 响应时间 | <200ms | <50ms（SQLite 就是快） |
| PWA 加载时间 | <2 秒 | 约 1.5 秒 |
| 硬件成本 | <$140 | $139（Raspberry Pi 5 档位） |
| 云端成本 | $0/月 | $0/月 |
| 隐私 | 不存储图像 | 仅元数据 |

训练时达到的 0.79 mAP50 在真实部署中依然站得住脚。模型能可靠地检测米饭、味噌汤、天妇罗和寿司等常见日式菜肴。对于较少出现的类别或摆盘不寻常的情况，性能会平缓下降，但这个系统的价值在于可见性，而非完美精度。知道"桌上可能有寿司"，远比一无所知有用得多。

系统追踪物品何时出现或消失，并在 30 秒轮询间隔内更新 PWA。实践中，170-250ms 的检测延迟意味着，一道菜被放上或拿走后的几秒内，库存更新就能到达后端。

---

## 经验教训

构建 FoodInsight 让我们学到了几条关于边缘 AI 部署的经验：

**1. 运动检测必不可少。** 没有它，Raspberry Pi 的 CPU 就会因以摄像头帧率运行 YOLO 推理而饱和。运动检测让负载在画面没有变化时降到接近零。

**2. 模型导出格式很重要。** 在 ARM CPU 上，NCNN 格式的运行速度明显快于 ONNX。导出步骤（`yolo export format=ncnn`）轻而易举，却对边缘性能至关重要。

**3. ByteTrack 需要调参。** 默认的跟踪器配置会导致 ID 闪烁。在跟踪器配置中加入 `fuse_score: true` 后，目标的持续跟踪趋于稳定。

**4. SQLite 被低估了。** 我们起初考虑用 Firebase 来实现"真正的"持久化，事实证明，对单设备应用而言 SQLite 更简单、更快、也更可靠。

**5. PWA 胜过原生应用。** 开发原生 iOS 和 Android 应用需要各自的代码库、应用商店审核和持续维护，而 PWA 用一套代码库就能在任何地方运行。

---

## 未来方向

现在这个版本的 FoodInsight 解决了可见性问题。以下几项扩展可以让它更有价值：

**营养追踪**：把检测到的食物接入营养数据库，估算份量大小，自动把用餐记录写入健康类应用。

**保质期感知**：追踪物品出现的时间，在存放过久时发出提醒，通过突出"尽快食用"的物品来减少食物浪费。

**多设备同步**：用支持同步的数据库（如 LiteFS 或 CRDT）替换 SQLite，让多个厨房点位汇入同一个仪表盘。

**WebSocket 实时更新**：用 WebSocket 连接消除轮询延迟，实时展示库存变化。

**定制模型训练**：用实际部署环境拍摄的照片进行训练，提高对高频出现的特定食物的检测精度。

---

## 结语

FoodInsight 证明，只要具备基础开发技能和普通硬件，实用的边缘 AI 触手可及。YOLO11、Raspberry Pi 与本地优先架构的组合，构建出了一个几年前还无法以可负担成本实现的系统。

这个项目为一段始于 2018 年 [12] 的旅程画上了句号——那一年我们首次在 UEC FOOD 100 数据集上训练 YOLO v2。八年之后，同一个数据集支撑起一套运行在 139 美元设备上的完整边缘 AI 系统。模型更小、更快、更准，部署更简单，成果也更有用。

核心洞见是：有用的 AI 并不需要云端规模。一台 139 美元的边缘设备运行一个 22MB 的模型，就能解决日常生活中真实而重要的问题。没有订阅费，没有数据离开你的网络，没有隐私顾虑。

我们从一个简单的问题出发："桌上有什么？"构建这个答案需要计算机视觉、目标跟踪、API 设计和 Web 开发，但结果优雅而简单：把摄像头对准食物，看一眼手机，就知道有什么可吃的。

代码已在 [github.com/bennycheung/FoodInsight](https://github.com/bennycheung/FoodInsight) [2] 开源。动手做一个属于你的吧，把它适配到你的厨房、餐厅或办公室午餐区。可见性问题无处不在，如今解决方案也一样。

---

## 参考文献

[1] Matsuda, Y., Hoashi, H., and Yanai, K. [*Recognition of Multiple-Food Images by Detecting Candidate Regions*（通过检测候选区域识别多种食物图像）](http://foodcam.mobi/dataset100.html). IEEE ICME, 2012.
  - 来自电气通信大学（University of Electro-Communications）的 UEC FOOD 100 数据集，包含 100 个日式食物类别、共 14,361 张图像。

[2] Benny Cheung. [*FoodInsight Project*（FoodInsight 项目）](https://github.com/bennycheung/FoodInsight). GitHub, 2026.
  - FoodInsight 完整源代码，包括边缘检测、后端 API 和消费端 PWA。

[3] Ultralytics. [*YOLO11 Documentation*（YOLO11 文档）](https://docs.ultralytics.com/). Ultralytics, 2024.
  - YOLO11 目标检测的全面文档，包括模型变体、训练和部署选项。

[4] Kleppmann, M., Wiggins, A., van Hardenberg, P., and McGranaghan, M. [*Local-First Software*（本地优先软件）](https://www.inkandswitch.com/local-first/). Ink & Switch, 2019.
  - 关于本地优先软件原则的奠基性文章：数据所有权、离线能力和默认保护隐私。

[5] Tencent. [*NCNN: High-Performance Neural Network Inference Framework*（NCNN：高性能神经网络推理框架）](https://github.com/Tencent/ncnn). GitHub, 2017.
  - 面向移动与嵌入式平台优化的神经网络推理，让 ARM 设备能够进行高效的基于 CPU 的推理。

[6] Zhang, Y., Sun, P., Jiang, Y., et al. [*ByteTrack: Multi-Object Tracking by Associating Every Detection Box*（ByteTrack：通过关联每个检测框进行多目标跟踪）](https://arxiv.org/abs/2110.06864). ECCV, 2022.
  - 高性能多目标跟踪方法，通过跨帧关联检测框实现目标的持续身份识别。

[7] Big Sky Software. [*HTMX*](https://htmx.org/). HTMX.org, 2020.
  - 让你直接在 HTML 属性中使用 AJAX、CSS 过渡、WebSocket 和服务器推送事件（Server-Sent Events）的库。

[8] Ramirez, S. [*FastAPI*](https://fastapi.tiangolo.com/). FastAPI, 2018.
  - 用于构建 API 的现代 Python Web 框架，带自动 OpenAPI 文档和 Pydantic 校验。

[9] SQLite Consortium. [*SQLite*](https://sqlite.org/). SQLite.org, 2000.
  - 自包含、无服务器的 SQL 数据库引擎，在全球数十亿设备中使用。

[10] Vue.js. [*Vue 3 Documentation*（Vue 3 文档）](https://vuejs.org/). Vue.js, 2020.
  - 用于构建用户界面的渐进式 JavaScript 框架，以 Composition API 带来更好的代码组织。

[11] Vite. [*Vite Documentation*（Vite 文档）](https://vitejs.dev/). Vite, 2020.
  - 下一代前端构建工具，提供极速开发服务器和优化的生产构建。

[12] Benny Cheung. [*YOLO for Real-Time Food Detection*（YOLO 实时食物检测）]({{ site.baseurl }}/yolo-for-real-time-food-detection). Benny's Mind Hack, 2018.
  - 记录使用 YOLO v2 与 Darknet 进行食物检测的原始博客文章，八年前的食物检测之旅正是从这里开始的。

---

**食物检测系列：**

* [YOLO 实时食物检测]({{ site.baseurl }}/yolo-for-real-time-food-detection)（2018 年——一切的起点）
* [You Only Look Once：食物检测的 8 年演进]({{ site.baseurl }}/from-yolo-v2-to-yolo11-food-detection-evolution)（训练模型）
* **>>** [FoodInsight：本地优先架构的边缘 AI 食物监测]({{ site.baseurl }}/foodinsight-edge-ai-food-monitoring)（部署系统）