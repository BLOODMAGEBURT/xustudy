### openssl升级最新版（Centos7）

#### 1、查看当前版本

```shell
# 查看版本
openssl version

# 查看路径
which openssl

# 安装依赖
 yum -y install perl perl-devel gcc gcc-c++
```

#### 2、下载最新的openssl

```shell
# 进入/usr/local/src/目录
cd /usr/local/src/
# 下载最新的openssl
wget https://github.com/openssl/openssl/archive/OpenSSL_1_1_1c.tar.gz
# 解压
tar -zxvf openssl-1.1.1c.tar.gz
```

#### 3、编译安装

```shell
# 进入解压后的目录
cd openssl-1.1.1c
# 配置指定安装路径
./config --prefix=/usr/local/openssl
# 编译安装
make && make install
```

#### 4、替换当前系统的旧版本

```shell
# 先保存原来的版本,原来的版本路径可以通过第一步的查看路径获得
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/lib64/openssl /usr/lib64/openssl.old
mv /usr/lib64/libssl.so /usr/lib64/libssl.so.old

# 建立软连接
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so /usr/lib64/libssl.so
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
# 链接生效
ldconfig -v 
```

#### 5、验证openssl的版本

```shell
openssl version
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20201221104135.png)

