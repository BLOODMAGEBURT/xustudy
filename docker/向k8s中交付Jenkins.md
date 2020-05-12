### 向k8s中交付Jenkins

示意图（交付Dubbo到k8s中，配置CICD）

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200512100407.png)

#### 1、安装Jenkins的准备工作

##### 1.1、准备镜像

> [jenkins镜像](https://hub.docker.com/r/jenkins/jenkins)
>
> 可以选择最新版本，也可以自选版本
>
> 我选择了一个老版本2.190.3
>
> docker pull jenkins/jenkins:2.190.3

##### 1.2、推送到本地仓库

```shell
docker tag  22b8b9a84dbe harbor.od.com/public/jenkins:v2.190.3

docker push harbor.od.com/public/jenkins:v2.190.3
```

##### 1.3、自定义jenkins的Dockerfile

```dockerfile
From harbor.od.com/public/jenkins:v2.190.3
USER root
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo 'Asia/Shanghai' > /etc/timezone
ADD id_rsa /root/.ssh/id_rsa
ADD config.json /root/.docker/config.json
ADD get-docker.sh /get-docker.sh
RUN echo " StrickHostKeyChecking no" >> /etc/ssh/ssh_config &&\
    /get-docker.sh
```



#### 2、在k8s集群里启动Jenkins容器



#### 3、配置Jenkins,安装插件

