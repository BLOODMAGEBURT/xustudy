### Centos7安装php5.2并安装mysql扩展

> 一个老项目，不支持php7,只能使用php5.2,
>
> 所以就开始安装吧

[TOC]



#### 1、步骤

##### 1、下载源码包

```shell
# 新建文件夹,并进入
mkdir /usr/local/php52
cd /usr/local/php52
# 下载源码，并解压
wget http://museum.php.net/php5/php-5.2.17.tar.gz
tar xf php-5.2.17.tar.gz
# 下载两个补丁包并打补丁，不然编译会报错
wget https://php-fpm.org/downloads/php-5.2.17-fpm-0.5.14.diff.gz
curl -o php-5.2.17.patch https://mail.gnome.org/archives/xml/2012-August/txtbgxGXAvz4N.txt

gzip -cd php-5.2.17-fpm-0.5.14.diff.gz | sudo patch -d php-5.2.17 -p1
cd php-5.2.17/
patch -p0 -b <../php-5.2.17.patch
```

##### 2、安装相关依赖

```
yum -y install gcc automake autoconf libtool gcc-c++ gd zlib zlib-devel openssl openssl-devel libxml2 libxml2-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libmcrypt libmcrypt-devel curl-devel
 
ln -s /usr/lib64/mysql /usr/lib/mysql
cp -rfp /usr/lib64/libldap* /usr/lib/
ln -s /usr/lib64/libjpeg.so /usr/lib/libjpeg.so
ln -s /usr/lib64/libpng.so /usr/lib/
```

##### 3、编译安装

```shell
./configure --prefix=/usr/local/php52 --with-iconv-dir=/usr/bin --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir --enable-xml --with-curl --enable-fpm --enable-fastcgi --enable-force-cgi-redirect --enable-mbstring --with-gd --enable-gd-native-ttf --enable-zip

make && make install
```

##### 4、拷贝php配置文件

```shell
cp php.ini-recommended /usr/local/php52/lib/php.ini
```

##### 5、添加用户

```shell
groupadd www
useradd -M -s /sbin/nologin -g www www

# 将php-fpm配置文件中的用户和组改为www这个用户
vim /usr/local/php52/etc/php-fpm.conf
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210111163800.png)

##### 6、启动php-fpm

```shell
/usr/local/php52/sbin/php-fpm start
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210111163934.png)

#### 2、安装mysql扩展

`Fatal error: Uncaught Error: Call to undefined function mysql_connect()`

> 首先确保mysql已经安装在 `/usr/local/mysql`目录下

如果php和mysql都已经安装完成了，可以使用phpize工具手动编译生成mysql.so扩展来解决

##### 1、进入php安装目录的ext/mysql目录下

```
# 如果是跟着我刚才的教程安装的，此时php安装目录结构如下
cd /usr/local/php52/php-5.2.17/ext/mysql
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210111164538.png)

##### 2、运行phpize，在该目录下生成一个configure文件

```
/usr/local/php52/bin/phpize
```

##### 3、运行configure

```shell
# 指明php-config文件位置（/usr/local/php52/bin/php-config）和mysql安装目录（/usr/local/mysql/）

./configure --with-php-config=/usr/local/php52/bin/php-config --with-mysql=/usr/local/mysql/
```

##### 4、编译安装，生成mysql.so

```shell
make && make install

# 此时自动生成mysql.so文件，并自动添加到默认php扩展目录下
# no-debug-non-zts-20060613最后这个文件夹名称可能不一样，各位看一下自己的目录
/usr/local/php52/lib/php/extensions/no-debug-non-zts-20060613
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210111165115.png)

##### 5、修改php配置文件，并重启php-fpm

```shell
vim /usr/local/php52/lib/php.ini

# 修改扩展目录文件夹(根据上一步确定具体目录)
extension_dir = "/usr/local/php52/lib/php/extensions/no-debug-non-zts-20060613"
# 添加一行
extension = mysql.so

# 重启php-fpm
/usr/local/php52/sbin/php-fpm restart
```

##### 6、验证一下扩展安装情况

```shell
/usr/local/php52/bin/php -m
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20210111165840.png)

####  3、老项目的一些问题

##### 1、`**Parse error**: syntax error, unexpected $end in`

如果碰到这个问题，可以修改php.ini配置文件

```shell
# short_open_tag 将这个配置打开On
; short_open_tag = Off ; previous value
short_open_tag = On ; new value

# 修改完配置文件记得，重启php-fpm
```

##### 2、大文件上传配置

```ini
file_uploads = on ;是否允许通过HTTP上传文件的开关。默认为ON即是开
upload_tmp_dir ;文件上传至服务器上存储临时文件的地方
upload_max_filesize = 8m ;允许上传文件大小的最大值。默认为2M
post_max_size = 8m ;表单POST给PHP的所能接收的最大值，包括表单里的所有值 默认为8M

;根据网上的资料，如果上传大于8M的文件，还要改一下时间的设置
max_execution_time = 600 ;每个PHP页面运行的最大时间值(秒)，默认30秒
max_input_time = 600 ;每个PHP页面接收数据所需的最大时间，默认60秒
memory_limit = 8m ;每个PHP页面所吃掉的最大内存，默认8M
把上述参数修改后，在网络所允许的正常情况下，就可以上传大体积文件了
max_execution_time = 600
max_input_time = 600
memory_limit = 32m
file_uploads = on
upload_tmp_dir = /tmp
upload_max_filesize = 32m
post_max_size = 32m
```

