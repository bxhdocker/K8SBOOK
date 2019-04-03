# 图解Kubernetes网络（三）

#### 集群动力学

由于Kubernetes（更通用的说法是分布式系统）天生具有不断变化的特性，因此它的Pod（以及Pod的IP）总是在改变。变化的原因可以是针对不可预测的Pod或节点崩溃而进行的滚动升级和扩展。这使得Pod IP不能直接用于通信。  
  
我们看一下Kubernetes Service，它是一个虚拟IP，并伴随着一组Pod IP作为Endpoint（通过标签选择器识别）。它们充当虚拟负载均衡器，其IP保持不变，而后端Pod IP可能会不断变化。  
[![2.png](http://dockone.io/uploads/article/20181218/49b016c98c39a365446a5af3963fbd1e.png)](http://dockone.io/uploads/article/20181218/49b016c98c39a365446a5af3963fbd1e.png)  
_Kubernetes Srvice对象的标签选择器_  
  
整个虚拟IP的实现实际上是一组iptables（最新版本可以选择使用IPVS，但这是另一个讨论）规则，由Kubernetes组件kube-proxy管理。 这个名字现在实际上是误导性的。 它在v 1.0之前确实是一个代理，并且由于其实现是内核空间和用户空间之间的不断复制，它变得非常耗费资源并且速度较慢。 现在，它只是一个控制器，就像Kubernetes中的许多其它控制器一样，它watch api server的endpoint的更改并相应地更新iptables规则。  
  
有了这些iptables规则，每当数据包发往Service IP时，它就进行DNAT（DNAT=目标网络地址转换）操作，这意味着目标IP从Service IP更改为其中一个Endpoint - Pod IP - 由iptables随机选择。这可确保负载均匀分布在后端Pod中。  
[![3.png](http://dockone.io/uploads/article/20181218/79811160b699c5e3a52ef21f63ea9d08.png)](http://dockone.io/uploads/article/20181218/79811160b699c5e3a52ef21f63ea9d08.png)  
_iptables DNAT_  
  
当这个DNAT发生时，这个信息存储在conntrack中——它是Linux连接跟踪表（存储iptables已完成的5元组翻译：protocol，srcIP，srcPort，dstIP，dstPort）。 这样当请求回来时，它可以取消DNAT，这意味着将源IP从Pod IP更改为Service IP。 这样，客户端就不用关心后台如何处理数据包流。  
[![4.png](http://dockone.io/uploads/article/20181218/84ad2825d163d97545ba04d94aa2937b.png)](http://dockone.io/uploads/article/20181218/84ad2825d163d97545ba04d94aa2937b.png)  
_conntrack表中的5元组实例_  
  
因此通过使用Kubernetes Service，我们可以使用相同的端口而不会发生任何冲突（因为我们可以将端口重新映射到Endpoint）。 这使服务发现变得非常容易。 我们可以使用内部DNS并对服务主机名进行硬编码。 我们甚至可以使用Kubernetes预设的服务主机和端口环境变量。  
  
**小建议**：采取第二种方法，你可节省大量不必要的DNS调用！  


#### 出站流量

到目前为止我们讨论的Kubernetes Service是在一个集群内工作。但是，在大多数实际情况中，应用程序需要访问一些外部api /网站。  
  
通常，节点可以同时具有私有IP和公共IP。对于互联网访问，这些公共和私有IP存在某种1：1的NAT，特别是在云环境中。  
  
对于从节点到某些外部IP的普通通信，源IP从节点的专用IP更改为其出站数据包的公共IP，入站的响应数据包则刚好相反。但是，当Pod发出与外部IP的连接时，源IP是Pod IP，云提供商的NAT机制不知道该IP。因此它将丢弃具有除节点IP之外的源IP的数据包。  
  
因此你可能也猜对了，我们将使用更多的iptables！这些规则也由kube-proxy添加，执行SNAT（源网络地址转换），即IP MASQUERADE。它告诉内核使用此数据包发出的网络接口的IP，代替源Pod IP。同时保留conntrack条目以进行反SNAT操作。  


#### 入站流量

到目前为止一切都很好。Pod可以互相交谈，也可以访问互联网。但我们仍然缺少关键部分 - 为用户请求流量提供服务。截至目前，有两种主要方法可以做到这一点：  


#### NodePort /云负载均衡器（L4 - IP和端口）

将服务类型设置为`NodePort`将会为服务分配范围为`30000-33000`的`nodePort`。即使在特定节点上没有运行Pod，此`nodePort`也会在每个节点上打开。此NodePort上的入站流量将再次使用iptables发送到其中一个Pod（该Pod甚至可能在其它节点上！）。  
  
云环境中的LoadBalancer服务类型将在所有节点之前创建云负载均衡器（例如ELB），命中相同的nodePort。  


#### Ingress（L7 - HTTP / TCP）

许多不同的工具，如Nginx，Traefik，HAProxy等，保留了http主机名/路径和各自后端的映射。通常这是基于负载均衡器和nodePort的流量入口点，其优点是我们可以有一个入口处理所有服务的入站流量，而不需要多个nodePort和负载均衡器。  


#### 网络策略

可以把它想象为Pod的安全组/ ACL。 NetworkPolicy规则允许/拒绝跨Pod的流量。确切的实现取决于网络层/CNI，但大多数只使用iptables。  
  
目前为止就这样了。 在[前面](http://dockone.io/article/3211)的[部分](http://dockone.io/article/3212)中，我们研究了Kubernetes网络的基础以及overlay网络的工作原理。 现在我们知道Service抽象是如何在一个动态集群内起作用并使服务发现变得非常容易。我们还介绍了出站和入站流量的工作原理以及网络策略如何对群集内的安全性起作用。

