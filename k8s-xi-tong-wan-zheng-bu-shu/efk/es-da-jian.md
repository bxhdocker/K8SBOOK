# ES搭建

**公司采用的kubernetes集群中的日志收集解决方案**

**为第三种**

将所有的Pod的日志都挂载到宿主机上，每台主机上单独起一个日志收集Pod,通过filebeat进行采集

**优点:**完全解耦，性能最高，管理起来最方便

**缺点:**需要统一日志收集规则，目录和输出方式



![](../../.gitbook/assets/image%20%2850%29.png)

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

下载安装包并安装

```text
wget -c https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.rpm
rpm -ivh elasticsearch-6.2.4.rpm
```

安装完后会报出

```text
###不在安装时启动，请执行以下语句以将ElasticSearch服务配置为使用SystemD自动启动
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
###您可以通过执行来启动ElasticSearch服务
sudo systemctl start elasticsearch.service
```

官方安装文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html\#rpm](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#rpm)

{% hint style="info" %}
在基于systemd的发行版上，安装脚本将尝试设置内核参数（例如 `vm.max_map_count`）; 您可以通过屏蔽systemd-sysctl.service单元来跳过此步骤
{% endhint %}



默认情况下，Elasticsearch服务不会在`systemd` 日记中记录信息。要启用`journalctl`日志记录，`--quiet`必须从文件中的`ExecStart`命令行中删除该选项`elasticsearch.service`。

当`systemd`启用了日志记录，日志信息使用可用`journalctl`的命令：

列出elasticsearch服务的日记帐分录：

```text
sudo journalctl --unit elasticsearch
```

 有关更多命令行选项，请查看`man journalctl`或[https://www.freedesktop.org/software/systemd/man/journalctl.html](https://www.freedesktop.org/software/systemd/man/journalctl.html)。

## 配置Elasticsearch

 Elasticsearch默认使用`/etc/elasticsearch`运行时配置。此目录的所有权以及此目录中的所有文件都设置为 `root:elasticsearch`打包安装，并且目录`setgid` 设置了标志，以便创建的所有文件和子目录`/etc/elasticsearch` 也使用此所有权创建（例如，如果使用密钥库创建[密钥库）工具](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)）。预计会对此进行维护，以便Elasticsearch进程可以通过组权限读取此目录下的文件。

 默认情况下，Elasticsearch从`/etc/elasticsearch/elasticsearch.yml`文件加载其配置 。[_配置Elasticsearch中_](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)介绍了此配置文件的格式。

#### 修改`/etc/elasticsearch/elasticsearch.yml配置文件`

```text
[root@centos elasticsearch]# egrep -v "#|^$" elasticsearch.yml
cluster.name: mzlog
node.name: node2
path.data: /data1/elastic/data  #修改存储路径
path.logs: /data1/elastic/logs  #修改日志路径
bootstrap.memory_lock: true  #启用内存锁
network.host: 10.125.0.45
```

注意：修改path.data和path.logs里面后的权限，属主要改为`elasticsearch用户`

#### 修改jvm.options里内存大小

使用6G内存

```text
-Xms6g
-Xmx6g
```

#### 

#### 修改/etc/sysconfig/elasticsearch系统配置

```text
[root@elasticsearch]#egrep -v "#|^$" /etc/sysconfig/elasticsearch|cat -n
1	ES_PATH_CONF=/etc/elasticsearch
2	PID_DIR=/data1/elastic
3	ES_STARTUP_SLEEP_TIME=5
```

修改/etc/security/limits.conf文件内容

```text
在最后增加以下内容
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
```

需要将elasticsearch替换为运行Elasticsearch程序的用户

在/etc/systemd/system/elasticsearch.service.d目录下创建一个文件override.conf，并添加下列内容

```text
[Service]
LimitMEMLOCK=infinity
```

详情我们可以参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html\#systemd

最后重新载入配置文件更新服务

```text
systemctl daemon-reload
systemctl start elasticsearch.service
```

查看elasticsearch的监听端口

```text
# netstat -tnlp |grep java
tcp  0      0 :::9200     :::*     LISTEN      7407/java           
tcp  0      0 :::9300     :::*     LISTEN      7407/java
```

curl命令发送请求来查看elasticsearch是否接收到了数据

```text
# curl http://localhost:9200/_search?pretty
```

```text
ELK默认端口号
elasticsearch：9200 9300
logstash     : 9301
kinaba       : 5601
```

## 故障处理

1[elasticsearch报错之 memory locking requested for elasticsearch process but memory is not locked](https://www.cnblogs.com/FengGeBlog/p/10266148.html)

解决方案：[https://www.cnblogs.com/FengGeBlog/p/10266148.html](https://www.cnblogs.com/FengGeBlog/p/10266148.html)



## ES使用指南

#### 从下面返加的JSON我们可以得到该节点的节点名，所属集群名，ES版本号，lucene版本号。

![](https://img-blog.csdn.net/20170925082840683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGVsaWNpb3VzaW9u/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### 1.查看集群的健康状态。

```text
http://127.0.0.1:9200/_cat/health?v
```

URL中\_cat表示查看信息，health表明返回的信息为集群健康信息，?v表示返回的信息加上头信息，跟返回JSON信息加上?pretty同理，就是为了获得更直观的信息

下面的信息包括：

集群的状态（status）：red红表示集群不可用，有故障。yellow黄表示集群不可靠但可用，一般单节点时就是此状态。green正常状态，表示集群一切正常。

节点数（node.total）：节点数，这里是2，表示该集群有两个节点。

数据节点数（node.data）：存储数据的节点数，这里是2。数据节点在Elasticsearch概念介绍有。

分片数（shards）：这是12，表示我们把数据分成多少块存储。

主分片数（pri）：primary shards，这里是6，实际上是分片数的两倍，因为有一个副本，如果有两个副本，这里的数量应该是分片数的三倍，这个会跟后面的索引分片数对应起来，这里只是个总数。

激活的分片百分比（active\_shards\_percent）：这里可以理解为加载的数据分片数，只有加载所有的分片数，集群才算正常启动，在启动的过程中，如果我们不断刷新这个页面，我们会发现这个百分比会不断加大。

#### 2.查看集群的索引数。

```text
http://127.0.0.1:9200/_cat/indices?v
```

通过该连接返回了集群中的所有索引，其中.kibana是kibana连接后在es建的索引，school是我自己添加的。

这些信息，包括

索引健康（health），green为正常，yellow表示索引不可靠（单节点），red索引不可用。与集群健康状态一致。

状态（status），表明索引是否打开。

索引名称（index），这里有.kibana和school。

uuid，索引内部随机分配的名称，表示唯一标识这个索引。

主分片（pri），.kibana为1，school为5，加起来主分片数为6，这个就是集群的主分片数。

文档数（docs.count），school在之前的演示添加了两条记录，所以这里的文档数为2。

已删除文档数（docs.deleted），这里统计了被删除文档的数量。

索引存储的总容量（store.size），这里school索引的总容量为6.4kb，是主分片总容量的两倍，因为存在一个副本。

主分片的总容量（pri.store.size），这里school的主分片容量是7kb，是索引总容量的一半。



#### 3.查看集群所在磁盘的分配状况

```text
http://127.0.0.1:9200/_cat/allocation?v
```

通过该连接返回了集群中的各节点所在磁盘的磁盘状况

返回的信息包括：

分片数（shards），集群中各节点的分片数相同，都是6，总数为12，所以集群的总分片数为12。

索引所占空间（disk.indices），该节点中所有索引在该磁盘所点的空间。

磁盘使用容量（disk.used），已经使用空间41.6gb

磁盘可用容量（disk.avail），可用空间4.3gb

磁盘总容量（disk.total），总共容量45.9gb

磁盘便用率（disk.percent），磁盘使用率90%



#### 4.查看集群的节点

```text
http://127.0.0.1:9200/_cat/nodes?v
```

通过该连接返回了集群中各节点的情况。这些信息中比较重要的是master列，带\*星号表明该节点是主节点。带-表明该节点是从节点。

## 补充默认设置

RPM还有一个系统配置文件（`/etc/sysconfig/elasticsearch`），允许您设置以下参数：

| `JAVA_HOME` | 设置要使用的自定义Java路径。 |
| :--- | :--- |
| `MAX_OPEN_FILES` | 最大打开文件数，默认为`65535`。 |
| `MAX_LOCKED_MEMORY` | 最大锁定内存大小。`unlimited`如果您使用`bootstrap.memory_lock`elasticsearch.yml中的选项，则 设置为。 |
| `MAX_MAP_COUNT` | 进程可能具有的最大内存映射区域数。如果您使用`mmapfs` 索引存储类型，请确保将其设置为较高的值。欲了解更多信息，请查看 [Linux内核文件](https://github.com/torvalds/linux/blob/master/Documentation/sysctl/vm.txt) 有关`max_map_count`。这是`sysctl`在启动Elasticsearch之前设置的。默认为`262144`。 |
| `ES_PATH_CONF` | 配置文件目录（其中必须包括`elasticsearch.yml`， `jvm.options`，和`log4j2.properties`文件）; 默认为 `/etc/elasticsearch`。 |
| `ES_JAVA_OPTS` | 您可能想要应用的任何其他JVM系统属性。 |
| `RESTART_ON_UPGRADE` | 在程序包升级时配置重新启动，默认为`false`。这意味着您必须在手动安装软件包后重新启动Elasticsearch实例。这样做的原因是为了确保群集中的升级不会导致连续的分片重新分配，从而导致高网络流量并缩短群集的响应时间。 |

使用的分发`systemd`要求通过`systemd`而不是通过`/etc/sysconfig/elasticsearch` 文件来配置系统资源限制。有关更多信息，请参阅[Systemd配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#systemd)。

### RPM目录布局

RPM将配置文件，日志和数据目录放置在基于RPM的系统的适当位置：

| 类型 | 描述 | 默认位置 | 设置 |
| :--- | :--- | :--- | :--- |
| **家** | Elasticsearch主目录或 `$ES_HOME` | `/usr/share/elasticsearch` |  |
| **箱子** | 二进制脚本，包括`elasticsearch`启动节点和`elasticsearch-plugin`安装插件 | `/usr/share/elasticsearch/bin` |  |
| **CONF** | 配置文件包括 `elasticsearch.yml` | `/etc/elasticsearch` | [`ES_PATH_CONF`](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#config-files-location) |
| **CONF** | 环境变量，包括堆大小，文件描述符。 | `/etc/sysconfig/elasticsearch` |  |
| **数据** | 节点上分配的每个索引/分片的数据文件的位置。可以容纳多个位置。 | `/var/lib/elasticsearch` | `path.data` |
| **日志** | 日志文件位置。 | `/var/log/elasticsearch` | `path.logs` |
| **插件** | 插件文件位置。每个插件都将包含在一个子目录中。 | `/usr/share/elasticsearch/plugins` |  |
| **回购** | 共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在此处指定的任何目录的任何子目录中。 | 未配置 | `path.repo` |

#### 后续步骤[编辑](https://github.com/elastic/elasticsearch/edit/7.1/docs/reference/setup/install/next-steps.asciidoc)

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

* 了解如何[配置Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)。
* 配置[重要的Elasticsearch设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)。
* 配置[重要的系统设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html)。

## 文章参考：

{% embed url="https://blog.csdn.net/genghaihua/article/details/81479619" %}

社区文档：[https://elasticsearch.cn/article/6395](https://elasticsearch.cn/article/6395)

