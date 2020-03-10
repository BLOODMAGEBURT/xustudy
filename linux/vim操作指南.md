## vim操作指南

##### 跳转操作

`Ctrl`+`f`和`Ctrl`+`b`是上下翻页

`Ctrl`+`u`和`Ctrl`+`d`是上下翻半页

M将光标移动的该页中部、`gg`回到文件顶部、G回到文件底部、`hjkl`移动光标

跳转到特定行使用     `：n`

#### 修改操作

- 撤销

  在vi中按u可以撤销一次操作

  u   撤销上一步的操作
  `Ctrl+r` 恢复上一步被撤销的操作

- 复制， 粘贴

  yy： 复制光标所在的那一行 
  nyy：n为数字，复制光标所在的向下n行，例如20yy则是复制20行 
  y1G：复制光标所在行到第一行的所有数据 
  yG：复制光标所在行到最后一行的所有数据 
  y0：复制光标所在的那个字符到改行行首的所有数据 

  y$：复制光标所在的那个字符到该行行尾的所有数据  

  p为将已复制的数据在光标下一行粘贴，P则为粘贴在光标上一行 

  yi{ 和 yi( ：可以复制{} ()之内的内容

- 删除

  `dd`：删除光标所在的那一行 
  `ndd`：n为数字，删除光标所在的向下n行，例如`20dd`则是删除20行
  
  di{ 或者 di( ： 可以删除 {} （）之内的内容
  
- 选择

  

## docker 操作指南

##### 查看配置

docker inspect id

## Linux 操作

##### 查看yum安装后的位置	

这里以`hdf5`软件包为例：

```shell
   首先采用 yum install hdf5安装hdf5

    #yum install hdf5

   第二步采用上面步骤1的命令

   ＃rpm -qa|grep hdf5 

   回车后输出  hdf5-1.8.7-1.el6.rf.x86_64 

  第三步采用上面步骤2的命令

  ＃rpm -ql hdf5-1.8.7-1.el6.rf.x86_64
```

##### FTP

1.安装

`yum install vsftpd`

`2、设置开机启动vsftpd ftp服务`

`chkconfig vsftpd on`

`3、启动vsftpd服务(默认ftp服务是没有启动的，用下面命令启动)`

```shell
service vsftpd start
```

##### `增加用户ftpuser，指向目录/home/wwwroot/ftpuser,禁止登录SSH权限`

`useradd -d /home/guide-backup/tech -g ftp -s /sbin/nologin xujianbo`

1、设置用户口令，密码输入两次

```shell
passwd xujianbo
```

2、`编辑文件chroot_list:`

`vi /etc/vsftpd/chroot_list`
`内容为ftp用户名,每个用户占一行,如：`

```shell
xujianbo
john
```