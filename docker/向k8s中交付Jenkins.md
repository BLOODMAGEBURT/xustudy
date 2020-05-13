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
COPY id_rsa /root/.ssh/id_rsa
COPY config.json /root/.docker/config.json
COPY get-docker.sh /get-docker.sh
RUN echo " StrictHostKeyChecking=no" >> /etc/ssh/ssh_config &&\
    /get-docker.sh
```

这个Dockerfile里我们主要做了以下几件事：

- 设置容器用户为root
- 设置容器内的时区为东八区
- 将ssh私钥加入容器（使用git拉代码时需要用到，配对的公钥配置在gitlab中）
- 容器中加入登陆harbor私有仓库的config文件，内含auth,可以免密登陆
- 在容器内部安装一个docker客户端
- 修改容器内ssh客户端的配置，访问新主机时，不用输入yes no

##### 1.4、生成本地密钥id_rsa

```shell
# -C 指定自己的邮箱
ssh-keygen -t rsa -b 2048 -C "xubobo@gmail.com" -N "" -f /root/.ssh/id_rsa
```

##### 1.5、打包镜像

```shell
# 创建文件夹
mkdir -p /data/dockerfile/jenkins
cd /data/dockerfile/jenkins
# 将本地文件，拷贝到此文件夹
cp /root/.ssh/id_rsa .
cp /root/.docker/config.json .
# 下载get-docker.sh
curl -fsSL get.docker.com -o get-docker.sh
chmod +x get-docker.sh
# 将上面的Dockerfile复制进来

# 先去harbor.od.com创建一个infra私有项目

# 执行打包镜像
docker build -t harbor.od.com/infra/jenkins:v2.190.3 .
# 推动镜像到私有仓库
docker push harbor.od.com/infra/jenkins:v2.190.3

# 测试运行jenkins容器
docker run --rm harbor.od.com/infra/jenkins:v2.190.3 ssh -T git@gitee.com
```



#### 2、在k8s集群里启动Jenkins容器

##### 2.1、创建infra命名空间

```shell
# 在任意运算节点上执行：
$ kubectl create namespace infra
# 由于是从harbor的私有仓库拉镜像，所以需要配置secret, 在dp中使用imagePullSecrets
$ kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=Harbor12345 -n infra
```

##### 2.2、准备共享存储

运维主机，以及所有运算节点上执行：

```shell
$ yum install nfs-utils -y 
```

配置NFS服务

```shell
# 运维主机上
vim /etc/exports
/data/nfs-volume 10.4.7.0/24(rw,no_root_squash)
```

启动NFS服务

```shell
# 运维主机上
mkdir -p /data/nfs-volume
systemctl start nfs
systemctl enable nfs
```

##### 2.3、jenkins的资源配置清单

```shell
# 在运维主机
mkdir /data/nfs-volume/jenkins_home
mkdir /data/k8s-yaml/jenkins
cd /data/k8s-yaml/jenkins

# 配置资源清单
```

 vim dp.yaml

```yaml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: jenkins
  namespace: infra
  labels:
    name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      name: jenkins
  template:
    metadata:
      labels:
        app: jenkins
        name: jenkins
    spec:
      volumes:
      - name: data
        nfs:
          server: hdss7-200
          path: /data/nfs-volume/jenkins_home
      - name: docker
        hostPath:
          path: /run/docker.sock
          type: ''
      containers:
      - name: jenkins
        image: harbor.od.com/infra/jenkins:v2.190.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx512m -Xms512m
        volumeMounts:
        - name: data
          mountPath: /var/jenkins_home
        - name: docker
          mountPath: /run/docker.sock
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```

vim svc.yaml

```yaml
kind: Service
apiVersion: v1
metadata:
  name: jenkins
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  selector:
    app: jenkins
```

vim ingress.yaml

```yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: jenkins
  namespace: infra
spec:
  rules:
  - host: jenkins.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: jenkins
          servicePort: 80
```

##### 2.4、启动jenkins

在任意运算节点执行

```shell
kubectl create -f http://k8s-yaml.od.com/jenkins/dp.yaml
kubectl create -f http://k8s-yaml.od.com/jenkins/svc.yaml
kubectl create -f http://k8s-yaml.od.com/jenkins/ingress.yaml
```

配置域名：在named中配置jenkins.od.com

#### 3、配置Jenkins,安装插件

##### 3.1、安全配置

Manage Jenkins - Configure Global Security - 

勾选：

- Allow anonymous read aeccess

去掉：

- Prevent Cross Site Request Forgery exploits

##### 3.2、安装CICD插件blue ocean

插件管理-下载blue ocean

