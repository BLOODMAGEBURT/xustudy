### 自定义View基础第三式——画笔Paint

[TOC]



#### 第一、引子概述

View上的内容是通过Canvas绘制出来的，但是Canvas中的大多数绘制方法都需要Paint作为参数。例如 `canvas.drawCircle(100,100,50,paint)` ，最后需要传递一个Paint,这是为什么呢？

因为画布本身只是呈现的一个载体，真正绘制出来的效果却要取决于画笔，就像同样白纸，要绘制一幅山水图，用毛笔画和用铅笔画的效果肯定是完全不同的，决定不同显示效果的并不是画布(Canvas), 而是画笔(Paint)。

 同样，在程序设计中也采用的类似的设计思想，画布的 draw 方法只是规定了所需要绘制的是什么东西，但具体绘制出什么效果则通过画笔来控制。
例如： `canvas.drawCircle(100, 100, 50, paint)`，这个方法说明了要在坐标 (100, 100) 的位置绘制一个半径为 50 的圆，但是这个圆具体要绘制成什么样子却没有明确的表明，圆的颜色，圆环还是圆饼等都没有明确的指示，而这些内容正存在于画笔之中。 

#### 第二、画笔介绍

##### 	2.1、初始化

 要使用画笔就要会创建画笔，创建一个画笔是非常简单的，在之前的文章中也有过简单的介绍。它有三种创建方法，如下： 

```java
// 1.创建一个默认画笔，使用默认的配置
Paint()
// 2.创建一个新画笔，并通过 flags 参数进行配置。
Paint(int flags)
    // 如果需要设置多个参数，参数之间用 | 进行连接即可
	Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG); 
// 3.创建一个新画笔，并复制参数中画笔的设置。就是复制一个画笔
Paint(Paint paint)
```

**不建议使用setFlags方法，这是因为 setFlags 方法会覆盖之前设置的内容** ，例如：

```java
Paint paint = new Paint();
paint.setFlags(Paint.ANTI_ALIAS_FLAG);
paint.setFlags(Paint.DITHER_FLAG);
Log.i(TAG, "paint isAntiAlias = " + paint.isAntiAlias());
Log.i(TAG, "paint isDither = " + paint.isDither());
```

输出结果：

```java
paint isAntiAlias = false
paint isDither = true
// 如果想要调整 flag 个人建议还是使用 paint 提供的一些封装方法，如:
 paint.setDither(true)
// 而不要自己手动去直接操作 flag。
```



##### 	2.2、画笔颜色

```java
// 下面两种设置方式是等价的，一种是 10 进制，一种是 16 进制
paint1.setAlpha(204);
paint2.setAlpha(0xCC);
// setARGB(), 4个参数的取值范围也是 0 - 255，对应 0x00 - 0xFF
paint1.setARGB(204, 255, 255, 0);
paint2.setARGB(0xCC, 0xFF, 0xFF, 0x00);
// setColor()
paint.setColor(Color.GREEN);
paint.setColor(0xFFE2A588);
```

**注意：**

 如果不设置 Alpha 通道，则默认Alpha通道为 0，即完全透明，如：0xE2A588，总共 6 位，没有 Alpha 通道，如果这样设置，则什么颜色也绘制不出来。 

##### 2.3、画笔宽度

```java
// 将画笔设置为描边
paint.setStyle(Paint.Style.STROKE);
// 设置线条宽度
paint.setStrokeWidth(120);
```

**注意：**

 **这条线的宽度是同时向两边进行扩展的，例如绘制一个圆时，将其宽度设置为 120 则会向外扩展 60 ，向内缩进 60，如下图所示。** 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160753.png)

 **因此如果绘制的内容比较靠近视图边缘，使用了比较粗的描边的情况下，一定要注意和边缘保持一定距离(`边距>StrokeWidth/2`) 以保证内容不会被剪裁掉。** 

**发际线模式 hairline mode**:

 在画笔宽度为 0 的情况下，使用 drawLine 或者使用描边模式(STROKE)也可以绘制出内容。只是绘制出的内容始终是 1 像素，不受画布缩放的影响。该模式被称为**hairline mode (发际线模式)**。 

>  如果你设置了画笔宽度为 1 像素，那么如果画布放大到 2 倍，1 像素会变成 2 像素。但如果是 0 像素，那么不论画布如何缩放，绘制出来的宽度依旧为 1 像素。 

```java
// 缩放 5 倍
canvas.scale(5, 5, 500, 500);

// 0 像素 (Hairline Mode)
paint.setStrokeWidth(0);
paint.setColor(0xFF7FC2D8);
canvas.drawCircle(500, 455, 40, paint);

// 1 像素
paint.setStrokeWidth(1);
paint.setColor(0xFF7FC2D8);
canvas.drawCircle(500, 545, 40, paint);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160815.png)



##### 2.4、画笔模式

```java
//填充
mPaint.setStyle(Paint.Style.FILL);
// 描边
mPaint.setStyle(Paint.Style.STROKE);
// 描边+填充
mPaint.setStyle(Paint.Style.FILL_AND_STROKE);
```

**示例程序：**

用一个简单的例子说明一下不同模式的区别。

```java
// 画笔初始设置
Paint paint = new Paint();
paint.setAntiAlias(true);
paint.setStrokeWidth(50);
paint.setColor(0xFF7FC2D8);

// 填充，默认
paint.setStyle(Paint.Style.FILL);
canvas.drawCircle(500, 200, 100, paint);

// 描边
paint.setStyle(Paint.Style.STROKE);
canvas.drawCircle(500, 500, 100, paint);

// 描边 + 填充
paint.setStyle(Paint.Style.FILL_AND_STROKE);
canvas.drawCircle(500, 800, 100, paint);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160837.png)



##### 2.5、画笔线段线帽(Paint.Cap，3种)

 画笔线帽(**Paint.Cap**)用于指定线段开始和结束时的效果。 

```java
// 它通过下面方式设置
paint.setStrokeCap(Paint.Cap.ROUND);
// 有3种效果
Paint.Cap.BUTT // 无线帽
Paint.Cap.SQUARE // 方形
Paint.Cap.ROUND // 圆形
```

```java
// 画笔初始设置
Paint paint = new Paint();
paint.setStyle(Paint.Style.STROKE);
paint.setAntiAlias(true);
paint.setStrokeWidth(80);
float pointX = 200;
float lineStartX = 320;
float lineStopX = 800;
float y;

// 默认
y = 200;
canvas.drawPoint(pointX, y, paint);
canvas.drawLine(lineStartX, y, lineStopX, y, paint);

// 无线帽(BUTT)
y = 400;
paint.setStrokeCap(Paint.Cap.BUTT);
canvas.drawPoint(pointX, y, paint);
canvas.drawLine(lineStartX, y, lineStopX, y, paint);

// 方形线帽(SQUARE)
y = 600;
paint.setStrokeCap(Paint.Cap.SQUARE);
canvas.drawPoint(pointX, y, paint);
canvas.drawLine(lineStartX, y, lineStopX, y, paint);

// 圆形线帽(ROUND)
y = 800;
paint.setStrokeCap(Paint.Cap.ROUND);
canvas.drawPoint(pointX, y, paint);
canvas.drawLine(lineStartX, y, lineStopX, y, paint);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160858.png)



##### 2.6、线段连接方式(拐角类型Paint.Join， 3种)

 画笔的连接方式(**Paint.Join**)是指两条连接起来的线段拐角显示方式。 

```java
// 通过下面方式设置连接类型
paint.setStrokeJoin(Paint.Join.ROUND);
```

它有三种样式：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160923.png)



##### 2.7、pathEffect(6种PathEffect，其中4种基础效果，2种叠加效果)

**PathEffect在绘制之前修改几何路径，它可以实现划线，自定义填充效果，自定义笔触的效果。PathEffect虽然名字看起来是和Path相关，但实际上它的效果可以作用于Canvas的各种绘制上，例如drawLine, drawRect, drawPath等等**

> **注意：PathEffect在部分情况下不支持硬件加速，需要关闭硬件加速**
>
> 1. `Canvas.drawLine()` 和 `Canvas.drawLines()` 方法画直线时，`setPathEffect()` 是不支持硬件加速的；
> 2. `PathDashPathEffect` 对硬件加速的支持也有问题，所以当使用 `PathDashPathEffect` 的时候，最好也把硬件加速关了。

 在 Android 中有 6 种 PathEffect，4 种基础效果，2 种叠加效果。 

| **PathEffect**     | **简介**                                               |
| ------------------ | ------------------------------------------------------ |
| CornerPathEffect   | 圆角效果，将尖角替换为圆角。                           |
| DashPathEffect     | 虚线效果，用于各种虚线效果。                           |
| PathDashPathEffect | Path 虚线效果，虚线中的间隔使用 Path 代替。            |
| DiscretePathEffect | 让路径分段随机偏移。                                   |
| SumPathEffect      | 两个 PathEffect 效果组合，同时绘制两种效果。           |
| ComposePathEffect  | 两个 PathEffect 效果叠加，先使用效果1，之后使用效果2。 |

```java
// 通过 setPathEffect 来设置效果
paint.setPathEffect(effect);
```

###### 2.7.1、CornerPathEffect

```java
// radius 为圆角半径大小，半径越大，path 越平滑。
CornerPathEffect(radius);
```

 **CornerPathEffect 也可以让手绘效果更加圆润。** 

>  一些简单的绘图场景或者签名场景中，一般使用 Path 来保存用户的手指轨迹，通过连续的 lineTo 来记录用户手指划过的路径，但是直接的 LineTo 会让转角看起来非常生硬，而使用 CornerPathEffect 效果则可以快速的让轨迹圆润起来。 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160951.png)

```java
public class CornerPathEffectTestView extends View {
    Paint mPaint = new Paint();
    PathEffect mPathEffect = new CornerPathEffect(200);
    Path mPath = new Path();

    public CornerPathEffectTestView(Context context) {
        this(context, null);
    }

    public CornerPathEffectTestView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        mPaint.setStrokeWidth(20);
        mPaint.setStyle(Paint.Style.STROKE);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                mPath.reset();
                mPath.moveTo(event.getX(), event.getY());
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(event.getX(), event.getY());
                break;
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                break;
        }
        postInvalidate();
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 绘制原始路径
        canvas.save();
        mPaint.setColor(Color.BLACK);
        mPaint.setPathEffect(null);
        canvas.drawPath(mPath, mPaint);
        canvas.restore();

        // 绘制带有效果的路径
        canvas.save();
        canvas.translate(0, canvas.getHeight() / 2);
        mPaint.setColor(Color.RED);
        mPaint.setPathEffect(mPathEffect);
        canvas.drawPath(mPath, mPaint);
        canvas.restore();
    }
}
```



###### 2.7.2、DashPathEffect

 DashPathEffect 用于实现虚线效果(适用于 STROKE 或 FILL_AND_STROKE 样式)。 

```java
// intervals：必须为偶数，用于控制显示和隐藏的长度。
// phase：相位。也可以理解为偏移量
DashPathEffect(float intervals[], float phase)
    
    
Path path_dash = new Path();
path_dash.lineTo(0, 1720);

canvas.save();
canvas.translate(980, 100);
paint.setPathEffect(new DashPathEffect(new float[]{200, 100}, 0));
canvas.drawPath(path_dash, paint);
canvas.restore();

canvas.save();
canvas.translate(400, 100);
paint.setPathEffect(new DashPathEffect(new float[]{200, 100}, 100));
canvas.drawPath(path_dash, paint);
canvas.restore();
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161011.png)

 **注意：intervals[] 中是允许设置多组数据的，每两个为一组，第一个表示显示长度，第二个表示隐藏长度。** 

###### 2.7.3、PathDashPathEffect

 这个也是实现类似虚线效果，只不过这个虚线中显示的部分可以指定为一个 Path(适用于 STROKE 或 FILL_AND_STROKE 样式)。 

```java
// shape: Path 图形
// advance: 图形占据长度
// phase: 相位差
// style: 转角样式
PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161035.png)

 **PathDashPathEffect.Style** 

 PathDashPathEffect 的最后一个参数是 PathDashPathEffect.Style，这个参数用于处理 Path 图形在转角处的样式。 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161059.png)



###### 2.7.4、DiscretePathEffect

 DiscretePathEffect 可以让 Path 产生随机偏移效果。 

> 至今未看到市面上使用此效果

```java
// segmentLength: 分段长度
// deviation: 偏移距离
DiscretePathEffect(float segmentLength, float deviation);
```



###### 2.7.5、SumPathEffect

 SumPathEffect 用于合并两种效果，它相当于两种效果都绘制一遍。 

```java
// 两种效果相加
SumPathEffect(PathEffect first, PathEffect second);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161122.png)



###### 2.7.6、ComposePathEffect

 ComposePathEffect 也是合并两种效果，只不过先应用一种效果后，再次叠加另一种效果，因此**交换参数最终得到的效果是不同的**。 

```java
// 构造一个 PathEffect, 其效果是首先应用 innerpe 再应用 outerpe (如: outer(inner(path)))。
ComposePathEffect(PathEffect outerpe, PathEffect innerpe);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161139.png)

##### 2.8、getFillPath()

```java
// 根据原始Path(src)获取预处理后的Path(dst)
paint.getFillPath(Path src, Path dst);
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161204.png)

 **Q: 可是我们拿到预处理后的 Path 有什么作用呢？** 

A: 尽管通常情况下我们用不到，但在一些特殊情况下还是有些作用的，可以通过下面的一个实例了解。

在我前段时间开源的一个库里面，需要实现下面这样的效果，一个弧形的 SeekBbar。

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161222.png)

 如上图，是一个非常粗的圆弧，有一个白色的描边，这个白色的描边效果就可以通过 getFillPath 轻松实现。 

```java
// 通过 getFillPath 来获得圆弧的实际区域, 存储到 mBorderPath 中
mArcPaint.getFillPath(mSeekPath, mBorderPath);
```

 直接用 getFillPath 获取到这个粗圆弧的边缘，存储到 mBorderPath 中，把 mBorderPath 用白色画笔绘制出来就可以实现上图中的描边效果啦。 