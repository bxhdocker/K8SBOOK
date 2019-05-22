# ES搭建

**公司采用的kubernetes集群中的日志收集解决方案**

**为第三种**

将所有的Pod的日志都挂载到宿主机上，每台主机上单独起一个日志收集Pod,通过filebeat进行采集

**优点:**完全解耦，性能最高，管理起来最方便

**缺点:**需要统一日志收集规则，目录和输出方式

## 官方简介：

由于es占用资源过大，使用独立机器建立

es官网:[https://www.elastic.co/cn/products/elasticsearch](https://www.elastic.co/cn/products/elasticsearch)

官方安装说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)

github地址：[https://github.com/elastic/elasticsearch](https://github.com/elastic/elasticsearch)

6.3版本下载：[https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-3-0](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-3-0)



## 安装

通过rpm包安装Elasticsearch

系统：centos7.4 64

安装版本：es6.2.0

安装要求

需要安装最新版本的Java

### JDK安装

查询yum源支持的jdk的rpm包

```text
yum list | grep jdk
```

安装jdk-1.8.0版本

```text
yum -y install java-1.8.0-openjdk*
```

安装后，执行java -version

```text
root@localhost ~]# java -version
openjdk version "1.8.0_144" 
OpenJDK Runtime Environment (build 1.8.0_144-b01)
```

安装成功。

_作者：Zzooper 来源：CSDN 原文：_[_https://blog.csdn.net/qq\_27739989/article/details/78047106_](https://blog.csdn.net/qq_27739989/article/details/78047106) __

### ES安装

下载安装

```text
wget -c https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.rpm
```

