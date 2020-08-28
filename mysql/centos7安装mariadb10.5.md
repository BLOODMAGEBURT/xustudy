### centos7安装mariadb10.5

#### 1、卸载旧版本

```shell
rpm -qa | grep mariadb
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200828103823.png)

使用yum命令删除以上三个

```shell
yum remove mariadb-server-5.5.60-1.el7_5.x86_64
 
yum remove mariadb-5.5.60-1.el7_5.x86_64
 
yum remove mariadb-libs-5.5.60-1.el7_5.x86_64

```

#### 2、创建MariaDB.repo

在目录下 /etc/yum.repos.d/ 创建文件： MariaDB.repo

以上是官方源，这里我们用阿里源，内容如下：

```repo
[mariadb]
name=MariaDB
baseurl=http://mirrors.aliyun.com/mariadb/yum/10.3/centos7-amd64
gpgkey= http://mirrors.aliyun.com/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck = 1
```

#### 3、安装MariaDB

```shell
yum install MariaDB-server MariaDB-client

```

#### 4、启动MariaDB

```shell
systemctl start mariadb
# 设置开机自启
systemctl enable mariadb
```

#### 5、进行简单配置

输入以下命令：

```shell
mysql_secure_installation
```

设置密码

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200828104637.png)

#### 6、测试登陆

```shell
mysql -uroot -p
```

#### 7、配置mariaDB相关字符集

##### 7.1、文件/etc/my.cnf

添加如下内容：

```mysql
[mysqld]
 
init_connect='SET collation_connection = utf8mb4_general_ci'
 
init_connect='SET NAMES utf8mb4'
 
character-set-server=utf8mb4
 
collation-server=utf8mb4_general_ci
 
skip-character-set-client-handshake

```

##### 7.2、文件文件/etc/my.cnf.d/mysql-clients.cnf

在[mysql]中添加

```mysql
default-character-set=utf8mb4
```

全部配置完成，重启mariadb

```
systemctl restart mariadb
```

之后进入MariaDB查看字符集

```mysql
mysql> show variables like "%character%";show variables like "%collation%";
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200828111614.png)

#### 8、添加用户、设置权限

创建用户命令

```mysql
mysql>create user xubobo@192.168.1.% identified by 'passwd';
```

或 直接创建用户并授权的命令

```mysql
mysql>

grant all privileges on *.* to xubobo@192.168.1.% identified by 'passwd';

```

或 授予外网登陆权限

```mysql
mysql>grant all privileges on *.* to username@'%' identified by 'password';

```

最后

```mysql
# 刷新权限
FLUSH PRIVILEGES;
```

**注意：**

其中只授予部分权限把 其中 all privileges或者all改为

select,insert,update,delete,create,drop,index,alter,grant,references,reload,shutdown,process,file其中一部分

```mysql
grant select,insert,update,delete on redmine1.* to jira@"%" identified by "jira";
# ON 子句中*.* 说明符的意思是“所有数据库，所有的表”
```

