# 转发K8S后端服务的四种方式

### ClusterIP <a id="clusterip"></a>

此类型会提供一个集群内部的虚拟IP（与Pod不在同一网段\)，以供集群内部的pod之间通信使用。ClusterIP也是Kubernetes service的默认类型。   
   
为了实现图上的功能主要需要以下几个组件的协同工作：   
apiserver：在创建service时，apiserver接收到请求以后将数据存储到etcd中。   
kube-proxy：k8s的每个节点中都有该进程，负责实现service功能，这个进程负责感知service，pod的变化，并将变化的信息写入本地的iptables中。   
iptables：使用NAT等技术将virtualIP的流量转至endpoint中。

![ClusterIP](https://img-blog.csdn.net/20170724161410421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl5aW5na2UxMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### NodePort <a id="nodeport"></a>

NodePort模式除了使用cluster ip外，也将service的port映射到每个node的一个指定内部port上，映射的每个node的内部port都一样。   
为每个节点暴露一个端口，通过nodeip + nodeport可以访问这个服务，同时服务依然会有cluster类型的ip+port。内部通过clusterip方式访问，外部通过nodeport方式访问。   


![](../.gitbook/assets/image%20%2839%29.png)

### loadbalance <a id="loadbalance"></a>

LoadBalancer在NodePort基础上，K8S可以请求底层云平台创建一个负载均衡器，将每个Node作为后端，进行服务分发。该模式需要底层云平台（例如GCE）支持。

### Ingress <a id="ingress"></a>

Ingress，是一种HTTP方式的路由转发机制，由Ingress Controller和HTTP代理服务器组合而成。Ingress Controller实时监控Kubernetes API，实时更新HTTP代理服务器的转发规则。HTTP代理服务器有GCE Load-Balancer、HaProxy、Nginx等开源方案。   
[详细说明请见http://blog.csdn.net/liyingke112/article/details/77066814](http://blog.csdn.net/liyingke112/article/details/77066814)   


![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20170810175951852?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl5aW5na2UxMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## service的三种端口 <a id="service&#x7684;&#x4E09;&#x79CD;&#x7AEF;&#x53E3;"></a>

### port <a id="port"></a>

service暴露在cluster ip上的端口，:port 是提供给集群内部客户访问service的入口。

### nodePort <a id="nodeport-1"></a>

nodePort是k8s提供给集群外部客户访问service入口的一种方式，:nodePort 是提供给集群外部客户访问service的入口。

### targetPort <a id="targetport"></a>

targetPort是pod上的端口，从port和nodePort上到来的数据最终经过kube-proxy流入到后端pod的targetPort上进入容器。

### port、nodePort总结 <a id="portnodeport&#x603B;&#x7ED3;"></a>

总的来说，port和nodePort都是service的端口，前者暴露给集群内客户访问服务，后者暴露给集群外客户访问服务。从这两个端口到来的数据都需要经过反向代理kube-proxy流入后端pod的targetPod，从而到达pod上的容器内。

