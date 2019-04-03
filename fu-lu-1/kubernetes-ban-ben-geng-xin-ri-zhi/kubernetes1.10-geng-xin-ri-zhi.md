# Kubernetes1.10更新日志

2018年3月26日，kubernetes1.10版本发布，这是2018年发布的第一个版本。该版本的Kubernetes主要提升了Kubernetes的成熟度、可扩展性与可插入性。

该版本提升了三大关键性功能的稳定度，分别为存储、安全与网络。

另外，此次新版本有几个值得注意的新增特性 

   `引入了外部kubectl凭证提供程序（处于alpha测试阶段）、`

   `在安装时将默认的DNS服务切换为CoreDNS（beta测试阶段）`

    `以及容器存储接口（简称CSI）与持久化本地卷的beta测试版。`

下面再分别说下三大关键更新。

### 存储 <a id="&#x5B58;&#x50A8;"></a>

* CSI（容器存储接口）迎来Beta版本，可以通过插件的形式安装存储。
* 持久化本地存储管理也迎来Beta版本。
* 对PV的一系列更新，可以自动阻止Pod正在使用的PVC的删除，阻止已绑定到PVC的PV的删除操作，这样可以保证所有存储对象可以按照正确的顺序被删除。

### 安全 <a id="&#x5B89;&#x5168;"></a>

* kubectl可以对接不同的凭证提供程序
* 各云服务供应商、厂商以及其他平台开发者现在能够发布二进制插件以处理特定云供应商IAM服务的身价验证

### 网络 <a id="&#x7F51;&#x7EDC;"></a>

* 将原来的kube-dns切换为CoreDNS

### 主要更新 10 个模块

**Node**

Node 方面的更新大多围绕控制层面：  


* Kubelet 动态配置（Dynamic Kubelet Configuration beta 版本）：可以在 Node 不下线的情况下改变 Kubelet 的配置；
* 可配置 Pod 中的容器是否共享一个 namespace（alpha 版本）；
* CRI 方面也做了一些改进，并升级到 v1alpha2 版本：支持 Windows 容器的配置； CRI 验证测试的 beta 版本。

**此外，资源管理 Working Group 在 1.10 版本中将 3 个功能升级到了 beta 版本：**

* **CPU Manager**
*   允许用户请求独有的 CPU 核，这将带来几个方面的性能提升；
*   对网络延时敏感的应用；
*   对 CPU 缓存敏感的应用。
* **Huge Pages**
*   允许 Pod 请求 2Mi 或者 1Gi 的 Huge Pages；
*   这将对内存有很大需求的应用带来好处。
* **Device Plugin**
*   提供了第三方设备资源发现的机制；
*  包括 GPU，FPGA，高性能 NIC，InfiniBand；
*   以及其他相似的需要特定设置的计算资源。

**Sig-storage**

Kubernetes 1.10 版本对本地存储和持久化存储都进行了增强：   


* **Mount namespace propagation** 允许容器以 rslave 的方式挂载存储卷，这样主机的挂载就可以在容器里面可见，或者以 rshared 的方式挂载，这样容器里面的挂载，主机也可以看到。
* **Local Ephemeral Storage Capacity Isolation** 允许设置本地临时存储的 requests 和 limits 。 另外，你也可以创建本地持久化存储，也就是说可以利用 attach 到本地的磁盘来创建 PV， 而不像以前那样，只能基于网络存储卷创建 PV。 
* **PV** 方面，1.10 版本包含 PV/PVC 保护， 如果 PVC/PV 正在被 Pod/PVC 使用，那么就不能直接删除 PVC/PV。
* 这个版本还包括本地持久化存储的 Topology Aware Volume Scheduling ，稳定版本的 Detailed storage metrics of internal state 以及 beta 版本的 Out-of-tree CSI Volume Plugins.

**Windows**

1.10 版本继续对 Windows 进行现有 feature 增强，包括容器 CPU 资源，镜像文件系统数据，和 flexvolumes。 还增加了 Windows 服务控制管理器（Windows service control manager）支持，对单容器 Pod 进行 Hyper-V 隔离实验性支持。

**OpenStack**

SIG-OpenStack 更新 OpenStack provider 。可以使用更新的 APIs，合并代码到一个库，和 Cloud Provider Working Group 合作制定长久计划，把 provider  相关的代码移动到一个单独的代码库里面，改善代码测试，增强与 OpenStack 开发社区的联系。

**API-machinery**

API Aggregation 在 1.10 中升级为稳定版本，可以用于生产。 Webhooks 也有了许多改进，包括对 alpha 版本 self-hosting authorizer webhooks 的支持。

**Auth**

Kubernetes 1.10 版本为新的 authentication 方法添加奠定了基础，包括 alpha 版本的 External client-go credential providers 和 TokenRequest API。另外，Pod Security Policy 允许管理员决定 Pods 可以跑在什么样的 contexts 中，还允许管理员限制 node 对 API 的访问。

**Azure**

Kubernetes 1.10 包含 alpha 版本的 Azure support for cluster-autoscaler， 还有 support for Azure Virtual Machine Scale Sets。

**CLI**

新版本包含了对 kubectl get and describe 的改进，以便更好地与 extensions 结合。为了更好的用户体验，server 端将会返回这些信息。

**Cluster Lifecycle**

这个版本包括对 beta 版本 out-of-process and out-of-tree cloud providers 的支持。

**Network**

**Kubernetes 1.10 关于网络的变化集中在控制层面**。用户现在可以对 Pod 的 resolv.conf 进行配置，而不是依赖 cluster DNS，这个是 beta 版本，用户还可以配置 NodePort IP 地址，也可以把默认的 DNS 插件改为 CoreDNS （beta 版本）

### 获取 <a id="&#x83B7;&#x53D6;"></a>

Kubernetes1.10已经可以通过[GitHub下载](https://github.com/kubernetes/kubernetes/releases/tag/v1.10.0)。

### 参考 <a id="&#x53C2;&#x8003;"></a>

[Kubernetes 1.10: Stabilizing Storage, Security, and Networking](http://blog.kubernetes.io/2018/03/kubernetes-1.10-stabilizing-storage-security-networking.html)



