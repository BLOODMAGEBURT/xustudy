### centos7 安装 docker

[TOC]

------



#### 1.环境说明

Centos7

#### 2.卸载旧版本

旧版本的 Docker 被叫做 `docker` 或 `docker-engine`，如果您安装了旧版本的 Docker ，您需要卸载掉它。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 3.安装

##### 3.1安装准备

为了方便添加软件源，支持 devicemapper 存储类型，安装如下软件包

```shell
# 先升级本地环境
sudo yum update
# 安装软件包
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

##### 3.2添加yum软件源

添加 Docker 稳定版本的 yum 软件源

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 也可以使用阿里云的源，速度会快一点
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

##### 3.3正式安装Docker

更新一下 yum 软件源的缓存，并安装 Docker。

```shell
sudo yum update
# 安装最新版Docker
sudo yum install docker-ce docker-ce-cli containerd.io
```

> **注意**：
>
> 默认的 docker 组是没有用户的（也就是说需要使用 sudo 才能使用 docker 命令）。
> 你可以将用户添加到 docker 组中（此用户就可以直接使用 docker 命令了）。

将当前登录用户加入docker用户组

```shell
sudo usermod -aG docker $USER

# 用户更新组信息后，重新登录系统即可生效。
```

##### 3.4启动Docker

```shell
# 配置docker开机自启
sudo systemctl enable docker
# 启动docker服务
sudo systemctl start docker
```

##### 3.5验证安装

验证 Docker CE 安装是否正确，可以运行 `hello-world` 镜像

```shell
 sudo docker run hello-world
```

如果如下图所示，即安装启动成功

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200108160623.png)

##### 3.6更新卸载Docker

使用 yum 管理，更新和卸载都很方便。

```shell
# 更新 Docker CE
sudo yum update docker-ce

# 卸载 Docker CE
sudo yum remove docker-ce

# 删除本地文件
# 注意docker 的本地文件，包括镜像(images), 容器(containers), 存储卷(volumes)等，都需要手工删除。
#默认目录存储在 /var/lib/docker。

sudo rm -rf /var/lib/docker
```

#### 4.修改国内源（可选，但建议）

> Docker很多镜像动不动就1G或几百M，官方经常掉线。所以只能换国内源。

国内的镜像源有以下：

- docker官方中国区：`https://registry.docker-cn.com`
- 网易：`http://hub-mirror.c.163.com`
- ustc：`http://docker.mirrors.ustc.edu.cn`
- 阿里云：`http://<你的ID>.mirror.aliyuncs.com`

在此，我们使用网易的源

通用的方法就是编辑`/etc/docker/daemon.json`：

```shell
# 进入文件
vim /etc/docker/daemon.json

# 修改文件

{
  "registry-mirrors" : [
    "http://hub-mirror.c.163.com"
  ],
  "insecure-registries" : [
    "registry.docker-cn.com",
    "docker.mirrors.ustc.edu.cn"
  ],
  "debug" : true,
  "experimental" : true
}

# 重启 docker 服务
sudo systemctl restart docker
```

#### 5.安装docker-compose（可选）

##### 5.1下载

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```



##### 5.2赋权限

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

##### 5.3实验是否正常

```shell
docker-compose

# 如下显示，即为正常
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109145250.png)