### Retrofit解析

[TOC]

#### 结构

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200318193404.png)

#### 1.接口是如何实现的

##### 1.1 接口的实现类从哪里来的

通过动态代理生成实现类（Proxy.newProxyInstance(classloader,interfaces [], invocationHandler )）

最终生成代理类的方法是：ProxyGenerator.genarateProxyClass(className, interefaces [], modifiers)

java官方内置的这套方案只针对接口实现动态代理，如果是抽象类则需要采用其他的实现方案。

##### 1.2 代理的其他实现方案

cglib方案

#### 2.如何处理请求

##### 2.1 如何解析并配置参数

```java
// 类型是 concorrentHashMap，是线程安全的
serviceMethodCache.get(method);
// 解析注解
serviceMethod.parseAnnocations(retrofit, method);
```

##### 2.2 如何实现自定义类型转换

自定义Converter

Converter.Factory

##### 2.3 如何动态切换`BaseURL`

#### 3.如何处理响应

##### 3.1 如何适配返回结果类型

##### 3.2 如何支持 `RxJava`

##### 3.3 如何支持 `Kotlin` 协程的 `Deferred`

##### 3.4 如何获取返回结果的泛型实参

##### 3.5 如何在反序列化时实例化对象

##### 3.6 如何支持 `Kotlin` 协程的 `suspend` 函数

#### 4.设计模式

##### 4.1 `Builder` 模式

##### 4.2 工厂模式

##### 4.3 适配器模式

##### 4.4 代理模式

#### 5.总结