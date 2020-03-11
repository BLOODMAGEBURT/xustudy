### Nginx操作指南

[TOC]



#### 1.安装

```shell
# centos7安装nginx

# 添加epel的yum源
yum install -y epel-release

# 安装ngin x
yum install nginx
```

#### 2.配置文件

```shell
# 配置文件的地址 /etc/nginx

# 网站文件存放默认目录
/usr/share/nginx/html

# 网站默认站点配置
/etc/nginx/conf.d/default.conf

# 自定义Nginx站点配置文件存放目录
/etc/nginx/conf.d/

# Nginx全局配置
/etc/nginx/nginx.conf
```

#### 3.配置Gzip压缩

##### 3.1修改配置文件

```shell
# 配置在nginx.conf中的http部分

##
# `gzip` Settings
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;
```

##### 3.2参数解析

> 添加gzip_min_length 256;指令，告诉Nginx不要压缩小于256字节的文件。这是非常小的文件，可以不用压缩
> gzip_types是表示压缩的文件类型，还可以添加web字体格式、ioc图标和SVG图像等其他类型文件

##### 3.3配置之后，重启nginx

```shell
# 重启nginx
systemctl restart nginx
```

