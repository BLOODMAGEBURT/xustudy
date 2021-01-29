### `Centos7`下同步系统时间

#### 1.设定系统时区

先执行命令`timedatectl status|grep 'Time zone'`查看当前时区，如果不是中国时区（Asia/Shanghai），则需要先设置为中国时区，否则时区不同会存在时差。

```shell
#已经是Asia/Shanghai，则无需设置
timedatectl status|grep 'Time zone'
```

如果不是上海，需要设置时区：

```shell
#设置硬件时钟调整为与本地时钟一致
timedatectl set-local-rtc 1
#设置时区为上海
timedatectl set-timezone Asia/Shanghai
```

#### 2.使用 `ntpdate` 同步时间

目前比较常用的做法就是使用`ntpdate`命令来同步时间，使用方法如下：

```shell
#安装ntpdate
yum -y install ntpdate
#同步时间
ntpdate -u  pool.ntp.org
#同步完成后,date命令查看时间是否正确
date
```

#### 3.定时同步

可能部分服务器过一段时间又会出现偏差，因此可以设置定时任务

```shell
#创建crontab任务
crontab -e
#添加定时任务, 每20分钟同步一次
*/20 * * * * /usr/sbin/ntpdate pool.ntp.org > /dev/null 2>&1 &
#重新加载crontab
systemctl reload crond.service
```

进阶-命令解析：

```shell
在Linux中：

0:表示键盘输入(stdin)

1:表示标准输出(stdout),系统默认是1

2:表示错误输出(stderr)

shell命令：command >/dev/null  2>&1  &  等同于   command 1>/dev/null 2>&1  &

1)command:表示shell命令或一个可执行的程序

2)>:表示重定向到

3)/dev/null:表示Linux的空设备文件

4)2:表示标准错误输出

5)&1:&表示等同于的意思,2>&1,表示2的输出重定向等同于1的重定向

6)&:表示后台执行这条指令

1>/dev/null:表示标准输出重定向到空设备文件,即不输出任何信息到终端。

2>&1:表示错误输出重定向等同于标准输出,因为之前标准输出已经重定向到了空设备文件,所以错误输出也重定向到空设备文件。

上述例子中的shell命令的意思就是在后台执行这个程序,并将错误输出2重定向到标准输出1,然后将标准输出1全部放到/dev/null文件,也就是清空.（shell命令：command >/dev/null  2>&1  &  等同于  command 1>/dev/null 2>&1  &）

" >/dev/null 2>&1 "常用来避免shell命令或者程序等运行中有内容输出。
```

