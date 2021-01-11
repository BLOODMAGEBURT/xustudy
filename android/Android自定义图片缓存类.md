### Android自定义图片缓存类

> **思路**：
>
> 首先，我们可以通过继承`LruCache`类来是实现图片缓存的存取。
>
> 其次，我们可以做的更精细，将内存缓存分为两级，一级是活动资源的硬缓存，二级是从一级淘汰下来的图片作软缓存。
>
> ​	两者之间的关系如下：
>
> ​	硬缓存容量有限，当淘汰之后就进入软缓存。
>
> ​	软缓存是通过`SoftReference`包裹的引用。
>
> ​	软缓存使用的`LinkedHashMap`,模仿`LruCache`实现。当软缓存的图片被取出时，要重新加入到硬缓存中。
>
> 后续：
>
> 如果还想做硬盘缓存，就可以将从软缓存中淘汰下来的图片，缓存到磁盘上做存储。

1、新建一个类，继承`LruCache`

```java
// 注意这里使用的LruCache是 AndroidUtil里的
public class BitmapLruCache extends LruCache<String, BitmapDrawable>{

} 
```

2、构造函数中初始化软缓存

```java
 // 注意这里使用的LruCache是 AndroidUtil里的
public class BitmapLruCache extends LruCache<String, BitmapDrawable>{
 // 创建一个软应用的缓存池
    LinkedHashMap<String, SoftReference<BitmapDrawable>> mSoftBitmapCache;
    int SOFT_CACHE_SIZE = 10;
    
    public BitmapLruCache(int maxSize) {
        super(maxSize);

        mSoftBitmapCache = new LinkedHashMap<String, SoftReference<BitmapDrawable>>(SOFT_CACHE_SIZE, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Entry<String, SoftReference<BitmapDrawable>> eldest) {
                if (size() > SOFT_CACHE_SIZE) {

                    // 回收bitmapDrawable
                    BitmapDrawable bitmapDrawable = eldest.getValue().get();
                    recycleBitmap(bitmapDrawable);
                    return true;
                }
                return false;
            }
        };
    }
} 
 
 
```

3、回收 drawable的方法

```java
private void recycleBitmap(BitmapDrawable bitmapDrawable) {
        if (bitmapDrawable != null) {
            Bitmap bitmap = bitmapDrawable.getBitmap();
            if (!bitmap.isRecycled()) {
                bitmap.recycle();
            }
        }
    }
```

4、重写`LurCache`的两个方法

`sizeOf(String key, BitmapDrawable drawable)`

```java
 // 回调：计算图片大小，添加元素进map时计算大小回调
    @Override
    protected int sizeOf(String key, BitmapDrawable drawable) {
        // 宽上的像素，乘以 高度
        Bitmap bitmap = drawable.getBitmap();

        return bitmap.getRowBytes() * bitmap.getHeight();
    }
```

`entryRemoved(boolean evicted, String key, BitmapDrawable oldValue, BitmapDrawable newValue)`

```java
// 回调：Entry被移除时回调

    @Override
    protected void entryRemoved(boolean evicted, String key, BitmapDrawable oldValue, BitmapDrawable newValue) {
        if (evicted) {
            // 容量满，被逐出, 移动到软缓存中
            mSoftBitmapCache.put(key, new SoftReference<>(oldValue));

        } else {
            // 自己主动删除时，回收bitmap， 调用remove()时
            recycleBitmap(oldValue);
        }
    }
```

5、获取缓存图片方法

`getDrawable()`

```java
public BitmapDrawable getDrawable(String url) {
        // 先从硬缓存中找
        BitmapDrawable bitmapDrawable = this.get(url);
        if (bitmapDrawable != null) {
            return bitmapDrawable;
        }
		// 如果找不到
        // 再从软缓存中找， 找到之后重新加入到硬缓存, 从软缓存中去掉
        SoftReference<BitmapDrawable> bitmapReference = mSoftBitmapCache.get(url);
        if (bitmapReference != null) {
            bitmapDrawable = bitmapReference.get();

            if (bitmapDrawable != null) {

                // 重新加入到硬缓存
                this.put(url, bitmapDrawable);
                // 从软缓存中去掉
                mSoftBitmapCache.remove(url);

                return bitmapDrawable;
            } else {
                mSoftBitmapCache.remove(url);
            }
        }

        return null;
    }
```

