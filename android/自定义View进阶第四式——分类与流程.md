### 自定义View进阶第四式——分类与流程

[TOC]



#### 第一、概述

 **本章节为什么要叫进阶篇？(虽然讲的是基础内容)，因为从本篇开始，将会逐渐揭开自定义View的神秘面纱，每一篇都将比上一篇内容更加深入，利用所学的知识能够制作更加炫酷自定义View，就像在台阶上一样，每一篇都更上一层，~~帮助大家一步步走向人生巅峰，出任CEO，迎娶白富美。~~ 误，是帮助大家更加了解那些炫酷的自定义View是如何制作的，达到举一反三的效果。** 

 自定义View绘制流程函数调用链(简化版) 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161304.png)



#### 第二、自定义View分类

 **自定义View分为了两类：** 

##### 2.1、自定义`ViewGroup`

 **自定义`ViewGroup`一般是利用现有的组件根据特定的布局方式来组成新的组件，大多继承自`ViewGroup`或各种Layout，包含有子View。** 

>  例如：应用底部导航条中的条目，一般都是上面图标(`ImageView`)，下面文字(`TextView`)，那么这两个就可以用自定义`ViewGroup`组合成为一个View，提供两个属性分别用来设置文字和图片，使用起来会更加方便。 



##### 2.2、自定义View

 **在没有现成的View，需要自己实现的时候，就使用自定义View，一般继承自View，`SurfaceView`或其他的View，不包含子View。** 

>  例如：制作一个支持自动加载网络图片的`ImageView`，制作图表等。 

 **注意： 自定义View在大多数情况下都有替代方案，利用图片或者组合动画来实现，但是使用后者可能会面临内存耗费过大，制作麻烦等诸多问题。** 

#### 第三、几个重要的函数

##### 3.1、构造函数

 构造函数是View的入口，可以用于**初始化一些的内容，和获取自定义属性**。 

 View的构造函数有四种重载分别如下: 

```java
public void SloopView(Context context) {}
public void SloopView(Context context, AttributeSet attrs) {}
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr) {}
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```

 **有四个参数的构造函数在`API21`的时候才添加上，暂不考虑。** 

**注意：即使你在View中使用了Style这个属性也不会调用三个参数的构造函数，所调用的依旧是两个参数的构造函数。**

**由于三个参数的构造函数第三个参数一般不用，暂不考虑，第三个参数的具体用法会在以后用到的时候详细介绍。**

 排除了两个之后，只剩下一个参数和两个参数的构造函数，他们的详情如下： 

```java
//一般在直接New一个View的时候调用。
public void SloopView(Context context) {}

//一般在layout文件中使用的时候会调用，关于它的所有属性(包括自定义属性)都会包含在attrs中传递进来。
public void SloopView(Context context, AttributeSet attrs) {}
```

 **以下方法调用的是一个参数的构造函数：** 

```java
//在Avtivity中
SloopView view = new SloopView(this);
```

 **以下方法调用的是两个参数的构造函数：** 

```java
//在layout文件中 - 格式为： 包名.View名
<com.sloop.study.SloopView
  android:layout_width"wrap_content"
  android:layout_height"wrap_content"/>
```



##### 3.2、测量View大小(`onMeasure`)

**Q: 为什么要测量View大小？**

**A: View的大小不仅由自身所决定，同时也会受到父控件的影响，为了我们的控件能更好的适应各种情况，一般会自己进行测量**

 测量View大小使用的是`onMeasure`函数，我们可以从`onMeasure`的两个参数中取出宽高的相关数据： 

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthsize = MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
    int widthmode = MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
    
    int heightsize = MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
    int heightmode = MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
}
```

从上面可以看出 `onMeasure` 函数中有 `widthMeasureSpec` 和 `heightMeasureSpec` 这两个 int 类型的参数， 毫无疑问他们是和宽高相关的， **但它们其实不是宽和高， 而是由宽、高和各自方向上对应的测量模式来合成的一个值：**

**测量模式一共有三种， 被定义在 Android 中的 View 类的一个内部类`View.MeasureSpec`中：**

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109161331.png)

 **PS: 实际上关于上面的东西了解即可，在实际运用之中只需要记住有三种模式，用 `MeasureSpec` 的 `getSize`是获取数值， `getMode`是获取模式即可。** 

**注意：**

 如果对View的宽高进行修改了，不要调用 `super.onMeasure( widthMeasureSpec, heightMeasureSpec);` 要调用 `setMeasuredDimension( widthsize, heightsize);` 这个函数。 

##### 3.3、确定View大小(`onSizeChanged`)

 这个函数在视图大小发生改变时调用。 

**Q: 在测量完View并使用`setMeasuredDimension`函数之后View的大小基本上已经确定了，那么为什么还要再次确定View的大小呢？**

**A: 这是因为View的大小不仅由View本身控制，而且受父控件的影响，所以我们在确定View大小的时候最好使用系统提供的`onSizeChanged`回调函数。**

```java
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
}
```

可以看出，它又四个参数，分别为 宽度，高度，上一次宽度，上一次高度。

这个函数比较简单，我们只需关注 宽度(w), 高度(h) 即可，这两个参数就是View最终的大小。

##### 3.4、确定子View布局位置(`onLayout`)

 **确定布局的函数是`onLayout`，它用于确定子View的位置，在自定义`ViewGroup`中会用到，他调用的是子View的layout函数。** 

 在自定义`ViewGroup`中，`onLayout`一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用以下函数设置子View位置。 

```java
child.layout(l, t, r, b);
```

四个参数分别为：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162134.png)



##### 3.5、绘制内容(`onDraw`)

 `onDraw`是实际绘制的部分，也就是我们真正关心的部分，使用的是Canvas绘图。 

```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
}
```



##### 3.6、对外提供操作方法和监听回调

 自定义完View之后，一般会对外暴露一些接口，用于控制View的状态等，或者监听View的变化. 

#### 第四、重点知识梳理

##### 4.1、自定义View分类

>  PS ：实际上`ViewGroup`是View的一个子类。 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162201.png)

##### 4.2、自定义View流程

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109162221.png)

