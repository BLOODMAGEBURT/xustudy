### `mysql`操作指南

- 重置root密码,安全模式重置法

  1. 停掉`MySQL  sudo service mysqld stop`

  2. 安全模式启动  `sudo mysqld_safe --skip-grant-tables --skip-networking &`

  3. root登录，无须密码：  `mysql -uroot`

  4. 重置密码

     ```mysql
     use mysql; 
     update user set authentication_string=password('实际密码') where user='root';
     	(或者5.7之前：update user set password=PASSWORD("123456") where User='root';  
     	8.0修改： ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '新密码'）
     flush privileges;
     ```

  5. 注释配置

     ```mysql
      vim /etc/my.cnf
      #注释掉
      skip-grant-tables
     ```

  6. 重启`mysql: service mysqld restart`

     ------

     

- 远程连接mysql

  端口号不是3306时 ,使用大写的-P指定端口号：  `mysql -h192.168.1.1 -P3308 -uroot -p`

  ------

  

- 开启远程访问

  > 天翼云阿里云腾讯云连不上：注意安全组`tcp 3306`端口规则
  > 注意查看防火墙是否关闭了3306端口
  > `MySQL`是否启动

  1. 例如，你想root使用123456从任何主机连接到`mysql`服务器。

     ​	

     ```mysql
     use mysql;
     GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
     FLUSH PRIVILEGES；
     ```

  2. 如果你想允许用户jack从`ip`为10.10.50.127的主机连接到`mysql`服务器，并使用654321作为密码

     ```mysql
     use mysql;
     GRANT ALL PRIVILEGES ON *.* TO 'jack'@’10.10.50.127’ IDENTIFIED BY '654321' WITH GRANT OPTION;
     FLUSH PRIVILEGES;
     ```

     ------

     

- 配置

  1. 配置文件地址

     `/etc/my.cof`

  2. 表名忽略大小写`mysqld`后面添加

     `lower_case_table_names=1`

  3. 链接`mysql`较慢，可以配置不去解析，在`mysqld`下面添加

     `skip-name-resolve`

  4. 支持中文，编码格式改为utf8

     ```shell
     修改 /etc/my.cnf
     [mysqld]
     collation-server = utf8_unicode_ci
     init-connect='SET NAMES utf8'
     character-set-server = utf8
     
     [client]
     default-character-set=utf8
     
     [mysql]
     default-character-set=utf8
     ```

      数据表调整

     `ALTER TABLE TABLE_NAME CONVERT TO CHARACTER set utf8`

     ------
     
     ### Docker安装Mysql
     
     #### 下载镜像
     
     `docker pull mariadb:latest`
     
     #### 运行镜像
     
     `docker run --name mysql -e MYSQL_ROOT_PASSWORD='xxxx'  -v /home/workxu/mysql/lib:/var/lib/mysql   -v /home/workxu/mysql/conf/my.cnf:/etc/mysql/my.cnf -d -i -p 3306:3306 --restart=always  --privileged=true  mariadb:latest`
     
     #### 进入容器
     
     `docker exec -ti mysql bash`
     
     #### 退出容器
     
     exit
     
     ------
     
     ### Centos7安装MariaDB
     
     #### 第一步，Yum安装
     
     ```shell
     yum install -y mariadb-server
     ```
     
     #### 第二步，启动服务，开机自启服务
     
     ```shell
     systemctl start mariadb
     
     # 开机自启
     systemctl enable mariadb
     ```
     
     #### 第三步，初始化配置
     
     ```shell
      # 交互执行初始化密码等操作
      mysql_secure_installation
      
      
      Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
      New password: <– 设置root用户的密码
      Re-enter new password: <– 再输入一次你设置的密码
     
      其他配置
     
      Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
     
      Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,
     
      Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
     
      Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车
      
     ```
     
     #### 第四步，配置`conf`文件,修改默认的`charset`
     
     ```shell
     # 更改 /etc/my.cnf.d/client.cnf 文件
     [client] 下加一行配置 default-character-set=utf8
     
     # 更改 /etc/my.cnf.d/mysql-clients.cnf 文件
     [mysql] 下加一行配置 default-character-set=utf8
     
     # 更改 /etc/my.cnf.d/server.cnf 配置
     [mysqld] 下加配置
     
     collation-server = utf8_general_ci
     init-connect='SET NAMES utf8'
     character-set-server = utf8
     sql-mode = TRADITIONAL
     ```
     
     