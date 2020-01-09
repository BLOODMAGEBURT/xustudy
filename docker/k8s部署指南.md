### `k8s`部署指南

> 准备工作
>
> 准备3台机器，系统`centos7`
>
>  一台master（192.168.1.145）
>
> 两台node（192.168.1.146，192.168.1.147）
>
> 关闭防火墙，关闭`selinux`
>
> 在三台机器的hosts中配置解析
>
> `192.168.1.145 k8s-master`
> `192.168.1.146 k8s-node1`
> `192.168.1.147 k8s-node2`

------

##### 配置master节点

- 配置`etcd`

  ```shell
  # 安装
  yum install etcd -y
  vim /etc/etcd/etcd.conf
   # 修改
  ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
  ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.145:2379"
   # 启动etcd服务
  systemctl start etcd.service
   # 设置开机自启
  systemctl enable etcd.service
  
  # 测试
  etcdctl set testdir/testkey0 0
  etcdctl get testdir/testkey0
  etcdctl -C http://192.168.1.145:2379 cluster-health
  
  ```

- 配置`kubernetes-master.x86_64`

  ```shell
  # 安装
  yum install kubernetes-master.x86_64 -y
  # 修改API-SERVER
  vim /etc/kubernetes/apiserver
  KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
  KUBE_API_PORT="--port=8080"
  KUBELET_PORT="--kubelet-port=10250" # 如果master还打算做node节点
  KUBE_ETCD_SERVER="--etcd-server=http://192.168.1.145:2379"
  
  # 修改 CONTROLLER-MANAGER, SCHEDULER
  vim /etc/kubernetes/config
  KUBE_MASTER="--master=http://192.168.1.145:8080"
  
  # 启动服务
  systemctl start kube-apiserver.service
  systemctl start kube-controller-manager.service
  systemctl start kube-scheduler.service
  # 设置开机自启
  systemctl enable kube-apiserver.service
  systemctl enable kube-controller-manager.service
  systemctl enable kube-scheduler.service
  
  # 验证
  kubectl get componentstatus
  # 返回结果
  NAME                 STATUS    MESSAGE             ERROR
  etcd-0               Healthy   {"health":"true"}   
  scheduler            Healthy   ok                  
  controller-manager   Healthy   ok 
  ```

------



#### 配置node节点

- ​	配置`kubernetes-node.x86_64`，在`192.168.1.145`

  ```shell
  # 安装
  yum install kubernetes-node.x86_64 -y
  
  # 修改
  vim /etc/kubernetes/kubelet
  KUBELET_ADDRESS="--address=192.168.1.145"
  	# 与上一个配置相同的port
  KUBELET_PORT="--PORT=10250" 
  	# 可以用ip,也可以用hosts解析过的主机名
  KUBELET_HOSTNAME="--hostname-override=k8s-master" 
  KUBELET_API_SERVER="--api-server=http://192.168.1.145:8080"
  
  # 启动服务
  systemctl start kubelet.service  # 启动时会自动启动docker服务
  systemclt start kube-proxy.service
  # 设置服务开机自启
  systemctl enable kubelet.service
  systemctl enable kube-proxy.service
  
  # 验证, 在master节点
  kubectl get nodes
      # 结果
      NAME         STATUS    AGE
      k8s-master   Ready     1m
  		
  ```

- 配置`kubernetes-node.x86_64`，在`192.168.1.146`

  ```shell
  # 安装
  yum install kubernetes-node.x86_64 -y
  
  # 修改
  vim /etc/kubernetes/config
  KUBE_MASTER="--master=http://192.168.1.145:8080"
  
  
  # 修改
  vim /etc/kubernetes/kubelet
  KUBELET_ADDRESS="--address=192.168.1.146"
  	# 与上一个配置相同的port
  KUBELET_PORT="--PORT=10250" 
  	# 可以用ip,也可以用hosts解析过的主机名
  KUBELET_HOSTNAME="--hostname-override=k8s-node1" 
  KUBELET_API_SERVER="--api-server=http://192.168.1.145:8080"
  
  # 启动服务
  systemctl start kubelet.service  # 启动时会自动启动docker服务
  systemclt start kube-proxy.service
  # 设置服务开机自启
  systemctl enable kubelet.service
  systemctl enable kube-proxy.service
  
  # 验证， 在maser节点
  kubectl get nodes
      # 结果
      NAME         STATUS     AGE
  	k8s-master   Ready      32m
  	k8s-node1    Ready     6m
  ```

- 配置flannel网络插件，所有node节点

  ```shell
  # 安装
  yum install flannel -y
  
  # 修改
  vim /etc/sysconfig/flanneld
  FLANNEL_ETCD_ENDPOINTS="http://192.168.1.145:2379"
   # key prefix 可不修改，在下面配置网段需要使用
   FLANNEL_ETCD_PREFIX="/atomic.io/network"
   
  # 配置网段,master节点
  etcdctl set /atomic.io/network/config '{"Network":"172.16.0.0/16"}'
  
  # 启动服务
  systemctl start flanneld.service
  # 设置开机自启
  systemctl enable flanneld.service
  
  # 重启docker服务，目的是使dokcer的网段与flannel保持一致
  systemctl restart docker
  
  # 验证
  ifconfig
  # 结果会多出来一个flannel网卡，ip地址在172.16.xxx.xxx
  # docker的网段也会变为 172.16.xxx.xxx
  
  
  # 所有node节点安装启动之后， 测试跨主机容器之间的通信，使用busybox
  docker pull busybox
  # 查看系统中的镜像，找到busybox的Image ID
  docker images
  # 运行busybox容器, 根据上一步查找的Image Id,替换成自己的
  docker run -it image-id
  # 此时已经进入镜像， 运行ifconfig,可以看到此镜像的ip
  ifconfig
  
  # 在镜像中ping 另外的两个ip，发现不通，这是因为docker的iptables规则drop不通
  ############
  # 解决方式就是在docker的启动文件中，配置iptables规则
  
  # 退出容器，查看docker的启动文件， systemctl status docker
  
  vim /usr/lib/systemd/system/docker.service
  # 在ExecStart之上加入下面一行，容器启动后，执行命令
  ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
  # 修改配置文件之后，重新加载生效
  systemctl daemon-reload
  ############
  ```

  ------

  

#### 常见资源使用

- pod资源

  ```shell
  # yaml文件, nginx_pod.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      app: web
  spec:
    containers:
      - name: nginx
  	  image: nginx:1.17.2
  	  ports:
  	    - containerPort: 80
  
  # 创建资源
  kubectl create -f nginx_pod.yaml
  # 此时会报错，No API token found for service account "default"
  # 修改 api server的配置来修复此错误, 去掉 ServiceAccount
  vim /etc/kubernetes/apiserver
  KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
  # 重启apiserver
  systemctl restart kube-apiserver.service
  # 重新创建资源
  kubectl create -f nginx_pod.yaml
  
  # 验证
  kubectl get pods
  # 结果
  NAME      READY     STATUS              RESTARTS   AGE
  nginx     0/1       ContainerCreating   0          2m
  
  # 但是状态一直是ContainerCreating，所以排错命令如下
  kubectl describe pod nginx
  # 镜像拉取错误
  ```

  ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160242.png)

  ```shell
  # 修改kubelet镜像下载地址,改为docker官方镜像
  # 先查看是nginx被调度到哪个节点了
  kubectl get pod nginx -o wide 
  # 修改 此node下的 kubelet
  vim /etc/kubernetes/kubelet
  KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=docker.io/tianyebj/pod-infrastructure:latest"
  # 重启 kubelet服务
  systemctl restart kubelet.service
  
  # 然后等一会就可以重新获取pods
  kubectl get pods
  # 结果 就是 running 状态了
  NAME      READY     STATUS    RESTARTS   AGE
  nginx     1/1       Running   0          28m
  
  # 总结
  # k8s中创建一个pod资源，k8s会控制docker至少启动两个容器，一个业务容器nginx,一个pod容器
  # 业务容器的网络类型是container，与pod容器共用一个ip
  
  ```



------

#### 私有仓库HARBOR



------

私有仓库registry

```shell
# 下载安装,在master节点
docker pull registry
# 启动容器, 会自动创建挂载目录 myregistry
docker run -d -p 5000:5000 --restart=always --name registry -v ~/k8s/myregistry:/var/lib/registry registry
# 上传镜像到私有仓库
    # 先打tag
    docker tag docker.io/tianyebj/pod-infrastructure:latest 192.168.1.145:5000/pod-infrastructure:latest
    # push到私有仓库
    docker push 192.168.1.145:5000/pod-infrastructure:latest
	# 上传有可能会报错 server gave HTTP response to HTTPS client
	vim /etc/docker/daemon.json
	{
	"insecure-registries":["192.168.1.145:5000"],
	}
	# 重启docker
	systemctl restart docker

```

- RC资源

  > 应用托管在`K8s`之后，`k8s`要保证应用能够持续运行，而这就是RC的工作内容
  >
  > 他会确保任何时间`k8s`中都有指定数量的pod在运行
  >
  > 在此基础上，RC还提供了一些更高级的特性
  >
  > 比如滚动升级，升级回滚等

  ```shell
  # nginx_rc.yaml
  apiVersion: v1
  kind: ReplicationController
  metadata:
    name: myweb
  spec:
    replicas: 2 # pod的启动数量
    selector:
      app: myweb
    template: # pod的启动模板
      metadata:
        labels:
          app: myweb
      spec:
        containers:
          - name: myweb
            image: 192.168.1.145:5000/nginx:1.17.2
            ports:
              - containerPort: 80
     
     
   # 创建rc
   kubectl create -f nginx_rc.yaml
   # 验证
   kubectl get rc
   kubectl get pods 
   
   # rc 与 pod 的关联是通过label 也即是 标签
   # selector lablels
   
   
   -------------------------------------------------------------------
   
   # rc的滚动升级 nginx_rc2.yaml
  apiVersion: v1
  kind: ReplicationController
  metadata:
    name: myweb2
  spec:
    replicas: 2 # pod的启动数量
    selector:
      app: myweb2
    template: # pod的启动模板
      metadata:
        labels:
          app: myweb2
      spec:
        containers:
          - name: myweb2
            image: 192.168.1.145:5000/nginx:1.17.3
            ports:
              - containerPort: 80
              
    # 升级命令-回滚命令也是这个
    kubectl rolling-update myweb -f nginx-rc2.yaml --update-period=30s
    
    # 升级过程中出现问题的回滚命令
    kubectl rolling-update myweb myweb2 --rollback
  ```

  