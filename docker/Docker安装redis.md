### Docker安装redis

#### 1、下载镜像

在[dockerHub](https://hub.docker.com/_/redis)查找redis镜像

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200918144145.png)

我使用的是6.0.8版本

```shell
docker pull redis:6.0.8
```

#### 2、准备配置文件

新建文件夹

```shell
mkdir /root/xubobo/data/redis/data

mkdir /root/xubobo/data/redis/conf/
```

[官网](http://download.redis.io/redis-stable/redis.conf)获取`redis.conf`配置文件

将配置文件复制到 `/root/xubobo/data/redis/conf/` 目录下

修改默认的配置文件

- bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问
- protected-mode no #默认yes，开启保护模式，限制为本地访问
- daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程（可选），改为yes会使配置文件方式启动redis失败
- dir  ./ #输入本地redis数据库存放文件夹（可选）
- appendonly yes #redis持久化（可选）
- requirepass #开启密码，修改密码保护，redis容易被入侵

#### 3、启动命令如下：

```shell
docker run -d --name=redis-xu  \
-v /root/xubobo/data/redis/data:/data \
-v /root/xubobo/data/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf  \
-p 6379:6379  --restart=always \
redis:6.0.8 redis-server /usr/local/etc/redis/redis.conf
```



