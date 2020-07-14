### Docker安装LNMP

Dockerfile

默认安装gd库

```dockerfile
FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd
```

根据dockerfile构建镜像

```shell
docker build -t xubobo/php-fpm:7.4 .
```

启动容器

```shell
docker run -d --name php-fpm-xu xubobo/php-fpm:7.4
```

拷贝配置文件

```
docker cp c999025a6624:/usr/local/etc/php-fpm.d/www.conf .
/usr/local/etc/php/php.ini-production php.ini
```

重新启动容器

```shell
# 先将上一个php容器停掉
# 由于使用同一个网络 web, 所以 --net web
docker run -d --name php-fpm-xu --restart always -p 9000:9000 --net web -v /home/data/www/:/home/data/www -v /root/xubobo/php/php.ini:/usr/local/etc/php/php.ini -v /root/xubobo/php/www.conf:/usr/local/etc/php-fpm.d/www.conf xubobo/php-fpm:7.4 
```

安装redis扩展

```shell
# 进入php-fpm容器
docker exec -it a1f21beddc35 /bin/bash

# 下载解压redis扩展源码
curl -L -o /tmp/reids.tar.gz https://codeload.github.com/phpredis/phpredis/tar.gz/5.0.2
cd /tmp
tar -xzf reids.tar.gz

# 第一次安装扩展之前需要先执行 docker-php-source extract 解压php源码包
docker-php-source extract
# 将redis扩展源码 移动到 php扩展源码路径
mv phpredis-5.0.2 /usr/src/php/ext/phpredis
# 安装扩展
docker-php-ext-install phpredis
# 如果要生效,退出容器，重启容器即可
exit
docker restart php-fpm-xu
```

安装pdo_mysql扩展

```shell
# 由于pdo_mysql扩展源码包已经包含在/usr/src/php/ext/ 路径下了
# 所以直接安装就行
docker-php-ext-install pdo_mysql mysqli

# 如果要生效,退出容器，重启容器即可
exit
docker restart php-fpm-xu
```

如果项目报错： php: mkdir() permission deny

```shell
# 在www.conf中查看php-fpm运行的用户
www-data

# 在php-fpm容器中，将项目用户修改为www-data
chown -R www-data:www-data saaspro
```

