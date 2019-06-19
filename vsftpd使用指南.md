---
typora-copy-images-to: img
---

### vsftpd使用指南

> 工作模式有两种

- 主动模式

  ![1560916512108](F:\xujianbo\xuStudy\img\1560916512108.png)

- 被动模式

  ![1560932355174](F:\xujianbo\xuStudy\img\1560932355174.png)

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

  