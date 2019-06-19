### centos7的防火墙配置

```shell
# Enable firewall
systemctl enable firewalld #开机自启动
systemctl start firewalld
systemctl status firewalld


//Disable firewall
systemctl disable firewalld #开机不启动
systemctl stop firewalld # 关闭防火墙
systemctl status firewalld
```

```shell
# 检查防火墙状态
firewall-cmd --state
firewall-cmd --list-all

```



```shell
# 永久的开放需要的端口 3000
sudo firewall-cmd --zone=public --add-port=6379/tcp --permanent
sudo firewall-cmd --reload
# 永久的开放需要的端口 批量
firewall-cmd --permanent --zone=public --add-port=100-500/tcp
sudo firewall-cmd --reload

# 去掉端口
sudo firewall-cmd --zone=public --remove-port=3000/tcp --permanent

# 添加服务名称 ftp
sudo firewall-cmd --permanent --zone=public --add-service=ftp
```

------

### 网络配置

- 静态ip

  `/etc/sysconfig/network-scripts/ifconfig.****`

  ```shell
  BOOTPROTO="static" #dhcp改为static 
  ONBOOT="yes" #开机启用本配置 
  IPADDR=192.168.7.106 #静态IP 
  GATEWAY=192.168.7.1 #默认网关 
  NETMASK=255.255.255.0 #子网掩码 
  DNS1=192.168.7.1 #DNS 配置 
  ```

  重启

  systemctl restart network

