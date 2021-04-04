### Handler机制

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

