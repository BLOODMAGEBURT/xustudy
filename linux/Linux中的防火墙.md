### Linux中的防火墙

#### 1.总体结构如下图：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200429105509.png)

#### 2.区别

[FirewallD](http://www.firewalld.org/) 即Dynamic Firewall Manager of Linux systems，Linux系统的动态防火墙管理器，是 iptables 的前端控制器，用于实现持久的网络流量规则。它提供命令行和图形界面，在大多数 Linux 发行版的仓库中都有。与直接控制 iptables 相比，使用 FirewallD 有两个主要区别：

1. FirewallD 使用区域和服务而不是链式规则。
2. FirewallD可以动态修改单条规则，而不需要像iptables那样，在修改了规则后必须得全部刷新才可以生效。

在RHEL7里有几种防火墙共存：FirewallD、iptables、ebtables，默认是使用FirewallD来管理netfilter子系统，不过底层调用的命令仍然是iptables等。

**FirewallD跟iptables比起来，不好的地方是每个服务都需要去设置才能放行，因为默认是拒绝。而iptables里默认是每个服务是允许，需要拒绝的才去限制。**

FirewallD自身并不具备防火墙的功能，而是和iptables一样需要通过内核的netfilter来实现，也就是说FirewallD和 iptables一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的netfilter，只不过FirewallD和iptables的结构以及使用方法不一样罢了。



iptables 服务在 /etc/sysconfig/iptables 中储存配置，而 FirewallD 将配置储存在 /usr/lib/firewalld/ 和 /etc/firewalld/ 中的各种 XML 文件里，使用 iptables 的时候每一个单独更改意味着清除所有旧有的规则和从 /etc/sysconfig/iptables 里读取所有新的规则，使用 firewalld 却不会再创建任何新的规则；仅仅运行规则中的不同。因此 FirewallD 可以在运行时改变设置而不丢失现行配置。

#### 3.区域管理

通过将网络划分成不同的区域，制定出不同区域之间的访问控制策略来控制不同程序区域间传送的数据流。例如，互联网是不可信任的区域，而内部网络是高度信任的区域。网络安全模型可以在安装，初次启动和首次建立网络连接时选择初始化。该模型描述了主机所连接的整个网络环境的可信级别，并定义了新连接的处理方式。有如下几种不同的初始化区域：

- 阻塞区域（block）：任何传入的网络数据包都将被阻止。
  任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。
- 工作区域（work）：相信网络上的其他计算机，不会损害你的计算机。
- 家庭区域（home）：相信网络上的其他计算机，不会损害你的计算机。
- 公共区域（public）：不相信网络上的任何计算机，只有选择接受传入的网络连接。
- 隔离区域（DMZ）：隔离区域也称为非军事区域，内外网络之间增加的一层网络，起到缓冲作用。对于隔离区域，只有选择接受传入的网络连接。
- 信任区域（trusted）：所有的网络连接都可以接受。
- 丢弃区域（drop）：任何传入的网络连接都被拒绝。
  任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。
- 内部区域（internal）：信任网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。
- 外部区域（external）：不相信网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。

注：FirewallD的默认区域是public。

FirewallD默认提供了九个zone配置文件：block.xml、dmz.xml、drop.xml、external.xml、 home.xml、internal.xml、public.xml、trusted.xml、work.xml

**他们都保存在“/usr/lib /firewalld/zones/”目录下。**

#### 4.配置方法

FirewallD的配置方法主要有三种：

1. firewall-config
2. firewall-cmd
3. 直接编辑xml文件

其中 firewall-config是图形化工具，firewall-cmd是命令行工具，而对于linux来说大家应该更习惯使用命令行方式的操作，所以 firewall-config（适合用于桌面版）我们就不给大家介绍了。

#### 5.常用操作

##### 5.1、查看区域信息:

```shell
$ firewall-cmd --get-active-zones

# 查看指定接口所属区域：
$ firewall-cmd --get-zone-of-interface=eth0

# 将接口添加到区域，默认接口都在public：
$ firewall-cmd --zone=public --add-interface=eth0

# 设置默认接口区域为public,也可以设置为别的
$ firewall-cmd --set-default-zone=public
```

##### 5.2、更新防火墙规则：

```shell
#两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务
$ firewall-cmd --reload
$ firewall-cmd --complete-reload
```

##### 5.3、打开端口：

```shell
# 查看所有打开的端口：
$ firewall-cmd --zone=public --list-ports

# 加入一个端口到区域：
$ firewall-cmd --zone=public --add-port=8080/tcp

# 移除一个端口
$ firewall-cmd --remove-port=3306/tcp # 阻止通过tcp访问3306

# 如果要永久生效
# 永久生效再加上 --permanent 然后reload防火墙
$ firewall-cmd --zone=public --add-port=8080/tcp --permanent
$ firewall-cmd --reload
```

##### 5.4、打开一个服务：

其实一个服务对应一个端口，每个服务对应/usr/lib/firewalld/services下面一个xml文件，服务需要在配置文件中添加

```shell
$ firewall-cmd --zone=public --add-service=smtp

# 移除服务
$ firewall-cmd --zone=public --remove-service=smtp

# 查看当前打开了哪些服务
firewall-cmd --list-services

# 查看还有哪些服务可以打开
firewall-cmd --get-services
```

##### 5.5、伪装IP

防火墙可以实现伪装IP的功能，下面的端口转发就会用到这个功能。

```shell
$ firewall-cmd --query-masquerade # 检查是否允许伪装IP
$ firewall-cmd --permanent --add-masquerade # 允许防火墙伪装IP
$ firewall-cmd --permanent --remove-masquerade # 禁止防火墙伪装IP
$ firewall-cmd --reload # 更新规则
```

##### 5.6、端口转发

端口转发可以将指定地址访问指定的端口时，将流量转发至指定地址的指定端口。

**转发的目的如果不指定ip的话就默认为本机，如果指定了ip却没指定端口，则默认使用来源端口。**

如果配置好端口转发之后不能用，可以检查下面两个问题：

- 比如我将80端口转发至8080端口，首先检查本地的80端口和目标的8080端口是否开放监听了
- 其次检查是否允许伪装IP，没允许的话要开启伪装IP

```shell
# 将80端口的流量转发至8080
$ firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080
# 将80端口的流量转发至
$ firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.0.1
# 将80端口的流量转发至192.168.0.1的8080端口
$ firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.0.1:toport=8080
```

端口转发的作用：

- 当我们想把某个端口隐藏起来的时候，就可以在防火墙上阻止那个端口访问，然后再开一个不规则的端口，之后配置防火墙的端口转发，将流量转发过去。
- 端口转发还可以做流量分发，一个防火墙拖着好多台运行着不同服务的机器，然后用防火墙将不同端口的流量转发至不同机器。

