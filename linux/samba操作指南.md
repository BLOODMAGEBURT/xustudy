### samba操作指南

> 解决的问题是：不同操作系统之间的文件共享问题， 例如，`linux`与`windows`之间

1. 部署安装 ，启动时开启两个服务 `nmb smb`

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152701.png)

   ```shell
   # samba服务依赖rpc服务，所以需要先安装rpc服务
   # 检测是否已安装rpc
   rpm -qa | grep rpc
   ```

2. 修改配置文件，启用网络发现   /etc/samba/smb.conf

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152740.png)

   

    ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152806.png)

3. 创建samba用户

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152830.png)

4. 创建共享文件夹，修改权限

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152854.png)

5. 如果无法连接，请记得开启防火墙的端口139和445

   ```shell
   firewall-cmd --zone=public --add-port=139/tcp --permanent
   firewall-cmd --zone=public --add-port=445/tcp --permanent
   firewall-cmd --reload
   ```

   

> ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20200109152919.png)

