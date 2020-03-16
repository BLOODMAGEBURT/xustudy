### centos7安装docker-compose

[TOC]

#### 1.先安装pip工具

```shell
yum install epel-release

yum install -y python-pip
```

#### 2.安装docker-compose

```shell
pip install -U docker-compose

# 如果安装过程中出现以下问题
# fatal error python.h no such file or directory，解决方式如下
# Python 3.6.6 : 
yum -y install python3-devel
pip install wheel
# Python 2 : 
yum -y install python-devel
pip install wheel
```

#### 3.bash命令自动补全配置

```shell
curl -L https://raw.githubusercontent.com/docker/compose/1.24.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

# 打开一个新窗口测试tab补全
docker-compose s
```

#### 4.卸载

```shell
pip uninstall docker-compose
```

#### 5.使用

##### 5.1、术语

首先介绍几个术语。

- 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

可见，一个项目可以由多个服务（容器）关联而成，`Compose` 面向项目进行管理。

##### 5.2、场景

最常见的项目是 web 网站，该项目应该包含 web 应用和缓存。

下面我们用 `Python` 来建立一个能够记录页面访问次数的 web 网站。

##### 5.3、web 应用

新建文件夹，在该目录中编写 `app.py` 文件

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

##### 5.4、编写Dockerfile

```dockerfile
FROM python:3.6-alpine
COPY . /code
WORKDIR /code
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple redis flask
CMD ["python", "app.py"]
```

##### 5.5、docker-compose.yml

编写 `docker-compose.yml` 文件，这个是 Compose 使用的主模板文件。

```yaml
version: '3'
services:

  web:
    build: .
    ports:
     - "5000:5000"

  redis:
    image: "redis:alpine"
```

##### 5.6、启动

```shell
# 在当前目录
docker-compose up
```

