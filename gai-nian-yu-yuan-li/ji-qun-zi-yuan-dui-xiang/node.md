# Node

## Node <a id="node"></a>

 Node 是 Pod 真正运行的主机，可以是物理机，也可以是虚拟机。为了管理 Pod，每个 Node 节点上至少要运行 container runtime（比如 `docker` 或者 `rkt`）、`kubelet` 和 `kube-proxy` 服务。

![](../../.gitbook/assets/image%20%2860%29.png)

### Node的状态 <a id="node&#x7684;&#x72B6;&#x6001;"></a>

每个 Node 都包括以下状态信息：

* 地址：包括 hostname、外网 IP 和内网 IP
* 条件（Condition）：包括 OutOfDisk、Ready、MemoryPressure 和 DiskPressure
* 容量（Capacity）：Node 上的可用资源，包括 CPU、内存和 Pod 总数
* 基本信息（Info）：包括内核版本、容器引擎版本、OS 类型等

Node包括如下状态信息：

* Address
  * HostName：可以被kubelet中的`--hostname-override`参数替代。
  * ExternalIP：可以被集群外部路由到的IP地址。
  * InternalIP：集群内部使用的IP，集群外部无法访问。
* Condition
  * OutOfDisk：磁盘空间不足时为`True`
  * Ready：Node controller 40秒内没有收到node的状态报告为`Unknown`，健康为`True`，否则为`False`。
  * MemoryPressure：当node没有内存压力时为`True`，否则为`False`。
  * DiskPressure：当node没有磁盘压力时为`True`，否则为`False`。
* Capacity
  * CPU
  * 内存
  * 可运行的最大Pod个数
* Info：节点的一些版本信息，如OS、kubernetes、docker等

### Node管理 <a id="node&#x7BA1;&#x7406;"></a>

不像其他的资源（如 Pod 和 Namespace），Node 本质上不是 Kubernetes 来创建的，Kubernetes 只是管理 Node 上的资源。虽然可以通过 Manifest 创建一个 Node 对象（如下 yaml 所示），但 Kubernetes 也只是去检查是否真的是有这么一个 Node，如果检查失败，也不会往上调度 Pod。

```text
kind: Node
apiVersion: v1
metadata:
  name: 10-240-79-157
  labels:
    name: my-first-k8s-node
```

这个检查是由 Node Controller 来完成的。Node Controller 负责

* 维护 Node 状态
* 与 Cloud Provider 同步 Node
* 给 Node 分配容器 CIDR
* 删除带有 `NoExecute` taint 的 Node 上的 Pods

默认情况下，kubelet 在启动时会向 master 注册自己，并创建 Node 资源。

禁止pod调度到该节点上

```text
kubectl cordon <node>
```

驱逐该节点上的所有pod

```text
kubectl drain <node>
```

查看node的资源情况

```text
kubectl top node
```

该命令会删除该节点上的所有Pod（DaemonSet除外），在其他node上重新启动它们，通常该节点需要维护时使用该命令。直接使用该命令会自动调用`kubectl cordon <node>`命令。当该节点维护完成，启动了kubelet后，再使用`kubectl uncordon <node>`即可将该节点添加到kubernetes集群中。

## Taints 和 tolerations <a id="taints-he-tolerations"></a>

Taints 和 tolerations 用于保证 Pod 不被调度到不合适的 Node 上，Taint 应用于 Node 上，而 toleration 则应用于 Pod 上（Toleration 是可选的）。

比如，可以使用 taint 命令给 node1 添加 taints：

```text
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value2:NoExecute
```

Taints 和 tolerations 的具体使用方法请参考 [调度器章节](https://kubernetes.feisky.xyz/he-xin-yuan-li/index-1/scheduler#Taints%20%E5%92%8C%20tolerations)。

