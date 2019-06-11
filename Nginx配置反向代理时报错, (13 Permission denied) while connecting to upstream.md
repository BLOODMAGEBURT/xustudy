### centos7中Nginx配置反向代理时报错, (13: Permission denied) while connecting to upstream

> It turns out my issue was due to **SELinux**.
>
> 是SELinux的配置问题引起的

nginx的error.log反应的情况如下：

![nginx](F:\xujianbo\xuStudy\img\nginx.jpg)



可以通过以下操作解决：

```shell
setsebool -P httpd_can_network_connect on
```

-P 参数是持久生效的意思



## Fixing 403 errors when using nginx with SELinux

> I was trying to configure up a new static content directory in nginx (so that I could use the [letsencrypt webroot domain verification method](https://community.letsencrypt.org/t/using-the-webroot-domain-verification-method/1445/47)), but kept getting 403 permission denied errors when accessing any files from the directory.
>
> Eventually tracked it down to SELinux blocking access to the new directory that I'd created because it wasn't part of the policy applied to nginx.
>
> The requests by nginx to read the file could be seen as being blocked in the `/var/log/audit/audit.log` file as:
>
> ![selinux403](F:\xujianbo\xuStudy\img\selinux403.jpg)

he quick fix: change the context of the new directory `/home/yuanxin` so that nginx can read it:

```shell
chcon -Rt httpd_sys_content_t /home/yuanxin/
```

After making this change, nginx can then read the directory and will serve the files from it without any errors.

然后就可以了

