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
> 在hosts中配置解析
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

  

