### Docker安装LNMP

Dockerfile

默认安装gd库

```dockerfile
FROM php:7.4-fpm
RUN usermod -u 1000 www-data && apt-get update && apt-get install -y \
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
docker cp c999025a6624:/usr/local/etc/php/php.ini-production php.ini
```

重新启动容器

```shell
# 先将上一个php容器停掉
# 由于使用同一个网络 web, 所以 --net web
docker run -d --name php-fpm-xu \
--restart always -p 9000:9000 \
-v /home/data/nginx/data:/home/data \
-v /home/data/php7.4/php.ini:/usr/local/etc/php/php.ini \
-v /home/data/php7.4/www.conf:/usr/local/etc/php-fpm.d/www.conf \
xubobo/php-fpm:7.4 
```

> **需要注意的是 PHP-FPM 容器的网站目录必须与 nginx 内的一致**，否则会有不必要的麻烦
>
> 比如：file not found "Primary script unknown" while reading response header from upstream"

> 另外这里使用了 PHP 官方的包，采用了7.4版本，各版本的 Tags 可以在[官方Docker Hub页面](https://hub.docker.com/_/php/)中查询。

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

#### NGINX的安装

准备nginx.conf

```nginx
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user www-data;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # gzip
    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 32k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
    gzip_disable "MSIE [1-6].";

    server_names_hash_bucket_size 128;
    client_max_body_size     100m;
    client_header_buffer_size 256k;
    large_client_header_buffers 4 256k;


    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

}
```

准备vhost文件

```nginx
server {
  listen 80;
  server_name www.saasv3.com saasv3.com;
  root /home/data/saaspro/;  # 请注意，这个地址是容器中的地址
  set $root root;
  index index.html index.htm index.php;

  location / {
    if ( !-e $request_filename) {
            rewrite ^(.*)jicai/([A-Za-z]+)/([A-Za-z0-9]+)\.html$ $1/index.php/privates/index/$2.html?uid=$3 last;
            rewrite  ^(.*)$  /index.php?s=/$1  last;
            break;
        }
  }

  location ~ \.php$ {
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
  }

  location ~ /\.ht {
    deny all;
  }
}

```



```shell
docker run -d --name nginx-xu \
-v /home/data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/data/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/data/nginx/data:/home/data \
-v /home/data/nginx/logs:/var/log/nginx \
--restart=always  --privileged=true --net=host \
nginx:1.19
```

如有必要，修改nginx的用户id

```shell
# 宿主机查看www-data的用户id
id www-data
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200817170919.png)

结果显示为1012

所以进入nginx容器，修改www-data的用户id和组id

```shell
# 进入容器
docker exec -it nginx-xu /bin/bash

# 修改用户和组id
usermod -u 1012 www-data
groupmod -g 1012 www-data

# 退出容器
exit

# 重启容器生效
docker restart nginx-xu
```

