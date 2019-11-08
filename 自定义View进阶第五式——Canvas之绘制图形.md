### 自定义View进阶第五式——Canvas之绘制图形

[TOC]



#### 第一、Canvas简介

Canvas我们可以称之为画布，能够在上面绘制各种东西，是安卓平台2D图形绘制的基础，非常强大。

**一般来说，比较基础的东西有两大特点:**

- 1.可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。
- 2.比较难用：各种方法太过基础，想要完美的将这些操作组合起来有一定难度。

不过不必担心，本系列文章不仅会介绍到Canvas的操作方法，还会简单介绍一些设计思路和技巧。

#### 第二、Canvas常用操作速查表

| 操作类型     | 相关API                                                      | 备注                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 绘制颜色     | drawColor,drawRGB,drawARGB                                   | 使用单一颜色填充整个画布                                     |
| 绘制基本形状 | drawPoint,drawPoints,drawLine,drawLines,drawRect,<br />drawRoundRect,drawOval,drawCircle,drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                |
| 绘制图片     | drawBitmap, drawPicture                                      |                                                              |
| 绘制文本     | drawText, drawPosText, drawTextOnPath                        |                                                              |
| 绘制路径     | drawPath                                                     |                                                              |
| 顶点操作     | drawVertics, drawBitmapMesh                                  | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪藏     | clipPath, clipRect                                           |                                                              |
| 画布快照     | save, restore, saveLayerXxx, restoreToCount, getSaveCount    |                                                              |
| 画布变换     | translate, scale, rotate, skew                               |                                                              |
| Matrix(矩阵) | getMatrix, setMatrix, concat                                 |                                                              |



#### 第三、Canvas详解

##### 3.1、绘制颜色

##### 3.2、创建画笔

##### 3.3、绘制点

##### 3.4、绘制直线

##### 3.5、绘制矩形

##### 3.6、绘制圆角矩形

##### 3.7、绘制椭圆

##### 3.8、绘制圆

##### 3.9、绘制圆弧

#### 第四、小栗子炒起来

##### 4.1、自定义一个饼图

