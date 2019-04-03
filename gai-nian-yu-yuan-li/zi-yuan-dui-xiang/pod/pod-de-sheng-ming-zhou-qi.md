# Pod 的生命周期

Pod被分配到一个Node上之后，就不会离开这个Node，直到被删除。当某个Pod失败，首先会被Kubernetes清理掉，之后ReplicationController将会在其它机器上（或本机）重建Pod，重建之后Pod的ID发生了变化，那将会是一个新的Pod。所以，Kubernetes中Pod的迁移，实际指的是在新Node上重建Pod。以下给出Pod的生命周期图。

![Pod&#x7684;&#x751F;&#x547D;&#x5468;&#x671F;&#x56FE;](http://images2015.cnblogs.com/blog/704717/201703/704717-20170304103856954-1091445106.png)

### lifecycle

创建资源对象时，可以使用lifecycle来管理容器在运行前和关闭前的一些动作。

lifecycle有两种回调函数：

```text
PostStart：容器创建成功后，运行前的任务，用于资源部署、环境准备等。
PreStop：在容器被终止前的任务，用于优雅关闭应用程序、通知其他系统等等。
```

**生命周期回调函数**：PostStart（容器创建成功后调研该回调函数）、PreStop（在容器被终止前调用该回调函数）

### 部署代码

以下示例中，定义了一个Pod，包含一个JAVA的web应用容器，其中设置了PostStart和PreStop回调函数。即在容器创建成功后，复制/sample.war到/app文件夹中。而在容器终止之前，发送HTTP请求到http://monitor.com:8080/waring，即向监控系统发送警告。

具体示例如下：

```text
......
containers:
- image: sample:v2  
     name: war
     lifecycle：
      postStart:
       exec:
         command:
          - “cp”
          - “/sample.war”
          - “/app”
      prestop:
       httpGet:
        host: monitor.com
        psth: /waring
        port: 8080
        scheme: HTTP
......

```

### Pod phase <a id="pod-phase"></a>

Pod 有一个 PodStatus 对象，其中包含一个 [PodCondition](https://kubernetes.io/docs/resources-reference/v1.7/#podcondition-v1-core) 数组。 PodCondition 数组的每个元素都有一个 `type` 字段和一个 `status` 字段。`type` 字段是字符串，可能的值有 PodScheduled、Ready、Initialized 和 Unschedulable。`status` 字段是一个字符串，可能的值有 True、False 和 Unknown

Pod 的 `status` 在信息保存在 [PodStatus](https://kubernetes.io/docs/resources-reference/v1.7/#podstatus-v1-core) 中定义，其中有一个 `phase` 字段。

Pod 的相位（phase）是 Pod 在其生命周期中的简单宏观概述。该阶段并不是对容器或 Pod 的综合汇总，也不是为了做为综合状态机。

Pod 相位的数量和含义是严格指定的。除了本文档中列举的状态外，不应该再假定 Pod 有其他的 `phase`值。

Kubernetes 以 `PodStatus.Phase` 抽象 Pod 的状态（但并不直接反映所有容器的状态）。可能的 Phase 包括

下面是 `phase` 可能的值：

* 挂起（Pending）： Pod 已经在 apiserver 中创建，但还没有调度到 Node 上面，但有一个或者多个容器镜像尚未创建。等待时间包括调度 Pod 的时间和通过网络下载镜像的时间，这可能需要花点时间。
* 运行中（Running）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
* 成功（Successed）：Pod 中的所有容器都被成功终止，并且不会再重启。
* 失败（Failed）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。
* 未知（Unkonwn）：因为某些原因无法取得 Pod 的状态， 通常是由于 apiserver 无法与 所在主机kubelet 通信导致

下图是Pod的生命周期示意图，从图中可以看到Pod状态的变化。

![Pod&#x7684;&#x751F;&#x547D;&#x5468;&#x671F;&#x793A;&#x610F;&#x56FE;&#xFF08;&#x56FE;&#x7247;&#x6765;&#x81EA;&#x7F51;&#x7EDC;&#xFF09;](https://jimmysong.io/kubernetes-handbook/images/kubernetes-pod-life-cycle.jpg)

### Pod 状态 <a id="pod-&#x72B6;&#x6001;"></a>

可以用 kubectl 命令查询 Pod Phase：

```text
$ kubectl get pod reviews-v1-5bdc544bbd-5qgxj -o jsonpath="{.status.phase}"
Running
```

PodSpec 中的 `restartPolicy` 可以用来设置是否对退出的 Pod 重启，可选项包括 `Always`、`OnFailure`、以及 `Never`。比如

* 单容器的 Pod，容器成功退出时，不同 `restartPolicy` 时的动作为
  * Always: 重启 Container; Pod `phase` 保持 Running.
  * OnFailure: Pod `phase` 变成 Succeeded.
  * Never: Pod `phase` 变成 Succeeded.
* 单容器的 Pod，容器失败退出时，不同 `restartPolicy` 时的动作为
  * Always: 重启 Container; Pod `phase` 保持 Running.
  * OnFailure: 重启 Container; Pod `phase` 保持 Running.
  * Never: Pod `phase` 变成 Failed.
* 2个容器的 Pod，其中一个容器在运行而另一个失败退出时，不同 `restartPolicy` 时的动作为
  * Always: 重启 Container; Pod `phase` 保持 Running.
  * OnFailure: 重启 Container; Pod `phase` 保持 Running.
  * Never: 不重启 Container; Pod `phase` 保持 Running.
* 2个容器的 Pod，其中一个容器停止而另一个失败退出时，不同 `restartPolicy` 时的动作为
  * Always: 重启 Container; Pod `phase` 保持 Running.
  * OnFailure: 重启 Container; Pod `phase` 保持 Running.
  * Never: Pod `phase` 变成 Failed.
* 单容器的 Pod，容器内存不足（OOM），不同 `restartPolicy` 时的动作为
  * Always: 重启 Container; Pod `phase` 保持 Running.
  * OnFailure: 重启 Container; Pod `phase` 保持 Running.
  * Never: 记录失败事件; Pod `phase` 变成 Failed.
* Pod 还在运行，但磁盘不可访问时
  * 终止所有容器
  * Pod `phase` 变成 Failed
  * 如果 Pod 是由某个控制器管理的，则重新创建一个 Pod 并调度到其他 Node 运行
* Pod 还在运行，但由于网络分区故障导致 Node 无法访问
  * Node controller等待 Node 事件超时
  * Node controller 将 Pod `phase` 设置为 Failed.
  * 如果 Pod 是由某个控制器管理的，则重新创建一个 Pod 并调度到其他 Node 运行

### 优雅删除资源对象

当用户请求删除含有pod的资源对象时（如RC、deployment等），K8S为了让应用程序优雅关闭（即让应用程序完成正在处理的请求后，再关闭软件），K8S提供两种信息通知：

```text
1）、默认：K8S通知node执行docker stop命令，docker会先向容器中PID为1的进程发送系统信号SIGTERM，然后等待容器中的应用程序终止执行，如果等待时间达到设定的超时时间，或者默认超时时间（30s），会继续发送SIGKILL的系统信号强行kill掉进程。
2）、使用pod生命周期（利用PreStop回调函数），它执行在发送终止信号之前。
```

 默认情况下，所有的删除操作的优雅退出时间都在30秒以内。kubectl delete命令支持--grace-period=的选项，以运行用户来修改默认值。0表示删除立即执行，并且立即从API中删除pod这样一个新的pod会在同时被创建。在节点上，被设置了立即结束的的pod，仍然会给一个很短的优雅退出时间段，才会开始被强制杀死

