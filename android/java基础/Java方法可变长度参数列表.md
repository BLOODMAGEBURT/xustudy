#### `Java`方法可变长度参数列表

方法参数中带有三个点 …

```java
public Constructor<T> getConstructor(Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException {
        return getConstructor0(parameterTypes, Member.PUBLIC);
    }
```

从`Java`5开始，叫可变长度参数列表，表示可以1个或多个`Class`类型的对象，或者是一个`Class`[].

多个参数可以使用 “，”分隔，也可以使用数组。

```java
clazz.getConstructor(new Class[]{
            Context.class, AttributeSet.class});
```

