### vsftpd使用指南

> 工作模式有两种

- 主动模式

  ![1560916512108](img\1560916512108.png)

- 被动模式

  ![1560932355174](img\1560932355174.png)

> 修改工作模式，修改默认端口号

> 添加ftp用户，并指定家目录 ，账号不能用于登录
>
> 要求：不能切换别的目录

- 添加用户，指定家目录，且不能用于shell登录

  ```shell
  # 创建一个 xubobo 用户  家目录为 /home/xubobo
  useradd -d /home/xubobo -s /sbin/nologin xubobo
  # 交互设置密码
  passwd xubobo
  ```

  

- 配置chroot， `/etc/vsftpd/vsftpd.conf`

  ```shell
  # 放开以下三行
  chroot_local_user=YES
  chroot_list_enable=YES
  # (default follows)
  chroot_list_file=/etc/vsftpd/chroot_list
  
  # 添加一行允许家目录具有写权限,不然会登录失败
  allow_writeable_chroot=YES
  
  ```

  > 虚拟用户管理
  >
  > 虚拟用户配置权限：主管有上传下载权限，员工只有下载权限
  
  
  
  ## 目标
  
  ![1561024421675](img\1561024421675.png)
  
  1. 准备工作，建文件设权限
  
     ![1561024497537](F:\xujianbo\xuStudy\img\1561024497537.png)
  
  2. 开启用户权限功能 /etc/vsftpd/vsftpd.conf，定义权限文件地址
  
     ![1561024599022](F:\xujianbo\xuStudy\img\1561024599022.png)
  
  3. 建立虚拟用户密码文件
  
     ![1561024707417](F:\xujianbo\xuStudy\img\1561024707417.png)
  
  4. pam认证
  
     ![1561024805762](F:\xujianbo\xuStudy\img\1561024805762.png)
  
  5. 不允许切出家目录，将用户添加到chroot_list中
  
     ![1561024868290](F:\xujianbo\xuStudy\img\1561024868290.png)
  
  6. 新建权限子配置文件，用于定义主管员工权限
  
     1. ![1561024934744](F:\xujianbo\xuStudy\img\1561024934744.png)
  
     2. 主管文件
  
        ![1561024965988](F:\xujianbo\xuStudy\img\1561024965988.png)
  
     3. 员工文件
  
        ![1561025009672](F:\xujianbo\xuStudy\img\1561025009672.png)

```shell
# 开启虚拟用户登录功能
# 定义权限文件
# 建立虚拟用户密码文件
# 设置pam认证
# 新建权限子配置文件，用于定义主管员工权限
```

