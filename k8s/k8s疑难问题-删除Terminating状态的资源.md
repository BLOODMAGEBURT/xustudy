### k8s疑难问题之一-删除Terminating状态的资源

> 删除pv的顺序： pod - pvc - pv
>
> 但是遇到pvc始终处于“Terminating”状态，而且delete不掉。

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200810174326.png)



**通用的解决方式是：**

```shell
kubectl patch pvc pvc-name -p '{"metadata":{"finalizers":null}}' -n namespace
```

一般来说，到这一步，基本也就删掉了。

如果跟我一样，我先把namesapce删掉了，此时执行命令就会报错：namespace not found

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200810174618.png)

所以，要进行重要的一步：新建namespace

```shell
# 新建namespace
kubectl create ns ns-870f85d80aab3294f825c1ac02e65e64

# 再次执行命令
kubectl patch pvc pvc-name -p '{"metadata":{"finalizers":null}}' -n namespace

# 此时，pvc应该已经删除了，新建的ns也就没用了，删掉即可
kubectl delete ns ns-870f85d80aab3294f825c1ac02e65e64
```



