#  Kubernetes 安装部署--使用lubeadm

## Kubernetes 安装部署

1. 实验系统和应用版本介绍

   - 操作系统环境

     ```shell
     [root@k8s-master ~]# cat /etc/redhat-release 
     CentOS Linux release 7.8.2003 (Core)
     关闭防火墙和selinux
     systemctl stop firewalld && systemctl disable firewalld
     sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
     setenforce 0
     ```

   - kubernetes 版本

     这里我使用的是1.17.0版本 后期演如何升级至1.18

   - 实验的服务器

     - Master : 10.0.1.6
     - Node1 : 10.0.1.7
     - Node2 : 10.0.1.8

     ```shell
     Master上执行
     [root@master ~]# hostnamectl set-hostname master
     Node1上执行
     [root@master ~]# hostnamectl set-hostname Node1
     Node2上执行
     [root@master ~]# hostnamectl set-hostname Node2
     cat >> /etc/hosts << EOF
     10.0.1.6 master
     10.0.1.7 node1
     10.0.1.8 node2
     EOF
     ```

2. 准备工作 （三个节点上同时执行）

   - 更改 centos7 国内仓库，加入 docker-ce 仓库，加入kubernetes 仓库，加入 docker 镜像加速配置文件，更改驱动程序为“systemd”，初始化时候会报错，k8s 和 docker 必须驱动是一样的，不然无法启动

     - 更换centos7 系统仓库

       ```shell
       [root@master ~]# yum install -y wget 
       [root@master ~]# cp /etc/yum.repos.d/CentOS-Base.repo{,.backup}
       wget -O /etc/yum.repos.d/CentOS-Base https://mirrors.aliyun.com/repo/Centos-7.repo
       yum makecache fast
       ```

     - 时间同步，不然加入节点时就报错

       ```shell
       yum install -y chrony vim tree net-tools epel-release
       
       vim /etc/chrony.conf
       server ntp1.aliyun.com iburst
       server ntp2.aliyun.com iburst
       
       systemctl enable chronyd.service && systemctl start chronyd.service
       chronyc sources
       timedatectl
       date
       ```

     - 加入docker-ce仓库

       ```
       yum install -y yum-utils device-mapper-persistent-data lvm2
       yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
       ```

     - 加入 kubernetes 仓库

       ```
       cat  > /etc/yum.repos.d/kubernetes.repo <<EOF
       [kubernetes]
       name=Kubernetes
       baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
       enabled=1
       gpgcheck=1
       repo_gpgcheck=1
       gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       EOF
       yum makecache fast
       ```

- 加入 docker 阿里云加速和更改驱动为 systemd

```
mkdir -p /etc/docker
cat > /etc/docker/daemon.json << EOF
{
"registry-mirrors": ["https://mvn9obgp.mirror.aliyuncs.com"],
"exec-opts":["native.cgroupdriver=systemd"]
}
EOF
- 关闭 swap

swapoff  -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

- 配置系统内核参数使流过网桥的流量也进入iptables/netfilter框架中

cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

1. 开始安装 ：准备工作完成后，开始安装 docker-ce；然后安装 kubeadm，kubelet，kubectl 工具，通过 kubeadm 参数安装 kubernetes

```
Master节点：
yum list kubeadm --showduplicates
yum install -y kubeadm-1.17.0 kubelet-1.17.0 kubectl-1.17.0 docker-ce
Node节点（两个节点一样）
yum install -y kubeadm-1.17.0 kubelet-1.17.0  docker-ce
systemctl start docker && systemctl enable docker && systemctl enable kubelet.service && systemctl start kubelet.service
```

1. 通过 kubeadm 加参数初始化 master 端，由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

   - Master节点设置 以及相关参数的解释

   ```
   kubeadm init \
   --control-plane-endpoint=10.0.1.6:6443 \
   --apiserver-advertise-address=10.0.1.6 \
   --image-repository registry.aliyuncs.com/google_containers \
   --kubernetes-version v1.17.0 \
   --service-cidr=192.168.0.0/16 \
   --pod-network-cidr=10.244.0.0/16
   ```

   ```
   –apiserver-bind-port int32 Default: 6443 可以通过这个参数指定API-server的工作端口，默认是6443。
   –config string 可以通过这个命令传入kubeadm的配置文件，需要注意的是，这个参数是实验性质的，不推荐使用。
   –dry-run 带了这个参数后，运行命令将会把kubeadm做的事情输出到标准输出，但是不会实际部署任何东西。强烈推荐！
   -h, --help 输出帮助文档。
   –node-name string 指定当前节点的名称。
   –pod-network-cidr string 通过这个值来设定pod网络的IP地址网段；设置了这个值以后，控制平面会自动给每个节点设置CIDRs（无类别域间路由，Classless Inter-Domain Routing）。
   –service-cidr string Default: “10.96.0.0/12” 设置service的CIDRs，默认为 10.96.0.0/12。
   –service-dns-domain string Default: “cluster.local” 设置域名称后缀，默认为cluster.local
   ```

   根据提示操作

   ```
   mkdir -p $HOME/.kube
   cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   chown $(id -u):$(id -g) $HOME/.kube/config
   ```

1. kubeadm token list

   ```
   openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
   这两条命令获取 token值 和 hash 参数
   kubeadm join 10.0.0.6:6443 --token 6spbm0.sc64hvqj4e19q8fe \
   --discovery-token-ca-cert-hash sha256:f4068403f61dcf48812c229a65f59c63685c5bb05d732a17f4783e3a713c5000
   
   或者使用以下方式 直接获取加入的命令
   
   kubeadm token create --print-join-command
   ```