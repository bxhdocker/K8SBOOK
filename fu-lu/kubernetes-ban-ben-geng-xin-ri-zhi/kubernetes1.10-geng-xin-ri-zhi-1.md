# Kubernetes1.11更新日志

**6 月 27 日，Kubernetes 2018 年的第二个版本 Kubernetes 1.11 正式发布，继续推动 Kubernetes 走向成熟**。Kubernetes 1.11 的可扩展性和灵活性更强，饱含着技术团队过去一年中在功能上作出的巨大努力。Kubernetes 1.11 增强了网络方面的主要功能，为 SIG-API Machinery 和 SIG-Node 提供了两个主要功能用于 beta 测试，持续增强过去两个版本关注的存储功能。Kubernetes 1.11 功能的更新为任何基础架构，云或内部部署都能嵌入到 Kubernetes 系统中增添了更多可能性。

Release Note  11 个重要更新

#### 1. SIG API Machinery

此次发布，SIG API Machinery 主要集中在 CustomResoures 方面。比如，CustomResources 的子资源现在进入 beta 版本，并且默认开启。根据这个变化， 对 /status 子资源的更新不允许修改除了  .status 外的所有其他字段（不像以前那样只允许对 .spec 和 .metadata 进行更新）。还有，在 /status 子资源 enable 的情况下， Required 和 Description 可用在 CRD OpenAPI 验证模式的根上。

另外，用户可以创建多版本的 CustomResourceDefinitions，但是不需要任何类型的自动转换，而且 CustomResourceDefinitions 现在允许通过 spec.additionalPrinterColumns 字段为 kubectl 提供附加列的规范。

#### 2. SIG Auth

这次发布周期的工作主要集中在升级现有功能，以及让用户理解安全相关功能。

1.9 版本引进的 RBAC，在这里升级为稳定版本， client-go credential plugins 也升级为 beta 版本，同时支持从外部插件获取 TLS 资格证书。Kubernetes 1.11 更容易看到事件信息，因为 API 请求信息的处理情况可以添加到 audit events 上了。

* Authorization 设置\`authorization.k8s.io/decision\` 注解，表示 authorization 决定（allow 或者 forbid），\`authorization.k8s.io/reason\` 注解，显示为什么会做这个决定的描述。
* PodSecurityPolicy admission 设置\`podsecuritypolicy.admission.k8s.io/admit-policy\` 和 \`podsecuritypolicy.admission.k8s.io/validate-policy\`注解，包含接纳 Pod 的策略名称。（PodSecurityPolicy 同时可以限制 hostPath volume mounts 为 read-only）

另外， NodeRestriction admission 插件阻止 kubelet 修改 Node API object 的 taints，这让我们更容易追踪那些正在被使用的 Node。



#### 3. SIG CLI

SIG CLI 主要重构了 kubectl 内部结构，提升了 kubectl 命令行的可组合性，可读性和可测性。这些重构将使团队能够在下一个版本中提取出一种实现 kubectl \(即插件\)的可扩展性的机制。

#### 4. SIG Cluster Lifecycle

这方面主要通过添加一组维护 kubeadm 配置文件的命令来提升 kubeadm 的用户体验，API 版本提升到了 v1alpha2。这些命令可以处理配置版本迁移，打印默认配置，并且列出和拉取启动一个集群需要的容器镜像。

**其他几个值得注意的变化：**

* CoreDNS 代替 kube-dns 成为默认的 DNS 提供商
* 提升了用户环境体验和支持除 docker 外其他 CRI 运行时
* 支持了 kubelet 结构化配置

#### 5. SIG Instrumentation

Kubernetes 1.11 版本中采用新的 Kubernetes 监控模型，弃用 Heapster。仍在使用 Heapster 做自动弹性伸缩的集群应该迁移到 metrics-server 和 custom metrics API。

#### 6. SIG Network

此次发布中网络部分重要的里程碑是基于 IPVS 的负载均衡和 CoreDNS 升级为 GA。IPVS 是一个集群内部，使用内核哈希表（in-kernel hash tables）的负载均衡的方案，替代先前的 iptables。 CoreDNS 是替代原来的 kube-dns 来做服务发现。

#### 7. SIG Node

Node 方面推动了一些特性开发，并且在一些关键主题上做了额外的提升。

动态调整 kubelet 配置特性升级到了 beta，默认启用，简化了节点对象的自我管理。配置与 CRI 一起工作的 kubelet 可以使用日志滚动特性（log rotation feature），该特性将在本版本中升级为 beta 。

cri-tools 项目已经到了 GA 阶段，该项目主要为操作人员提供一致的工具，使他们能够独立于所选择的容器运行时对生产中的节点进行调试和检测。

平台方面，和 SIG-Windows 配合，kubelet 在 Windows 系统下的支持得到了很大的提升，资源管理方面也做了一些提升。特别是在 Linux 下支持 sysctls 的功能已进入 beta 阶段。

  
8. SIG OpenStack

SIG OpenStack 在继续完善测试，其中 11 个验收测试涵盖了广泛的场景和用例。在 1.11 版本周期中，我们给 test-grid 的报告已经将 OpenStack cloud provider 限定为 Kubernetes 版本发布的一个门控任务。  


新功能包括改进 Keystone 服务和 Kubernetes RBAC 之间的集成，以及整个提供商代码库中的许多稳定性和兼容性改进。

#### 9. SIG Scheduling

Pod 优先级和抢占升级到了 Beta，默认启用。注意这个功能变化对运维特别重要。团队同样在努力提升 scheduler 的性能和可靠性。

#### 10. SIG Storage

存储方面升级了以前版本的两个特性，同时引入了三个 alpha 版本的新特性。

* StorageProtection feature：阻止删除那些正在被 Pod 使用的 PVC，以及已经绑定到 PVC 上面的 PV，现在这个 feature 已经 GA。
* Volume resizing feature：允许在 Pod 重启的时候对 volume 进行扩容，现在是 beta，默认是打开状态。

新的 alpha 特性包括：

* Onlize volume resizing： 不用重启 Pod 的情况下，支持对 fs 进行扩容；
* Dynamic max volume per node count： 限制每个节点的最大 attached volume 个数（仅限：AWS EBS 和 GCE PD）；
* Provide environment variables expansion in sub path mount： 支持用 Downward API 环境变量创建 Subpath volume derectories。

  
11. SIG Windows

此版本支持更多用于 Windows 上的 Pod 和 container 的 Kubernetes API，其中包括：  


* Pod，Container，日志文件系统的监控信息；
* run\_as\_user 安全文本；
* Azure 磁盘的本地持久化卷和 fstype。

Windows Server 1803 版的改进还为 Kubernetes v1.11 带来了新的存储功能，其中包括：

* ConfigMap 和 Secret 的卷挂载
* SMB 和 iSCSI 存储的 Flexvolume 插件也可以在外部的 Microsoft/K8s-Storage-Plugins 项目中找到。

##                                          **关键功能更新**

**值得注意的是，新版本中两个备受期待的功能进入了 GA 阶段，即：基于 IPVS 的集群内负载均衡和 CoreDNS 作为集群 DNS 附加选项**，这意味着增加了生产应用程序的可扩展性和灵活性。接下来让我们深入了解 Kubernetes 1.11 的一些关键功能。

  
基于 IPVS 的集群内服务负载均衡进入到 GA 阶段

新版本中，基于 IPVS 的集群内服务负载均衡功能已趋于稳定。IPVS（IP 虚拟服务器）提供了高性能的内核内负载均衡，其编程接口比 iptables 更简单。这个改变为集群范围内包含 Kubernetes Service 模型的分布式负载均衡提供了更好的网络吞吐，更好的编程延迟和更好的扩展性。IPVS 还不是默认设置，但集群可以在产品流量中使用。

  
CoreDNS 进入 GA 阶段

CoreDNS 现在可用作集群 DNS 附加选项，在使用 kubeadm 时是默认选项。CoreDNS 是一个灵活的，可扩展的权威 DNS 服务器，并集成在 Kubernetes API 中。CoreDNS 比以前的 DNS 服务器拥有更少的移动部件，因为它是单可执行文件和单进程，并且通过创建自定义 DNS 条目来支持灵活的用例。它也是用 Go 语言编写的，具有内存安全性。

  
动态 Kubelet 配置升级到 Beta 阶段

通过此功能，可以在运行的集群中部署新的 Kubelet 配置。目前，Kubelet 可以通过命令行标志进行配置，这使得更新正在运行的集群中的 Kubelet 配置变得困难。借助此 beta 功能，用户可以通过 API server 在运行的群集中配置 Kubelet 。

  
CustomResourceDefinitions 现在可以定义多个版本

CustomResourceDefinitions 不再局限于定义单一版本的自定义资源，这是一项难以解决的限制。现在，利用此测试版功能，可以定义资源的多个版本。在未来，这将扩大到支持一些自动转换; 目前，此功能允许自定义资源作者“以安全更改进行升级，例如 v1beta1 到 v1”，并为有变化的资源创建迁移路径。

CustomResourceDefinitions 现在支持 “status” 和 “scale” 子资源，这些子资源与监控和高可用性框架相集成。这两项更改提高了使用 CustomResourceDefinitions 在生产中运行云原生应用程序的能力。  
  
CSI 的增强

容器存储接口（CSI）在过去几个版本中一直是一个主要问题。在 1.10 版本发布之后，1.11 版本继续增强 CSI 功能。1.11 版本将原始块卷的 alpha 支持添加到 CSI，将 CSI 与新的 kubelet 插件注册机制集成在一起，并且更容易将密钥传递给 CSI 插件。

  
新的存储功能

支持在线调整 Persistent Volume 的大小已被引入作为 alpha 功能。这使用户可以增加 PV 的大小，而无需先终止 Pod 并卸载卷。用户将更新 PVC 以请求新的尺寸，kubelet 将调整 PVC 的文件系统尺寸。  
  
作为 alpha 功能引入了对动态最大卷计数的支持。此新功能使 in-tree 卷插件能够指定可以附加到节点的最大卷数，并允许限制因节点类型而异。以前，这些限制是通过硬编码或通过环境变量进行配置的。

StorageObjectInUseProtection 特性现在很稳定，可以防止删除绑定到 PVC 的 PV，以及 Pod 使用的 PVC。这一保护措施将有助于防止删除当前绑定到活跃 Pod 的 PV 或 PVC 的问题。

  
正式上线

Kubernetes 1.11 目前已经可通过 GitHub 进行下载 \[1\]。要开始使用 Kubernetes，请点击此处 \[2\] 参阅相关交互式教程。您也可以利用 Kubeadm 安装 1.11 版本。1.11.0 版本将以 Deb 与 RPM 软件包的形式提供，并于 6 月 28 日通过 Kubeadm 集群安装器进行安装 \[3\]。

### 总结-K8S 1.11版本更新

1.Pod 优先级和抢占升级到了 Beta，默认启用

2.网络方面:基于 IPVS 的负载均衡和 CoreDNS 升级为 GA,IPVS 替代先前的 iptables,CoreDNS 是替代 kube-dns 来做服务发现, CoreDNS现在是v1.1.3

3.Kubernetes 1.11 版本中采用新的 Kubernetes 监控模型，弃用 Heapster。仍在使用 Heapster 做自动弹性伸缩的集群应该迁移到 metrics-server 和 custom metrics API

4.动态调整 kubelet 配置特性升级到了 beta，默认启用，简化了节点对象的自我管理

5.存储方面升级了以前版本的两个特性，同时引入了三个 alpha 版本的新特性。

* StorageProtection feature：阻止删除那些正在被 Pod 使用的 PVC，以及已经绑定到 PVC 上面的 PV，现在这个 feature 已经 GA。
* Volume resizing feature：允许在 Pod 重启的时候对 volume 进行扩容，现在是 beta，默认是打开状态。

   新的 alpha 特性包括：

* Onlize volume resizing： 不用重启 Pod 的情况下，支持对 fs 进行扩容；
* Dynamic max volume per node count： 限制每个节点的最大 attached volume 个数（仅限：AWS EBS 和 GCE PD）；
* Provide environment variables expansion in sub path mount： 支持用 Downward API 环境变量创建 Subpath volume derectories。

6. [为Pod中的容器提供RunAsGroup功能](https://github.com/kubernetes/features/issues/213) 作为Kubernetes用户，我应该能够为每个容器基础上的容器中运行的容器指定用户ID和组ID，类似于docker允许使用docker运行选项-u ，--user =“”用户名或UID

7.PodSecurityPolicy admission 设置\`podsecuritypolicy.admission.k8s.io/admit-policy\` 和 \`podsecuritypolicy.admission.k8s.io/validate-policy\`注解，包含接纳 Pod 的策略名称。（PodSecurityPolicy 同时可以限制 hostPath volume mounts 为 read-only）

Known Issues

1.基于 IPVS 的 kube-proxy 不支持用于终止 pod 的优雅关闭连接，这个问题将在未来的版本中修复

2.在某些环境中，需要将 kube-proxy 配置为重载主机名

3.Vertical Pod Autoscaler 将在 1.12 中彻底改变，因此 1.11 中的 VPA（alpha）用户将收到警告，他们将无法自动将其 VPA 配置从 1.11 迁移到 1.12

**弃用:**

 1.etcd 2作为后端已被弃用，并且在Kubernetes 1.13中将删除支持

 2.InfluxDB集群监视已被弃用, 将在v1.12中删除

3. kubelet `--rotate-certificates`标志现已被弃用，并将在未来版本中删除。现在可以通过`.RotateCertificates`kubelet的配置文件中的字段启用kubelet证书轮换功能

 4.Kubelets将不再设置`externalID`。自v1.1以来，此功能已被弃用

 5.`service.alpha.kubernetes.io/tolerate-unready-endpoints`已弃用。用户应该使用Service.spec.publishNotReadyAddresses

 6.`kubectl rolling-update`现在已被弃用。`kubectl rollout`改为使用

 7.与Alpha ScheduleDaemonSetPods功能标志关联的DaemonSet调度已被删除

 8.`METADATA_AGENT_VERSION`配置选项已被删除

### 升级前须知

1.包含具有不正确大小写字段的 JSON 配置文件将不再有效，升级前必须更正这些文件

2. kube-apiserver `--storage-version`标志已被删除;

3. kubectl：此客户端版本需要`apps/v1`API

4. Pod优先和预占现在默认启用  `PriorityClass`API被提升为`scheduling.k8s.io/v1beta1`

5.`PersistentVolumeLabel`准入控制器现在默认情况下禁用

6. CRI：更新容器日志路径的文档。容器日志路径已从containername\_attempt＃.log更改为containername / attempt＃.log

7. Kubelet已弃用的--allow-privileged标志现在默认为true

  


```text
总结-K8S 1.11版本更新
1.Pod 优先级和抢占升级到了 Beta，默认启用
2.网络方面:基于 IPVS 的负载均衡和 CoreDNS 升级为 GA,IPVS 替代先前的 iptables,CoreDNS 是替代 kube-dns 来做服务发现, CoreDNS现在是v1.1.3
3.Kubernetes 1.11 版本中采用新的 Kubernetes 监控模型，弃用 Heapster。仍在使用 Heapster 做自动弹性伸缩的集群应该迁移到 metrics-server 和 custom metrics API
4.动态调整 kubelet 配置特性升级到了 beta，默认启用，简化了节点对象的自我管理
5.存储方面升级了以前版本的两个特性，同时引入了三个 alpha 版本的新特性。
   StorageProtection feature：阻止删除那些正在被 Pod 使用的 PVC，以及已经绑定到 PVC 上面的 PV，现在这个 feature 已经 GA。
   Volume resizing feature：允许在 Pod 重启的时候对 volume 进行扩容，现在是 beta，默认是打开状态。
  新的 alpha 特性包括：
    Onlize volume resizing： 不用重启 Pod 的情况下，支持对 fs 进行扩容；
    Dynamic max volume per node count： 限制每个节点的最大 attached volume 个数（仅限：AWS EBS 和 GCE PD）；
    Provide environment variables expansion in sub path mount： 支持用 Downward API 环境变量创建 Subpath volume derectories。
6. 为Pod中的容器提供RunAsGroup功能
 作为Kubernetes用户，能够为每个容器基础上的容器中运行的容器指定用户ID和组ID，类似于docker允许使用docker运行选项-u ，--user =“”用户名或UID
7.PodSecurityPolicy admission 设置`podsecuritypolicy.admission.k8s.io/admit-policy` 和 `podsecuritypolicy.admission.k8s.io/validate-policy`注解，包含接纳 Pod 的策略名称。（PodSecurityPolicy 同时可以限制 hostPath volume mounts 为 read-only）
Known Issues
1.基于 IPVS 的 kube-proxy 不支持用于终止 pod 的优雅关闭连接，这个问题将在未来的版本中修复
2.在某些环境中，需要将 kube-proxy 配置为重载主机名
3.Vertical Pod Autoscaler 将在 1.12 中彻底改变，因此 1.11 中的 VPA（alpha）用户将收到警告，他们将无法自动将其 VPA 配置从 1.11 迁移到 1.12
弃用:
 1.etcd 2作为后端已被弃用，并且在Kubernetes 1.13中将删除支持
 2.InfluxDB集群监视已被弃用, 将在v1.12中删除
 3. kubelet --rotate-certificates标志现已被弃用，并将在未来版本中删除。现在可以通过.RotateCertificateskubelet的配置文件中的字段启用kubelet证书轮换功能
 4.Kubelets将不再设置externalID。自v1.1以来，此功能已被弃用
 5.service.alpha.kubernetes.io/tolerate-unready-endpoints已弃用。用户应该使用Service.spec.publishNotReadyAddresses
 6.kubectl rolling-update现在已被弃用。kubectl rollout改为使用
 7.与Alpha ScheduleDaemonSetPods功能标志关联的DaemonSet调度已被删除
 8.METADATA_AGENT_VERSION配置选项已被删除
升级前须知
1.包含具有不正确大小写字段的 JSON 配置文件将不再有效，升级前必须更正这些文件
2. kube-apiserver --storage-version标志已被删除;
3. kubectl：此客户端版本需要apps/v1API
4. Pod优先和预占现在默认启用  PriorityClassAPI被提升为scheduling.k8s.io/v1beta1
5.PersistentVolumeLabel准入控制器现在默认情况下禁用
6. CRI：更新容器日志路径的文档。容器日志路径已从containername_attempt＃.log更改为containername / attempt＃.log
7. Kubelet已弃用的--allow-privileged标志现在默认为true
```

参考文献：

\[1\]https://github.com/kubernetes/kubernetes/releases/tag/v1.11.0

\[2\] https://kubernetes.io/docs/tutorials/

\[3\] https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

\[4\] https://kubernetes.io/blog/2018/06/27/kubernetes-1.11-release-announcement/

[https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md)

