#### 动态设置TextView的DrawableLeft等属性

有两种方式：置Drawable显示在text的左、上、右、下位置

1. `setCompoundDrawables(left, top, right, bottom)`
2. `setCompoundDrawablesWithIntrinsicBounds(left, top, right, bottom)`

**两者之间的联系：**

`setCompoundDrawablesWithIntrinsicBounds` 最终还是调用的 `setCompoundDrawables`，只是在此之前调用了setBounds()方法。

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20201230123652.png)

**根据两者之间的联系，我们也能看出两者之间的区别：**

`setCompoundDrawables` 画的drawable的宽高是按drawable.setBound()设置的宽高，
所以才有The Drawables must already have had setBounds(Rect) called，即使用之前必须使用Drawable.setBounds设置Drawable的长宽。

而`setCompoundDrawablesWithIntrinsicBounds`是画的drawable的宽高是按drawable固定的宽高，
所以才有The Drawables' bounds will be set to their intrinsic bounds.即通过*getIntrinsicWidth()*与*getIntrinsicHeight()*获得。

**所以一般建议使用：**

setCompoundDrawablesWithIntrinsicBounds，这样你即无需设置Drawables的bounds了。