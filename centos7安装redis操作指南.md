### centos7安装redis操作指南

> yum install redis时碰到 no package redis available centos 7 
>
> 先添加仓库 yum install epel-release
>
> 再升级yum ,运行 yum update，再次尝试就可以了

- 安装redis

  yum install redis

- 启动redis

  systemctl start redis.service

- 设置redis开机启动

  systemctl enable redis.service

- 另外，为了可以使Redis能被远程连接，需要修改配置文件，路径为/etc/redis.conf

  注释这一行：#bind 127.0.0.1

- 推荐给Redis设置密码，取消注释这一行：

  #requirepass foobared

  foobared即当前密码，可以自行修改为

  requirepass 密码

------

如果bind注释之后，也重启了redis还是无法远程链接的话，请查看防火墙规则是否未开放6379

------

## 创建服务

> 用service来管理服务的时候，是在/etc/init.d/目录中创建一个脚本文件，来管理服务的启动和停止，在systemctl中，也类似，文件目录有所不同，在/lib/systemd/system目录下创建一个脚本文件redis.service，里面的内容如下：

​		

~~~shell
[Unit]
Description=Redis
After=network.target

[Service]
ExecStart=/usr/sbin/redis-server /etc/redis.conf  --daemonize yes
ExecStop=/usr/sbin/redis-cli -h 127.0.0.1 -p 6379 shutdown

[Install]
WantedBy=multi-user.target

```````````````````````````````````
[Unit] 表示这是基础信息 
Description 是描述 
After 是在那个服务后面启动，一般是网络服务启动后启动 
[Service] 表示这里是服务信息 
ExecStart 是启动服务的命令 
ExecStop 是停止服务的指令 
[Install] 表示这是是安装相关信息 
WantedBy 是以哪种方式启动：multi-user.target表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。
``````````````````````````````````````
~~~

## 创建软链接

> 创建软链接是为了下一步系统初始化时自动启动服务

```shell
ln -s /lib/systemd/system/redis.service /etc/systemd/system/multi-user.target.wants/redis.service
```

## 刷新配置

```shell
# 刚刚配置的服务需要让systemctl能识别，就必须刷新配置
$ systemctl daemon-reload

```

## 启动、重启、停止

```shell
$ systemctl start redis
$ systemctl restart redis
$ systemctl stop redis
$ systemctl stop redis
```

