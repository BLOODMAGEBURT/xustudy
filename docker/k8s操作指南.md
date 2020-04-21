### `k8s`操作指南

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200417175440.png)

#### 1、命令自动补全

```shell
# k8s 命令自动补全 
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc


# You can also use a shorthand alias for kubectl that also works with completion:
alias kb=kubectl
complete -F __start_kubectl kb
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

# Get commands with basic output
kubectl get services                          # List all services in the namespace
kubectl get pods --all-namespaces             # List all pods in all namespaces
kubectl get pods -o wide                      # List all pods in the current namespace, with more details
kubectl get deployment my-dep                 # List a particular deployment
kubectl get pods                              # List all pods in the namespace
kubectl get pod my-pod -o yaml                # Get a pod's YAML

# Describe commands with verbose output
kubectl describe nodes my-node
kubectl describe pods my-pod

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
>
> `Ingress` 实际是就是 `Nginx` + 一段 `go` 的脚本
>
> 是 `k8s`  的标准资源之一， 他其实是一组 基于 域名和 `url` 路径，把用户的请求转发至指定 `service`资源的规则

