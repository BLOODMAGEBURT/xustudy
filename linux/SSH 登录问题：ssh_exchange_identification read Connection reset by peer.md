### SSH 登录问题：ssh_exchange_identification: read: Connection reset by peer

- 原因

  - 估计是被ip被服务器拉黑

    

- 解决思路

  - 放开所有ip访问

    

- 解决步骤分两步

1. 将sshd：ALL添加到/etc/hosts.allow

   vim /etc/hosts.allow

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200107172932.png)

2. 重启sshd

   service sshd restart

