### samba操作指南

> 解决的问题是：不同操作系统之间的文件共享问题

1. 部署安装 ，启动时开启两个服务 nmb smb

   ![1561109576918](F:\xujianbo\xuStudy\img\1561109576918.png)

   ```shell
   # samba服务依赖rpc服务，所以需要先安装rpc服务
   # 检测是否已安装rpc
   rpm -qa | grep rpc
   ```

2. 修改配置文件，启用网络发现   /etc/samba/smb.conf

   ![1561109606357](F:\xujianbo\xuStudy\img\1561109606357.png)

   

   ![1561109624712](F:\xujianbo\xuStudy\img\1561109624712.png)

3. 创建samba用户

   ![1561109646006](F:\xujianbo\xuStudy\img\1561109646006.png)

4. 创建共享文件夹，修改权限

   ![1561109658263](F:\xujianbo\xuStudy\img\1561109658263.png)
   
5. 如果无法连接，请记得开启防火墙的端口139和445

   ```shell
   firewall-cmd --zone=public --add-port=139/tcp --permanent
   firewall-cmd --zone=public --add-port=445/tcp --permanent
   firewall-cmd --reload
   ```

   

> ![1561109674085](F:\xujianbo\xuStudy\img\1561109674085.png)

