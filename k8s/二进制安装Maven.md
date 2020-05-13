### 二进制安装Maven

> [Maven官网](https://maven.apache.org/docs/history.html)
>
> [下载地址](https://archive.apache.org/dist/maven/maven-3/)

#### 1、下载解压

在200运维主机：

```shell
# 下载3.6.1版本
cd /opt/src/
wget https://archive.apache.org/dist/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz

# 解压到目录
tar zxf apache-maven-3.6.1-bin.tar.gz -C /data/nfs-volume/jenkins_home/

# 新建文件夹,文件夹8u23是jenkenins容器中的java版本（进入jenkins容器查看版本 java -version）
mv /data/nfs-volume/jenkins_home/apache-maven-3.6.1 /data/nfs-volume/jenkins_home/maven-3.6.1-8u232

```

##### 2、配置

设置国内源

```xml
# 配置
/data/nfs-volume/jenkins_home/maven-3.6.1-8u232/conf/setting.xml

<mirror>  
  <id>alimaven</id>  
  <name>aliyun maven</name>  
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>;  
  <mirrorOf>*</mirrorOf>          
</mirror>

```

