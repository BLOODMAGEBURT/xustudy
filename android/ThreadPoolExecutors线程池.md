### ThreadPoolExecutors线程池

[TOC]

#### 1.提交的流程

先使用核心线程 **core**

核心线程满后，添加到队列中等待（如果队列不满，就等待核心线程执行完上个任务后 从队列中取任务） **max-core**

队列满后，创建非核心线程执行任务 **queue** 

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210401221841.png)





**注意：** 

一个线程池最多能执行 queue + max 个任务，超过的话会触发拒绝策略

当队列不满时，**不会**创建非核心线程，队列中的任务等待核心线程取任务

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210401222239.png)

#### 2.执行的流程

先执行核心线程

如果有非核心线程 则再执行非核心线程

然后从队列中取任务 执行队列中的任务

#### 3.线程的存活时间

核心线程一直存活

非核心线程执行完任务后，根据初始化线程池时设置的存活时间确定存活时效

#### 4.execute和submit的区别

其实submit方法中最终也是调用的execute来执行任务

只是submit有返回值，返回一个future类型的runnable

而execute没有返回值