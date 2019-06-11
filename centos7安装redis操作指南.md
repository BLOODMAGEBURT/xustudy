### centos7安装redis操作指南

> yum install redis时碰到 no package redis available centos 7 
>
> 先添加仓库 yum install epel-release
>
> 再升级yum ,运行 yum update，再次尝试就可以了

- 安装redis

  yum install redis

- 启动redis

  systemctl start redis.service

- 设置redis开机启动

  systemctl enable redis.service

- 另外，为了可以使Redis能被远程连接，需要修改配置文件，路径为/etc/redis.conf

  注释这一行：#bind 127.0.0.1

- 推荐给Redis设置密码，取消注释这一行：

  #requirepass foobared

  foobared即当前密码，可以自行修改为

  requirepass 密码

------

如果bind注释之后，也重启了redis还是无法远程链接的话，请查看防火墙规则是否未开放6379