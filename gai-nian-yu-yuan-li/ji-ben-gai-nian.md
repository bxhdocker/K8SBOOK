# 基本概念总结

Kubernetes中概念的简要概述

* Cluster : 集群是指由Kubernetes使用一系列的物理机、虚拟机和其他基础资源来运行你的应用程序。
* Node : 一个node就是一个运行着Kubernetes的物理机或虚拟机，并且pod可以在其上面被调度。.
* [**Pod** :](https://www.kubernetes.org.cn/kubernetes-pod) 一个pod对应一个由相关容器和卷组成的容器组 （[**了解Pod详情**](https://www.kubernetes.org.cn/kubernetes-pod)）
* Label : 一个label是一个被附加到资源上的键/值对，譬如附加到一个Pod上，为它传递一个用户自定的并且可识别的属性.Label还可以被应用来组织和选择子网中的资源（了解Label详情）
* selector是一个通过匹配labels来定义资源之间关系得表达式，例如为一个负载均衡的service指定所目标Pod.（了解selector详情）
* [**Replication Controller**](https://www.kubernetes.org.cn/replication-controller-kubernetes) : replication controller 是为了保证一定数量被指定的Pod的复制品在任何时间都能正常工作.它不仅允许复制的系统易于扩展，还会处理当pod在机器在重启或发生故障的时候再次创建一个（[**了解Replication Controller详情**](https://www.kubernetes.org.cn/replication-controller-kubernetes)）
* [**Service** ](https://www.kubernetes.org.cn/kubernetes-services): 一个service定义了访问pod的方式，就像单个固定的IP地址和与其相对应的DNS名之间的关系。（[**了解Service详情**](https://www.kubernetes.org.cn/kubernetes-services)）
* [**Volume**](https://www.kubernetes.org.cn/kubernetes-volumes): 一个volume是一个目录，可能会被容器作为未见系统的一部分来访问。（[**了解Volume详情**](https://www.kubernetes.org.cn/kubernetes-volumes)）
* Kubernetes volume 构建在Docker Volumes之上,并且支持添加和配置volume目录或者其他存储设备。
* Secret : Secret 存储了敏感数据，例如能允许容器接收请求的权限令牌。
* Name : 用户为Kubernetes中资源定义的名字
* Namespace : Namespace 好比一个资源名字的前缀。它帮助不同的项目、团队或是客户可以共享cluster,例如防止相互独立的团队间出现命名冲突
* Annotation : 相对于label来说可以容纳更大的键值对，它对我们来说可能是不可读的数据，只是为了存储不可识别的辅助数据，尤其是一些被工具或系统扩展用来操作的数据

![](../.gitbook/assets/image%20%288%29.png)

### Container <a id="container"></a>

Container（容器）是一种便携式、轻量级的操作系统级虚拟化技术。它使用namespace隔离不同的软件运行环境，并通过镜像自包含软件的运行环境，从而使得容器可以很方便的在任何地方运行。

由于容器体积小且启动快，因此可以在每个容器镜像中打包一个应用程序。这种一对一的应用镜像关系拥有很多好处。使用容器，不需要与外部的基础架构环境绑定, 因为每一个应用程序都不需要外部依赖，更不需要与外部的基础架构环境依赖。完美解决了从开发到生产环境的一致性问题。

容器同样比虚拟机更加透明，这有助于监测和管理。尤其是容器进程的生命周期由基础设施管理，而不是由容器内的进程对外隐藏时更是如此。最后，每个应用程序用容器封装，管理容器部署就等同于管理应用程序部署。

在Kubernetes必须要使用Pod来管理容器，每个Pod可以包含一个或多个容器。

在kubernetes中，镜像的下载策略为：

　　　　**Always**：每次都下载最新的镜像

　　　　**Never**：只使用本地镜像，从不下载

　　　　**IfNotPresent**：只有当本地没有的时候才下载镜像

　　Pod被分配到Node之后会根据镜像下载策略进行镜像下载，可以根据自身集群的特点来决定采用何种下载策略。无论何种策略，都要确保Node上有正确的镜像可用

### Pod <a id="pod"></a>

Pod是一组紧密关联的容器集合，它们共享PID、IPC、Network和UTS namespace，是Kubernetes调度的基本单位。Pod的设计理念是支持多个容器在一个Pod中共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

![](../.gitbook/assets/image%20%28121%29.png)

在Kubernetes中，所有对象都使用manifest（yaml或json）来定义，比如一个简单的nginx服务可以定义为nginx.yaml，它包含一个镜像为nginx的容器：

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

### Node <a id="node"></a>

Node是Pod真正运行的主机，可以物理机，也可以是虚拟机。为了管理Pod，每个Node节点上至少要运行container runtime（比如docker或者rkt）、`kubelet`和`kube-proxy`服务。

![](../.gitbook/assets/image%20%2813%29.png)

### Namespace <a id="namespace"></a>

Namespace是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的pods, services, replication controllers和deployments等都是属于某一个namespace的（默认是default），而node, persistentVolumes等则不属于任何namespace。

### Service <a id="service"></a>

Service是应用服务的抽象，通过labels为应用提供负载均衡和服务发现。匹配labels的Pod IP和端口列表组成endpoints，由kube-proxy负责将服务IP负载均衡到这些endpoints上。

每个Service都会自动分配一个cluster IP（仅在集群内部可访问的虚拟地址）和DNS名，其他容器可以通过该地址或DNS来访问服务，而不需要了解后端容器的运行。

![](https://kubernetes.feisky.xyz/zh/introduction/media/14731220608865.png)

```text
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 8078 # the port that this service should serve on
    name: http
    # the container on each pod to connect to, can be a name
    # (e.g. 'www') or a number (e.g. 80)
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

Service的虚拟IP是由Kubernetes虚拟出来的内部网络，外部是无法寻址到的。但是有些服务又需要被外部访问到，例如web前段。这时候就需要加一层网络转发，即外网到内网的转发。Kubernetes提供了NodePort、LoadBalancer、Ingress三种方式。

　　　　**NodePort**，在之前的Guestbook示例中，已经延时了NodePort的用法。NodePort的原理是，Kubernetes会在每一个Node上暴露出一个端口：nodePort，外部网络可以通过（任一Node）\[NodeIP\]:\[NodePort\]访问到后端的Service。

　　　　**LoadBalancer**，在NodePort基础上，Kubernetes可以请求底层云平台创建一个负载均衡器，将每个Node作为后端，进行服务分发。该模式需要底层云平台（例如GCE）支持。

　　　　**Ingress**，是一种HTTP方式的路由转发机制，由Ingress Controller和HTTP代理服务器组合而成。Ingress Controller实时监控Kubernetes API，实时更新HTTP代理服务器的转发规则。HTTP代理服务器有GCE Load-Balancer、HaProxy、Nginx等开源方案。

### Label <a id="label"></a>

Label是识别Kubernetes对象的标签，以key/value的方式附加到对象上（key最长不能超过63字节，value可以为空，也可以是不超过253字节的字符串）。

Label不提供唯一性，并且实际上经常是很多对象（如Pods）都使用相同的label来标志具体的应用。

Label定义好后其他对象可以使用Label Selector来选择一组相同label的对象（比如ReplicaSet和Service用label来选择一组Pod）。Label Selector支持以下几种方式：

* 等式，如`app=nginx`和`env!=production`
* 集合，如`env in (production, qa)`
* 多个label（它们之间是AND关系），如`app=nginx,env=test`

### Annotations <a id="annotations"></a>

Annotations是key/value形式附加于对象的注解。不同于Labels用于标志和选择对象，Annotations则是用来记录一些附加信息，用来辅助应用部署、安全策略以及调度策略等。比如deployment使用annotations来记录rolling update的状态。

