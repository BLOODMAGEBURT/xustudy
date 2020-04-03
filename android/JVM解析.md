### JVM解析

[TOC]

#### 1、jvm学习方向

##### 1.1 为什么要打破双亲委派机制 （类加载）

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200401232730.png)

**类加载器分类**：

a .启动类加载器

*负责加载JRE核心类库，比如rt.jar charsets.jar*

b .扩展类加载器

*负责加载JRE扩展目录ext中jar类包*

c .系统类加载器

*负责加载classpath路径下的类包*

d .用户自定义加载器

##### 1.2 运行时数据区（内存结构）

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200402232725.png)

- 程序计数器

  指向 *当前线程* 正在执行的字节码指令的地址（行号）

  目的：记录每个线程的执行进度

  <img src="https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200402234359.png" style="zoom:50%;" />

- 本地方法栈

   栈的特性就是先进后出，而方法调用正好满足这个特性

  <img src="https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200402235015.png" style="zoom:50%;" />

- 虚拟机栈

  存储 **当前线程** 运行方法所需的数据、指令、返回地址

  类中的每一个方法对应一个栈帧

  <img src="https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200402235739.png" style="zoom:50%;" />

  

##### 1.3 GC怎么调优？

