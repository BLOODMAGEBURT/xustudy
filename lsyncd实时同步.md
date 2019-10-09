#### `lsyncd`实时同步[Live Syncing (Mirror) Daemon]

> `Lysncd 实际上是lua语言封装了 inotify 和 rsync 工具，采用了 Linux 内核（2.6.13 及以后）里的 inotify 触发机制，然后通过rsync去差异同步，达到实时的效果。我认为它最令人称道的特性是，完美解决了 inotify + rsync海量文件同步带来的文件频繁发送文件列表的问题 —— 通过时间延迟或累计触发事件次数实现。另外，它的配置方式很简单，lua本身就是一种配置语言，可读性非常强。lsyncd也有多种工作模式可以选择，本地目录cp，本地目录rsync，远程目录rsyncssh`

1. 安装

   ```shell
   # 先安装 epel-release
   yum install -y epel-release
   # 安装 lsyncd
   yum install -y lsyncd
   
   ```

2. 配置

   ~~~shell
   # 配置文件
   vim /etc/lsyncd.conf
   # 按需修改
   ````````````````````````````````````
   # setttings是全局设置
       logfile 定义日志文件
       stausFile 定义状态文件
       nodaemon=true 表示不启用守护模式，默认
       statusInterval 将lsyncd的状态写入上面的statusFile的间隔，默认10秒
       inotifyMode 指定inotify监控的事件，默认是CloseWrite，还可以是Modify或CloseWrite or Modify
       maxProcesses 同步进程的最大个数。假如同时有20个文件需要同步，而maxProcesses = 8，则最大能看到有8个rysnc进程
       maxDelays 累计到多少所监控的事件激活一次同步，即使后面的delay延迟时间还未到
   # sync是局域设置
       default.rsync ：本地目录间同步，使用rsync，也可以达到使用ssh形式的远程rsync效果，或daemon方式连接远程rsyncd进程；
       default.direct ：本地目录间同步，使用cp、rm等命令完成差异文件备份；
       default.rsyncssh ：同步到远程主机目录，rsync的ssh模式，需要使用key来认证
   `````````````````````````````````````
   
   settings {
           logfile = "/var/log/lsyncd/lsyncd.log",
           statusFile = "/var/log/lsyncd/lsyncd.status"
   }
   sync {
       default.rsyncssh,
       source = "/root/linsir", --源目录
       host = "192.168.2.16", --目的主机
       targetdir = "/root/remote", --远程目录
       delete = true,
       delay = 0,
       excludeFrom = "/etc/rsyncd.d/rsync_exclude.lst",
       rsync = {
              binary = "/usr/bin/rsync",
              archive = true, --归档
               compress = true, --压缩
               verbose = true, 
              owner = true,   --属主
               perms = true,   --权限
               _extra = {"--bwlimit=2000"},
               },
           ssh = {
               port = 3322
               }
   }
   ~~~

3. 配置免密登录

   ~~~shell
   # 三步走
   # 第一步，在本地电脑生成密钥
   ````````````````````````
   一路使用enter键确认
   ````````````````````````
   ssk-keygen
   # 第二步，将公钥复制到远程服务器中
   ssh-copy-id -i ~/.ssh/id_rsa.pub  user@192.168.x.xxx
   
   ```````````
   注意: ssh-copy-id 将key写到远程机器的 ~/ .ssh/authorized_key.文件中
   ```````````
   # 第三步: 登录到远程机器不用输入密码
   ssh user@192.168.x.xxx
   ~~~

4. 启动`lsyncd`服务

   ```shell
   # 启动
   systemctl start lsyncd
   # 开机启动
   systemctl enable lsyncd
   ```

   

