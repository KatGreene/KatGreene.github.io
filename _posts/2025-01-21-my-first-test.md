---
layout:       post
title:        "你好！正式开张"
author:       "KatGreene"
header-style: text
catalog:      true
tags:
    - Web
    - UE
---

> 这是一条测试推文。

# C++ Section

## 01 EngineTypes.h

```cpp
enum EMaterialShadingModel : int
{
	MSM_Unlit					UMETA(DisplayName="Unlit"),
	MSM_DefaultLit				UMETA(DisplayName="Default Lit"),
	MSM_Subsurface				UMETA(DisplayName="Subsurface"),
	MSM_PreintegratedSkin		UMETA(DisplayName="Preintegrated Skin"),
	MSM_ClearCoat				UMETA(DisplayName="Clear Coat"),
	MSM_SubsurfaceProfile		UMETA(DisplayName="Subsurface Profile"),
	MSM_TwoSidedFoliage			UMETA(DisplayName="Two Sided Foliage"),
	MSM_Hair					UMETA(DisplayName="Hair"),
	MSM_Cloth					UMETA(DisplayName="Cloth"),
	MSM_Eye						UMETA(DisplayName="Eye"),
	MSM_SingleLayerWater		UMETA(DisplayName="SingleLayerWater"),
	MSM_ThinTranslucent			UMETA(DisplayName="Thin Translucent"),
	MSM_Strata					UMETA(DisplayName="Substrate", Hidden),

	// KATTOON START #001
	MSM_KatToon					UMETA(DisplayName="KatToon"),
	MSM_KatToonSkin				UMETA(DisplayName="KatToon Skin"),
	MSM_KatToonHair				UMETA(DisplayName="KatToon Hair"),
	// KATTOON END #001
	
	/** Number of unique shading models. */
	MSM_NUM						UMETA(Hidden),
	/** Shading model will be determined by the Material Expression Graph,
		by utilizing the 'Shading Model' MaterialAttribute output pin. */
	MSM_FromMaterialExpression	UMETA(DisplayName="From Material Expression"),
	MSM_MAX
};
```

# 1.1 渲染流水线

## 整体流程
- 应用阶段：
	- 由CPU完成，准备模型数据、摄像机光源数据、贴图数据等；
	- 进行粗粒度的遮挡剔除；
	- 将渲染设置与绘制顺序、渲染目标、渲染状态等提交给GPU（调用DrawCall)；
- 几何阶段：
	- 由GPU完成，包含了将顶点从模型空间变换到屏幕空间的过程；
	- 顶点着色器（顶点的变换与着色）、曲面细分着色器、几何着色器依次执行；
	- 进行投影（正交与透视）、裁剪（CVV和正面背面剔除）、屏幕映射；
- 光栅化阶段：
	- 由GPU完成，包含了将顶点组装为三角形，并进行三角形遍历；
	- 将顶点数据进行三角形插值，生成片元数据；
	- 产生的结果为一个个片元，并且进行硬件抗锯齿MSAA；
- 逐片元操作：
	- 由GPU完成，将每一个片元进行计算，得到最终片元的颜色；
	- 然后进行颜色混合，这一步进行Alpha、Depth、Stencil测试，并按照Blend设置输出颜色；
	- 最后输出到缓冲区FrameBuffer或者RenderTexture；
- 后处理：
	- 进行HDR色调映射、边缘检测、泛光、景深与模糊、抗锯齿等。

## 思考问题
### 应用阶段的粗粒度剔除是如何实现的？
- 八叉树
- BSP树
- K-D树
- BVH

### 逐片元阶段的各个测试的顺序是什么？
- 裁剪测试：如果一个点在裁剪区域外，我希望它彻底不渲染，也不要对缓冲区造成任何影响，所以优先级最高
- Alpha测试：优先于模板测试，当我们希望用一个不规则的图片进行模板的裁剪的时候，应该使用Alpha测试先过滤掉那些透明的片段，我们只希望剩下的不透明片段来更新模板缓存
- 模板测试：我希望对模板值进行一些更新，所以哪怕我被其它东西挡住，导致这个片段显示不出来，我也要把模板值更新进去
- 深度测试：最后决定这个片段是否渲染

# 字体渲染测试

あいうえお	かきくけこ	さしすせそ	たちつて

丕丞両並丨丶丼丿乗乛乜乩亀亍亓亘亜亝亟亨亳亵亶亸仂

耲耱懿韂蘸鹳蘼囊霾
氍饕躔躐髑
镵镶穰鳤
瓤饔
鬻
鬟趱攫攥颧
躜
罐鼹鼷
癯麟蠲

矗蠹醾
躞
衢鑫
灞襻
纛鬣攮
囔
馕
戆
蠼
爨
齉