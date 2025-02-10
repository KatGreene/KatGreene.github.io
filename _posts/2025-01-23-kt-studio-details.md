---
layout:       post
title:        "KT Studio 开发记录（TA技术美术）"
author:       "KatGreene"
header-style: text
catalog:      true
tags:
    - KT Studio
    - TA
---

> 本文记录我在KT Studio里做了什么画面效果、渲染优化等。**欢迎前往游戏内实机体验~**
> 
> 如果这些简短的记录能为你提供灵感，我将十分甚至九分高兴！
> 
> 同时也欢迎<a href = "https://katgreene.github.io/about/">联系我</a>进行深入♂交流。

## 一、角色渲染部分

<img src="/img/404-bg.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 好看！*

[//]: # (<div style="height: 15px;"></div>)

### 1. 基础角色效果

参考：

<a href = "https://www.bilibili.com/video/BV1kBBKYRE6Q/">【Unity/虚幻5/Blender】3种引擎 绝区零风格 卡通渲染 星见雅 完整流程</a>

<a href = "https://www.bilibili.com/video/BV1CN411C7qx">【Unity/虚幻5/Blender】3种引擎 崩坏: 星穹铁道风格 卡通渲染 从球谐光照到眉毛透过刘海 完整流程</a>

<a href = "https://www.bilibili.com/video/BV1HN4y1X7uS">【Unity开源项目】纳西妲仿原神渲染预设工程</a>

<a href = "https://www.bilibili.com/video/BV1Qk4y1g7Fe">【Unity URP】仿星穹铁道渲染，代码看简介</a>

基于《崩坏：星穹铁道》的角色渲染框架，复现原游戏内的基础卡通角色渲染。

这些效果包括：
- 对角色进行兰伯特**二分阴影**处理，并采样Ramp贴图；
- 脸部阴影的**SDF**阴影处理、脖子部位的**阴影区**处理；
- 复现阴影分界线**偏移**与**固定阴影区**；
- 复现衣物的**金属高光和非金属高光**效果；
- 后处理**Bloom**效果与**色调映射**；
- 角色法线外扩**描边的平滑与粗细偏移**；
- 角色**眼睛眉毛透过刘海效果**，并且兼容多角色；
- 角色表情贴图的使用；
- ······

### 2. 进阶角色效果

在《崩坏：星穹铁道》的角色渲染框架之上，提升了许多其它效果。

📝**方案设计原则：**
- 🚫避免重新手动打直UV；
- 🚫避免重绘基础色贴图；
- 尽可能减少手动绘制Mask贴图；
- 尽可能兼容原来的光照贴图、顶点色、LUT等的格式。

#### **头发⭐**

**i. 仿手绘动态高光：**

- 采用动态的仿手绘动态高光方案：高光**随着视角运动**，并且相比直接画高光贴图拥有**更高的清晰度**；
- 高光多方向适应：同一个Mesh内可以拥有**多个高光束**，并拥有不同的方向和基础位置；当角色头骨旋转时，高光偏移方向也能够正确跟随；
- 高光**抗变形**：将动态高光与物体法线解耦，防止物体运动变形时导致的高光分裂与变形。

<img src="/img/ta/blog-ta-2025-01-30 160113.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色头发高光总览*

<div style="height: 15px;"></div>

**ii. 高光抖动与笔触：**
- 根据高光方向**自动计算抖动采样UV**，保证高光**抖动方向正确**；

**iii. 次高光：**
- 在基础高光（高频、低频高光）之下增加**仿手绘次高光**，一般为暗色，对比突出主高光；
- 采样适当的混合方式，避免颜色变脏变灰。

**iv. 高光流色：**
- 仿照原画，实现高光**颜色渐变流动**效果，并且随时间、视角变化。

**v. 阴影笔触：**
- 使用抖动贴图采样的另外通道，单独控制**漫射分界线**偏移与软硬的抖动。

<img src="/img/ta/hair-jitter.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色头发笔触总览*

<div style="height: 15px;"></div>

**vi. 制作与对齐：**
- 编写Blender脚本，加速制作流程，其中辅助UV可自动生成，无需重新打直主UV；
- 对齐Blender与Unity内的高光效果，方便DCC内LookDev。

<div style="height: 15px;"></div>

#### **眼睛⭐**
- 融合**视差方案**，搭配**一张贴图与一次采样**，实现自定义的丰富眼睛效果；
- 动态效果包括：①**色散流动层**效果、②**闪烁流动层**效果；
- 效果会随时间和视角变换**整体旋转**；
- 无需在DCC绘制任何Mask，在引擎中可实时调整位置、大小等参数；
- 配合不同的特效贴图，可以实现其它更多效果，如星星月亮银河、烟花流云彩蝶等等。

<img src="/img/ta/blog-ta-2025-01-30 160358.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

<div style="display: flex; justify-content: space-around;">

<img src="/img/ta/blog-ta-2025-02-10 095901.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">

<img src="/img/ta/blog-ta-2025-02-10 095831.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">

</div>

*- 角色动态眼睛总览*

<div style="height: 15px;"></div>

#### 阴影
- **接受场景阴影**：接受场景中一定范围内的物体投影；
- 排除角色**自投影**：去除角色往自己身上投射的丑陋阴影（阴影空间偏移方法，暂未采用Per-Object Shadow）；
- 屏幕空间角色**自投影**：改为使用屏幕空间深度方法，绘制角色自投影；
- **刘海投影**：使用额外深度方法，绘制清晰且形状稳定的刘海投影；除此之外，还新增了仿原画的刘海阴影膨胀效果。
  
<img src="/img/ta/blog-ta-2025-01-30 155717.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色阴影总览*

<div style="height: 15px;"></div>

#### 法线
- 头部使用法线**球面化**，并在一定范围外混合**原法线**，保持长发区的法线正确；
- 眼部使用法线**平面化**，适配眼部在背光阴影时的亮度变化，防止阴影区眼白过亮导致的阴间效果；
- 计划改为在DCC软件中传递，可靠性更高且节省性能。

<img src="/img/ta/blog-ta-2025-01-30 155858.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色法线总览*

<div style="height: 15px;"></div>

#### 边缘光
- 屏幕空间边缘光：适配**FOV、屏幕尺寸、摄像机距离**。当这些参数变化时，边缘光的区域能在一定范围内自动缩放，防止边缘光区域过大或过小；
- 逆光增强：当**逆光观察**角色时，屏幕空间边缘光会适当**增强**；
- 待优化：平面反射的相机视角下，V向量不准确导致逆光增强异常。

<div style="display: flex; justify-content: space-around;">

<img src="/img/ta/blog-ta-2025-01-30 155954.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">

<img src="/img/ta/blog-ta-2025-01-30 160010.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">

</div>

*- 角色边缘光总览*

<div style="height: 15px;"></div>

#### 丝袜
- 重写了丝袜渲染效果，采用**高光油丝袜**的渲染思路；
- 丝袜的高光可以跟随视角和灯光旋转，并且有一定的横向抖动。

<img src="/img/ta/blog-ta-2025-01-30 160311.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色丝袜总览*

<div style="height: 15px;"></div>

#### 皮质高光
- 新增**皮质高光**效果，同时兼容原来的所有高光。

<img src="/img/ta/blog-ta-2025-01-30 161340.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 角色皮质高光总览*

<div style="height: 15px;"></div>

#### 阴影内效果
- 丰富当角色处在阴影内的效果，包括阴影内较柔和的次二分阴影、更加柔和的高光。

<div style="display: flex; justify-content: space-around;">

<img src="/img/ta/blog-ta-2025-02-03 155307.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">

<img src="/img/ta/blog-ta-2025-02-03 155323.jpg" alt="KT Studio角色展示" style="width:49%; height:auto; border-radius: 8px">
</div>

*- 角色阴影内效果总览*

<div style="height: 15px;"></div>

#### 多光源
- 角色能够接受场景主光和额外光源的颜色；
- 角色同一时间内只接受**唯一主光**产生的阴影，其它光源均不造成任何阴影（仅随距离进行亮度衰减）；
- 待优化：灯光染色导致贴图颜色变脏发灰，需要优化染色结果的饱和度范围。

<div style="height: 15px;"></div>

## 二、场景渲染部分

<img src="/img/home-bg-silwolf-1.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 高度定制化渲染的场景*

[//]: # (<div style="height: 15px;"></div>)

### 1. 基础场景效果

基于《崩坏：星穹铁道》的场景渲染框架，复现原游戏内的基础场景渲染渲染。

这些效果包括：
- 基于原来的基础色、金属度、法线、光滑度贴图，还原基础的PBR渲染效果；
- 融合基础的深度雾与高度雾；
- ······

### 2. 进阶场景效果

在《崩坏：星穹铁道》的场景渲染框架之上，提升了许多其它效果。

这些效果包括：

#### 天空盒⭐
- 低成本大气散射：实时**快速拟合Mie散射**部分；
- 地平线遮挡效果：落日时增加地平线对散射的**遮挡斜切**效果；
- 低成本**末地光柱**效果：仿照Minecraft主流光影在末地的渲染效果，并采用低成本的方法复现，无需Ray marching；
- 平面映射**焦散云**：天空采样平面映射后的焦散贴图，类似卷云或极光效果；
- 星星点点：流动的**星光效果**。

[//]: # (<div style="display: flex; justify-content: space-around;">)

<img src="/img/ta/blog-ta-2025-01-30 160602.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

<img src="/img/ta/blog-ta-2025-01-30 160641.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

[//]: # (</div>)

*- 程序化天空盒总览*

<div style="height: 15px;"></div>

#### 环境光⭐
- 低成本环境光漫射：基于**法线与光照方向**，采样环境漫射颜色Ramp图，代替球谐光照方案；同时，这张Ramp图也会控制环境光照的**染色强度**，无需动态生成Ramp图；
- 低成本环境光镜面反射：基于**反射向量方向与直接高光结果**，采样环境高光颜色Ramp图，代替采样天空球CubeMap的方案；同样的，这张Ramp图也无需动态生成；

<img src="/img/ta/blog-ta-2025-01-30 160836.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 低成本环境光总览*

<div style="height: 15px;"></div>

#### 平面反射
- 在角色展示界面与水面处加入平面反射，采用直接反转相机的方案，并斜截视锥体，最终在屏幕空间中采样。

#### 水面渲染
- 水面反射：ULTRA画质下启用**平面反射**，其它画质下采样静态环境捕获**CubeMap**并融合灯光颜色；
- 水面焦散旋转：根据**世界位置与灯光方向**采样焦散，使其能够随灯光方向变化；
- 其它水面效果：法线扰动、竖直方向散射、岸边浪、水面与水下阴影、折射等等。

<img src="/img/ta/blog-ta-2025-01-30 161007.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 水面渲染总览*

<div style="height: 15px;"></div>

#### 体积光（待优化）
- 非常耗费性能的体积光，采用主流方案并且还未优化。

#### 场景深度描边
- 采用主流深度描边方案，提升场景手绘感效果。

### 3. 音乐可视化

#### FDTD贴图生成
- 采用Compute Shader，基于传统的FDTD法模拟音乐波动和扩散效果；
- 一共分为五个源激励，由玩家与音乐控制，分别代表五个音乐可视化区。

#### UV采样
- 使用专用的UV采样此FDTD贴图，产生音乐波动效果与旋律弦效果；
- 同时接受C#程序端传入的信息，跟随音乐进行闪烁。

<img src="/img/ta/bg-music-01.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 音乐可视化总览*

<div style="height: 15px;"></div>

## 三、性能测试与优化

### 1. 性能分级与目标

#### 性能分级

部分参与分级的特性：渲染分辨率缩放、阴影分辨率、体积光、场景描边、Depth Prepass、MSAA

  | 等级    | 分辨率缩放 | 阴影分辨率 | 体积光 | 场景描边 | Depth Prepass | MSAA |
    |-------|-------|-------|-----|------|---------------|------|
  | ULTRA | 1.0   | 4096  | 开   | 开    | 开             | 4x   |
  | HIGH  | 1.0   | 4096  | 关   | 开    | 开             | 4x   |
  | MID   | 0.95  | 2048  | 关   | 开    | 开             | 关    |
  | LOW   | 0.85  | 2048  | 关   | 关    | 关             | 关    |

#### 目标平台
部分参与测试的指标：平均FPS、1%LowFPS、平均功耗、峰值功耗、平均GPU负载

- A档：中高端PC (RTX 3080Laptop)  —— ULTRA画质  3840x2160
- B档：中高端安卓手机 (Adreno 750) —— HIGH画质   2340x1080
- C档：中低端安卓平板 (Adreno 650) —— MID画质    2800x1840

  | 档位 | 平均FPS  | 1%LowFPS | 平均功耗 | 峰值功耗 | 平均GPU负载 |
      |----|--------|----------|------|------|---------|
  | A档 | &gt;60 | &gt;55   | -    | -    | -       |
  | B档 | &gt;55 | &gt;30   | <5W  | <10W | <80%    |
  | C档 | &gt;45 | &gt;25   | <4W  | <8W  | <80%    |

### 2. 实机测试

#### 测试场景
- ①号 基准场景：默认场景 —— 面数：-；场景物体数：-；阴影级联数：-；动态光源数：1；平面反射：全分辨率；
- ②号 标杆场景：空间站 —— 面数：-；场景物体数：-；阴影级联数：-；动态光源数：1；

#### 测试结果
每轮跑测10分钟，之后散热10分钟，再进行下一轮；共进行3轮。

- A档：中高端PC —— ULTRA画质

| 场景 | 平均FPS | 1%LowFPS | 平均功耗 | 峰值功耗 | 平均GPU负载 |
      |----|-------|----------|------|------|---------|
| ①号 |       |          |      |      |         |
| ②号 |       |          |      |      |         |

<div style="height: 15px;"></div>

- B档：中高端安卓手机 —— HIGH画质

| 场景 | 平均FPS | 1%LowFPS | 平均功耗 | 峰值功耗 | 平均GPU负载 |
      |----|-------|----------|------|------|---------|
| ①号 |       |          |      |      |         |
| ②号 |       |          |      |      |         |

<div style="height: 15px;"></div>

- C档：中低端安卓平板 —— MID画质

| 场景 | 平均FPS | 1%LowFPS | 平均功耗 | 峰值功耗 | 平均GPU负载 |
      |----|-------|----------|------|------|---------|
| ①号 |       |          |      |      |         |
| ②号 |       |          |      |      |         |

<div style="height: 15px;"></div>

### 3. 后续优化

## 四、工具链编写

### 1. 场景导入小连招

开始 → Blender导入 → 

→ Python插件：清理无关物体、精简重复贴图、导出映射表 →

→ Blender导出FBX →

→ Python脚本：贴图库中查找对应DDS贴图、转换为PNG格式 →

→ Unity导入FBX →

→ Unity工具：根据映射表自动上贴图、生成材质变体、清理多余贴图 → 结束

### 2. 角色导入小连招

<div style="height: 225px;"></div>



























