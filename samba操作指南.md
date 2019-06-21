### samba操作指南

> 解决的问题是：不同操作系统之间的文件共享问题

1. 部署安装 ，启动时开启两个服务 nmb smb

   ![1561110165189](img/1561110165189.png)

   ```shell
   # samba服务依赖rpc服务，所以需要先安装rpc服务
   # 检测是否已安装rpc
   rpm -qa | grep rpc
   ```

2. 修改配置文件，启用网络发现   /etc/samba/smb.conf

   ![1561110201660](img/1561110201660.png)

   

    ![1561110225713](img/1561110225713.png)

3. 创建samba用户

   ![1561110251707](img/1561110251707.png)

4. 创建共享文件夹，修改权限

   ![1561110272164](img/1561110272164.png)

5. 如果无法连接，请记得开启防火墙的端口139和445

   ```shell
   firewall-cmd --zone=public --add-port=139/tcp --permanent
   firewall-cmd --zone=public --add-port=445/tcp --permanent
   firewall-cmd --reload
   ```

   

> ![1561110292990](img/1561110292990.png)

