### Linux配置免密登录

[TOC]



#### 第一、Linux下生成密钥

使用 `ssh-keygen`  然后一路回车，在用户根目录生成 `.ssh` 文件夹

进入 `.ssh` 文件夹显示如下：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200107173114.png)

> authorized_keys:存放远程免密登录的公钥,主要通过这个文件记录多台机器的公钥
>
> `id_rsa` : 生成的私钥文件
>
> `id_rsa.pub` ： 生成的公钥文件
>
> know_hosts : 已知的主机公钥清单
>
> **如果希望ssh公钥生效需满足至少下面两个条件：**
>
> 1) .ssh目录的权限必须是700 
>
> 2) .ssh/authorized_keys文件权限必须是600



#### 第二、免密登录原理

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200107174247.png)



#### 第三、配置免密登录

##### 3.1、通过ssh-copy-id的方式

> 命令： `ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.1.135` 
>
> 或者： `ssh-copy-id  root@192.168.1.135`

实例：

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.1.135

# 然后输入 192.168.1.135的密码即可

# 如果提示命令不存在，安装即可
yum -y install openssh-clients
```



##### 3.2、通过`scp`将内容写到对方的文件中

> 命令：`scp -p ~/.ssh/id_rsa.pub root@host:/root/.ssh/authorized_keys`

实例：

```shell
scp -p ~/.ssh/id_rsa.pub root@192.168.1.135:/root/.ssh/authorized_keys
# 然后输入 192.168.1.135的密码即可
```

#### 第四、配置服务器别名

```shell
# 新建config文件
cd ~/.ssh/
vim config

# 填写服务器配置
Host tx
Hostname 140.143.163.99
Port 22
User root
Identityfile ~/.ssh/id_rsa

# 之后登录就可以使用
ssh tx

```

