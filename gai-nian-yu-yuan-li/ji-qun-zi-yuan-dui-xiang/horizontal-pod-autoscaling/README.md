# Horizontal Pod Autoscaling

## Horizontal Pod Autoscaling <a id="horizontal-pod-autoscaling"></a>

应用的资源使用率通常都有高峰和低谷的时候，如何削峰填谷，提高集群的整体资源利用率，让service中的Pod个数自动调整呢？这就有赖于Horizontal Pod Autoscaling了，顾名思义，使Pod水平自动缩放。这个Object（跟Pod、Deployment一样都是API resource）也是最能体现kubernetes之于传统运维价值的地方，不再需要手动扩容了，终于实现自动化了，还可以自定义指标，没准未来还可以通过人工智能自动进化呢！

HPA属于Kubernetes中的**autoscaling** SIG（Special Interest Group），其下有两个feature：

* [Arbitrary/Custom Metrics in the Horizontal Pod Autoscaler\#117](https://github.com/kubernetes/features/issues/117)
* [Monitoring Pipeline Metrics HPA API \#118](https://github.com/kubernetes/features/issues/118)

Kubernetes自1.2版本引入HPA机制，到1.6版本之前一直是通过kubelet来获取监控指标来判断是否需要扩缩容，1.6版本之后必须通过API server、Heapseter或者kube-aggregator来获取监控指标。

对于1.6以前版本中开启自定义HPA请参考[Kubernetes autoscaling based on custom metrics without using a host port](https://medium.com/@marko.luksa/kubernetes-autoscaling-based-on-custom-metrics-without-using-a-host-port-b783ed6241ac)，对于1.7及以上版本请参考[Configure Kubernetes Autoscaling With Custom Metrics in Kubernetes 1.7 - Bitnami](https://docs.bitnami.com/kubernetes/how-to/configure-autoscaling-custom-metrics/)。

### HPA解析 <a id="hpa&#x89E3;&#x6790;"></a>

Horizontal Pod Autoscaling仅适用于Deployment和ReplicaSet，在V1版本中仅支持根据Pod的CPU利用率扩所容，在v1alpha版本中，支持根据内存和用户自定义的metric扩缩容。

Horizontal Pod Autoscaling，简称HPA，是Kubernetes中实现POD水平自动伸缩的功能。自动扩展主要分为两种:

**水平扩展\(scale out\)**，针对于实例数目的增减 

**垂直扩展\(scal up\)**，即单个实例可以使用的资源的增减, 比如增加cpu和增大内存

**HPA属于前者。它可以根据CPU使用率或应用自定义metrics自动扩展Pod数量\(支持 replication controller、deployment 和 replica set\)**

如果你不想看下面的文章可以直接看下面的示例图，组件交互、组件的配置、命令示例，都画在图上了。

Horizontal Pod Autoscaling由API server和controller共同实现。

![](../../../.gitbook/assets/image%20%28116%29.png)



图片 - horizontal-pod-autoscaler

### Metrics支持 <a id="metrics&#x652F;&#x6301;"></a>

在不同版本的API中，HPA autoscale时可以根据以下指标来判断：

* autoscaling/v1
  * CPU
* autoscaling/v1alpha1
  * 内存
  * 自定义metrics
    * kubernetes1.6起支持自定义metrics，但是必须在kube-controller-manager中配置如下两项：
      * `--horizontal-pod-autoscaler-use-rest-clients=true`
      * `--api-server`指向[kube-aggregator](https://github.com/kubernetes/kube-aggregator)，也可以使用heapster来实现，通过在启动heapster的时候指定`--api-server=true`。查看[kubernetes metrics](https://github.com/kubernetes/metrics)
  * 多种metrics组合
    * HPA会根据每个metric的值计算出scale的值，并将最大的那个值作为扩容的最终结果。

### 监控数据获取

     **Heapster**: heapster收集Node节点上的cAdvisor数据，并按照kubernetes的资源类型来集合资源。但是在_**v1.11中已经被废弃**_（heapster监控数据可用，但HPA不再从heapster拿数据）

    **metric-server**: 在v1.8版本中引入，官方将其作为heapster的替代者。metric-server依赖于kube-aggregator，因此需要在apiserver中开启相关参数。v1.11中HPA从metric-server获取监控数据

### 工作流程

1.创建HPA资源，设定目标CPU使用率限额，以及最大、最小实例数, 一定要设置Pod的资源限制参数: request, 否则HPA不会工作。 

2.控制管理器每隔30s\(可以通过–horizontal-pod-autoscaler-sync-period修改\)查询metrics的资源使用情况 

3.然后与创建时设定的值和指标做对比\(平均值之和/限额\)，求出目标调整的实例个数

4.目标调整的实例数不能超过1中设定的最大、最小实例数，如果没有超过，则扩容；超过，则扩容至最大的实例个数

_以上查考：_[_https://blog.csdn.net/u011230692/article/details/86037169_](https://blog.csdn.net/u011230692/article/details/86037169)\_\_

### 如何部署：

* 1.6-1.8版本默认使用heapster
* 1.11版本及以上默认使用metric-server（需单独安装，并开启参数）

如果为1.8及以下的k8s集群，指标正常，如果为1.11集群，需要执行如下操作

![](../../../.gitbook/assets/image%20%2859%29.png)

### 使用kubectl管理 <a id="&#x4F7F;&#x7528;kubectl&#x7BA1;&#x7406;"></a>

Horizontal Pod Autoscaling作为API resource也可以像Pod、Deployment一样使用kubeclt命令管理，使用方法跟它们一样，资源名称为`hpa`。

```text
kubectl create hpa
kubectl get hpa
kubectl describe hpa
kubectl delete hpa
```

有一点不同的是，可以直接使用`kubectl autoscale`直接通过命令行的方式创建Horizontal Pod Autoscaler。

用法如下：

```text
kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS
[--cpu-percent=CPU] [flags] [options]
```

举个例子：

```text
kubectl autoscale deployment foo --min=2 --max=5 --cpu-percent=80
```

为Deployment foo创建 一个autoscaler，当Pod的CPU利用率达到80%的时候，RC的replica数在2到5之间。

**注意** ：如果为ReplicaSet创建HPA的话，无法使用rolling update，但是对于Deployment来说是可以的，因为Deployment在执行rolling update的时候会自动创建新的ReplicationController。

### 实验测试hpa-例子

1.部署和运行php-apache并将其暴露成为服务

![](../../../.gitbook/assets/image%20%2823%29.png)

2.创建hpa

![](../../../.gitbook/assets/image%20%2874%29.png)

3.或者使用yaml文件

![](../../../.gitbook/assets/image%20%28123%29.png)

4.向php-apache服务增加负载，验证自动扩缩容

启动一个容器，并通过一个循环向php-apache服务器发送无限的查询请求（请在另一个终端中运行以下命令）

![](../../../.gitbook/assets/image%20%2883%29.png)

5.观察HPA是否生效

![](../../../.gitbook/assets/image%20%2835%29.png)

HPA由一个控制循环实现，循环周期由–horizontal-pod-autoscaler-sync-period 标志指定，默认是30秒，每个周期内，controller-manager会查询HPA中定义的metric的资源利用率。

 如上例子，pod的request定义为200M，而HPA定义的target为50%，即HPA将通过增加或者减少Pod副本的数量（通过Deployment）以保持所有Pod的平均CPU利用率在50%以内\(即200\*0.5=100M以内），循环周期到达时，获取pod的1分钟内的平均cpu利用率（从heaspter\),发现超过了100M，为332M，于是通过下面的公式，决定最终的pod数量 

`TargetNumOfPods = ceil(sum(CurrentPodsCPUUtilization) / Target) 1` 

即 332/50 =6.xxx ceil为向上取整，得到7。如果得到的结果大于10，则为10 因为每次HPA生效都会创建或者删除pod，而这些操作其实会影响到metric监控值，如创建pod会暂时性的升高cpu，因此每次扩容都要间隔3分钟，缩容需要间隔5分钟。且需要满足：avg（CurrentPodsConsumption）/ Target下降9%，进行缩容，增加至10%才进行扩容 

这样做好处是： 

1、判断的精度高，不会频繁的扩缩pod，造成集群压力大。 

2、避免频繁的扩缩pod，防止应用访问不稳定 

实现hpa的条件： 

1、hpa不能autoscale daemonset类型control

2、要实现autoscale，pod必须设置request

原文：[https://yasongxu.gitbook.io/container-monitor/](https://yasongxu.gitbook.io/container-monitor/)

### 什么是 Horizontal Pod Autoscaling？ <a id="&#x4EC0;&#x4E48;&#x662F;-horizontal-pod-autoscaling&#xFF1F;"></a>

利用 Horizontal Pod Autoscaling，kubernetes 能够根据监测到的 CPU 利用率（或者在 alpha 版本中支持的应用提供的 metric）自动的扩容 replication controller，deployment 和 replica set。

Horizontal Pod Autoscaler 作为 kubernetes API resource 和 controller 的实现。Resource 确定 controller 的行为。Controller 会根据监测到用户指定的目标的 CPU 利用率周期性得调整 replication controller 或 deployment 的 replica 数量。

### Horizontal Pod Autoscaler 如何工作？ <a id="horizontal-pod-autoscaler-&#x5982;&#x4F55;&#x5DE5;&#x4F5C;&#xFF1F;"></a>

Horizontal Pod Autoscaler 由一个控制循环实现，循环周期由 controller manager 中的 `--horizontal-pod-autoscaler-sync-period` 标志指定（默认是 30 秒）。

在每个周期内，controller manager 会查询 HorizontalPodAutoscaler 中定义的 metric 的资源利用率。Controller manager 从 resource metric API（每个 pod 的 resource metric）或者自定义 metric API（所有的metric）中获取 metric。

* 每个 Pod 的 resource metric（例如 CPU），controller 通过 resource metric API 获取 HorizontalPodAutoscaler 中定义的每个 Pod 中的 metric。然后，如果设置了目标利用率，controller 计算利用的值与每个 Pod 的容器里的 resource request 值的百分比。如果设置了目标原始值，将直接使用该原始 metric 值。然后 controller 计算所有目标 Pod 的利用率或原始值（取决于所指定的目标类型）的平均值，产生一个用于缩放所需 replica 数量的比率。 请注意，如果某些 Pod 的容器没有设置相关的 resource request ，则不会定义 Pod 的 CPU 利用率，并且 Aucoscaler 也不会对该 metric 采取任何操作。
* 对于每个 Pod 自定义的 metric，controller 功能类似于每个 Pod 的 resource metric，只是它使用原始值而不是利用率值。
* 对于 object metric，获取单个度量（描述有问题的对象），并与目标值进行比较，以产生如上所述的比率。

HorizontalPodAutoscaler 控制器可以以两种不同的方式获取 metric ：直接的 Heapster 访问和 REST 客户端访问。

当使用直接的 Heapster 访问时，HorizontalPodAutoscaler 直接通过 API 服务器的服务代理子资源查询 Heapster。需要在集群上部署 Heapster 并在 kube-system namespace 中运行。

Autoscaler 访问相应的 replication controller，deployment 或 replica set 来缩放子资源。

Scale 是一个允许您动态设置副本数并检查其当前状态的接口。

### API Object <a id="api-object"></a>

Horizontal Pod Autoscaler 是 kubernetes 的 `autoscaling` API 组中的 API 资源。当前的稳定版本中，只支持 CPU 自动扩缩容，可以在`autoscaling/v1` API 版本中找到。

在 alpha 版本中支持根据内存和自定义 metric 扩缩容，可以在`autoscaling/v2alpha1` 中找到。`autoscaling/v2alpha1` 中引入的新字段在`autoscaling/v1` 中是做为 annotation 而保存的。

### 在 kubectl 中支持 Horizontal Pod Autoscaling <a id="&#x5728;-kubectl-&#x4E2D;&#x652F;&#x6301;-horizontal-pod-autoscaling"></a>

Horizontal Pod Autoscaler 和其他的所有 API 资源一样，通过 `kubectl` 以标准的方式支持。

我们可以使用`kubectl create`命令创建一个新的 autoscaler。

我们可以使用`kubectl get hpa`列出所有的 autoscaler，使用`kubectl describe hpa`获取其详细信息。

最后我们可以使用`kubectl delete hpa`删除 autoscaler。

另外，可以使用`kubectl autoscale`命令，很轻易的就可以创建一个 Horizontal Pod Autoscaler。

例如，执行`kubectl autoscale rc foo —min=2 —max=5 —cpu-percent=80`命令将为 replication controller _foo_ 创建一个 autoscaler，目标的 CPU 利用率是`80%`，replica 的数量介于 2 和 5 之间。

### 滚动更新期间的自动扩缩容 <a id="&#x6EDA;&#x52A8;&#x66F4;&#x65B0;&#x671F;&#x95F4;&#x7684;&#x81EA;&#x52A8;&#x6269;&#x7F29;&#x5BB9;"></a>

目前在Kubernetes中，可以通过直接管理 replication controller 或使用 deployment 对象来执行 [滚动更新](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller)，该 deployment 对象为您管理基础 replication controller。

Horizontal Pod Autoscaler 仅支持后一种方法：Horizontal Pod Autoscaler 被绑定到 deployment 对象，它设置 deployment 对象的大小，deployment 负责设置底层 replication controller 的大小。

Horizontal Pod Autoscaler 不能使用直接操作 replication controller 进行滚动更新，即不能将 Horizontal Pod Autoscaler 绑定到 replication controller，并进行滚动更新（例如使用`kubectl rolling-update`）。

这不行的原因是，当滚动更新创建一个新的 replication controller 时，Horizontal Pod Autoscaler 将不会绑定到新的 replication controller 上。

### 支持多个 metric <a id="&#x652F;&#x6301;&#x591A;&#x4E2A;-metric"></a>

Kubernetes 1.6 中增加了支持基于多个 metric 的扩缩容。您可以使用`autoscaling/v2alpha1` API 版本来为 Horizontal Pod Autoscaler 指定多个 metric。然后 Horizontal Pod Autoscaler controller 将权衡每一个 metric，并根据该 metric 提议一个新的 scale。在所有提议里最大的那个 scale 将作为最终的 scale。

### 支持自定义 metric <a id="&#x652F;&#x6301;&#x81EA;&#x5B9A;&#x4E49;-metric"></a>

**注意：** Kubernetes 1.2 根据特定于应用程序的 metric ，通过使用特殊注释的方式，增加了对缩放的 alpha 支持。

在 Kubernetes 1.6中删除了对这些注释的支持，有利于`autoscaling/v2alpha1` API。 虽然旧的收集自定义 metric 的旧方法仍然可用，但是这些 metric 将不可供 Horizontal Pod Autoscaler 使用，并且用于指定要缩放的自定义 metric 的以前的注释也不在受 Horizontal Pod Autoscaler 认可。

Kubernetes 1.6增加了在 Horizontal Pod Autoscale r中使用自定义 metric 的支持。

您可以为`autoscaling/v2alpha1` API 中使用的 Horizontal Pod Autoscaler 添加自定义 metric 。

Kubernetes 然后查询新的自定义 metric API 来获取相应自定义 metric 的值。

### 前提条件 <a id="&#x524D;&#x63D0;&#x6761;&#x4EF6;"></a>

为了在 Horizontal Pod Autoscaler 中使用自定义 metric，您必须在您集群的 controller manager 中将 `--horizontal-pod-autoscaler-use-rest-clients` 标志设置为 true。然后，您必须通过将 controller manager 的目标 API server 设置为 API server aggregator（使用`--apiserver`标志），配置您的 controller manager 通过 API server aggregator 与API server 通信。 Resource metric API和自定义 metric API 也必须向 API server aggregator 注册，并且必须由集群上运行的 API server 提供。

您可以使用 Heapster 实现 resource metric API，方法是将 `--api-server` 标志设置为 true 并运行 Heapster。 单独的组件必须提供自定义 metric API（有关自定义metric API的更多信息，可从 [k8s.io/metrics repository](https://github.com/kubernetes/metrics) 获得）。



## 自定义指标HPA <a id="&#x81EA;&#x5B9A;&#x4E49;&#x6307;&#x6807;hpa"></a>

Kubernetes中不仅支持CPU、内存为指标的HPA，还支持自定义指标的HPA，例如QPS。

本文中使用的yaml文件见[manifests/HPA](https://github.com/rootsongjc/kubernetes-handbook/tree/master/manifests/HPA)。

### 设置自定义指标 <a id="&#x8BBE;&#x7F6E;&#x81EA;&#x5B9A;&#x4E49;&#x6307;&#x6807;"></a>

**kubernetes1.6**

> 在kubernetes1.6集群中配置自定义指标的HPA的说明已废弃。

在设置定义指标HPA之前需要先进行如下配置：

* 将heapster的启动参数 `--api-server` 设置为 true
* 启用custom metric API
* 将kube-controller-manager的启动参数中`--horizontal-pod-autoscaler-use-rest-clients`设置为true，并指定`--master`为API server地址，如`--master=http://172.20.0.113:8080`

在kubernetes1.5以前很容易设置，参考[1.6以前版本的kubernetes中开启自定义HPA](https://medium.com/@marko.luksa/kubernetes-autoscaling-based-on-custom-metrics-without-using-a-host-port-b783ed6241ac)，而在1.6中因为取消了原来的annotation方式设置custom metric，只能通过API server和kube-aggregator来获取custom metric，因为只有两种方式来设置了，一是直接通过API server获取heapster的metrics，二是部署[kube-aggragator](https://github.com/kubernetes/kube-aggregator)来实现。

我们将在kubernetes1.8版本的kubernetes中，使用聚合的API server来实现自定义指标的HPA。

**kuberentes1.7+**

确认您的kubernetes版本在1.7或以上，修改以下配置：

* 将kube-controller-manager的启动参数中`--horizontal-pod-autoscaler-use-rest-clients`设置为true，并指定`--master`为API server地址，如\`--master=[http://172.20.0.113:8080](http://172.20.0.113:8080/)
* 修改kube-apiserver的配置文件apiserver，增加一条配置`--requestheader-client-ca-file=/etc/kubernetes/ssl/ca.pem --requestheader-allowed-names=aggregator --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --proxy-client-cert-file=/etc/kubernetes/ssl/kubernetes.pem --proxy-client-key-file=/etc/kubernetes/ssl/kubernetes-key.pem`，用来配置aggregator的CA证书。

已经内置了`apiregistration.k8s.io/v1beta1` API，可以直接定义APIService，如：

```text
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1.custom-metrics.metrics.k8s.io
spec:
  insecureSkipTLSVerify: true
  group: custom-metrics.metrics.k8s.io
  groupPriorityMinimum: 1000
  versionPriority: 5
  service:
    name: api
    namespace: custom-metrics
  version: v1alpha1
```

**部署Prometheus**

使用[prometheus-operator.yaml](https://github.com/rootsongjc/kubernetes-handbook/blob/master/manifests/HPA/prometheus-operator.yaml)文件部署Prometheus operator。

**注意：**将镜像修改为你自己的镜像仓库地址。

这产生一个自定义的API：[http://172.20.0.113:8080/apis/custom-metrics.metrics.k8s.io/v1alpha1](http://172.20.0.113:8080/apis/custom-metrics.metrics.k8s.io/v1alpha1)，可以通过浏览器访问，还可以使用下面的命令可以检查该API：

```text
$ kubectl get --raw=apis/custom-metrics.metrics.k8s.io/v1alpha1
```

### 参考 <a id="&#x53C2;&#x8003;"></a>

* HPA说明：[https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* HPA详解：[https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* 自定义metrics开发：[https://github.com/kubernetes/metrics](https://github.com/kubernetes/metrics)
* 1.7版本的kubernetes中启用自定义HPA：[Configure Kubernetes Autoscaling With Custom Metrics in Kubernetes 1.7 - Bitnami](https://docs.bitnami.com/kubernetes/how-to/configure-autoscaling-custom-metrics/)
* [1.6以前版本的kubernetes中开启自定义HPA](https://medium.com/@marko.luksa/kubernetes-autoscaling-based-on-custom-metrics-without-using-a-host-port-b783ed6241ac)
* [1.7版本的kubernetes中启用自定义HPA](https://docs.bitnami.com/kubernetes/how-to/configure-autoscaling-custom-metrics/)
* [Horizontal Pod Autoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
* [Kubernetes 1.8: Now with 100% Daily Value of Custom Metrics](https://blog.openshift.com/kubernetes-1-8-now-custom-metrics/)
* [Arbitrary/Custom Metrics in the Horizontal Pod Autoscaler\#117](https://github.com/kubernetes/features/issues/117)

[https://www.centos.bz/2018/07/kubernetes-pod%E7%9A%84%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9/](https://www.centos.bz/2018/07/kubernetes-pod%E7%9A%84%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9/)

{% embed url="https://2.创建hpa" %}

