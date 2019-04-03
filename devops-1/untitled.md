# NSQ

## 一，简介

分布式消息队列 NSQ，NSQ是google开发，用go实现的消息队列服务，

NSQ可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。NSQ具有分布式、去中心化的拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。NSQ非常容易配置和部署，且具有最大的灵活性，支持众多消息协议

两种比较常用的消息队列——NSQ和Kafka

下面上一张Nsq与其他mq的对比图

![](../.gitbook/assets/image%20%2862%29.png)

图片来自golang2017开发者大会

### 1 消息队列的作用 <a id="&#x6D88;&#x606F;&#x961F;&#x5217;&#x7684;&#x4F5C;&#x7528;"></a>

1. 解耦，将一个流程加入一层数据接口拆分成两个部分，上游专注通知，下游专注处理
2. 缓冲，应对流量的突然上涨变更，消息队列有很好的缓冲削峰作用
3. 异步，上游发送消息以后可以马上返回，处理工作交给下游进行
4. 广播，让一个消息被多个下游进行处理
5. 冗余，保存处理的消息，防止消息处理失败导致的数据丢失

### 2 NSQ介绍 <a id="NSQ"></a>

#### 2.1 组件

NSQ主要包含3个组件：

* nsqd：在服务端运行的守护进程，负责接收，排队，投递消息给客户端。能够独立运行，不过通常是由 nsqlookupd 实例所在集群配置的
* nsqlookup：为守护进程，负责管理拓扑信息并提供发现服务。客户端通过查询 nsqlookupd 来发现指定话题（topic）的生产者，并且 nsqd 节点广播话题（topic）和通道（channel）信息
* nsqadmin：一套WEB UI，用来汇集集群的实时统计，并执行不同的管理任务

![](../.gitbook/assets/image%20%28143%29.png)

#### 2.2 特性

1. 消息默认不可持久化，虽然系统支持消息持久化存储在磁盘中（通过 **–mem-queue-size** ），不过默认情况下消息都在内存中
2. 消息最少会被投递一次，假设成立于 nsqd 节点没有错误
3. 消息无序，是由重新队列\(requeues\)，内存和磁盘存储的混合导致的，实际上，节点间不会共享任何信息。它是相对的简单完成疏松队列
4. 支持无 SPOF 的分布式拓扑，nsqd 和 nsqadmin 有一个节点故障不会影响到整个系统的正常运行
5. 支持requeue，延迟消费机制
6. 消息push给消费者

#### 内存限制

NSQ通过提供–mem-queue-size的配置选项来设置内存中的消息队列的大小。如果超过消息队列的大小，消息将写入硬盘。细心的读者会发现，通过设置内存消息队列的大小低到某个值时（如1或0）可以提高消息投递的可靠性。后端的硬盘队列用于非正常退出时恢复消息投递。

对于消息投递的可靠性，正常退出时消息会安全的被持久化到硬盘中，包括内存队列、投递中队列、延迟队列以及各种内部缓存。

注意，每个topic和channel的名称后面如果是以\#ephemeral结尾，那么缓存中的消息不被持久化到硬盘中而且超过内存消息队列大小时，消息将被丢弃。这使消费者在订阅一个channel时可以无需消息投递可靠性。这种临时的channel在无客户端连接时会自动消失。对于一个临时的topic，意味着至少有一个channel被创建、消费和删除（通常也是临时的channel）。

#### 消息投递的可靠性

NSQ保证消息投递至少一次，有可能重复投递。消费者应该自行保证消息处理的幂等性。

这个可靠性也作为消息协议的一部分，处理流程如下：

1.客户端声明准备好接收消息

2.NSQ发送一个消息并临时保存消息到本地

3.客户端发送FIN或者REQ表明处理成功或者失败。如果客户端未在规定的时间内作出响应，NSQ会根据设置的超时时间来自动把消息重新入队列。

这就保证了消息只有在NSQ非正常关闭时会发生丢失。这种情况下，所有内存的信息都会丢失。

如果防止消息丢失是非常重要的，即使是非正常退出，也是可以减少影响度的。一种方式是增加nsqd节点（部署在不同host），来备份消息。因为客户端处理消息是幂等的所以多次消息投递不影响下游系统，以保证任何单个节点故障不至于引起消息丢失。

关键是NSQ要提供构建的模块去支持多种生产用例和可配置程度的耐久性

#### 消除单点故障（SPOF）

NSQ采用的是分布式的设计方式。客户端通过长连接，连接所有指定topic的所有nsqd实例，这里没有中间人，没有单点故障。

只要有足够的客户端消费者连接到所有的生产者以保障大量的消息处理，就能保证所有的信息最终都可以被处理。

对于nsqlookupd，可以通过多个实例来实现高可用。他们之间不需要之间通信，数据是会最终一致的。消费者通过配置的nsqlookupd实例来获取所有信息并将其汇总

#### 2.3 流程

单个nsqd实例一次可以处理多个消息流。消息流被称为“topics”，一个topic有一个或者多个“channels”。每个channel会接收来自topic的所有消息。实际上，一个channel对应下游个服务消费的一个topic

topics和channels并不是预先配置的。topics是在首次发布或者订阅的时候创建的，channels是在首次订阅的时候创建的。

topics和channels的所有缓存数据都相互独立，目的是为了防止一个“慢”消费者造成消息积压而影响其他topic或channel。

一个channel通常有多个消费者连接，假如所有消费者都是在准备接收消息状态，每个消息会被随机投递到消费者中。

![](../.gitbook/assets/image%20%2888%29.png)

综述，消息是从topic广播到channel，然后从channel投递到消费者中。

NSQ还有一个辅助工具nsqlookupd，它提供了服务发现功能，使消费者能够订阅感兴趣的topic所在的所有nsqd的地址。同时在配置方面，使消费者和生产者解耦，他们不需要知道彼此，只需要通过nsqlookupd来建立联系，降低复杂性。

在底层实现中，每个nsqd和nsqlookupd都建立了长连接，定期推送自己的状态信息。nsqlookupd通过这些信息来判断哪个nsqd地址应该返回给消费者。

当添加一个新的消费者时，只要给nsqd客户端初始化nsqlookupd的实例地址。所以在新增消费者或者生产者时都无需修改任何配置，大大减少了复杂度。

需要重点强调的是，nsqd和nsqlookupd的守护进程是相互独立的，在兄弟节点之间没有通信和协作。

我们认为通过某种方式去观察、管理这些节点是非常重要的。我们构建了nsqadmin来处理这件事情，它提供了web界面来浏览topics/channels/consumers的层级、检查深度，以及其他的信息。而且还提供了一些管理员命令，比如移除和清空channel。

n[sq详细介绍](https://wcccode.github.io/2016/11/06/nsq-design.html)

单个nsqd可以有多个Topic，每个Topic又可以有多个Channel。Channel能够接收Topic所有消息的副本，从而实现了消息多播分发；而Channel上的每个消息被分发给它的订阅者，从而实现负载均衡，所有这些就组成了一个可以表示各种简单和复杂拓扑结构的强大框架。

以上摘抄于[分布式消息队列-NSQ-和-Kafka-对比](https://www.liuin.cn/2018/07/11/%E5%88%86%E5%B8%83%E5%BC%8F%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97-NSQ-%E5%92%8C-Kafka-%E5%AF%B9%E6%AF%94/)

**核心概念**

topic: topic是nsq的消息发布的逻辑`关键词`。当程序初次发布带topic的消息时,如果topic不存在,则会被创建。

channels: 当生产者每次发布消息的时候,消息会采用多播的方式被拷贝到各个channel中,channel起到队列的作用。

messages: 数据流的形式。

## 二，安装

 官方文档网址： [http://nsq.io/overview/quick\_start.html](http://nsq.io/overview/quick_start.html) 

下载地址： [http://nsq.io/deployment/installing.html](http://nsq.io/deployment/installing.html)   
目前的稳定版是V1.0.0，根据自己实际情况选择安装方式（写于2019/3/8）

```text
#设置主机名
[root@nsq3 ~] hostnamectl set-hostname nsq3
#创建安装目录
[root@nsq3 ~]#mkdir /home/nsq
[root@nsq3 ~]#cd /home/nsq/
[root@nsq3 ~]#mkdir config log data
#下载文件
[root@nsq3 ~]#wget https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz
#解压
[root@nsq3 ~]#tar xzvf nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz
[root@nsq3 ~]#ln -s nsq-1.0.0-compat.linux-amd64.go1.8 nsq
```

### 配置nsq文件（以nsq3 10.1.194.245为例）

```text
vim /home/nsq/config/nsqd.cfg
内容如下
```

```text
## log verbosity level: debug, info, warn, error, or fatal
log-level = "info"

## unique identifier (int) for this worker (will default to a hash of hostname)
# id = 5150

## <addr>:<port> to listen on for TCP clients
tcp_address = "0.0.0.0:4150"

## <addr>:<port> to listen on for HTTP clients
http_address = "0.0.0.0:4151"

## <addr>:<port> to listen on for HTTPS clients
# https_address = "0.0.0.0:4152"

## address that will be registered with lookupd (defaults to the OS hostname)
broadcast_address = "10.1.194.245"

## cluster of nsqlookupd TCP addresses
nsqlookupd_tcp_addresses = [
    "10.1.194.243:4160",
    "10.1.194.244:4160",
    "10.1.194.245:4160"
]

## duration to wait before HTTP client connection timeout
http_client_connect_timeout = "2s"

## duration to wait before HTTP client request timeout
http_client_request_timeout = "5s"

## path to store disk-backed messages
data_path = "/home/nsq/data"

## number of messages to keep in memory (per topic/channel)
mem_queue_size = 10000

## number of bytes per diskqueue file before rolling
max_bytes_per_file = 104857600

## number of messages per diskqueue fsync
sync_every = 2500

## duration of time per diskqueue fsync (time.Duration)
sync_timeout = "2s"


## duration to wait before auto-requeing a message
msg_timeout = "60s"

## maximum duration before a message will timeout
max_msg_timeout = "15m"

## maximum size of a single message in bytes
max_msg_size = 1024768

## maximum requeuing timeout for a message
max_req_timeout = "1h"

## maximum size of a single command body
max_body_size = 5123840


## maximum client configurable duration of time between client heartbeats
max_heartbeat_interval = "60s"

## maximum RDY count for a client
max_rdy_count = 2500

## maximum client configurable size (in bytes) for a client output buffer
max_output_buffer_size = 65536

## maximum client configurable duration of time between flushing to a client (time.Duration)
max_output_buffer_timeout = "1s"


## UDP <addr>:<port> of a statsd daemon for pushing stats
# statsd_address = "127.0.0.1:8125"

## prefix used for keys sent to statsd (%s for host replacement)
statsd_prefix = "nsq.%s"

## duration between pushing to statsd (time.Duration)
statsd_interval = "60s"

## toggle sending memory and GC stats to statsd
statsd_mem_stats = true


## message processing time percentiles to keep track of (float)
e2e_processing_latency_percentiles = [
    100.0,
    99.0,
    95.0
]

## calculate end to end latency quantiles for this duration of time (time.Duration)
e2e_processing_latency_window_time = "10m"


## path to certificate file
tls_cert = ""

## path to private key file
tls_key = ""

## set policy on client certificate (require - client must provide certificate,
##  require-verify - client must provide verifiable signed certificate)
# tls_client_auth_policy = "require-verify"

## set custom root Certificate Authority
# tls_root_ca_file = ""

## require client TLS upgrades
tls_required = false

## minimum TLS version ("ssl3.0", "tls1.0," "tls1.1", "tls1.2")
tls_min_version = ""

## enable deflate feature negotiation (client compression)
deflate = true

## max deflate compression level a client can negotiate (> values == > nsqd CPU usage)
max_deflate_level = 6

## enable snappy feature negotiation (client compression)
snappy = true
```

### 配置nsqlookupd.cfg文件

```text
vim /home/nsq/config/nsqlookupd.cfg
```

```text
cat > /home/nsq/config/nsqlookupd.cfg << EOF  #也可以用这种方式导入，可删除此行
## log verbosity level: debug, info, warn, error, or fatal
log-level = "info"

## <addr>:<port> to listen on for TCP clients
tcp_address = "0.0.0.0:4160"

## <addr>:<port> to listen on for HTTP clients
http_address = "0.0.0.0:4161"

## address that will be registered with lookupd (defaults to the OS hostname)
broadcast_address = "10.1.194.245"


## duration of time a producer will remain in the active list since its last ping
inactive_producer_timeout = "300s"

## duration of time a producer will remain tombstoned if registration remains
tombstone_lifetime = "45s"
```

### \*如果nsqadmin在此节点上，配置nsqadmin文件nsqadmin.cfg

```text
## log verbosity level: debug, info, warn, error, or fatal
log-level = "info"

## <addr>:<port> to listen on for HTTP clients
http_address = "0.0.0.0:4171"

## graphite HTTP address
graphite_url = ""

## proxy HTTP requests to graphite
proxy_graphite = false

## prefix used for keys sent to statsd (%s for host replacement, must match nsqd)
statsd_prefix = "nsq.%s"

## format of statsd counter stats
statsd_counter_format = "stats.counters.%s.count"

## format of statsd gauge stats
statsd_gauge_format = "stats.gauges.%s"

## time interval nsqd is configured to push to statsd (must match nsqd)
statsd_interval = "60s"

## HTTP endpoint (fully qualified) to which POST notifications of admin actions will be sent
notification_http_endpoint = ""


## nsqlookupd HTTP addresses
nsqlookupd_http_addresses = [
    "10.1.194.243:4161",
    "10.1.194.244:4161",
    "10.1.194.245:4161"
]

## nsqd HTTP addresses (optional)
#nsqd_http_addresses = [
#    "127.0.0.1:4151"
#]

```

### Systemd 管理启动 配置

#### 配置nsqd.service

```text
cat > /etc/systemd/system/nsqd.service << EOF
[Unit]
Description=NSQD
After=network.target
[Service]
LimitCORE=infinity
LimitNOFILE=100000
LimitNPROC=100000
WorkingDirectory=/home/nsq
ExecStart=/home/nsq/nsq/bin/nsqd -config=/home/nsq/config/nsqd.cfg
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=10s
User=root
[Install]
WantedBy=multi-user.target
EOF
```

#### 配置nsqlookupd.service

```text
cat > /etc/systemd/system/nsqlookupd.service << EOF
[Unit]
Description=NSQLookupD
After=network.target
[Service]
LimitCORE=infinity
LimitNOFILE=100000
LimitNPROC=100000
WorkingDirectory=/home/nsq
ExecStart=/home/nsq/nsq/bin/nsqlookupd -config=/home/nsq/config/nsqlookupd.cfg
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=10s
User=root
[Install]
WantedBy=multi-user.target
EOF
```

#### 配置nsqadmin.service文件

```text
cat > /etc/systemd/system/nsqadmin.service << EOF
  171  [Unit]
  172  Description=NSQAdmin
  173  After=network.target
  174  [Service]
  175  LimitCORE=infinity
  176  LimitNOFILE=100000
  177  LimitNPROC=100000
  178  WorkingDirectory=/home/nsq
  179  ExecStart=/home/nsq/nsq/bin/nsqadmin -config=/home/nsq/config/nsqadmin.cfg
  180  ExecReload=/bin/kill -HUP $MAINPID
  181  Type=simple
  182  KillMode=process
  183  Restart=on-failure
  184  RestartSec=10s
  185  User=root
  186  [Install]
  187  WantedBy=multi-user.target
  188  EOF
```

#### 开机启动

```text
systemctl daemon-reload
systemctl start nsqd
systemctl enable nsqd;systemctl enable nsqlookupd;systemctl enable nsqadmin
systemctl start nsqlookupd
systemctl start nsqadmin
#查看状态
systemctl status nsqd
systemctl status nsqlookupd
```

## 三，使用

域名地址：

```text
NSQ  做好域名解析
nsqlookup的地址：

http:
nsq1.darren.com:4161
nsq2.darren.com:4161
nsq3.darren.com:4161
tcp:
nsq1.darren.com:4160
nsq2.darren.com:4160
nsq3.darren.com:4160
nsqd的地址：

http:
nsq1.darren.com:4151
nsq2.darren.com:4151
nsq3.darren.com:4151
tcp:
nsq1.darren.com:4150
nsq2.darren.com:4150
nsq3.darren.com:4150
nsqadmin的地址

http://nsqadmin.darren.com:4171
```

默认情况下, 可以在浏览器输入:[**http://nsqadmin.darren.com:4171**](http://nsqadmin.darren.com:4171) 打开监控面板

![](../.gitbook/assets/image%20%2827%29.png)

由于nsq提供了一个unix-like的工具,所以我们可以在终端使用以下命令进行消息的发送测试:

```text
$ curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'
$ curl -d 'hello world 2' 'http://127.0.0.1:4251/put?topic=test'
```

发送后, 可以在监控面板观察页面数据的变化



**NSQ Topic和Channel命名规范**

Channel命名规范：{环境} \_ {产品线英文/拼音简写} \_ {终端类型} {业务模块英文/拼音简写}   
Channel命名示例：PRE\_SDYXMALL\_APP\_ORDER

Topic命名规范：{环境} \_ {产品线英文/拼音简写} \_ {业务模块英文/拼音简写} \_ {消息主题} \_ {事件类型}   
Topic命名示例：PRE\_SDYXMALL\_ORDER\_STATUS\_CHANGE





#### 使用go编写NSQ消费端的例子 <a id="&#x4F7F;&#x7528;go&#x7F16;&#x5199;nsq&#x6D88;&#x8D39;&#x7AEF;&#x7684;&#x4F8B;&#x5B50;"></a>

consumer.go

```text
package queue

import (
    "fmt"
    "github.com/nsqio/go-nsq"
    "log"
    "os"
)

type logger interface {
    Output(int, string) error
}

type Consumer struct {
    client      *nsq.Consumer
    config      *nsq.Config
    nsqds       []string
    nsqlookupds []string
    concurrency int
    channel     string
    topic       string
    level       nsq.LogLevel
    log         logger
    err         error
}

//初始化消费端
func NewConsumer(topic, channel string) *Consumer {
    return &Consumer{
        log:         log.New(os.Stderr, "", log.LstdFlags),
        config:      nsq.NewConfig(),
        level:       nsq.LogLevelInfo,
        channel:     channel,
        topic:       topic,
        concurrency: 1,
    }
}

func (c *Consumer) SetLogger(log logger, level nsq.LogLevel) {
    c.level = level
    c.log = log
}

func (c *Consumer) SetMap(options map[string]interface{}) {
    for k, v := range options {
        c.Set(k, v)
    }
}

func (c *Consumer) Set(option string, value interface{}) {
    switch option {
    case "topic":
        c.topic = value.(string)
    case "channel":
        c.channel = value.(string)
    case "concurrency":
        c.concurrency = value.(int)
    case "nsqd":
        c.nsqds = []string{value.(string)}
    case "nsqlookupd":
        c.nsqlookupds = []string{value.(string)}
    case "nsqds":
        s, err := strings(value)
        if err != nil {
            c.err = fmt.Errorf("%q: %v", option, err)
            return
        }
        c.nsqds = s
    case "nsqlookupds":
        s, err := strings(value)
        if err != nil {
            c.err = fmt.Errorf("%q: %v", option, err)
            return
        }
        c.nsqlookupds = s
    default:
        err := c.config.Set(option, value)
        if err != nil {
            c.err = err
        }
    }
}

func (c *Consumer) Start(handler nsq.Handler) error {

    if c.err != nil {
        return c.err
    }

    client, err := nsq.NewConsumer(c.topic, c.channel, c.config)
    if err != nil {
        return err
    }
    c.client = client
    client.SetLogger(c.log, c.level)
    client.AddConcurrentHandlers(handler, c.concurrency)
    return c.connect()
}

//连接到nsqd
func (c *Consumer) connect() error {

    if len(c.nsqds) == 0 && len(c.nsqlookupds) == 0 {
        return fmt.Errorf(`at least one "nsqd" or "nsqlookupd" address must be configured`)
    }

    if len(c.nsqds) > 0 {
        err := c.client.ConnectToNSQDs(c.nsqds)
        if err != nil {
            return err
        }
    }
    if len(c.nsqlookupds) > 0 {
        err := c.client.ConnectToNSQLookupds(c.nsqlookupds)
        if err != nil {
            return err
        }
    }
    return nil
}

//stop and wait
func (c *Consumer) Stop() error {
    c.client.Stop()
    <-c.client.StopChan
    return nil
}

func strings(v interface{}) ([]string, error) {
    switch v.(type) {
    case []string:
        return v.([]string), nil
    case []interface{}:
        var ret []string
        for _, e := range v.([]interface{}) {
            s, ok := e.(string)
            if !ok {
                return nil, fmt.Errorf("string expected")
            }
            ret = append(ret, s)
        }
        return ret, nil
    default:
        return nil, fmt.Errorf("strings expected")
    }
}

```

Go

main.go

```text

package main

import (
    "fmt"
    "github.com/nsqio/go-nsq"
    "your_go_path/project/queue"
)

func main() {
    done := make(chan bool)
    c := queue.NewConsumer("test", "testchan2")
    c.Set("nsqds", []string{"192.168.139.134:4150", "192.168.139.134:4250"})
    c.Set("concurrency", 15)
    c.Set("max_attempts", 10)
    c.Set("max_in_flight", 150)
    err := c.Start(nsq.HandlerFunc(func(msg *nsq.Message) error {
        fmt.Println("customer2:", string(msg.Body))
        return nil
    }))
    if err != nil {
        fmt.Errorf(err.Error())
    }
    <-done
}


```

Go

#### 使用总结 <a id="&#x4F7F;&#x7528;&#x603B;&#x7ED3;"></a>

nsq大部分情况基本能满足我们作为消息队列的要求,而且性能与单点故障处理能力也比较出色.

但它不适用的地方主要有:

> 有顺序要求的消息
>
> 不支持副本集的集群方式

```text
举例说明
[Topic里的AURAYOU随具体的业务线而确定]

订单消息通知
1、消息Topic:AURAYOU_ORDER_STATUS_CHANGE
更新订单状态的消息格式

{
      "head":{
         "msgId":"2017082115231500000001",   //消息ID（22位）    全局唯一  当前时间格式化为年月日时分秒+8位随机数
         "sourceIp":"225.0.0.1",    //消息发布系统的来源IP
         "createdAt":143000000   //消息发布时间戳（秒）
      },
      "businessData":{
        "salesChannelId":3,
        "payOrderId":"111111",
        "orderId":"222222",
        "userId":11111,
        "orderStatus":8,
        "orderType":0,
        "appStoreId":"1"
      }
 }
2、消息Topic:AURAYOU_PAYORDER_STATUS_CHANGE
更新订单支付状态的消息格式

{
      "head":{
         "msgId":"2017082115231500000001",   //消息ID（22位）    全局唯一  当前时间格式化为年月日时分秒+8位随机数
         "sourceIp":"225.0.0.1",    //消息发布系统的来源IP
         "createdAt":143000000   //消息发布时间戳（秒）
      },
      "businessData":{
        "salesChannelId":3,
        "payOrderId":"111111",
        "orderIdList":["222222","3333333"],
        "userId":123434,
        "orderStatus":8,
        "orderType":0,
        "appStoreId":"1"
      }
 }
```



## 参考文档

极客学院 [nsq指南](http://wiki.jikexueyuan.com/project/nsq-guide/design.html)

[NSQ消息发送机制](https://blog.csdn.net/u013735511/article/details/82555419)





