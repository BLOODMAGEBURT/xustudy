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

绘制颜色是填充整个画布，常用于绘制底色。

```java
canvas.drawColor(Color.BLUE)
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162311.png)



##### 3.2、创建画笔

要想绘制内容，首先需要先创建一个画笔，如下：

```java
// 1.创建一个画笔
private Paint mPaint = new Paint();

// 2.初始化画笔
private void initPaint() {
	mPaint.setColor(Color.BLACK);       //设置画笔颜色
	mPaint.setStyle(Paint.Style.FILL);  //设置画笔模式为填充
	mPaint.setStrokeWidth(10f);         //设置画笔宽度为10px
}

// 3.在构造函数中初始化
public SloopView(Context context, AttributeSet attrs) {
   super(context, attrs);
   initPaint();
}
```

在创建完画笔之后，就可以在Canvas中绘制各种内容了。

##### 3.3、绘制点

可以绘制一个点，也可以绘制一组点。

```java
canvas.drawPoint(200, 200, mPaint);     //在坐标(200,200)位置绘制一个点
canvas.drawPoints(new float[]{          //绘制一组点，坐标位置由float数组指定
      500,500,
      500,600,
      500,700
},mPaint);
```

关于坐标原点默认在左上角，水平向右为x轴增大方向，竖直向下为y轴增大方向。

##### 3.4、绘制直线

绘制直线需要两个点，初始点和结束点，同样绘制直线也可以绘制一条或者绘制一组：

```java
canvas.drawLine(300,300,500,600,mPaint);    // 在坐标(300,300)(500,600)之间绘制一条直线
canvas.drawLines(new float[]{               // 绘制一组线 每四数字(两个点的坐标)确定一条线
    100,200,200,200,
    100,300,200,300
},mPaint);
```



##### 3.5、绘制矩形

我们都知道，确定一个矩形最少需要四个数据，就是**对角线的两个点**的坐标值，这里一般采用**左上角和右下角**的两个点的坐标。

关于绘制矩形，Canvas提供了三种重载方法.

```java
// 第一种
canvas.drawRect(100,100,800,400,mPaint);

// 第二种
Rect rect = new Rect(100,100,800,400);
canvas.drawRect(rect,mPaint);

// 第三种
RectF rectF = new RectF(100,100,800,400);
canvas.drawRect(rectF,mPaint);
```

看到这里,相信很多观众会产生一个疑问，**为什么会有Rect和RectF两种？两者有什么区别吗？**

答案当然是存在区别的，**两者最大的区别就是精度不同，Rect是int(整形)的，而RectF是float(单精度浮点型)的**。

##### 3.6、绘制圆角矩形

绘制圆角矩形也提供了两种重载方式，如下：

```java
// 第一种
RectF rectF = new RectF(100,100,800,400);
canvas.drawRoundRect(rectF,30,30,mPaint);

// 第二种,在API21加入
canvas.drawRoundRect(100,100,800,400,30,30,mPaint);
```

**与矩形相比，圆角矩形多出来了两个参数rx 和 ry**，这两个参数是干什么的呢？

但是，**半径只需要一个参数，但这里怎么会有两个呢？**

好吧，让你发现了，**这里圆角矩形的角实际上不是一个正圆的圆弧，而是椭圆的圆弧，这里的两个参数实际上是椭圆的两个半径**，他们看起来个如下图：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162329.png)



**红线标注的 rx 与 ry 就是两个半径，也就是相比绘制矩形多出来的那两个参数。**

##### 3.7、绘制椭圆

相对于绘制圆角矩形，绘制椭圆就简单的多了，因为他只需要一个矩形作为参数:

```java
// 第一种
RectF rectF = new RectF(100,100,800,400);
canvas.drawOval(rectF,mPaint);

// 第二种
canvas.drawOval(100,100,800,400,mPaint);
```

同样，以上两种方法效果完全一样，但一般使用第一种。

绘制椭圆实际上就是绘制一个矩形的内切图形，原理如下，就不多说了：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162349.png)

**PS**： 如果你传递进来的是一个长宽相等的矩形(即正方形)，那么绘制出来的实际上就是一个圆。

##### 3.8、绘制圆

绘制圆形也比较简单, 如下：

```java
canvas.drawCircle(500,500,400,mPaint);  // 绘制一个圆心坐标在(500,500)，半径为400 的圆。
```



##### 3.9、绘制圆弧

绘制圆弧就比较神奇一点了，为了理解这个比较神奇的东西，我们先看一下它需要的几个参数：

```java
// 第一种
public void drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint){}
    
// 第二种
public void drawArc(float left, float top, float right, float bottom, float startAngle,
            float sweepAngle, boolean useCenter, @NonNull Paint paint) {}
```

从上面可以看出，相比于绘制椭圆，绘制圆弧还多了三个参数：

```java
startAngle  // 开始角度
sweepAngle  // 扫过角度
useCenter   // 是否使用中心
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162416.png)



可以发现使用了中心点之后绘制出来类似于一个扇形，而不使用中心点则是圆弧起始点和结束点之间的连线加上圆弧围成的图形。这样中心点这个参数的作用就很明显了，不必多说想必大家试一下就明白了。 

#### 第四、小栗子炒起来

##### 4.1、自定义一个饼图

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162437.png)

