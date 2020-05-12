### Linux修改本机时间和时区

在linux中与时间相关的文件有

> /etc/localtime
>
> /etc/timezone

其中，/etc/localtime是用来描述本机时间，而 /etc/timezone是用来描述本机所属的时区。

#### 1、修改本机时间

```shell
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

在/usr/share/zoneinfo下存放着不同时区格式的时间文件，执行以下命令，可以将本机时间调整至目标时区的时间格式。
**但是！调整了时间格式，本机所属的时区是保持不变的！**

#### 2、修改本机时区

在linux中，有一些程序会自己计算时间，不会直接采用带有时区的本机时间格式，会根据UTC时间和本机所属的时区等计算出当前的时间。
比如jdk应用，时区为“Etc/UTC”，本机时间改为北京时间，通过java代码中new 出来的时间还是utc时间，所以必须得修正本机的时区。

```shell
echo 'Asia/Shanghai' >/etc/timezone
```

#### 3、验证本机时间和时区

```shell
date -R 
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200512135603.png)