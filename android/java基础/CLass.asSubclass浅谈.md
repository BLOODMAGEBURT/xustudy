#### `CLass.asSubclass`浅谈

```java
public <U> Class<? extends U> asSubclass(Class<U> clazz)  
```

这是`java.lang.Class`中的一个方法，作用是将调用这个方法的class对象转换成由`clazz`参数所表示的class对象的某个子类。举例来说，

```java
List<String> strList = new ArrayList<String>();  
Class<? extends List> strList_cast = strList.getClass().asSubclass(List.class); 
```

上面的代码是将`strList.getClass()获取到的class转化为` Class<? extends List> 这么做似乎没有什么意义，因为我们很清楚`strList.getClass()`获取的class对象就是`ArrayList`，它当然是`List.class`的一个子类；

但有些情况下，我们并不能确知一个 `class`对象的类型，典型的情况就是 `Class.forName()`获取到的对象，例如，在 `Android`中：

```java
 Class<? extends View> clazz = Class.forName(name, false,
                        context.getClassLoader()).asSubclass(View.class);
```

当 `name`是View的子类时，正常执行，当不是View的子类时，会抛出 `ClassCastException` ，这时我们可以做别的处理。

**作用：**

`asSubclass`用户窄化未知的Class类型的范围。