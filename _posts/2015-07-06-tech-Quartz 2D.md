---
layout: post
title: Quartz 2D
category: 技术
tags: Quartz 2D
keywords: Quartz 2D
description: Quartz 2D 编程指南
---

## Graphics Context（绘制目标）
是一个数据类型(`CGContextRef`)，用于封装Quartz绘制图像到输出设备的信息，定义了基本的绘制属性，如颜色、裁减区域、线条宽度和样式信息、字体信息、混合模式等。设备可以是PDF文件、bitmap或者显示器的窗口上。

Quartz提供了以下几种类型的Graphics Context：

1. Bitmap Graphics Context
2. PDF Graphics Context
3. Window Graphics Context
4. Layer Context
5. Post Graphics Context

### 在iOS中的视图Graphics Context进行绘制

在iOS应用程序中，如果要在屏幕上进行绘制，需要创建一个UIView对象，并实现它的`drawRect:`方法。视图的`drawRect:`方法在视图显示在屏幕上及它的内容需要更新时被调用。在调用自定义的`drawRect:`后，视图对象自动配置绘图环境以便代码能立即执行绘图操作。作为配置的一部分，视图对象将为当前的绘图环境创建一个`Graphics Context`。我们可以通过调用`UIGraphicsGetCurrentContext`函数来获取这个`Graphics Context`。

UIKit默认的坐标系统与Quartz不同。在UIKit中，原点位于左上角，y轴正方向为向下。UIView通过将修改Quartz的Graphics Context的CTM(当前变换矩阵)[原点平移到左下角，同时将y轴反转(y值乘以-1)]以使其与UIView匹配。

## Quartz 2D 数据类型
除了 Graphics Context 之外，Quartz 2D API还定义一些数据类型。Quartz 2D使用这些数据类型来创建对象，通过操作这些对象来获取特定的图形。

下面列出了Quartz 2D包含的数据类型：

- CGPathRef：用于向量图，可创建路径，并进行填充或描画(stroke)
- CGImageRef：用于表示bitmap图像和基于采样数据的bitmap图像遮罩。
- CGLayerRef：用于表示可用于重复绘制(如背景)和幕后(offscreen)绘制的绘画层
- CGPatternRef：用于重绘图
- CGShadingRef、CGGradientRef：用于绘制渐变
- CGFunctionRef：用于定义回调函数，该函数包含一个随机的浮点值参数。当为阴影创建渐变时使用该类型
- CGColorRef, CGColorSpaceRef：用于告诉Quartz如何解释颜色
- CGImageSourceRef,CGImageDestinationRef：用于在Quartz中移入移出数据
- CGFontRef：用于绘制文本
- CGPDFDictionaryRef, CGPDFObjectRef, CGPDFPageRef, CGPDFStream, CGPDFStringRef, and CGPDFArrayRef：用于访问PDF的元数据
- CGPDFScannerRef, CGPDFContentStreamRef：用于解析PDF元数据
- CGPSConverterRef：用于将PostScript转化成PDF。在iOS中不能使用。

## 图形状态

Quartz通过修改当前图形状态(current graphics state)来修改绘制操作的结果。图形状态包含用于绘制程序的参数。绘制程序根据这些绘图状态来决定如何渲染结果。例如，当你调用设置填充颜色的函数时，你将改变存储在当前绘图状态中的颜色值。

Graphics Context包含一个绘图状态栈。当Quartz创建一个Graphics Context时，栈为空。当保存图形状态时，Quartz将当前图形状态的一个副本压入栈中。当还原图形状态时，Quartz将栈顶的图形状态出栈。出栈的状态成为当前图形状态。

可使用函数`CGContextSaveGState`来保存图形状态，`CGContextRestoreGState`来还原图形状态。

## 路径(Path)

### 构建块(Building Block)

- 点   `CGContextMoveToPoint`
- 直线 `CGContextAddLineToPoint`
- 弧   `CGContextAddArc`、`CGContextAddArcToPoint`:为矩形创建内切弧
- 曲线 `CGContextAddCurveToPoint`
- 闭合路径 `CGContextClosePath`
- 椭圆 `CGContextAddEllipseInRect`
- 矩形 `CGContextAddRect`

### 创建路径

1. 在开始绘制路径前，调用函数`CGContextBeginPath；`来标记Quartz
2. 直线、弧、曲线开始于当前点。空路径没有当前点；必须调用`CGContextMoveToPoint`来设置第一个子路径的起始点，或者调用一个便利函数来隐式地完成该任务。
3. 如果要闭合当前子路径，调用函数`CGContextClosePath`。随后路径将开始一个新的子路径，即使不显示设置一个新的起始点。
4. 当绘制弧时，Quartz将在当前点与弧的起始点间绘制一条直线。
5. 添加椭圆和矩形的Quartz程序将在路径中添加新的闭合子路径。
6. 必须调用绘制函数来填充或者描边一条路径，因为创建路径时并不会绘制路径。

在绘制路径后，将清空图形上下文。如果想保留路径，特别是在绘制复杂场景时，需要反复使用。基于此，Quartz提供了两个数据类型来创建可复用路径—`CGPathRef`和`CGMutablePathRef`。可以调用函数`CGPathCreateMutable`来创建可变的`CGPath`对象，并可向该对象添加直线、弧、曲线和矩形。Quartz提供了一个类似于操作图形上下文的`CGPath`的函数集合。这些路径函数操作`CGPath`对象，而不是图形上下文。这些函数包括：

- `CGPathCreateMutable`，取代`CGContextBeginPath`
- `CGPathMoveToPoint`，取代`CGContextMoveToPoint`
- `CGPathAddLineToPoint`，取代`CGContexAddLineToPoint`
- `CGPathAddCurveToPoint`，取代`CGContexAddCurveToPoint`
- `CGPathAddEllipseInRect`，取代`CGContexAddEllipseInRect`
- `CGPathAddArc`，取代`CGContexAddArc`
- `CGPathAddRect`，取代`CGContexAddRect`
- `CGPathCloseSubpath`，取代`CGContexClosePath`

### 裁剪路径

```objc
CGContextBeginPath(context);
CGContextAddArc(context, w/2, h/2, ((w>h) ? h : w)/2, 0, 2*PI, 0);
CGContextClosePath(context);
CGContextClip(context);
```






