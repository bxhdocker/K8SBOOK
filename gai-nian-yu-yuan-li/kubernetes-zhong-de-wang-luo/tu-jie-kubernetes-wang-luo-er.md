# 图解Kubernetes网络（二）

 我们漫谈了Kubernetes的网络模型。我们观察了数据包是如何在同一节点上的pod 间和跨节点的 pod 间流动的。我们也注意到了Linux网桥和路由表在这个过程中所扮演的角色。  
  
今天我们将进一步阐述这些概念，并阐述Overlay网络是如何工作的。我们也将理解Kubernetes千变万化的Pod是如何从运行的应用中抽象出来，并在幕后处理的。  


#### Overlay 网络

Overlay网络不是默认必须的，但是它们在特定场景下非常有用。比如当我们没有足够的IP空间，或者网络无法处理额外路由，抑或当我们需要Overlay提供的某些额外管理特性。一个常见的场景是当云提供商的路由表能处理的路由数是有限制时。例如，AWS路由表最多支持50条路由才不至于影响网络性能。因此如果我们有超过50个Kubernetes节点，AWS路由表将不够。这种情况下，使用Overlay网络将帮到我们。  
  
本质上来说，Overlay就是在跨节点的本地网络上的包中再封装一层包。你可能不想使用Overlay网络，因为它会带来由封装和解封所有报文引起的时延和复杂度开销。通常这是不必要的，因此我们应当在知道为什么我们需要它时才使用它。  
  
为了理解Overlay网络中流量的流向，我们拿[Flannel](http://github.com/coreos/flannel)做例子，它是CoreOS 的一个开源项目。  
[![2.gif](http://dockone.io/uploads/article/20180121/2da4d778f04c46ccf4dceab80c5a62dc.gif)](http://dockone.io/uploads/article/20180121/2da4d778f04c46ccf4dceab80c5a62dc.gif)  
_Kubernetes Node with route table（cross node pod-to-pop Traffic flow with flannel overlay network）_  
  
这里我们注意到它和之前我们看到的设施是一样的，只是在root netns中新增了一个虚拟的以太网设备，称为flannel0。它是虚拟扩展网络Virtual Extensible LAN（VXLAN）的一种实现，但是在Linux上，它只是另一个网络接口。  
  
从`pod1`到`pod4`（在不同节点）的数据包的流向类似如下：  
  
1、它由`pod1`中netns的`eth0`网口离开，通过`vethxxx`进入root netns。  
  
2、然后被传到`cbr0`，cbr0通过发送ARP请求来找到目标地址。  
  
3a、由于本节点上没有Pod拥有`pod4`的IP地址，因此网桥把数据包发送给了`flannel0`，因为节点的路由表上`flannel0`被配成了Pod网段的目标地址。  
  
3b、flanneld daemon和Kubernetes apiserver或者底层的etcd通信，它知道所有的Pod IP，并且知道它们在哪个节点上。因此Flannel创建了Pod IP和Node IP之间的映射（在用户空间）。  
  
`flannel0`取到这个包，并在其上再用一个UDP包封装起来，该UDP包头部的源和目的IP分别被改成了对应节点的IP，然后发送这个新包到特定的VXLAN端口（通常是8472）。  
[![3.png](http://dockone.io/uploads/article/20180121/4d5b77faa9892a0993deda5bed432421.png)](http://dockone.io/uploads/article/20180121/4d5b77faa9892a0993deda5bed432421.png)  
_Packet-in-packet encapsulation（notice the packet is encapsulated from 3c to 6b in previous diagram）_  
  
尽管这个映射发生在用户空间，真实的封装以及数据的流动发生在内核空间，因此仍然是很快的。  
  
3c、封装后的包通过`eth0`发送出去，因为它涉及了节点间的路由流量。  
  
4、包带着节点IP信息作为源和目的地址离开本节点。  
  
5、云提供商的路由表已经知道了如何在节点间发送报文，因此该报文被发送到目标地址`node2`。  
  
6a、包到达node2的`eth0`网卡，由于目标端口是特定的VXLAN端口，内核将报文发送给了 `flannel0`。  
  
6b、`flannel0`解封报文，并将其发送到 root 命名空间下。  
  
从这里开始，报文的路径和我们之前在[Part 1](http://dockone.io/article/3211) 中看到的非Overlay网络就是一致的了。  
  
6c、由于IP forwarding开启着，内核按照路由表将报文转发给了`cbr0`。  
  
7、网桥获取到了包，发送ARP请求，发现目标IP属于`vethyyy`。  
  
8、包跨过管道对到达`pod4`。  
  
这就是Kubernetes中Overlay网络的工作方式，虽然不同的实现还是会有细微的差别。有个常见的误区是，当我们使用Kubernetes，我们就不得不使用Overlay网路。事实是，这完全依赖于特定场景。因此请确保在确实需要的场景下才使用。  
  
本部分到此结束。在[前一部分](http://dockone.io/article/3211)我们学习了Kubernetes网络的基础知识。现在我们知道了Overlay网络是如何工作的。在下一部分，我们将看到Pod创建和删除过程中网络变化是如何发生的，以及出站和进站流量是如何流动的。

