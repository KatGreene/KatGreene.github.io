---
layout:       post
title:        "KT Studio 开发记录（技术美术相关）"
author:       "KatGreene"
header-style: text
catalog:      true
tags:
    - KT Studio
    - TA
---

> 本文记录了我在 KT Studio 里做了什么画面效果、渲染优化等等；
> 
> 如果这些简短的记录能为你提供灵感，我将十分甚至九分高兴！同时也欢迎联系我进行深入♂交流。

- 参考过的教程：

## 角色渲染部分

<img src="/img/404-bg.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 好看！*

[//]: # (<div style="height: 15px;"></div>)

### 基础角色渲染

基于《崩坏：星穹铁道》的角色渲染框架，复现原游戏内的基础卡通角色渲染。

这些效果包括：
- 对角色进行兰伯特二分阴影处理，并采样Ramp贴图；
- 脸部阴影的SDF二分阴影处理、脖子部位的阴影区处理；
- 复现衣物的金属高光和非金属高光效果；
- 后处理Bloom效果与色调映射；
- 角色平滑法线描边；
- 角色眼睛眉毛透过刘海效果；
- ······

### 进阶角色渲染

在《崩坏：星穹铁道》的角色渲染框架之上，提升了许多其它效果。

这些效果包括：

#### 阴影
- 接受场景阴影：接受场景中一定范围内的物体投影；
- 排除角色自投影：去除角色往自己身上投射的丑陋阴影；
- 屏幕空间角色自投影：改为使用屏幕空间深度方法，绘制角色自投影；
- 刘海投影：采样屏幕空间的投影，使用额外深度方法，绘制清晰且不变形的刘海投影；除此之外，还新增了仿原画的刘海阴影膨胀效果。
  
<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色阴影总览*

<div style="height: 15px;"></div>

#### 法线
- 头部使用法线球面化，并在一定范围外混合原法线，保持长发区的法线正确；
- 眼部使用法线平面化，适配眼部在背光阴影时的亮度变化，防止阴影区眼白过亮导致的阴间效果。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色法线总览*

<div style="height: 15px;"></div>

#### 边缘光
- 屏幕空间边缘光：适配FOV、屏幕尺寸、摄像机距离。当这些参数变化时，边缘光的区域能在一定范围内自动缩放，防止边缘光区域过大或过小；
- 逆光增强：当逆光观察角色时，屏幕空间边缘光会适当增强。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色边缘光总览*

<div style="height: 15px;"></div>

#### 动态头发高光
- 采用动态的仿手绘动态高光方案：高光随着视角运动，并且相比直接画高光贴图拥有更高的清晰度；
- 高光多方向适应：同一个Mesh内可以拥有多个高光束，并拥有不同的方向和基础位置；当角色头骨旋转时，高光偏移方向也能够正确跟随；
- 高光抗变形：将动态高光与物体法线解耦，防止物体运动变形时导致的高光分裂与变形。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色头发高光总览*

<div style="height: 15px;"></div>

#### 丝袜
- 重写了丝袜渲染效果，采用高光油丝袜的渲染思路。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色丝袜总览*

<div style="height: 15px;"></div>

#### 动态眼睛
- 融合视差方案，并搭配贴图实现自定义的丰富眼睛效果。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色动态眼睛总览*

<div style="height: 15px;"></div>

#### 多光源
- 角色能够接受场景主光和额外光源的颜色；
- 角色同一时间内只接受唯一主光产生的阴影，其它光源均不造成任何阴影（仅随距离进行亮度衰减）。


<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 角色多光源总览*

<div style="height: 15px;"></div>

## 场景渲染部分

<img src="/img/home-bg-silwolf-1.jpg" alt="KT Studio角色展示" style="width:100%; height:auto; border-radius: 8px">

*- 高度定制化渲染！*

[//]: # (<div style="height: 15px;"></div>)

### 基础场景渲染

基于《崩坏：星穹铁道》的场景渲染框架，复现原游戏内的基础场景渲染渲染。

这些效果包括：
- 基于原来的基础色、金属度、法线、光滑度贴图，还原基础的PBR渲染效果；
- 融合基础的深度雾与高度雾；
- ······

### 进阶场景渲染

在《崩坏：星穹铁道》的场景渲染框架之上，提升了许多其它效果。

这些效果包括：

#### 程序化天空盒
- 低成本大气散射：实时快速拟合Mie散射部分；
- 地平线遮挡效果：落日时增加地平线对散射的遮挡斜切效果；
- 低成本末地光柱效果：仿照Minecraft主流光影在末地的渲染效果，并采用低成本的方法复现，无需Ray marching；
- 平面映射焦散云：天空采样平面映射后的焦散贴图，类似卷云或极光效果；
- 星星点点：流动的星光效果。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 程序化天空盒总览*

#### 低成本环境光
- 环境光漫射：基于法线与光照方向，采样环境漫射颜色Ramp图，代替球谐光照方案；同时，这张Ramp图也会融合实时光照信息，因此无需动态生成Ramp图；
- 环境光镜面反射：基于反射向量方向与直接高光结果，采样环境高光颜色Ramp图，代替环境探针采样天空球的CubeMap方案；同样的，这张Ramp图也无需动态生成；

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 低成本环境光总览*

<div style="height: 15px;"></div>

#### 平面反射
- 在角色展示界面与水面处加入平面反射，采用直接反转相机的方案，并斜截视锥体，最终在屏幕空间中采样。

#### 水面渲染
- 水面反射：ULTRA画质下启用平面反射，其它画质下采样静态环境捕获Cubemap并融合灯光颜色；
- 水面焦散旋转：根据世界位置与灯光方向采样焦散，使其能够随灯光方向变化；
- 其它水面效果：法线扰动、竖直方向散射、岸边浪、水面与水下阴影、折射等等。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 水面渲染总览*

<div style="height: 15px;"></div>

#### 高成本体积光（待优化）
- 非常耗费性能的体积光，采用主流方案并且还未优化。

#### 场景深度描边
- 采用主流深度描边方案，提升场景手绘效果。

## 音乐可视化部分

### FDTD贴图生成
- 采用Compute Shader，基于传统的FDTD法模拟音乐波动和扩散效果；
- 一共分为五个源激励，可由玩家或C#独立控制，分别代表五个音乐可视化区。

### UV采样
- 使用专用的UV采样此FDTD贴图，产生音乐波动效果与旋律弦效果；
- 同时接受C#程序端传入的信息，跟随音乐进行闪烁。

<img src="/img/ktstudio_icon_rounded.png" alt="KT Studio角色展示" style="width:15%; height:auto; border-radius: 8px">

*- 音乐可视化总览*

<div style="height: 15px;"></div>

## 性能测试与优化

### 性能分级与目标

### 实机测试

### 后续优化

## 工具链编写

### 场景导入小连招

### 角色导入小连招

<div style="height: 2025px;"></div>



























