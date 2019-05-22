# Pod概述

### **什么是Pod**

一个[**Pod**](https://www.kubernetes.org.cn/tags/pod)（就像一群鲸鱼，或者一个豌豆夹）相当于一个共享context的配置组，在同一个context下，应用可能还会有独立的cgroup隔离机制，一个Pod是一个容器环境下的“逻辑主机”，它可能包含一个或者多个紧密相连的应用，这些应用可能是在同一个物理主机或虚拟机上。

Pod 的context可以理解成多个linux命名空间的联合

* PID 命名空间（同一个Pod中应用可以看到其它进程）
* 网络 命名空间（同一个Pod的中的应用对相同的IP地址和端口有权限）
* IPC 命名空间（同一个Pod中的应用可以通过VPC或者POSIX进行通信）
* UTS 命名空间（同一个Pod中的应用共享一个主机名称）

同一个Pod中的应用可以共享磁盘，磁盘是Pod级的，应用可以通过文件系统调用，额外的，一个Pod可能会定义顶级的cgroup隔离，这样的话绑定到任何一个应用（好吧，这句是在没怎么看懂，就是说Pod，应用，隔离）

由于docker的架构，一个Pod是由多个相关的并且共享磁盘的容器组成，Pid的命名空间共享还没有应用到Docker中

和相互独立的容器一样，Pod是一种相对短暂的存在，而不是持久存在的，正如我们在Pod的生命周期中提到的，Pod被安排到结点上，并且保持在这个节点上直到被终止（根据重启的设定）或者被删除，当一个节点死掉之后，上面的所有Pod均会被删除。特殊的Pod永远不会被转移到的其他的节点，作为替代，他们必须被replace.

### 理解Pod <a id="&#x7406;&#x89E3;pod"></a>

Pod是kubernetes中你可以创建和部署的最小也是最简的单位。一个Pod代表着集群中运行的一个进程。

Pod中封装着应用的容器（有的情况下是好几个容器），存储、独立的网络IP，管理容器如何运行的策略选项。Pod代表着部署的一个单位：kubernetes中应用的一个实例，可能由一个或者多个容器组合在一起共享资源。

> [Docker](https://www.docker.com/)是kubernetes中最常用的容器运行时，但是Pod也支持其他容器运行时。

在Kubrenetes集群中Pod有如下两种使用方式：

* **一个Pod中运行一个容器**。“每个Pod中一个容器”的模式是最常见的用法；在这种使用方式中，你可以把Pod想象成是单个容器的封装，kuberentes管理的是Pod而不是直接管理容器。
* **在一个Pod中同时运行多个容器**。一个Pod中也可以同时封装几个需要紧密耦合互相协作的容器，它们之间共享资源。这些在同一个Pod中的容器可以互相协作成为一个service单位——一个容器共享文件，另一个“sidecar”容器来更新这些文件。Pod将这些容器的存储资源作为一个实体来管理。

[Kubernetes Blog](http://blog.kubernetes.io/) 有关于Pod用例的详细信息，查看：

* [The Distributed System Toolkit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html)
* [Container Design Patterns](http://blog.kubernetes.io/2016/06/container-design-patterns.html)

每个Pod都是应用的一个实例。如果你想平行扩展应用的话（运行多个实例），你应该运行多个Pod，每个Pod都是一个应用实例。在Kubernetes中，这通常被称为replication



Pod 是一组紧密关联的容器集合，它们共享 IPC、Network 和 UTS namespace，是 Kubernetes 调度的基本单位。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。+

![](../../../.gitbook/assets/image%20%2882%29.png)

Pod 的特征

* 包含多个共享 IPC、Network 和 UTC namespace 的容器，可直接通过 localhost 通信
* 所有 Pod 内容器都可以访问共享的 Volume，可以访问共享数据
* 无容错性：直接创建的 Pod 一旦被调度后就跟 Node 绑定，即使 Node 挂掉也不会被重新调度（而是被自动删除），因此推荐使用 Deployment、Daemonset 等控制器来容错
* 优雅终止：Pod 删除的时候先给其内的进程发送 SIGTERM，等待一段时间（grace period）后才强制停止依然还在运行的进程
* 特权容器（通过 SecurityContext 配置）具有改变系统配置的权限（在网络插件中大量应用）

> Kubernetes v1.8+ 还支持容器间共享 PID namespace，需要 docker &gt;= 1.13.1，并配置 kubelet `--docker-disable-shared-pid=false`。

### API 版本对照表 <a id="api-&#x7248;&#x672C;&#x5BF9;&#x7167;&#x8868;"></a>

| Kubernetes 版本 | Core API 版本 | 默认开启 |
| :--- | :--- | :--- |
| v1.5+ | core/v1 | 是 |

#### Pod中如何管理多个容器 <a id="pod&#x4E2D;&#x5982;&#x4F55;&#x7BA1;&#x7406;&#x591A;&#x4E2A;&#x5BB9;&#x5668;"></a>

Pod中可以同时运行多个进程（作为容器运行）协同工作。同一个Pod中的容器会自动的分配到同一个 node 上。同一个Pod中的容器共享资源、网络环境和依赖，它们总是被同时调度。

注意在一个Pod中同时运行多个容器是一种比较高级的用法。只有当你的容器需要紧密配合协作的时候才考虑用这种模式。例如，你有一个容器作为web服务器运行，需要用到共享的volume，有另一个“sidecar”容器来从远端获取资源更新这些文件，如下图所示：

![pod diagram](https://jimmysong.io/kubernetes-handbook/images/pod-overview.png)

Pod中可以共享两种资源：网络和存储。

**网络**

每个Pod都会被分配一个唯一的IP地址。Pod中的所有容器共享网络空间，包括IP地址和端口。Pod内部的容器可以使用`localhost`互相通信。Pod中的容器与外界通信时，必须分配共享网络资源（例如使用宿主机的端口映射）。

**存储**

可以Pod指定多个共享的Volume。Pod中的所有容器都可以访问共享的volume。Volume也可以用来持久化Pod中的存储资源，以防容器重启后文件丢失。

### Pod 定义 <a id="pod-&#x5B9A;&#x4E49;"></a>

通过 [yaml 或 json 描述 Pod](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#pod-v1-core) 和其内容器的运行环境以及期望状态，比如一个最简单的 nginx pod 可以定义为

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

> 在生产环境中，推荐使用 Deployment、StatefulSet、Job 或者 CronJob 等控制器来创建 Pod，而不推荐直接创建 Pod。

#### Docker 镜像支持 <a id="docker-&#x955C;&#x50CF;&#x652F;&#x6301;"></a>

目前，Kubernetes 仅支持使用 Docker 镜像来创建容器，但并非支持 [Dockerfile](https://docs.docker.com/engine/reference/builder/) 定义的所有行为。如下表所示

| Dockerfile 指令 | 描述 | 支持 | 说明 |
| :--- | :--- | :--- | :--- |
| ENTRYPOINT | 启动命令 | 是 | containerSpec.command |
| CMD | 命令的参数列表 | 是 | containerSpec.args |
| ENV | 环境变量 | 是 | containerSpec.env |
| EXPOSE | 对外开放的端口 | 否 | 使用 containerSpec.ports.containerPort 替代 |
| VOLUME | 数据卷 | 是 | 使用 volumes 和 volumeMounts |
| USER | 进程运行用户以及用户组 | 是 | securityContext.runAsUser/supplementalGroups |
| WORKDIR | 工作目录 | 是 | containerSpec.workingDir |
| STOPSIGNAL | 停止容器时给进程发送的信号 | 是 | SIGKILL |
| HEALTHCHECK | 健康检查 | 否 | 使用 livenessProbe 和 readinessProbe 替代 |
| SHELL | 运行启动命令的 SHELL | 否 | 使用镜像默认 SHELL 启动命令 |

#### Pod和Controller <a id="pod&#x548C;controller"></a>

Controller可以创建和管理多个Pod，提供副本管理、滚动升级和集群级别的自愈能力。例如，如果一个Node故障，Controller就能自动将该节点上的Pod调度到其他健康的Node上。

包含一个或者多个Pod的Controller示例：

* [Deployment](https://jimmysong.io/kubernetes-handbook/concepts/deployment.html)
* [StatefulSet](https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html)
* [DaemonSet](https://jimmysong.io/kubernetes-handbook/concepts/daemonset.html)

通常，Controller会用你提供的Pod Template来创建相应的Pod。

### Pod Templates <a id="pod-templates"></a>

Pod模版是包含了其他object的Pod定义，例如[Replication Controllers](https://jimmysong.io/kubernetes-handbook/concepts/replicaset.html)，[Jobs](https://jimmysong.io/kubernetes-handbook/concepts/job.html)和 [DaemonSets](https://jimmysong.io/kubernetes-handbook/concepts/daemonset.html)。Controller根据Pod模板来创建实际的Pod。





