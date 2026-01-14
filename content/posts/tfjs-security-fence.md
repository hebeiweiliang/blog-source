
---
title: "无需买硬件！用 TensorFlow.js + 旧手机搭建 0 成本“AI 电子围栏”安防系统"
date: 2026-01-14T10:00:00+08:00
draft: false
tags: ["TensorFlow.js", "AI", "Edge Computing", "安防", "JavaScript"]
categories: ["AI实战 Lab"]
author: "InfiniteWei"
description: "家里有闲置的旧手机或笔记本？别卖废品！本文教你使用 TensorFlow.js 和 COCO-SSD 模型，3分钟将其变身为具备“电子围栏”功能的智能安防摄像头。纯前端运行，数据隐私 100% 安全。"
cover:
    image: "https://images.unsplash.com/photo-1550751827-4bd374c3f58b?q=80&w=2070&auto=format&fit=crop"
    alt: "AI Security System"
    caption: "纯前端实现的 AI 监控"
---

在这个万物互联的时代，一套智能安防系统动辄几百上千元。但作为开发者，我们完全可以用代码“白嫖”现有的算力。

今天，我将带大家用 **TensorFlow.js** 和轻量级目标检测模型 **COCO-SSD**，把你的旧手机或闲置笔记本变成一个**带有“电子围栏”报警功能的 AI 监控哨**。

最重要的是：**它完全在本地运行，不需要上传视频流，0 延迟，且 100% 保护你的隐私。**

## 🚀 在线体验 (Live Demo)

别光听我说，直接体验一下。请允许浏览器**开启摄像头权限**（数据仅在本地处理，绝不会上传）。

> **⚠️ 注意：** 建议使用 PC 或 Android 手机 Chrome 浏览器体验。保持静止，尝试用手或身体进入画面中央的红框区域。

<div style="background: #f5f5f5; border-radius: 10px; padding: 20px; text-align: center; border: 2px dashed #333; margin: 20px 0;">
    <h3>🚧 AI 电子围栏实验室</h3>
    <p>当前模型：COCO-SSD (Lite)</p>
    <br/>
    <a href="/apps/fence/index.html" target="_blank" style="background: #007bff; color: white; padding: 12px 25px; text-decoration: none; border-radius: 5px; font-weight: bold; font-size: 16px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); display: inline-block;">
        👉 点击全屏运行 AI 电子围栏
    </a>
    <p style="font-size: 12px; color: #666; margin-top: 10px;">检测到 "Person" 进入红框区域将触发红色警报</p>
</div>

---

## 🛠️ 技术原理

为什么我们可以通过浏览器实现实时监控？核心技术栈如下：

1.  **TensorFlow.js:** Google 开发的可以在浏览器运行的机器学习库，支持 WebGL/WebGPU 加速，利用显卡进行矩阵运算。
2.  **COCO-SSD (MobileNet):** 一个预训练好的轻量级对象检测模型。它能识别 80 种常见物体（人、猫、狗、手机等），虽然精度不如 YOLOv8，但在网页端能跑出 30+ FPS 的流畅度。
3.  **Canvas API:** 用于绘制那个“红色的电子围栏”以及检测框。

### 核心逻辑图解

<div class="mermaid">
graph LR
    A[摄像头流 WebCam] --> B(视频帧 Frame);
    B --> C{COCO-SSD 模型};
    C -->|输出| D[预测结果 Objects];
    D --> E{逻辑判断};
    E -->|物体坐标 ∈ 围栏区域| F[🚨 触发报警];
    E -->|物体在区域外| G[安全监控中];
    style F fill:#f96,stroke:#333,stroke-width:2px
    style G fill:#bbf,stroke:#333,stroke-width:2px
</div>

## 💻 核心代码实现

不需要复杂的 Python 环境配置，只要一个 HTML 文件即可。

### 1. 引入模型库

我们直接使用 CDN 引入（或者如我一样下载到本地以提高加载速度）：

```html
<script src="[https://cdn.jsdelivr.net/npm/@tensorflow/tfjs](https://cdn.jsdelivr.net/npm/@tensorflow/tfjs)"></script>
<script src="[https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd](https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd)"></script>

```

### 2. 定义“电子围栏”

电子围栏本质上就是画面中的一个坐标区域 `(x, y, width, height)`。

```javascript
// 定义围栏区域 (画面中央的一个矩形)
const fence = {
    x: 100,
    y: 100,
    width: 440,
    height: 280
};

function drawFence(ctx) {
    ctx.strokeStyle = 'red';
    ctx.lineWidth = 4;
    ctx.strokeRect(fence.x, fence.y, fence.width, fence.height);
    
    ctx.fillStyle = 'rgba(255, 0, 0, 0.1)';
    ctx.fillRect(fence.x, fence.y, fence.width, fence.height);
}

```

### 3. 碰撞检测（报警算法）

这是本系统的灵魂。我们需要判断模型检测到的**物体边界框（Bounding Box）是否与我们的围栏**发生了重叠。

这里我写了一个简单的判断逻辑：只要物体的中心点进入围栏，就算入侵。

```javascript
model.detect(video).then(predictions => {
    predictions.forEach(prediction => {
        // 只针对 'person' 进行报警，你可以改成 'cat' 防止猫主子偷吃
        if (prediction.class === 'person') {
            const [x, y, width, height] = prediction.bbox;
            
            // 计算物体中心点
            const centerX = x + width / 2;
            const centerY = y + height / 2;

            // 判断是否在围栏内
            if (centerX > fence.x && 
                centerX < fence.x + fence.width && 
                centerY > fence.y && 
                centerY < fence.y + fence.height) {
                
                triggerAlarm(); // 触发报警函数
            }
        }
    });
});

```

## 💡 扩展应用场景

有了这个原型，你完全可以根据需求进行魔改：

* **防猫咪上桌：** 将检测对象改为 `cat`，把围栏设置在餐桌区域，连上蜂鸣器，打造“自动驱猫神器”。
* **危险区域管控：** 工地或厨房重地，识别到人进入即发出语音提示。
* **快递监控：** 识别 `box` 或 `backpack`，当快递员把包裹放在门口特定区域时，通过 Telegram Bot 发送通知给你的手机。

## ⚠️ 隐私与性能说明

很多读者担心：“网页开摄像头会不会泄露我的生活画面？”

请放心，**TensorFlow.js 的所有计算均在 Client 端（你的浏览器）完成**。你可以拔掉网线（离线）运行本页面的 Demo，它依然能工作。这就是端侧 AI (Edge AI) 的最大魅力——数据不出域，隐私更安全。

---

**如果你觉得这个项目有趣，欢迎收藏本站！**
下一期，我将教大家如何把这个检测结果通过 WebHook 对接到飞书/钉钉机器人，实现真正的远程推送。

<div style="margin-top:50px; padding: 20px; background: #fafafa; border: 1px solid #eee; text-align:center; color:#999;">
(AdSense 广告位 - 建议上线后在此处插入代码)
</div>

<script src="https://www.google.com/search?q=https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>

<script>
mermaid.initialize({ startOnLoad: true });
</script>
