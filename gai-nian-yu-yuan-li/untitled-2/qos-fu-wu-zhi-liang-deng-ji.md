# QoS（服务质量等级）

QoS（Quality of Service），大部分译为“服务质量等级”，又译作“服务质量保证”，是作用在 Pod 上的一个配置，当 Kubernetes 创建一个 Pod 时，它就会给这个 Pod 分配一个 QoS 等级，可以是以下等级之一：+

* **Guaranteed**：Pod 里的每个容器都必须有内存/CPU 限制和请求，而且值必须相等。如果我们配置了limit，没有配置request，默认会以limit的值来定义request
* **Burstable**：Pod 里至少有一个容器有内存或者 CPU 请求且不满足 Guarantee 等级的要求，即内存/CPU 的值设置的不同。
* **BestEffort**：容器必须没有任何内存或者 CPU 的限制或请求。

关于request和limit参考 [pod其他设置](https://darren.gitbook.io/project/~/edit/drafts/-LUF8vUYyGJ55B0Pam-g/gai-nian-yu-yuan-li-1/zi-yuan-dui-xiang/pod/pod-qi-ta-she-zhi#zi-yuan-xian-zhi)

该配置不是通过一个配置项来配置的，而是通过配置 CPU/内存的 `limits` 与 `requests` 值的大小来确认服务质量等级的。使用 `kubectl get pod -o yaml` 可以看到 pod 的配置输出中有 `qosClass` 一项。该配置的作用是为了给资源调度提供策略支持，调度算法根据不同的服务质量等级可以确定将 pod 调度到哪些节点上。

在Kubernetes中，POD的QoS服务质量一共有三个级别，如下图所示：

![](../../.gitbook/assets/image%20%2816%29.png)

这三个QoS级别介绍，可以看下面表格：

| QoS级别 | QoS介绍 |
| :--- | :--- |
| BestEffort | POD中的所有容器都没有指定CPU和内存的requests和limits，那么这个POD的QoS就是BestEffort级别 |
| Burstable | POD中只要有一个容器，这个容器requests和limits的设置同其他容器设置的不一致，那么这个POD的QoS就是Burstable级别 |
| Guaranteed | POD中所有容器都必须统一设置了limits，并且设置参数都一致，如果有一个容器要设置requests，那么所有容器都要设置，并设置参数同limits一致，那么这个POD的QoS就是Guaranteed级别 |

为了更清楚的了解这三个QoS级别，下面我们举例说明。



![](../../.gitbook/assets/image%20%2834%29.png)

![](../../.gitbook/assets/image%20%2842%29.png)

![](../../.gitbook/assets/image%20%2892%29.png)

![](../../.gitbook/assets/image%20%28162%29.png)

![](../../.gitbook/assets/image%20%2838%29.png)

QoS级别决定了kubernetes处理这些POD的方式，我们以内存资源为例：

1、当NODE节点上内存资源不够的时候，QoS级别是BestEffort的POD会最先被kill掉；当NODE节点上内存资源充足的时候，QoS级别是BestEffort的POD可以使用NODE节点上剩余的所有内存资源。

2、当NODE节点上内存资源不够的时候，如果QoS级别是BestEffort的POD已经都被kill掉了，那么会查找QoS级别是Burstable的POD，并且这些POD使用的内存已经超出了requests设置的内存值，这些被找到的POD会被kill掉；当NODE节点上内存资源充足的时候，QoS级别是Burstable的POD会按照requests和limits的设置来使用。

3、当NODE节点上内存资源不够的时候，如果QoS级别是BestEffort和Burstable的POD都已经被kill掉了，那么会查找QoS级别是Guaranteed的POD，并且这些POD使用的内存已经超出了limits设置的内存值，这些被找到的POD会被kill掉；当NODE节点上内存资源充足的时候，QoS级别是Burstable的POD会按照requests和limits的设置来使用。

从容器的角度出发，为了限制容器使用的CPU和内存，是通过cgroup来实现的，目前kubernetes的QoS只能管理CPU和内存，所以kubernetes现在也是通过对cgroup的配置来实现QoS管理的。

![](../../.gitbook/assets/image%20%2869%29.png)



原文链接：[https://blog.csdn.net/horsefoot/article/details/52091077](https://blog.csdn.net/horsefoot/article/details/52091077)

