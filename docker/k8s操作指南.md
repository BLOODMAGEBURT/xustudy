### `k8s`操作指南



#### 1、命令自动补全

```shell
# k8s 命令自动补全 
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```



#### 2、集群操作

获取集群状态

```shell
kubectl get cs
kubectl get clusterrolebinding
kubectl get nodes -o wide
```



#### 3、pod操作

```shell
# 查看yaml文件
kubectl get pod xxx -o yaml
# 获取帮助文档 sevice.metadata的详解
kubectl explain service.metadata

```

#### 4、harbor

```shell
# harbor修改配置文件之后重启
docker-compose down
cd /opt/harbor
./prepate
docker-compose up -d 
```

#### 5、`pod`内容器之间通信 （flannel）

```
#三种模式
gw
Vxlan
Vxlan-true
```

#### 6、服务发现（`coredns`）

> 服务发现：
>
> 就是将`svc`的 名字  与  `clusterIp`  自动的关联在一起

```shell
# 以容器的方式，向k8s中交付软件（coredns）
# 下载
docker pull coredns/coredns:1.6.5

```

#### 7、服务暴露 (Ingress)

> 如何使得服务在`K8S`集群 外 被访问和使用呢？