### centos7中Supervisor安装使用

- yum安装

  yum install supervisor

- 启动supervisor服务

  ```shell
  sudo supervisord -c /etc/supervisord.conf
  sudo supervisorctl -c /etc/supervisord.conf
  ```

- 配置

  ```shell
  vim /etc/supervisord.conf
  
  [program:lightreader]
  command=/home/workspace/LightReader/venv/bin/gunicorn -w 2 -b 0.0.0.0:5000 debug_server:app --reload -t 500 -D --access-logfile log/gunicorn.log
  directory=/home/workspace/LightReader
  user=root
  autostart=true
  autorestart=true
  stopasgroup=true
  killasgroup=true
  ```

  以上启动命令的含义为：

  - -w 50 开启50个进程
  - 0.0.0.0:5000 定义5000端口
  - vuln:app debug_server为项目的文件名，如上面的debug_server.py文件名，app为debug_server.py代码中 app = Flask(__name__)
  - –realod 监听到项目文件变动自动重启gunicorn 使之生效
  - -t 500 配置每个请求的超时时间为500秒
  - -D 让命令后台执行
  - –access-logfile log/gunicorn.log 将请求日志保存到该文件中
  - `command`，`directory`和`user`设置告诉supervisor如何运行应用程序

  

