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

#### 修改/etc/sysconfig/elasticsearch系统配置

```text
[root@elasticsearch]#egrep -v "#|^$" /etc/sysconfig/elasticsearch|cat -n
1	ES_PATH_CONF=/etc/elasticsearch
2	PID_DIR=/data1/elastic
3	ES_STARTUP_SLEEP_TIME=5
```

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

