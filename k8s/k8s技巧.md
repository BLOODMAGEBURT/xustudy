### k8s技巧

#### 1、使用kubectl远程连接apiserver

准备k8s用户

签发证书与私钥

```shell
cd /opt/certs
vim admin-csr.json
```

admin-csr.json

```json
{
    "CN": "cluster-admin",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

签发证书

```shell
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client admin-csr.json |cfssl-json -bare admin
```

做kubeconfig配置

任意运算节点

```shell
scp root@hdss7-200:/opt/certs/ca.pem .
scp root@hdss7-200:/opt/certs/admin.pem .
scp root@hdss7-200:/opt/certs/admin-key.pem .
```

创建UA四步骤

```shell
# set-cluster
kubectl config set-cluster myk8s --certificate-authority=./ca.pem --embed-certs=true --server=https://10.4.7.10:7443 --kubeconfig=config

# set-credentials
kubectl config set-credentials cluster-admin --client-certificate=./admin.pem --client-key=./admin-key.pem --embed-certs=true --kubeconfig=config

# set-context
kubectl config set-context myk8s-context --cluster=myk8s --user=cluster-admin --kubeconfig=config

# use-context
kubectl config use-context myk8s-context --kubeconfig=config
```

**会生成一个config文件**

绑定集群角色,赋予ua权限

```shell
kubectl create clusterrolebinding myk8s-admin --clusterrole=cluster-admin --user=cluster-admin
```



拷贝kubectl，拷贝config文件

```shell
# 将config文件复制到/root/.kube/下，如果没有则先创建
cp config /root/.kube/

# 将运算节点的kubectl客户端拷贝过来
scp root@hdss7-21:/opt/kubernetes/server/bin/kubectl /usr/bin/

# 将配置加到环境变量
echo "export KUBECONFIG=/root/.kube/config" >> /etc/profile
source /etc/profile
```

然后就可以使用了

