# etcd概念与原理

## Etcd解析 <a id="etcd&#x89E3;&#x6790;"></a>

 Etcd是CoreOS基于Raft开发的分布式key-value存储，可用于服务发现、共享配置以及一致性保障（如数据库选主、分布式锁等），Etcd是Kubernetes集群中的一个十分重要的组件，用于保存集群所有的网络配置和对象的状态信息。在后面具体的安装环境中，我们安装的etcd的版本是v3.1.5，整个kubernetes系统中一共有两个服务需要用到etcd用来协同和存储配置，分别是：

* 网络插件flannel、对于其它网络插件也需要用到etcd存储网络的配置信息
* kubernetes本身，包括各种对象的状态和元信息配置

**注意**：flannel操作etcd使用的是v2的API，而kubernetes操作etcd使用的v3的API，所以在下面我们执行`etcdctl`的时候需要设置`ETCDCTL_API`环境变量，该变量默认值为2。

{% hint style="danger" %}
 提醒一下，etcd2作为后端已被弃用，并且在Kubernetes 1.13中将删除支持
{% endhint %}

#### Etcd主要功能

* 基本的key-value存储
* 监听机制
* key的过期及续约机制，用于监控和服务发现
* 原子CAS和CAD，用于分布式锁和leader选举

### 原理 <a id="&#x539F;&#x7406;"></a>

Etcd使用的是raft一致性算法来实现的，是一款分布式的一致性KV存储，主要用于共享配置和服务发现。关于raft一致性算法请参考[该动画演示](http://thesecretlivesofdata.com/raft/)。

关于Etcd的原理解析请参考[Etcd 架构与实现解析](http://jolestar.com/etcd-architecture/)。

[关于Etcd的的一致性](https://darren.gitbook.io/project/~/edit/drafts/-LG3zLujRGWx7xpuBUvd/gai-nian-yu-yuan-li/he-xin-zu-jian-yuan-li/etcd/etcd-ji-yu-raft-de-yi-zhi-xing)

### 使用Etcd存储Flannel网络信息 <a id="&#x4F7F;&#x7528;etcd&#x5B58;&#x50A8;flannel&#x7F51;&#x7EDC;&#x4FE1;&#x606F;"></a>

我们在安装Flannel的时候配置了`FLANNEL_ETCD_PREFIX="/kube-centos/network"`参数，这是Flannel查询etcd的目录地址。

查看Etcd中存储的flannel网络信息：

```text
$ etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem ls /kube-centos/network -r
2018-01-19 18:38:22.768145 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
/kube-centos/network/config
/kube-centos/network/subnets
/kube-centos/network/subnets/172.30.31.0-24
/kube-centos/network/subnets/172.30.20.0-24
/kube-centos/network/subnets/172.30.23.0-24
```

查看flannel的配置：

```text
$ etcdctl --ca-file=/etc/kubernetes/ssl/ca.pem --cert-file=/etc/kubernetes/ssl/kubernetes.pem --key-file=/etc/kubernetes/ssl/kubernetes-key.pem get /kube-centos/network/config
2018-01-19 18:38:22.768145 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
{ "Network": "172.30.0.0/16", "SubnetLen": 24, "Backend": { "Type": "host-gw" } }
```

### 使用Etcd存储Kubernetes对象信息 <a id="&#x4F7F;&#x7528;etcd&#x5B58;&#x50A8;kubernetes&#x5BF9;&#x8C61;&#x4FE1;&#x606F;"></a>

Kubernetes使用etcd v3的API操作etcd中的数据。所有的资源对象都保存在`/registry`路径下，如下：

```text
ThirdPartyResourceData
apiextensions.k8s.io
apiregistration.k8s.io
certificatesigningrequests
clusterrolebindings
clusterroles
configmaps
controllerrevisions
controllers
daemonsets
deployments
events
horizontalpodautoscalers
ingress
limitranges
minions
monitoring.coreos.com
namespaces
persistentvolumeclaims
persistentvolumes
poddisruptionbudgets
pods
ranges
replicasets
resourcequotas
rolebindings
roles
secrets
serviceaccounts
services
statefulsets
storageclasses
thirdpartyresources
```

如果你还创建了CRD（自定义资源定义），则在此会出现CRD的API。

#### 查看集群中所有的Pod信息 <a id="&#x67E5;&#x770B;&#x96C6;&#x7FA4;&#x4E2D;&#x6240;&#x6709;&#x7684;pod&#x4FE1;&#x606F;"></a>

例如我们直接从etcd中查看kubernetes集群中所有的pod的信息，可以使用下面的命令：

```text
ETCDCTL_API=3 etcdctl get /registry/pods --prefix -w json|python -m json.tool
```

此时将看到json格式输出的结果，其中的`key`使用了`base64`编码，关于etcdctl命令的详细用法请参考[使用etcdctl访问kubernetes数据](https://jimmysong.io/kubernetes-handbook/guide/using-etcdctl-to-access-kubernetes-data.html)。

### Etcd V2与V3版本API的区别 <a id="etcd-v2&#x4E0E;v3&#x7248;&#x672C;api&#x7684;&#x533A;&#x522B;"></a>

Etcd V2和V3之间的数据结构完全不同，互不兼容，也就是说使用V2版本的API创建的数据只能使用V2的API访问，V3的版本的API创建的数据只能使用V3的API访问。这就造成我们访问etcd中保存的flannel的数据需要使用`etcdctl`的V2版本的客户端，而访问kubernetes的数据需要设置`ETCDCTL_API=3`环境变量来指定V3版本的API。

### Etcd数据备份 <a id="etcd&#x6570;&#x636E;&#x5907;&#x4EFD;"></a>

我们安装的时候指定的Etcd数据的存储路径是`/var/lib/etcd`，一定要对该目录做好备份。

   更多关于[Etcd v2与v3存储区别](https://darren.gitbook.io/project/~/edit/drafts/-LG3zLujRGWx7xpuBUvd/gai-nian-yu-yuan-li/he-xin-zu-jian-yuan-li/etcd/etcd-v2-yu-v3-cun-chu)

### Etcd 使用注意事项 <a id="etcd-&#x4F7F;&#x7528;&#x6CE8;&#x610F;&#x4E8B;&#x9879;"></a>

1. Etcd cluster 初始化的问题

   如果集群第一次初始化启动的时候，有一台节点未启动，通过v3的接口访问的时候，会报告Error: Etcdserver: not capable 错误。这是为兼容性考虑，集群启动时默认的API版本是2.3，只有当集群中的所有节点都加入了，确认所有节点都支持v3接口时，才提升集群版本到v3。这个只有第一次初始化集群的时候会遇到，如果集群已经初始化完毕，再挂掉节点，或者集群关闭重启（关闭重启的时候会从持久化数据中加载集群API版本），都不会有影响。

2. Etcd 读请求的机制

   v2 quorum=true 的时候，读取是通过raft进行的，通过cli请求，该参数默认为true。

   v3 --consistency=“l” 的时候（默认）通过raft读取，否则读取本地数据。sdk 代码里则是通过是否打开：WithSerializable option 来控制。

   一致性读取的情况下，每次读取也需要走一次raft协议，能保证一致性，但性能有损失，如果出现网络分区，集群的少数节点是不能提供一致性读取的。但如果不设置该参数，则是直接从本地的store里读取，这样就损失了一致性。使用的时候需要注意根据应用场景设置这个参数，在一致性和可用性之间进行取舍。

3. Etcd 的 compact 机制

   Etcd 默认不会自动 compact，需要设置启动参数，或者通过命令进行compact，如果变更频繁建议设置，否则会导致空间和内存的浪费以及错误。Etcd v3 的默认的 backend quota 2GB，如果不 compact，boltdb 文件大小超过这个限制后，就会报错：”Error: etcdserver: mvcc: database space exceeded”，导致数据无法写入。

### etcd的问题 <a id="etcd&#x7684;&#x95EE;&#x9898;"></a>

```text
当前 Etcd 的raft实现保证了多个节点数据之间的同步，但明显的一个问题就是扩充节点不能解决容量问题。
 要想解决容量问题，只能进行分片，但分片后如何使用raft同步数据？
只能实现一个 multiple group raft，每个分片的多个副本组成一个虚拟的raft group，通过raft实现数据同步。
当前实现了multiple group raft的有 TiKV 和 Cockroachdb，但尚未一个独立通用的。
理论上来说，如果有了这套 multiple group raft，后面挂个持久化的kv就是一个分布式kv存储，挂个内存kv就是分布式缓存，挂个lucene就是分布式搜索引擎。
当然这只是理论上，要真实现复杂度还是不小。
```

注： 部分转自[jolestar](http://jolestar.com/etcd-architecture/)和[infoq](http://www.infoq.com/cn/articles/etcd-interpretation-application-scenario-implement-principle).

### Etcd，Zookeeper，Consul 比较 <a id="etcd&#xFF0C;zookeeper&#xFF0C;consul-&#x6BD4;&#x8F83;"></a>

* Etcd 和 Zookeeper 提供的能力非常相似，都是通用的一致性元信息存储，都提供watch机制用于变更通知和分发，也都被分布式系统用来作为共享信息存储，在软件生态中所处的位置也几乎是一样的，可以互相替代的。二者除了实现细节，语言，一致性协议上的区别，最大的区别在周边生态圈。Zookeeper 是apache下的，用java写的，提供rpc接口，最早从hadoop项目中孵化出来，在分布式系统中得到广泛使用（hadoop, solr, kafka, mesos 等）。Etcd 是coreos公司旗下的开源产品，比较新，以其简单好用的rest接口以及活跃的社区俘获了一批用户，在新的一些集群中得到使用（比如kubernetes）。虽然v3为了性能也改成二进制rpc接口了，但其易用性上比 Zookeeper 还是好一些。
* 而Consul 的目标则更为具体一些，Etcd 和 Zookeeper 提供的是分布式一致性存储能力，具体的业务场景需要用户自己实现，比如服务发现，比如配置变更。而Consul 则以服务发现和配置变更为主要目标，同时附带了kv存储。

### Etcd 的周边工具 <a id="etcd-&#x7684;&#x5468;&#x8FB9;&#x5DE5;&#x5177;"></a>

1. **Confd**

   在分布式系统中，理想情况下是应用程序直接和 Etcd 这样的服务发现/配置中心交互，通过监听 Etcd 进行服务发现以及配置变更。但我们还有许多历史遗留的程序，服务发现以及配置大多都是通过变更配置文件进行的。Etcd 自己的定位是通用的kv存储，所以并没有像 Consul 那样提供实现配置变更的机制和工具，而 Confd 就是用来实现这个目标的工具。

   Confd 通过watch机制监听 Etcd 的变更，然后将数据同步到自己的一个本地存储。用户可以通过配置定义自己关注哪些key的变更，同时提供一个配置文件模板。Confd 一旦发现数据变更就使用最新数据渲染模板生成配置文件，如果新旧配置文件有变化，则进行替换，同时触发用户提供的reload脚本，让应用程序重新加载配置。

   Confd 相当于实现了部分 Consul 的agent以及consul-template的功能，作者是kubernetes的Kelsey Hightower，但大神貌似很忙，没太多时间关注这个项目了，很久没有发布版本，我们着急用，所以fork了一份自己更新维护，主要增加了一些新的模板函数以及对metad后端的支持。[confd](https://github.com/yunify/confd)

2. **Metad**

   服务注册的实现模式一般分为两种，一种是调度系统代为注册，一种是应用程序自己注册。调度系统代为注册的情况下，应用程序启动后需要有一种机制让应用程序知道『我是谁』，然后发现自己所在的集群以及自己的配置。Metad 提供这样一种机制，客户端请求 Metad 的一个固定的接口 /self，由 Metad 告知应用程序其所属的元信息，简化了客户端的服务发现和配置变更逻辑。

   Metad 通过保存一个ip到元信息路径的映射关系来做到这一点，当前后端支持Etcd v3，提供简单好用的 http rest 接口。 它会把 Etcd 的数据通过watch机制同步到本地内存中，相当于 Etcd 的一个代理。所以也可以把它当做Etcd 的代理来使用，适用于不方便使用 Etcd v3的rpc接口或者想降低 Etcd 压力的场景。 [metad](https://github.com/yunify/metad)

### 参考 <a id="&#x53C2;&#x8003;"></a>

* [etcd官方文档](https://coreos.com/etcd/docs/latest)
* [etcd v3命令和API](http://blog.csdn.net/u010278923/article/details/71727682)
* [Etcd 架构与实现解析](http://jolestar.com/etcd-architecture/)
* [Etcd website](https://coreos.com/etcd/)
* [Etcd github](https://github.com/coreos/etcd/)
* [Projects using etcd](https://github.com/coreos/etcd/blob/master/Documentation/production-users.md)
* [http://jolestar.com/etcd-architecture/](http://jolestar.com/etcd-architecture/)
* [etcd从应用场景到实现原理的全方位解读](http://www.infoq.com/cn/articles/etcd-interpretation-application-scenario-implement-principle)

