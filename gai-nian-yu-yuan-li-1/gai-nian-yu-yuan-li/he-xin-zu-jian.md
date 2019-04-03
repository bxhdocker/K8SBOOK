# 核心组件

![](../../.gitbook/assets/image%20%2871%29.png)

![&#x6838;&#x5FC3;&#x7EC4;&#x4EF6;&#x539F;&#x7406;](../../.gitbook/assets/image%20%28142%29.png)

Kubernetes主要由以下几个核心组件组成：

* etcd保存了整个集群的状态；
* apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
* controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
* scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
* kubelet负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；_（ 负责管理_[_pods_](https://www.kubernetes.org.cn/kubernetes-pod)_和它们上面的容器，images镜像、volumes、etc）_
* Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
* kube-proxy负责为Service提供cluster内部的服务发现和负载均衡

### 组件通信 <a id="&#x7EC4;&#x4EF6;&#x901A;&#x4FE1;"></a>

Kubernetes 多组件之间的通信原理为

* apiserver 负责 etcd 存储的所有操作，且只有 apiserver 才直接操作 etcd 集群
* apiserver 对内（集群中的其他组件）和对外（用户）提供统一的 REST API，其他组件均通过 apiserver 进行通信
  * controller manager、scheduler、kube-proxy 和 kubelet 等均通过 apiserver watch API 监测资源变化情况，并对资源作相应的操作
  * 所有需要更新资源状态的操作均通过 apiserver 的 REST API 进行
* apiserver 也会直接调用 kubelet API（如 logs, exec, attach 等），默认不校验 kubelet 证书，但可以通过 `--kubelet-certificate-authority` 开启（而 GKE 通过 SSH 隧道保护它们之间的通信）

比如典型的创建 Pod 的流程为

![](https://kubernetes.feisky.xyz/zh/components/images/workflow.png)

 **简单说明下Create Pod的过程:**   
首先用户通过命令行工具向apiserver提交创建Pod的请求，apiserver将请求的Pod信息对象存储到etcd中；然后scheduler周期性的通过apiserver访问etcd中需要创建的Pod的信息，并且通过调度算法和策略给需要创建的Pod分配Node；最后kubelet周期性的访问需要创建的Pod的信息，通过docker创建并且启动一个容器。

1. 用户通过 REST API 创建一个 Pod
2. apiserver 将其写入 etcd
3. scheduluer 检测到未绑定 Node 的 Pod，开始调度并更新 Pod 的 Node 绑定
4. kubelet 检测到有新的 Pod 调度过来，通过 container runtime 运行该 Pod
5. kubelet 通过 container runtime 取到 Pod 状态，并更新到 apiserver 中

### 端口号 <a id="&#x7AEF;&#x53E3;&#x53F7;"></a>

![ports](https://kubernetes.feisky.xyz/zh/components/images/ports.png)

#### Master node\(s\) <a id="master-nodes"></a>

| Protocol | Direction | Port Range | Purpose |
| :--- | :--- | :--- | :--- |
| TCP | Inbound | 6443\* | Kubernetes API server |
| TCP | Inbound | 8080 | Kubernetes API insecure server |
| TCP | Inbound | 2379-2380 | etcd server client API |
| TCP | Inbound | 10250 | Kubelet API |
| TCP | Inbound | 10251 | kube-scheduler healthz |
| TCP | Inbound | 10252 | kube-controller-manager healthz |
| TCP | Inbound | 10253 | cloud-controller-manager healthz |
| TCP | Inbound | 10255 | Read-only Kubelet API |
| TCP | Inbound | 10256 | kube-proxy healthz |

#### Worker node\(s\) <a id="worker-nodes"></a>

| Protocol | Direction | Port Range | Purpose |
| :--- | :--- | :--- | :--- |
| TCP | Inbound | 4194 | Kubelet cAdvisor |
| TCP | Inbound | 10248 | Kubelet healthz |
| TCP | Inbound | 10249 | kube-proxy metrics |
| TCP | Inbound | 10250 | Kubelet API |
| TCP | Inbound | 10255 | Read-only Kubelet API |
| TCP | Inbound | 10256 | kube-proxy healthz |
| TCP | Inbound | 30000-32767 | NodePort Services\*\* |

#### 核心组件 <a id="&#x6838;&#x5FC3;&#x7EC4;&#x4EF6;"></a>

![](https://kubernetes.feisky.xyz/zh/architecture/images/core-packages.png)

### Kubernetes 版本 <a id="kubernetes-&#x7248;&#x672C;"></a>

Kubernetes 的稳定版本在发布后会继续支持 9 个月。每个版本的支持周期为：

| Kubernetes version | Release month（ 发布月份） | End-of-life-month |
| :--- | :--- | :--- |
| v1.6.x | March 2017 | December 2017 |
| v1.7.x | June 2017 | March 2018 |
| v1.8.x | September 2017 | June 2018 |
| v1.9.x | December 2017 | September 2018 |
| v1.10.x | March 2018 | December 2018 |

### 参考文档 <a id="&#x53C2;&#x8003;&#x6587;&#x6863;"></a>

* [Master-Node communication](https://kubernetes.io/docs/concepts/architecture/master-node-communication/)
* [Core Kubernetes: Jazz Improv over Orchestration](https://blog.heptio.com/core-kubernetes-jazz-improv-over-orchestration-a7903ea92ca)
* [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports)

