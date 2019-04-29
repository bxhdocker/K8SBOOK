# 非K8S主机部署Filebat

**filebeat介绍**

　　Filebeat由两个主要组成部分组成：prospector和 harvesters。这些组件一起工作来读取文件并将事件数据发送到您指定的output。

什么是harvesters？  
　　harvesters负责读取单个文件的内容。harvesters逐行读取每个文件，并将内容发送到output中。每个文件都将启动一个harvesters。harvesters负责文件的打开和关闭，这意味着harvesters运行时，文件会保持打开状态。如果在收集过程中，即使删除了这个文件或者是对文件进行重命名，Filebeat依然会继续对这个文件进行读取，这时候将会一直占用着文件所对应的磁盘空间，直到Harvester关闭。默认情况下，Filebeat会一直保持文件的开启状态，直到超过配置的close\_inactive参数，Filebeat才会把Harvester关闭。

关闭Harvesters会带来的影响：  
 　　file Handler将会被关闭，如果在Harvester关闭之前，读取的文件已经被删除或者重命名，这时候会释放之前被占用的磁盘资源。  
 　　当时间到达配置的scan\_frequency参数，将会重新启动为文件内容的收集。  
 　　如果在Havester关闭以后，移动或者删除了文件，Havester再次启动时，将会无法收集文件数据。  
 　　当需要关闭Harvester的时候，可以通过close\_\*配置项来控制。

什么是Prospector？

　　Prospector负责管理Harvsters，并且找到所有需要进行读取的数据源。如果input type配置的是log类型，Prospector将会去配置度路径下查找所有能匹配上的文件，然后为每一个文件创建一个Harvster。每个Prospector都运行在自己的Go routine里。

　　Filebeat目前支持两种Prospector类型：log和stdin。每个Prospector类型可以在配置文件定义多个。log Prospector将会检查每一个文件是否需要启动Harvster，启动的Harvster是否还在运行，或者是该文件是否被忽略（可以通过配置 ignore\_order，进行文件忽略）。如果是在Filebeat运行过程中新创建的文件，只要在Harvster关闭后，文件大小发生了变化，新文件才会被Prospector选择到。

**filebeat工作原理**　　

Filebeat可以保持每个文件的状态，并且频繁地把文件状态从注册表里更新到磁盘。这里所说的文件状态是用来记录上一次Harvster读取文件时读取到的位置，以保证能把全部的日志数据都读取出来，然后发送给output。如果在某一时刻，作为output的ElasticSearch或者Logstash变成了不可用，Filebeat将会把最后的文件读取位置保存下来，直到output重新可用的时候，快速地恢复文件数据的读取。在Filebaet运行过程中，每个Prospector的状态信息都会保存在内存里。如果Filebeat出行了重启，完成重启之后，会从注册表文件里恢复重启之前的状态信息，让FIlebeat继续从之前已知的位置开始进行数据读取。

Prospector会为每一个找到的文件保持状态信息。因为文件可以进行重命名或者是更改路径，所以文件名和路径不足以用来识别文件。对于Filebeat来说，都是通过实现存储的唯一标识符来判断文件是否之前已经被采集过。　　

如果在你的使用场景中，每天会产生大量的新文件，你将会发现Filebeat的注册表文件会变得非常大。这个时候，你可以参考（[the section called “Registry file is too large?](https://www.elastic.co/guide/en/beats/filebeat/current/faq.html#reduce-registry-size)[edit](https://github.com/elastic/beats/edit/5.1/filebeat/docs/faq.asciidoc)），来解决这个问题。

**安装filebeat服务**

下载和安装key文件

```text
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

创建yum源文

```text
[root@localhost ~]# vim /etc/yum.repos.d/elk-elasticsearch.repo
[elastic-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-m
```

开始安装

```text
yum install filebeat
```

启动服务

```text
systemctl start filebeat
systemctl status filebeat
```

**收集日志**

这里我们先以收集docker日志为例，简单来介绍一下filebeat的配置文件该如何编写。具体内容如下

```text
[root@localhost ~]# grep "^\s*[^# \t].*$" /etc/filebeat/filebeat.yml 
filebeat.prospectors:
- input_type: log
  paths:
    - /var/lib/docker/containers/*/*.log
output.elasticsearch:
  hosts: ["192.168.58.128:9200"]
```

和我们看的一样，其实并没有太多的内容。我们采集/var/lib/docker/containers/\*/\*.log，即filebeat所在节点的所有容器的日志。输出的位置是我们ElasticSearch的服务地址，这里我们直接将log输送给ES，而不通过Logstash中转。

再启动之前，我们还需要向ES提交一个filebeat index template，以便让elasticsearch知道filebeat输出的日志数据都包含哪些属性和字段。filebeat.template.json这个文件安装完之后就有，无需自己编写，找不到的同学可以通过find查找。加载模板到elasticsearch中：

```text
[root@localhost ~]# curl -XPUT 'http://192.168.58.128:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json
{
  "acknowledged" : true
}
```

重启服务

```text
systemctl restart filebeat
```

提示：如果你启动的是一个filebeat容器，需要将/var/lib/docker/containers目录挂载到该容器中；

**Kibana配置**

如果上面配置都没有问题，就可以访问Kibana，不过这里需要添加一个新的index pattern。按照manual中的要求，对于filebeat输送的日志，我们的index name or pattern应该填写为："filebeat-\*"。

