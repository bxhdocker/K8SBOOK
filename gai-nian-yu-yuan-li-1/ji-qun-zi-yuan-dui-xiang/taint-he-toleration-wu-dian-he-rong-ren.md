# Taint和Toleration（污点和容忍

## Taint和Toleration（污点和容忍） <a id="taint&#x548C;toleration&#xFF08;&#x6C61;&#x70B9;&#x548C;&#x5BB9;&#x5FCD;&#xFF09;"></a>

Taint（污点）和 Toleration（容忍）可以作用于 node 和 pod 上，其目的是优化 pod 在集群间的调度，这跟节点亲和性类似，只不过它们作用的方式相反，具有 taint 的 node 和 pod 是互斥关系，而具有节点亲和性关系的 node 和 pod 是相吸的。另外还有可以给 node 节点设置 label，通过给 pod 设置 `nodeSelector` 将 pod 调度到具有匹配标签的节点上。

Taint 和 toleration 相互配合，可以用来避免 pod 被分配到不合适的节点上。每个节点上都可以应用**一个或多个** taint ，这表示对于那些不能容忍这些 taint 的 pod，是不会被该节点接受的。如果将 toleration 应用于 pod 上，则表示这些 pod 可以（但不要求）被调度到具有相应 taint 的节点上。

### 示例 <a id="&#x793A;&#x4F8B;"></a>

以下分别以为 node 设置 taint 和为 pod 设置 toleration 为例。

### 为 node 设置 taint <a id="&#x4E3A;-node-&#x8BBE;&#x7F6E;-taint"></a>

为 node1 设置 taint：

```text
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

删除上面的 taint：

```text
kubectl taint nodes node1 key1:NoSchedule-
kubectl taint nodes node1 key1:NoExecute-
kubectl taint nodes node1 key2:NoSchedule-
```

查看 node1 上的 taint：

```text
kubectl describe nodes node1
```

### 为 pod 设置 toleration <a id="&#x4E3A;-pod-&#x8BBE;&#x7F6E;-toleration"></a>

只要在 pod 的 spec 中设置 tolerations 字段即可，可以有多个 `key`，如下所示：

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
- key: "node.alpha.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

* `value` 的值可以为 `NoSchedule`、`PreferNoSchedule` 或 `NoExecute`。
* `tolerationSeconds` 是当 pod 需要被驱逐时，可以继续在 node 上运行的时间。

详细使用方法请参考[官方文档](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)。

### 参考 <a id="&#x53C2;&#x8003;"></a>

* [Taints and Tolerations - kuberentes.io](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

