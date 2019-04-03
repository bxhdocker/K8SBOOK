# Kubernetes1.12更新日志

## Kubernetes1.12更新日志 <a id="kubernetes112&#x66F4;&#x65B0;&#x65E5;&#x5FD7;"></a>

Kubernetes 1.12中包含38项功能。

> https://github.com/kubernetes/features/milestone/11

该版本发布继续关注Kubernetes的稳定性，主要是内部改进和一些功能的毕业。该版本中毕业的功能有安全性和Azure的关键功能。此版本中还有两个毕业的值得注意的新增功能：Kubelet TLS Bootstrap和 Azure Virtual Machine Scale Sets（AVMSS）支持。

这些新功能意味着更高的安全性、可用性、弹性和易用性，可以更快地将生产应用程序推向市场。该版本还表明Kubernetes在开发人员方面日益成熟。

下面是该版本中的一些关键功能介绍。

### Kubelet TLS Bootstrap GA <a id="kubelet-tls-bootstrap-ga"></a>

我们很高兴地宣布[Kubelet TLS Bootstrap GA](https://github.com/kubernetes/features/issues/43)。在Kubernetes 1.4中，我们引入了一个API，用于从集群级证书颁发机构（CA）请求证书。此API的初衷是为kubelet启用TLS客户端证书的配置。此功能允许kubelet将自身引导至TLS安全集群。最重要的是，它可以自动提供和分发签名证书。

之前，当kubelet第一次运行时，必须在集群启动期间在带外进程中为其提供客户端凭据。负担是运营商提供这些凭证的负担。由于此任务对于手动执行和复杂自动化而言非常繁重，因此许多运营商为所有kubelet部署了具有单个凭证和单一身份的集群。这些设置阻止了节点锁定功能的部署，如节点授权器和NodeRestriction准入控制器。

为了缓解这个问题，[SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth)引入了一种方法，让kubelet生成私钥，CSR用于提交到集群级证书签署过程。v1（GA）标识表示生产加固和准备就绪，保证长期向后兼容。

除此之外，[Kubelet服务器证书引导程序和轮换](https://github.com/kubernetes/features/issues/267)正在转向测试版。目前，当kubelet首次启动时，它会生成一个自签名证书/密钥对，用于接受传入的TLS连接。此功能引入了一个在本地生成密钥，然后向集群API server发出证书签名请求以获取由集群的根证书颁发机构签名的关联证书的过程。此外，当证书接近过期时，将使用相同的机制来请求更新的证书。

### 稳定支持Azure Virtual Machine Scale Sets（VMSS）和Cluster-Autoscaler <a id="&#x7A33;&#x5B9A;&#x652F;&#x6301;azure-virtual-machine-scale-sets&#xFF08;vmss&#xFF09;&#x548C;cluster-autoscaler"></a>

Azure Virtual Machine Scale Sets（VMSS）允许您创建和管理可以根据需求或设置的计划自动增加或减少的同类VM池。这使您可以轻松管理、扩展和负载均衡多个VM，从而提供高可用性和应用程序弹性，非常适合可作为Kubernetes工作负载运行的大型应用程序。

凭借这一新的稳定功能，Kubernetes支持[使用Azure VMSS扩展容器化应用程序](https://github.com/kubernetes/features/issues/514)，包括[将其与cluster-autoscaler集成的功能](https://github.com/kubernetes/features/issues/513)根据相同的条件自动调整Kubernetes集群的大小。

### 其他值得注意的功能更新 <a id="&#x5176;&#x4ED6;&#x503C;&#x5F97;&#x6CE8;&#x610F;&#x7684;&#x529F;&#x80FD;&#x66F4;&#x65B0;"></a>

* [`RuntimeClass`](https://github.com/kubernetes/features/issues/585)是一个新的集群作用域资源，它将容器运行时属性表示为作为alpha功能发布的控制平面。
* [Kubernetes和CSI的快照/恢复功能](https://github.com/kubernetes/features/issues/177)正在作为alpha功能推出。这提供了标准化的API设计（CRD），并为CSI卷驱动程序添加了PV快照/恢复支持。
* [拓扑感知动态配置](https://github.com/kubernetes/features/issues/561)现在处于测试阶段，存储资源现在可以感知自己的位置。这还包括对[AWS EBS](https://github.com/kubernetes/features/issues/567)和[GCE PD](https://github.com/kubernetes/features/issues/558)的beta支持。
* [可配置的pod进程命名空间共享](https://github.com/kubernetes/features/issues/495)处于测试阶段，用户可以通过在PodSpec中设置选项来配置pod中的容器以共享公共PID命名空间。
* [根据条件的taint节点](https://github.com/kubernetes/features/issues/382)现在处于测试阶段，用户可以通过使用taint来表示阻止调度的节点条件。
* Horizo​ntal Pod Autoscaler中的[任意/自定义指标](https://github.com/kubernetes/features/issues/117)正在转向第二个测试版，以测试一些其他增强功能。这项重新设计的Horizo​ntal Pod Autoscaler功能包括对自定义指标和状态条件的支持。
* 允许[Horizo​ntal Pod Autoscaler更快地达到适当大小](https://github.com/kubernetes/features/issues/591)正在转向测试版。
* [Pod的垂直缩放](https://github.com/kubernetes/features/issues/21)现在处于测试阶段，使得可以在其生命周期内改变pod上的资源限制。
* [通过KMS进行静态加密](https://github.com/kubernetes/features/issues/460)目前处于测试阶段。增加了多个加密提供商，包括Google Cloud KMS、Azure Key Vault、AWS KMS和Hashicorp Vault，它们会在数据存储到etcd时对其进行加密。



新亮点。

**一、Kubelet证书轮换**  


Kubelet证书轮换功能现已进入beta状态。这一功能可以在当前证书到期时自动续订密钥和kubelet API服务器的证书。在官方1.12文档发布之前，您可以在此处阅读有关此功能的测试版文档：

> https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping.md\#kubelet-configuration

二、网络策略：CIDR选择器和egress规则

有两个以前的beta功能现在已是stable状态：其中一个是ipBlock选择器，它允许根据CIDR表示法中的网络地址指定ingress/egress规则。第二个则可以通过指定egress规则来过滤离开pod的流量。以下示例说明了这两个功能的使用：

> apiVersion: networking.k8s.io/v1
>
> kind: NetworkPolicy
>
> metadata:
>
>   name: network-policy
>
>   namespace: default
>
> spec:
>
>   podSelector:
>
>     matchLabels:
>
>       role: app
>
>   policyTypes:
>
>   - Egress
>
>   egress:
>
>   - to:
>
>     - ipBlock:
>
>         cidr: 10.0.0.0/24
>
> \(...\)

egress和ipBlock以前都是beta功能，它们已经在Kubernetes官方的网络策略文档中了：

> https://kubernetes.io/docs/concepts/services-networking/network-policies/

三、挂载命名空间传播

挂载命名空间传播，即挂载卷  rshared ，从而容器内的任何挂载都能反映在root（= host）挂载命名空间中，这一功能现已是stable状态。您可以在Kubernetes卷文档中阅读有关此功能的更多信息：

> https://kubernetes.io/docs/concepts/storage/volumes/\#mount-propagation

四、按条件创建Taint Nodes

在Kubernetes1.8中，这一功能还是早期alpha版本，现在此功能已升级为beta。启用它的featureflag，节点控制器可以根据节点条件创建taints，并使调度器根据taints而不是条件来过滤节点。官方文档在此：

> https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/\#taint-nodes-by-condition

五、具有自定义指标的水平pod自动伸缩器

虽然HPA中对自定义指标的支持一直是beta状态，但1.12版增加了各种增强功能，例如可以根据监控管道中的可用标签选择指标。如果您对基于Prometheus、Sysdig或Datadog等监控系统提供的应用程序级指标自动调整pod感兴趣，我建议您查看 HPA中外部指标的设计方案：

> https://github.com/kubernetes/community/blob/master/contributors/design-proposals/autoscaling/hpa-external-metrics.md

六、RuntimeClass

RuntimeClass是一个新的集群范围的资源，“它将容器运行时属性表示到控制平面”。换言之，它可以让用户通过提供PodSpec中的runtimeClass，选择和配置（每个pod）特定容器运行时（如Docker、RKT或Virtlet）。这一功能还处于早期alpha阶段，更多信息可以参阅此处：

> https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/containers/runtime-class.md

七、资源配额优先级

资源配额让管理员可以限制命名空间中的资源消耗。这一功能在多个租户（用户／团队）共享集群中的可用计算和存储资源时尤其实用。beta版的资源配额优先级允许管理员根据pod的PriorityClass，确定配额范围，从而调整命名空间内的资源分配。你可以在这里了解更多细节：

> https://kubernetes.io/docs/concepts/policy/resource-quotas/\#resource-quota-per-priorityclass

八、卷快照

Kubernetes 1.12中最令人的兴奋的存储功能之一，是持久性卷快照（尽管它还在alpha阶段）。此功能允许用户在任何CSI存储提供商支持的特定时间点创建和恢复快照。此次更新添加了三个新的API资源作为此功能的一部分：

* VolumeSnapshotClass定义如何配置现有卷的快照；
* VolumeSnapshotContent表示现有快照；
* VolumeSnapshot允许用户请求持久卷的新快照

下面是示例：

> apiVersion: snapshot.storage.k8s.io/v1alpha1
>
> kind: VolumeSnapshot
>
> metadata:
>
>   name: new-snapshot-test
>
> spec:
>
>   snapshotClassName: csi-hostpath-snapclass
>
>   source:
>
>     name: pvc-test
>
>     kind: PersistentVolumeClaim

详细信息可查看Github 上的1.12 文档分支：

> https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/storage/volume-snapshots.md

九、拓扑感知动态配置

另一个与存储相关的功能，拓扑感知动态配置。这一功能在Kubernetes 1.11中初次引入，并在1.12中被提升为beta状态。它解决了在跨多个区域的集群中动态配置卷的一些限制，其中单区存储后端无法从所有节点全局访问。

十、对Azure的增强支持

在Kubernetes 1.12中，有两项关于在Azure中运行Kubernetes的增强：

* 集群自动伸缩

  Azure 的集群自动伸缩器支持已升级为稳定版。这将允许基于全局资源，自动扩展Kubernetes集群中的Azure节点数。

* Azure可用区支持

  Kubernetes 1.12添加了Azure可用区（AZ）的alpha支持。可用区域中的节点将添加标签  failure-domain.beta.kubernetes.io/zone=&lt;region&gt;-&lt;AZ&gt; ，并为Azure托管磁盘存储类添加拓扑感知配置。

### 可用性 <a id="&#x53EF;&#x7528;&#x6027;"></a>

Kubernetes 1.12可以[在GitHub上下载](https://github.com/kubernetes/kubernetes/releases/tag/v1.12.0)。要开始使用Kubernetes，请查看这些[交互式教程](https://kubernetes.io/docs/tutorials/)。您也可以使用[Kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)来安装1.12。

### 参考 <a id="&#x53C2;&#x8003;"></a>

* [Kubernetes 1.12: Kubelet TLS Bootstrap and Azure Virtual Machine Scale Sets \(VMSS\) Move to General Availability](https://kubernetes.io/blog/2018/09/27/kubernetes-1.12-kubelet-tls-bootstrap-and-azure-virtual-machine-scale-sets-vmss-move-to-general-availability/)

