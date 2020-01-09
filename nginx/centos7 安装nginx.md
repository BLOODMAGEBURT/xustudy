#### centos7 安装nginx

## 	环境说明

CentOS 7

```
$ cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
```



##     步骤

1. ###     **添加yum源**

   ​	Nginx 不在默认的 yum 源中，可以使用 epel 或者官网的 yum 源，本例使用官网的 yum 源。

   ```shell
   $ sudo rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
   
   安装完 yum 源之后，可以查看一下。
   
   $ sudo yum repolist
   Loaded plugins: fastestmirror, langpacks
   Loading mirror speeds from cached hostfile
    * base: mirrors.aliyun.com
    * extras: mirrors.aliyun.com
    * updates: mirrors.aliyun.com
   repo id                          repo name                          status
   base/7/x86_64                    CentOS-7 - Base                    9,911
   extras/7/x86_64                  CentOS-7 - Extras                    368
   nginx/x86_64                     nginx repo                           108
   updates/7/x86_64                 CentOS-7 - Updates                 1,041
   repolist: 11,428
   
   可以发现 nginx repo 已经安装到本机了。
   ```

2. ###  **安装**

    yum 安装 Nginx，非常简单，一条命令

   ```shell
   sudo yum install nginx
   ```

3. ### 配置 Nginx 服务

   1. 设置开机启动

      ```shell
      sudo systemctl enable nginx
      ```

   2. 启动服务

      ```shell
       sudo systemctl start nginx
      ```

   3. 停止服务

      ```shell
       sudo systemctl stop nginx
      ```

   4. 重启服务

      ```shell
       sudo systemctl restart nginx
      ```

   5. 重新加载，因为一般重新配置之后，不希望重启服务，这时可以使用重新加载。

      ```shell
      sudo systemctl reload nginx
      ```

4. ### 打开防火墙端口

   默认 CentOS7 使用的防火墙 firewalld 是关闭 http 服务的（打开 80 端口）。

   ```shell
   $ sudo firewall-cmd --zone=public --permanent --add-service=http
   success
   $ sudo firewall-cmd --reload
   success
   
   打开之后，可以查看一下防火墙打开的所有的服务
   
   $ sudo sudo firewall-cmd --list-service
   ssh dhcpv6-client http
   可以看到，系统已经打开了 http 服务。
   ```



























