# Pod解析

Pod是kubernetes中可以创建的最小部署单元。

V1 core版本的Pod的配置模板见[Pod template](https://jimmysong.io/kubernetes-handbook/manifests/template/pod-v1-template.yaml)。

### Pod的动机 <a id="pod&#x7684;&#x52A8;&#x673A;"></a>

#### 管理 <a id="&#x7BA1;&#x7406;"></a>

Pod是一个服务的多个进程的聚合单位，pod提供这种模型能够简化应用部署管理，通过提供一个更高级别的抽象的方式。Pod作为一个独立的部署单位，支持横向扩展和复制。共生（协同调度），命运共同体（例如被终结），协同复制，资源共享，依赖管理，Pod都会自动的为容器处理这些问题。

#### 资源共享和通信 <a id="&#x8D44;&#x6E90;&#x5171;&#x4EAB;&#x548C;&#x901A;&#x4FE1;"></a>

Pod中的应用可以共享网络空间（IP地址和端口），因此可以通过`localhost`互相发现。因此，pod中的应用必须协调端口占用。每个pod都有一个唯一的IP地址，跟物理机和其他pod都处于一个扁平的网络空间中，它们之间可以直接连通。

Pod中应用容器的hostname被设置成Pod的名字。

Pod中的应用容器可以共享volume。Volume能够保证pod重启时使用的数据不丢失。

### **Pod的使用**

Pod可以作为垂直应用整合的载体，但是它的主要特点是支持同地协作，同地管理程序，例如：

* 内容管理系统，文件和数据加载，本地缓存等等
* 日志和检查点备份，压缩，循环，快照等等
* 数据交换监控，日志追踪，日志记录和监控适配器，以及事件发布等等
* 代理，网桥，适配器
* 控制，管理，配置，更新

总体来说，独立的Pod不会去加载多个相同的应用实例

 详细说明请看： [The Distributed System ToolKit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html).

### 考虑过的其他方案

**为什么不直接在一个容器上运行所有的应用？**

1. 透明，[**Pod**](https://www.kubernetes.org.cn/tags/pod)中的容器对基础设施可见使的基础设施可以给容器提供服务，例如线程管理和资源监控，这为用户提供很多便利
2. 解耦软件依赖关系,独立的容器可以独立的进行重建和重新发布，Kubernetes 甚至会在将来支持独立容器的实时更新
3. 易用，用户不需要运行自己的线程管理器，也不需要关心程序的信号以及异常结束码等
4. 高效，因为基础设施承载了更多的责任，所以容器可以更加高效

**为什么不支持容器的协同调度**

容器的协同调度可以提供，但是它不具备Pod的大多数优点，比如资源共享，IPC，选择机制，简单管理等

### **Pod的持久性**

Pod并不是被设计成一个持久化的资源，它不会在调度失败，节点崩溃，或者其他回收中（比如因为资源的缺乏，或者其他的维护中）幸存下来

总体来说，用户因该直接去创建Pod，并且一直使用controller\(replication controller\),即使是一个节点的情况，这是因为controller提供了集群范围内的自我修复，以及复制还有展示管理

集群API的使用是用户的主要使用方式，这是相对普遍的在如下云管理平台中（ Borg, Marathon, Aurora, and Tupperware.）

**Pod的直接暴露是如下操作变得更容器**

1. 调度和管理的易用性
2. 在没有代理的情况下通过API可以对Pod进行操作
3. Pod的生命周期与管理器的生命周期的分离
4. 解偶控制器和服务，后段管理器仅仅监控Pod
5. 划分清楚了Kubelet级别的功能与云平台级别的功能，kubelet 实际上是一个Pod管理器
6. 高可用，当发生一些删除或者维护的过程时，Pod会自动的在他们被终止之前创建新的替代

目前对于宠物的最佳实践是，创建一个副本等于1和有对应service的一个replication控制器。如果你觉得这太麻烦，请在这里留下你的意见。

### 容器的终止

因为pod代表着一个集群中节点上运行的进程，让这些进程不再被需要，优雅的退出是很重要的（与粗暴的用一个KILL信号去结束，让应用没有机会进行清理操作）。用户应该能请求删除，并且在室进程终止的情况下能知道，而且也能保证删除最终完成。当一个用户请求删除pod，系统记录想要的优雅退出时间段，在这之前Pod不允许被强制的杀死，TERM信号会发送给容器主要的进程。一旦优雅退出的期限过了，KILL信号会送到这些进程，pod会从API服务器其中被删除。如果在等待进程结束的时候，Kubelet或者容器管理器重启了，结束的过程会带着完整的优雅退出时间段进行重试。

一个示例流程：

1. 用户发送一个命令来删除Pod，默认的优雅退出时间是30秒

2. API服务器中的Pod更新时间，超过该时间Pod被认为死亡

3. 在客户端命令的的里面，Pod显示为”Terminating（退出中）”的状态

4. （与第3同时）当Kubelet看到Pod标记为退出中的时候，因为第2步中时间已经设置了，它开始pod关闭的流程

i. 如果该Pod定义了一个停止前的钩子，其会在pod内部被调用。如果钩子在优雅退出时间段超时仍然在运行，第二步会意一个很小的优雅时间断被调用

ii. 进程被发送TERM的信号

5. （与第三步同时进行）Pod从service的列表中被删除，不在被认为是运行着的pod的一部分。缓慢关闭的pod可以继续对外服务，当负载均衡器将他们轮流移除。

6. 当优雅退出时间超时了，任何pod中正在运行的进程会被发送SIGKILL信号被杀死。

7. Kubelet会完成pod的删除，将优雅退出的时间设置为0（表示立即删除）。pod从API中删除，不在对客户端可见。

默认情况下，所有的删除操作的优雅退出时间都在30秒以内。kubectl delete命令支持–graceperiod=的选项，以运行用户来修改默认值。0表示删除立即执行，并且立即从API中删除pod这样一个新的pod会在同时被创建。在节点上，被设置了立即结束的的pod，仍然会给一个很短的优雅退出时间段，才会开始被强制杀死。

### Pod中容器的特权模式 <a id="pod&#x4E2D;&#x5BB9;&#x5668;&#x7684;&#x7279;&#x6743;&#x6A21;&#x5F0F;"></a>

从kubernetes1.1版本开始，pod中的容器就可以开启previleged模式，在容器定义文件的 `SecurityContext`下使用 `privileged` flag。 这在使用Linux的网络操作和访问设备的能力时是很有用的。容器内进程可获得近乎等同于容器外进程的权限。在不需要修改和重新编译kubelet的情况下就可以使用pod来开发节点的网络和存储插件。

如果master节点运行的是kuberentes1.1或更高版本，而node节点的版本低于1.1版本，则API server将也可以接受新的特权模式的pod，但是无法启动，pod将处于pending状态。

执行 `kubectl describe pod FooPodName`，可以看到为什么pod处于pending状态。输出的event列表中将显示： `Error validating pod "FooPodName"."FooPodNamespace" from api, ignoring: spec.containers[0].securityContext.privileged: forbidden '<*>(0xc2089d3248)true'`

如果master节点的版本低于1.1，无法创建特权模式的pod。如果你仍然试图去创建的话，你得到如下错误：

`The Pod "FooPodName" is invalid. spec.containers[0].securityContext.privileged: forbidden '<*>(0xc20b222db0)true`

