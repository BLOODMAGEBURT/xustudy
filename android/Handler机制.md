### Handler机制

[TOC]



#### 1.handler构造函数初始化mLooper和mQueue

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403215203.png)

#### 2.Looper.myLooper()调用的是ThreadLocal.get();

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403215401.png)

sThreadLocal是Looper类的静态变量，直接初始化的

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403220011.png)

mQueue是MessageQueue是在Looper构造函数中初始化的

messageQueue是一个**链表**实现的队列，入队时 根据message的（当前时间+delay时间）**排序**

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403220118.png)

#### 3.Looper.loop()是启动了MessageQueue

无限循环从queue中取消息msg

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403220914.png)

获取消息后分发消息

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403221854.png)



queue.next()也是无限循环

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210403221011.png)

#### 4.子线程中使用handler

##### 4.1、先调用Looper.prepare()，设置looper

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210404104230.png)

##### 4.2、使用刚才的looper对象，创建handler

```java
new Thread(new Runnable() {
			public void run() {
				Looper.prepare();  // 此处获取到当前线程的Looper，并且prepare()
				Handler handler = new Handler(){
					@Override
					public void handleMessage(Message msg) {
						Toast.makeText(getApplicationContext(), "handler msg", Toast.LENGTH_LONG).show();
					}
				};
				handler.sendEmptyMessage(1);
				
			};
		}).start();
```

##### 4.3、启动Looper

```java
new Thread(new Runnable() {
			public void run() {
				Looper.prepare();
				Handler handler = new Handler(){
					@Override
					public void handleMessage(Message msg) {
						Toast.makeText(getApplicationContext(), "handler msg", Toast.LENGTH_LONG).show();
					}
				};
				handler.sendEmptyMessage(1);
        // 启动Looper
				Looper.loop();
			};
		}).start();
```

#### 5.常见面试题

##### 5.1、子线程中维护的Looper,消息队列无消息时的处理方案是什么？有什么用？主线程能退出吗？

线程睡眠(nativePollOnce())与唤醒机制

Looper.quit();

退出时唤醒线程（nativeWake()）继续循环，当检测到需要退出时，next()返回null 跳出next()循环

当queue.next()返回null时，loop()循环判断 msg == null 也跳出循环，loop()执行完毕。



主线程的Looper不能退出，调用Looper.quit()时会报异常

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210404130455.png)

在ActivityThread中，4大组件都依赖主线程的Handler

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210404131041.png)

##### 5.2、应该怎么创建Message?



##### 5.3、Looper死循环为什么不会导致应用卡死？