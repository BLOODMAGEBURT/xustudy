#### 自定义View基础第一式——坐标系

------

[TOC]



##### 第一、屏幕坐标系与数学坐标系的区别

 由于移动设备一般定义屏幕左上角为坐标原点，向右为x轴增大方向，向下为y轴增大方向， 所以在手机屏幕上的坐标系与数学中常见的坐标系是稍微有点差别的，详情如下： 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160337.png)

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160400.png)

 **实际屏幕上的默认坐标系如下：** 

>  PS: 假设其中棕色部分为手机屏幕 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160429.png)

##### 第二、View的坐标系是相对于父控件而言的

 **注意：View的坐标系统是相对于父控件而言的.** 

```java
getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```

**如下图所示：**

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160454.png)

##### 第三、MotionEvent的get与getRaw的区别

```java
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();
```

 **如下图所示：** 

>  PS:其中相同颜色的内容是对应的，其中为了显示方便，蓝色箭头向左稍微偏移了一点. 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160516.png)

