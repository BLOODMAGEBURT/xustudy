### 制作`Dubbo`微服务的底包镜像

在运维主机200上：

```shell
/data/dockerfile/jre8/Dockerfile
```

##### 1、自定义`Dockerfile`

```dockerfile
FROM harbor.od.com/public/jre:8u112
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\
    echo 'Asia/Shanghai' > /etc/timezone
COPY config.yml /opt/prom/config.yml
COPY jmx_javaagent-0.3.1.jar /opt/prom/
WORKDIR /opt/project_dir
COPY entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]
```

`config.yml`

```yml
---
rules:
  - pattern: '.*'
```

`jmx_javaagent-0.3.1.jar`

```shell
# 用来采集监控jvm的信息，发给promethues
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.1/jmx_prometheus_javaagent-0.3.1.jar -O jmx_javaagent-0.3.1.jar
```

`entrypoint.sh`

```shell
#!/bin/sh
M_OPTS="-Duser.timezone=Asia/Shanghai -javaagent:/opt/prom/jmx_javaagent-0.3.1.jar=$(hostname -i):${M_PORT:-"12346"}:/opt/prom/config.yml"
C_OPTS=${C_OPTS}
JAR_BALL=${JAR_BALL}
exec java -jar ${M_OPTS} ${C_OPTS} ${JAR_BALL}
```

给shell添加执行权限

`chmod +x entrypoint.sh`

##### 2、`build`镜像

先在`harbor`中新建一个base公开仓库

执行构建

```shell
docker build -t harbor.od.com/base/jre8:8u112 .
```

推送到harbor仓库

```shell
docker push harbor.od.com/base/jre8:8u112
```

