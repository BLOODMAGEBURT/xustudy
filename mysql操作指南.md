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

     ```mysql
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

     