### crontab 运行 service 命令不起作用

> 需要特别注环境变量的设置，因为我们手动执行某个任务时，是在当前shell环境下进行的，程序当然能找到环境变量，而系统自动执行任务调度时，是不会加载任何环境变量的，因此，就需要在crontab文件中指定任务运行所需的所有环境变量，这样，系统执行任务调度时就没有问题了。

- 例如,以下任务就找不到service

  ​	`52 10 * * * service httpd restart`

  需要写全路径

  有可能是 `/sbin/service` ， 也有可能是  `/usr/sbin/service`

  使用以下命令确定：

  `which service`

  我的系统返回的是：`/sbin/service`

- 正确的路径的service

  `52 10 * * * /sbin/service httpd restart`