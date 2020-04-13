### node节点docker重启之后，一直NOTREADY

#### 1. 查看node上kubelet的日志

```shell
# 查看日志
journalctl -fu kubelet
# 报错信息
User "system:node:ecs-b11c-0003.novalocal" cannot get resource "leases" in API group "coordination.k8s.io" in the namespace "kube-node-lease": can only access node lease with the same name as the requesting node

nodes "k8s-node02-loadblancer" is forbidden: node "ecs-b11c-0003.novalocal" is not allowed to modify node "k8s-node02-loadblancer"

```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200409114510.png)