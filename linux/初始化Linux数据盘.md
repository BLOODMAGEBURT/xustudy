### 初始化Linux数据盘

[TOC]

> 本操作以该场景为例，当云主机挂载了一块新的数据盘时，使用fdisk分区工具将该数据盘设为主分区，分区方式默认设置为MBR，文件系统设为ext4格式，挂载在“/mnt/sdc”下，并设置开机启动自动挂载。

#### 1、执行以下命令，查看新增磁盘

```shell
fdisk -l 
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162025.png)

表示当前的云主机有两块磁盘，“/dev/xvda”是系统盘，“/dev/xvdb”是新增数据盘

#### 2、执行以下命令，进入fdisk模式，开始对新增数据盘执行分区操作

```shell
# fdisk 新增数据盘， 以新挂载的数“/dev/xvdb”为例
fdisk /dev/xvdb 
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162158.png)

#### 3、输入【n】，按【Enter】，开始新建分区

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162232.png)

#### 4、输入【p】，按【Enter】，开始创建一个主分区

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162255.png)

#### 5、输入主分区编号，按【Enter】

本步骤中以【1】为例。

​                              屏幕回显如下：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162336.png)

#### 6、以选择默认初始磁柱编号2048为例，按【Enter】

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162419.png)

“Last sector”表示截止磁柱区域，可以选择2048-20971519，默认为20971519

#### 7、以选择默认截止磁柱编号20971519为例，按【Enter】

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162505.png)

**表示分区完成，即为10GB的数据盘新建了1个分区**

#### 8、输入【p】，按【Enter】，查看新建分区

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162557.png)

#### 9、输入【w】，按【Enter】。 将分区结果写入分区表中，分区创建完毕

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162624.png)

表示分区创建完成。如果之前分区操作有误，请输入【q】，则会退出fdisk分区工具，之前的分区结果将不会被保留

#### 10、执行以下命令，将新建分区文件系统设为系统所需格式

```shell
# mkfs -t 文件系统格式 /dev/xvdb1,以设置文件系统为“ext4”为例：
mkfs -t ext4 /dev/xvdb1
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806162748.png)

格式化需要等待一段时间，请观察系统运行状态，不要退出。

**注意：不同文件系统支持的分区个数和分区大小不同，请根据您的业务需求选择合适的文件系统。**

#### 11、执行如下命令，新建挂载点

```shell
# mkdir 挂载点

# 以新建挂载点“/mnt/sdc”为例：

mkdir /mnt/sdc
```

#### 12、执行以下命令，将新建分区挂载到步骤11中新建的挂载点下

```shell
# mount /dev/xvdb1 挂载点

# 以挂载新建分区至“/mnt/sdc”为例：

mount /dev/xvdb1 /mnt/sdc
```

#### 13、执行以下命令，查看挂载结果

```shell
df -TH
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806163039.png)

表示新建分区“/dev/xvdb1”已挂载至“/mnt/sdc”。

#### 14、设置开机自动挂载磁盘

> 如果您需要在云主机系统启动时自动挂载磁盘，不能采用在 /etc/fstab直接指定 /dev/xvdb1的方法，因为云中设备的顺序编码在关闭或者开启云主机过程中可能发生改变，例如/dev/xvdb1可能会变成/dev/xvdb2。推荐使用UUID来配置自动挂载数据盘。
>
> 说明：磁盘的UUID（universally unique identifier）是Linux系统为磁盘分区提供的唯一的标识字符串。

##### 14.1、执行如下命令，查询磁盘UUID

```shell
# blkid 磁盘分区
# 以查询磁盘分区“/dev/xvdb1”的UUID为例：
blkid /dev/xvde1 
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200806163259.png)

##### 14.2、执行以下命令，使用VI编辑器打开【fstab】文件

```shell
vi /etc/fstab

# 添加以下内容
UUID=1851e23f-1c57-40ab-86bb-5fc5fc606ffa /mnt/sdc      ext4 defaults     0   2

# 保存，退出即可
```

#### 15、磁盘管理常用命令

```shell
# 列出系统上的所有磁盘列表，及关系
lsblk
# lsblk 可以看成“ list block device ”的缩写，就是列出所有储存设备的意思！这个工具软件真的很好用喔！来瞧一瞧！
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200814101422.png)

```shell
# 查看磁盘id
blkid /dev/sdb1
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200814101551.png)

```shell
# 查看磁盘详细信息
fdisk -l 
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200814102337.png)

```shell
# 查看磁盘挂载情况
df -TH
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200814102522.png)