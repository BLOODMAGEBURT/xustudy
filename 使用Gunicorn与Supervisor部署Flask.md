### 使用Gunicorn与Supervisor部署Flask

[TOC]



> 假如我们的项目目录在： /root/xubobo/project
>
> 项目结构如下图
>
> ![项目结构](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20191224152058.png)



#### 1. 安装virtualenv，激活虚拟环境

```shell
# 进入项目目录
cd /root/xubobo/project

# pip下载virtualenv
pip install virtualenv

# 新建虚拟环境
virtualenv venv

# 激活虚拟环境
source venv/bin/activate

# 激活环境之后，会在命令行前出现(venv字样)
```

<img src="https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20191224151143.png" style="zoom:100%;" />

#### 2. 安装flask, 测试运行

```shell
# 安装flask
pip install flask

# 测试flask
python app.py
# 如果显示的是 Running on -----------等等，说明flask安装启动成功
```

### 3. 安装使用gunicorn

```shell
# 下载安装gunicorn
pip install gunicorn

# 使用gunicorn启动项目
gunicorn -w 3 -b 0.0.0.0:5000 --reload app:app 

------------------------------------------------------------
注意：
-w 3 开启3个进程
-b 0.0.0.0:5000 定义监听5000端口
app:app 启动文件app.py ：项目中的flask应用变量名
–realod 监听到项目文件变动自动重启gunicorn 使之生效
--------------------------------------------------------------
```

#### 4. 安装配置supervisor

> 实现程序的后台守护运行
>
> 也可以实现开机自动重启

```shell
# 下载安装supervisor， 请注意此时所在目录还是项目目录 /root/xubobo/project
pip install supervisor

# 初始化配置文件
echo_supervisord_conf > supervisor.conf

# 此时目录下多了一个配置文件， 修改此配置文件
vim supervisor.conf
# 在配置文件最底部加入如下配置
[program: githook]
command=/root/xubobo/githook/venv/bin/gunicorn -w 3 -b 0.0.0.0:5000 app:app   ; 启动命令
directory=/root/xubobo/githook                                   ; 项目的文件夹路径
startsecs=0                                                      ; 启动时间
stopwaitsecs=0                                                   ; 终止等待时间
autostart=true                                                   ; 是否自动启动
autorestart=true                                                 ; 是否自动重启
stdout_logfile=/data/python/SMT/log/gunicorn.log                 ; log 日志
stderr_logfile=/data/python/SMT/log/gunicorn.err                 ; 错误日志

# 然后保存退出vim界面

# 指定配置文件来启动supervisord
supervisord -c supervisor.conf

# 检查状态
supervisorctl -c supervisor.conf status
# 可以看到 githook 应用已经是RUNNING状态了
```

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com//mkdown20191224162111.png)

##### 4.1 supervisor的常用命令

```shell
# 通过配置文件启动supervisor
supervisord -c supervisor.conf       

# 察看supervisor的状态
supervisorctl -c supervisor.conf status

# 重新载入 配置文件
supervisorctl -c supervisor.conf reload

# 启动指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf start [all]|[appname]

# 关闭指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]
```

