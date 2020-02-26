### centos7编译安装php7.4.3

[TOC]

#### A.安装php

##### 1.下载源码

```shell
wget https://www.php.net/distributions/php-7.4.3.tar.gz

# 可以使用国内镜像站，加速下载速度
wget http://php.net/get/php-7.4.3.tar.gz/from/a/mirror
```

##### 2.解压包

```shell
tar -zxvf php-7.4.3.tar.gz
```

##### 3.安装依赖

```shell
sudo yum install openssl-devel libxml2-devel bzip2-devel glibc glibc-devel\
   libcurl-devel libjpeg-devel libpng-devel freetype-devel \
   libmcrypt-devel recode-devel libicu-devel libzip-devel\
   libxml2-devel sqlite-devel bzip2-devel libcurl-devel libicu-devel
   
# rpm包安装oniguruma
sudo yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/o/oniguruma-5.9.5-3.el7.x86_64.rpm
sudo yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/o/oniguruma-devel-5.9.5-3.el7.x86_64.rpm

# 如果上面这两个包不安装，编译的时候会提示No package ‘oniguruma’ found。
```

##### 4.进入源代码包文件夹，编译、安装

```shell
# 进入源码包
cd php-7.4.3/

# 生成配置文件
./configure --prefix=/usr/local/php7.4 --with-config-file-scan-dir=/usr/local/php7.4/etc/php.d --with-config-file-path=/usr/local/php7.4/etc --enable-mbstring --enable-zip --enable-bcmath --enable-pcntl --enable-ftp --enable-xml --enable-shmop --enable-soap --enable-intl --with-openssl --enable-exif --enable-calendar --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-opcache --enable-fpm --enable-session --enable-sockets --enable-mbregex --enable-wddx --with-curl --with-iconv  --enable-gd-jis-conv --with-openssl --with-pdo-mysql=mysqlnd --with-gettext=/usr --with-zlib=/usr --with-bz2=/usr --with-recode=/usr --with-xmlrpc --with-mysqli=mysqlnd

# 编译
make
# 安装
make install
```

##### 5.复制php-fpm配置文件

```shell
cp /usr/local/php7.4/etc/php-fpm.conf.default /usr/local/php7.4/etc/php-fpm.conf
cp /usr/local/php7.4/etc/php-fpm.d/www.conf.default /usr/local/php7.4/etc/php-fpm.d/www.conf
```

##### 6.复制php配置文件

```shell
cp php.ini-production /usr/local/php7.4/etc/php.ini
```

##### 7.为php添加环境变量

```shell
# 编辑 /etc/profile，文件末尾添加一行
PATH=/usr/local/php7.4/bin:/usr/local/php7.4/sbin:$PATH
```

##### 8.将php添加到sudo环境变量

```shell
# 编辑 /etc/sudoers中的 Defaults secure_path，加上PHP路径 :
/usr/local/php7.4/bin:/usr/local/php7.4/sbin:
```

##### 9.使环境变量生效

```shell
source /etc/profile
```

##### 10.启动PHP:

```shell
php-fpm

# 也可以使用特定配置文件启动
php-fpm -y /usr/local/php7.4/etc/php-fpm.d/9009.conf
# 其中 9009.conf 文件是www.conf复制而来，并将监听端口改为9009
```

##### 11.查看版本，查看扩展

```shell
# 查看版本
php -v
# 查看扩展
php -m
```

#### B.重新编译gd库添加jpeg，png支持

```shell
# 重新编译php7.4.3，不加 --with-gd这些配置
# 进入php源码目录
./configure --prefix=/usr/local/php7.4 --with-config-file-scan-dir=/usr/local/php7.4/etc/php.d --with-config-file-path=/usr/local/php7.4/etc --enable-mbstring --enable-zip --enable-bcmath --enable-pcntl --enable-ftp --enable-xml --enable-shmop --enable-soap --enable-intl --with-openssl --enable-exif --enable-calendar --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-opcache --enable-fpm --enable-session --enable-sockets --enable-mbregex --enable-wddx --with-curl --with-iconv  --enable-gd-jis-conv --with-openssl --with-pdo-mysql=mysqlnd --with-gettext=/usr --with-zlib=/usr --with-bz2=/usr --with-recode=/usr --with-xmlrpc --with-mysqli=mysqlnd
# make&make install


# gd的源码目录（php源码解压目录）
cd /home/www/php-7.4.3/ext/gd 
# 生成configure命令
/usr/local/php7.4/bin/phpize
# 配置
./configure --with-php-config=/usr/local/php7.4/bin/php-config -with-png=/usr/local/png --with-freetype=/usr/local/freetype --with-jpeg=/usr/local/jpeg -with-zlib --with-gd

# 编译
make
# 安装
make install

# 修改php.ini文件,添加扩展
vim /usr/local/php7.4/etc/php.ini
# 添加一句
extension=gd.so

# 重启php-fpm
php-fpm
```

#### C.安装redis扩展

```shell
# 下载
wget http://pecl.php.net/get/redis-3.1.6.tgz
# 解压
tar zxf redis-3.1.6.tgz

# phpize
whereis phpize
/usr/local/php7.4/bin/phpize
# 如果报错执行以下
yum install autoconf
# 配置
./configure --with-php-config=/usr/local/php7.4/bin/php-config
# 编译和安装
make & make install

# 修改php.ini
vim /usr/local/php7.4/etc/php.ini
extension=redis.so

# 验证
php -m

```

