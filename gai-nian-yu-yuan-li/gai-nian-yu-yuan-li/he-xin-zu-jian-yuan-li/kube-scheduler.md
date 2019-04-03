# kube-scheduler

## kube-scheduler <a id="kube-scheduler"></a>

kube-scheduler 负责分配调度 Pod 到集群内的节点上，它监听 kube-apiserver，查询还未分配 Node 的 Pod，然后根据调度策略为这些 Pod 分配节点（更新 Pod 的 `NodeName` 字段）。

调度器需要充分考虑诸多的因素：

* 公平调度
* 资源高效利用
* QoS
* affinity 和 anti-affinity
* 数据本地化（data locality）
* 内部负载干扰（inter-workload interference）
* deadlines

## 调度策略

Kubernetes的调度策略分为Predicates（预选策略）和Priorites（优选策略），整个调度过程分为两步：  
预选策略，是强制性规则，遍历所有的Node，按照具体的预选策略筛选出符合要求的Node列表，如没有Node符合Predicates策略规则，那该Pod就会被挂起，直到有Node能够满足；优选策略，在第一步筛选的基础上，按照优选策略为待选Node打分排序，获取最优者；

## 1.指定 Node 节点调度

有三种方式指定 Pod 只运行在指定的 Node 节点上

* NodeName
* nodeSelector：只调度到匹配指定 label 的 Node 上
* nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作
* podAffinity：调度到满足条件的 Pod 所在的 Node 上

1.0，NodeName示例

`Pod.spec.nodeName`用于强制约束将Pod调度到指定的Node节点上，这里说是“调度”，但其实指定了nodeName的Pod会直接跳过Scheduler的调度逻辑，直接写入PodList列表，该匹配规则是强制匹配。

例子：

```text
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      nodeName: k8s.node1 #指定调度节点为k8s.node1，写ip也可以
      containers:
      - name: tomcat
        image: tomcat:8.0
        ports:
        - containerPort: 8080
```

### 1.1、nodeSelector 示例

首先给 Node 打上标签

```text
kubectl label nodes node-01 disktype=ssd
```

然后在 daemonset 中指定 nodeSelector 为 `disktype=ssd`：

```text
spec:
  nodeSelector:
    disktype: ssd
```

### 1.2、nodeAffinity 示例

nodeAffinity 目前支持两种：requiredDuringSchedulingIgnoredDuringExecution 和 preferredDuringSchedulingIgnoredDuringExecution，分别代表必须满足条件和优选条件。比如下面的例子代表调度到包含标签 `kubernetes.io/e2e-az-name` 并且值为 e2e-az1 或 e2e-az2 的 Node 上，并且优选还带有标签 `another-node-label-key=another-node-label-value` 的 Node。

```text
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: gcr.io/google_containers/pause:2.0
```

### 1.3、podAffinity 示例

podAffinity 基于 Pod 的标签来选择 Node，仅调度到满足条件 Pod 所在的 Node 上，支持 podAffinity 和 podAntiAffinity。这个功能比较绕，以下面的例子为例：

* 如果一个 “Node 所在 Zone 中包含至少一个带有 `security=S1` 标签且运行中的 Pod”，那么可以调度到该 Node
* 不调度到 “包含至少一个带有 `security=S2` 标签且运行中 Pod” 的 Node 上

```text
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: gcr.io/google_containers/pause:2.0
```

## 2、Taints 和 tolerations

Taints 和 tolerations 用于保证 Pod 不被调度到不合适的 Node 上，其中 Taint 应用于 Node 上，而 toleration 则应用于 Pod 上。

目前支持的 taint 类型

* NoSchedule：新的 Pod 不调度到该 Node 上，不影响正在运行的 Pod
* PreferNoSchedule：soft 版的 NoSchedule，尽量不调度到该 Node 上
* NoExecute：新的 Pod 不调度到该 Node 上，并且删除（evict）已在运行的 Pod。Pod 可以增加一个时间（tolerationSeconds），

然而，当 Pod 的 Tolerations 匹配 Node 的**所有** Taints 的时候可以调度到该 Node 上；当 Pod 是已经运行的时候，也不会被删除（evicted）。另外对于 NoExecute，如果 Pod 增加了一个 tolerationSeconds，则会在该时间之后才删除 Pod。

比如，假设 node1 上应用以下几个 taint

```text
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

下面的这个 Pod 由于没有 tolerate`key2=value2:NoSchedule` 无法调度到 node1 上

```text
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```

而正在运行且带有 tolerationSeconds 的 Pod 则会在 600s 之后删除

```text
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 600
- key: "key2"
  operator: "Equal"
  value: "value2"
  effect: "NoSchedule"
```

注意，DaemonSet 创建的 Pod 会自动加上对 `node.alpha.kubernetes.io/unreachable` 和 `node.alpha.kubernetes.io/notReady` 的 NoExecute Toleration，以避免它们因此被删除。

## 3、优先级调度（priorityClass）

从 v1.8 开始，kube-scheduler 支持定义 Pod 的优先级，从而保证高优先级的 Pod 优先调度，默认是disable的。

Kubernetes中的抢占式调度，简称为Preemption，1.10后已经废弃[rescheduler](https://github.com/kubernetes-incubator/descheduler)，改用优先级priorityClass

在最新版的1.11又改为默认开启了

| Kubernetes Version | Priority and Preemption State | Enabled by default |
| :--- | :--- | :--- |
| 1.8 | alpha | no |
| 1.9 | alpha | no |
| 1.10 | alpha | no |
| 1.11 | beta | yes |

3.1、开启方法为

* apiserver 配置 `--feature-gates=PodPriority=true` 和 `--runtime-config=scheduling.k8s.io/v1alpha1=true`
* kube-scheduler 配置 `--feature-gates=PodPriority=true`
* kubelet配置 `--feature-gates=PodPriority=true`

以api server为例

```text
···以上省略····修改ExecStart字段   k8s1.10为例
ExecStart={{ bin_dir }}/kube-apiserver \
  --runtime-config=scheduling.k8s.io/v1alpha1=true  \
  --enable-admission-plugins=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,NodeRestriction,Priority \
  --feature-gates=PodPriority=true \
  ····以下省略····
```

反过来，把上面的参数删除并重启，即可disable。

有个问题：如果我开启了这个特性，并且创建了一些PriorityClass，然后还给某些Pod使用了，这个时候我再disable掉这个特性，会不会有问题？

答案是否定的！disable后，那些之前设置的Pod Priority field还会继续存在，但是并没什么用处了，Preemption是关闭的。当然，你也不能给新的Pods引用PriorityClass了。

原文链接 [PriorityClass](https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/#rescheduler-guaranteed-scheduling-of-critical-add-ons)

3.2、使用系统定义优先级 优先级为20亿， pod必须在`kube-system`命名空间（通过标志可配置）中运行

```text
priorityClassName: system-node-critical
```

3.3、使用自定义优先级

在指定 Pod 的优先级之前需要先定义一个 PriorityClass（非 namespace 资源），如

```text
apiVersion: v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

{% hint style="warning" %}
注意：PriorityClass是非namespace隔离的，是global的。因此metadata下面是不能设置namespace field的
{% endhint %}

其中

* apiVersion: scheduling.k8s.io/v1alpha1 \# enable的时候需要配置的runtime-config参数
* metadata.name: 设置PriorityClass的名字；
* `value` 32-bit 整型值，值越大代表优先级越高，但是必须小于等于 1 billion，大于 1 billion的值是给集群内的critical system Pods保留的，表示该Priority的Pod是不能被抢占的。
* `globalDefault` 用于未配置 PriorityClassName 的 Pod，整个集群中应该只有一个 PriorityClass 将其设置为 true，如果没有一个PriorityClass的该字段为true，则那些没有明确引用PriorityClass的Pod的Priority value就为最低值0
* description: String，写给人看的备注，Kubernetes不做处理；

然后，在 PodSpec 中通过 PriorityClassName 设置 Pod 的优先级：

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

{% hint style="warning" %}
如果Pod.spec. priorityClassName中指定的PriorityClass不存在，则Pod会创建失败；  
 前面也提到，创建Pod的时候Priority Admission Controller会根据PriorityClassName找到对应的PriorityClass，并将其value设置给Pod.spec.priority
{% endhint %}

{% hint style="danger" %}
注意：

* PriorityClass只会影响那些还没创建的Pod，一旦Pod创建完成，那么admission Controller就已经将Pod Spec中应用的PriorityClassName对应的PriorityClass的value设置到Pod的Priority field了。意味着你再修改PriorityClass的任何field，包括globalDefault，也不会影响已经创建完成的Pod。
* 如果你删除某个PriorityClass，那么不会影响已经引用它的Pod Priority，但你不能用它来创建新的Pod了。这其实是显而易见的。
{% endhint %}

#### Preemption当前还存在的问题

* 因为抢占式调度evict低优先级Pod时，有一个优雅终止时间\(默认30s\),如果该Node上需要evict多个低优先级的Pod，那么可能会需要很长的时间后，最终Pod才能调度到该Node上并启动运行，那么问题来了，这么长时间的过去了，这个Node是否此时此刻还是最适合这个Pod的呢？不一定！而且在大规模且创建Pod频繁的集群中，这种结果是经常的。意味着，当初合正确的调度决定，在真正落实的时候却一定时正确的了。
* 还不支持preempted pod时考虑PDB，计划会在beta版中实现；
* 目前premmpted pod时没考虑Pending pod和victims pod的亲和性：如果该pending pod要调度到的node上需要evict的lower Priority pods和该pending pod是亲和的，目前直接evict lower Priority pods就可能会破坏这种pod亲和性。
* 不支持跨节点抢占。比如pending Pod M要调度到Node A，而该Pending Pod M又与“同Node A一个zone的Node B上的Pod N”是基于zone topology反亲和性的，目前的Alpha版本就会导致Pod M继续Pending不能成功调度。如果后续支持跨界点抢占，就能将lower Priority的Pod N从Node B上evict掉，从而保证了反亲和性。

### 多调度器 <a id="&#x591A;&#x8C03;&#x5EA6;&#x5668;"></a>

如果默认的调度器不满足要求，还可以部署自定义的调度器。并且，在整个集群中还可以同时运行多个调度器实例，通过 `podSpec.schedulerName` 来选择使用哪一个调度器（默认使用内置的调度器）。

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  # 选择使用自定义调度器 my-scheduler
  schedulerName: my-scheduler
  containers:
  - name: nginx
    image: nginx:1.10
```

调度器的示例参见 [这里](https://kubernetes.feisky.xyz/zh/plugins/scheduler.html)。

### 调度器扩展 <a id="&#x8C03;&#x5EA6;&#x5668;&#x6269;&#x5C55;"></a>

kube-scheduler 还支持使用 `--policy-config-file` 指定一个调度策略文件来自定义调度策略，比如

```text
{
"kind" : "Policy",
"apiVersion" : "v1",
"predicates" : [
    {"name" : "PodFitsHostPorts"},
    {"name" : "PodFitsResources"},
    {"name" : "NoDiskConflict"},
    {"name" : "MatchNodeSelector"},
    {"name" : "HostName"}
    ],
"priorities" : [
    {"name" : "LeastRequestedPriority", "weight" : 1},
    {"name" : "BalancedResourceAllocation", "weight" : 1},
    {"name" : "ServiceSpreadingPriority", "weight" : 1},
    {"name" : "EqualPriority", "weight" : 1}
    ],
"extenders":[
    {
        "urlPrefix": "http://127.0.0.1:12346/scheduler",
        "apiVersion": "v1beta1",
        "filterVerb": "filter",
        "prioritizeVerb": "prioritize",
        "weight": 5,
        "enableHttps": false,
        "nodeCacheCapable": false
    }
    ]
}
```

### 其他影响调度的因素 <a id="&#x5176;&#x4ED6;&#x5F71;&#x54CD;&#x8C03;&#x5EA6;&#x7684;&#x56E0;&#x7D20;"></a>

* 如果 Node Condition 处于 MemoryPressure，则所有 BestEffort 的新 Pod（未指定 resources limits 和 requests）不会调度到该 Node 上
* 如果 Node Condition 处于 DiskPressure，则所有新 Pod 都不会调度到该 Node 上
* 为了保证 Critical Pods 的正常运行，当它们处于异常状态时会自动重新调度。Critical Pods 是指
  * annotation 包括 `scheduler.alpha.kubernetes.io/critical-pod=''  在1.10弃用，1.12版本中删除`
  * tolerations 包括 `[{"key":"CriticalAddonsOnly", "operator":"Exists"}]`

### 启动 kube-scheduler 示例 <a id="&#x542F;&#x52A8;-kube-scheduler-&#x793A;&#x4F8B;"></a>

```text
kube-scheduler --address=127.0.0.1 --leader-elect=true --kubeconfig=/etc/kubernetes/scheduler.conf
```

### kube-scheduler 工作原理 <a id="kube-scheduler-&#x5DE5;&#x4F5C;&#x539F;&#x7406;"></a>

kube-scheduler 调度原理：

```text
For given pod:

    +---------------------------------------------+
    |               Schedulable nodes:            |
    |                                             |
    | +--------+    +--------+      +--------+    |
    | | node 1 |    | node 2 |      | node 3 |    |
    | +--------+    +--------+      +--------+    |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+

    Pred. filters: node 3 doesn't have enough resource

    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+
    |             remaining nodes:                |
    |   +--------+                 +--------+     |
    |   | node 1 |                 | node 2 |     |
    |   +--------+                 +--------+     |
    |                                             |
    +-------------------+-------------------------+
                        |
                        |
                        v
    +-------------------+-------------------------+

    Priority function:    node 1: p=2
                          node 2: p=5

    +-------------------+-------------------------+
                        |
                        |
                        v
            select max{node priority} = node 2
```

kube-scheduler 调度分为两个阶段，predicate 和 priority

* predicate：过滤不符合条件的节点
* priority：优先级排序，选择优先级最高的节点

predicates 策略

* PodFitsPorts：同 PodFitsHostPorts
* PodFitsHostPorts：检查是否有 Host Ports 冲突
* PodFitsResources：检查 Node 的资源是否充足，包括允许的 Pod 数量、CPU、内存、GPU 个数以及其他的 OpaqueIntResources
* HostName：检查 `pod.Spec.NodeName` 是否与候选节点一致
* MatchNodeSelector：检查候选节点的 `pod.Spec.NodeSelector` 是否匹配
* NoVolumeZoneConflict：检查 volume zone 是否冲突
* MaxEBSVolumeCount：检查 AWS EBS Volume 数量是否过多（默认不超过 39）
* MaxGCEPDVolumeCount：检查 GCE PD Volume 数量是否过多（默认不超过 16）
* MaxAzureDiskVolumeCount：检查 Azure Disk Volume 数量是否过多（默认不超过 16）
* MatchInterPodAffinity：检查是否匹配 Pod 的亲和性要求
* NoDiskConflict：检查是否存在 Volume 冲突，仅限于 GCE PD、AWS EBS、Ceph RBD 以及 ISCSI
* GeneralPredicates：分为 noncriticalPredicates 和 EssentialPredicates。noncriticalPredicates 中包含 PodFitsResources，EssentialPredicates 中包含 PodFitsHost，PodFitsHostPorts 和 PodSelectorMatches。
* PodToleratesNodeTaints：检查 Pod 是否容忍 Node Taints
* CheckNodeMemoryPressure：检查 Pod 是否可以调度到 MemoryPressure 的节点上
* CheckNodeDiskPressure：检查 Pod 是否可以调度到 DiskPressure 的节点上
* NoVolumeNodeConflict：检查节点是否满足 Pod 所引用的 Volume 的条件

priorities 策略

* SelectorSpreadPriority：优先减少节点上属于同一个 Service 或 Replication Controller 的 Pod 数量
* InterPodAffinityPriority：优先将 Pod 调度到相同的拓扑上（如同一个节点、Rack、Zone 等）
* LeastRequestedPriority：优先调度到请求资源少的节点上
* BalancedResourceAllocation：优先平衡各节点的资源使用
* NodePreferAvoidPodsPriority：alpha.kubernetes.io/preferAvoidPods 字段判断, 权重为 10000，避免其他优先级策略的影响
* NodeAffinityPriority：优先调度到匹配 NodeAffinity 的节点上
* TaintTolerationPriority：优先调度到匹配 TaintToleration 的节点上
* ServiceSpreadingPriority：尽量将同一个 service 的 Pod 分布到不同节点上，已经被 SelectorSpreadPriority 替代 \[默认未使用\]
* EqualPriority：将所有节点的优先级设置为 1\[默认未使用\]
* ImageLocalityPriority：尽量将使用大镜像的容器调度到已经下拉了该镜像的节点上 \[默认未使用\]
* MostRequestedPriority：尽量调度到已经使用过的 Node 上，特别适用于 cluster-autoscaler\[默认未使用\]

> **代码入口路径**
>
> 与 Kubernetes 其他组件的入口不同 \(其他都是位于 `cmd/` 目录\)，kube-schedular 的入口在 `plugin/cmd/kube-scheduler`

### 总结

考虑以下两种情况：

* 集群中有新增节点，想要让集群中的节点的资源利用率比较均衡一些，想要将一些高负载的节点上的pod驱逐到新增节点上，这是kuberentes的scheduer所不支持的，需要使用如[rescheduler](https://github.com/kubernetes-incubator/descheduler)这样的插件来实现。写法如下

```text
    scheduler.alpha.kubernetes.io/critical-pod: '' #Rescheduler 最后被驱逐
    scheduler.alpha.kubernetes.io/tolerations: |   #在1.10后已经废弃，改用优先级priorityClass
       [{"key": "dedicated", "value": "master", "effect": "NoSchedule" },
        {"key":"CriticalAddonsOnly", "operator":"Exists"}]
```

* 想要运行一些大数据应用，设计到资源分片，pod需要与数据分布达到一致均衡，避免个别节点处理大量数据，而其它节点闲置导致整个作业延迟，这时候可以考虑使用[kube-arbitritor](https://github.com/kubernetes-incubator/kube-arbitrator)。

### 参考文档 <a id="&#x53C2;&#x8003;&#x6587;&#x6863;"></a>

* [Pod Priority and Preemption](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/)
* [Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)
* [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)
* [Advanced Scheduling in Kubernetes](http://blog.kubernetes.io/2017/03/advanced-scheduling-in-kubernetes.html)

