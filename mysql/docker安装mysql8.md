### docker安装mysql8

#### 1、下载镜像

```shell
docker pull mysql:8.0.21
```

#### 2、新建挂载文件夹

```shell
# 数据文件夹
mkdir -p /root/xubobo/data/mysql/data
# 配置文件夹
mkdir -p /root/xubobo/data/mysql/conf
```

#### 3、启动测试容器

查看mysql镜像id

```shell
docker images
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200910144051.png)

```shell
docker run -it --name mysql-xu -e MYSQL_ROOT_PASSWORD=123456 3646af3dc14a
```

拷贝容器中的配置文件到宿主机

```shell
docker cp mysql-xu:/etc/mysql/my.cnf /root/xubobo/data/mysql/conf/

docker cp mysql-xu:/etc/mysql/conf.d /root/xubobo/data/mysql/conf/

```

销毁测试容器

```shell
docker stop mysql-xu
docker rm mysql-xu
```

#### 4、启动正式容器

```shell
docker run -it -d --name mysql-xu \
-e MYSQL_ROOT_PASSWORD=liushaxianzi2580MS \
-v /root/xubobo/data/mysql/conf/my.cnf:/etc/mysql/my.cnf \
-v /root/xubobo/data/mysql/conf/conf.d:/etc/mysql/conf.d \
-v /root/xubobo/data/mysql/data:/var/lib/mysql \
-p 3306:3306 --restart=always \
3646af3dc14a
```

修改容器时区，与宿主机时区保持一致

```shell
docker cp /etc/localtim mysql-xu:/etc/localtime
```

#### 5、连接mysql数据库

如果出现以下问题：

`authentication plugin 'caching_sha2_password' cannot be loaded`

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200910152207.png)

进入容器，执行以下命令：

```shell
# 进入容器
docker exec -it mysql-xu /bin/bash

# 进入数据库
mysql -uroot -p

# 调整用户属性 根据实际情况 修改username ip passwd
ALTER USER 'username'@'ip_address' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```

退出容器，重启容器

```shell
# 容器内执行
exit;
# 宿主机执行
docker restart mysql-xu
```

再次连接，成功了

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200910153123.png)