# 图解Kubernetes网络（一）

 本文阐述了Kubernetes网络模型，并详细描述了Kubernetes Pods在节点内和节点间的通信方式，帮助读者在碰到Kubernetes网络问题时从容应对。  
  
你一直在Kubernetes集群中运行一系列服务并已从中获益，或者你正打算这么做。尽管有一系列工具能帮助你建立并管理集群，你仍困惑于集群底层是如何工作的，以及出现问题该如何处理。我曾经就是这样的。

![](../../.gitbook/assets/image%20%2843%29.png)

  
诚然Kubernetes对初学者来说已足够易用，但我们仍然不得不承认，它的底层实现异常复杂。Kubernetes由许多部件组成，如果你想对失败场景做好应对准备，那么你必须知道各部件是如何协调工作的。其中一个最复杂，甚至可以说是最关键的部件就是网络。  
  
因此我着手精确理解Kubernetes网络是如何工作的。我阅读了许多文章，看了很多演讲，甚至浏览了代码库。以下就是我的所得。  


#### Kubernetes网络模型

核心点是，Kubernetes网络有一个重要的基本设计原则：  
  
**每个Pod拥有唯一的IP**。  
  
这个Pod IP被该Pod内的所有容器共享，并且其它所有Pod都可以路由到该Pod。你可曾注意到，你的Kubernetes节点上运行着一些"pause"容器？它们被称作“沙盒容器（sandbox containers）"，其唯一任务是保留并持有一个网络命名空间（netns），该命名空间被Pod内所有容器共享。通过这种方式，即使一个容器死掉，新的容器创建出来代替这个容器，Pod IP也不会改变。这种IP-per-pod模型的巨大优势是，Pod和底层主机不会有IP或者端口冲突。我们不用担心应用使用了什么端口。  
  
这点满足后，Kubernetes唯一的要求是，这些Pod IP可被其它所有Pod访问，不管那些Pod在哪个节点。  


**节点内通信**

第一步是确保同一节点上的Pod可以相互通信，然后可以扩展到跨节点通信、internet上的通信，等等。

![Kubernetes Node&#xFF08;root network namespace&#xFF09;](../../.gitbook/assets/image%20%28120%29.png)

在每个Kubernetes节点（本场景指的是Linux机器）上，都有一个根（root）命名空间（root是作为基准，而不是超级用户）--root netns。  
  
最主要的网络接口 `eth0` 就是在这个root netns下。

![Kubernetes Node&#xFF08;pod network namespace&#xFF09;](../../.gitbook/assets/image%20%28163%29.png)

  
类似的，每个Pod都有其自身的netns，通过一个虚拟的以太网对连接到root netns。这基本上就是一个管道对，一端在root netns内，另一端在Pod的nens内。  
  
我们把Pod端的网络接口叫 `eth0`，这样Pod就不需要知道底层主机，它认为它拥有自己的根网络设备。另一端命名成比如 `vethxxx`。你可以用`ifconfig` 或者 `ip a` 命令列出你的节点上的所有这些接口。

![Kubernetes Node&#xFF08;linux network bridge&#xFF09;](../../.gitbook/assets/image%20%28145%29.png)

  
节点上的所有Pod都会完成这个过程。这些Pod要相互通信，就要用到linux的以太网桥 `cbr0` 了。Docker使用了类似的网桥，称为`docker`。  
  
你可以用 `brctl show` 命令列出所有网桥。

[![5.gif](http://dockone.io/uploads/article/20180121/e585af8eb1d0245815d6e430f58e016a.gif)](http://dockone.io/uploads/article/20180121/e585af8eb1d0245815d6e430f58e016a.gif)  
_Kubernetes Node（same node pod-to-pod communication）_  
  
假设一个网络数据包要由`pod1`到`pod2`。

1. 它由`pod1`中netns的`eth0`网口离开，通过`vethxxx`进入root netns。
2. 然后被传到`cbr0`，`cbr0`使用ARP请求，说“谁拥有这个IP”，从而发现目标地址。
3. `vethyyy`说它有这个IP，因此网桥就知道了往哪里转发这个包。
4. 数据包到达`vethyyy`，跨过管道对，到达`pod2`的netns。

  
这就是同一节点内容器间通信的流程。当然也可以用其它方式，但是无疑这是最简单的方式，同时也是Docker采用的方式。  


**节点间通信**

正如我前面提到，Pod也需要跨节点可达。Kubernetes不关心如何实现。我们可以使用L2（ARP跨节点），L3（IP路由跨节点，就像云提供商的路由表），Overlay网络，或者甚至信鸽。无所谓，只要流量能到达另一个节点的期望Pod就好。每个节点都为Pod IPs分配了唯一的CIDR块（一段IP地址范围），因此每个Pod都拥有唯一的IP，不会和其它节点上的Pod冲突。  
  
大多数情况下，特别是在云环境上，云提供商的路由表就能确保数据包到达正确的目的地。我们在每个节点上建立正确的路由也能达到同样的目的。许多其它的网络插件通过自己的方式达到这个目的。  
  
这里我们有两个节点，与之前看到的类似。每个节点有不同的网络命名空间、网络接口以及网桥。  
[![6.gif](http://dockone.io/uploads/article/20180121/c56fb68ab8e146619d51807feafeae98.gif)](http://dockone.io/uploads/article/20180121/c56fb68ab8e146619d51807feafeae98.gif)

  
_Kubernetes Nodes with route table（cross node pod-to-pod communication）_  
  
假设一个数据包要从`pod1`到达`pod4`（在不同的节点上）。  


1. 它由`pod1`中netns的`eth0`网口离开，通过`vethxxx`进入root netns。
2. 然后被传到`cbr0`，`cbr0`通过发送ARP请求来找到目标地址。
3. 本节点上没有Pod拥有`pod4`的IP地址，因此数据包由`cbr0` 传到 主网络接口 `eth0`.
4. 数据包的源地址为`pod1`，目标地址为`pod4`，它以这种方式离开`node1`进入电缆。
5. 路由表有每个节点的CIDR块的路由设定，它把数据包路由到CIDR块包含`pod4`的IP的节点。
6. 因此数据包到达了`node2`的主网络接口`eth0`。现在即使`pod4`不是`eth0`的IP，数据包也仍然能转发到`cbr0`，因为节点配置了IP forwarding enabled。节点的路由表寻找任意能匹配`pod4` IP的路由。它发现了 `cbr0` 是这个节点的CIDR块的目标地址。你可以用`route -n`命令列出该节点的路由表，它会显示`cbr0`的路由，类型如下：

   ![](../../.gitbook/assets/image%20%2827%29.png)

7. 网桥接收了数据包，发送ARP请求，发现目标IP属于`vethyyy`。
8. 数据包跨过管道对到达`pod4`。

  
这就是Kubernetes网络的基础。下次你碰到问题，务必先检查这些网桥和路由表。  
  
先说到这里。在下一部分，我们将看到[Overlay网络是如何工作的](https://medium.com/@ApsOps/an-illustrated-guide-to-kubernetes-networking-part-2-13fdc6c4e24c)，Pod创建和删除过程中网络变化是如何发生的，以及出站和进站流量是如何流动的。

原文：[http://dockone.io/article/3211](http://dockone.io/article/3211)  


