#### Subprocess报`FileNotFoundError`

代码如下：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160119.png)

运行时报错，`FileNotFoundError: pipenv`

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160136.png)

解决方案：

> 因为`pipenv`找不到，所以需要指定全路径

​	

```shell
which pipenv
# 结果显示
/root/anaconda3/bin/pipenv
# 因此修改代码中pipenv为全路径的，可成功运行
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109160154.png)

> 另外，报`FileNotFoundError` 的错误，有的有可以通过在`subprocess.Popen()`中配置shell=True 参数来修正
>
> 只是，我使用之后并没有效果



