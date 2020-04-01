### Handler解析

[TOC]

#### 1、使用handler中遇到的问题

##### 1.1 不能在子线程中更新UI

##### 1.2 使用不当引起OOM

##### 1.3 Message优化

##### 1.4 子线程中创建handler需要准备looper

##### 1.5 handler处理完消息时，页面销毁了，引起空指针异常

#### 2、handler整体架构

##### 2.1 handler的作用

- 处理延时任务（异步）
- 线程间通信



#### 3、handler源码分析



#### 4、hander框架手写