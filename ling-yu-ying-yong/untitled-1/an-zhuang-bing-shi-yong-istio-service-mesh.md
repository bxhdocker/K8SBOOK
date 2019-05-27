# 安装并试用Istio service mesh

官方文档地址 [快速开始](https://istio.io/docs/setup/kubernetes/)

本文根据官网的文档整理而成，步骤包括安装**istio 1.0**并创建一个bookinfo的微服务来测试istio的功能。

### 部署结构 <a id="&#x90E8;&#x7F72;&#x7ED3;&#x6784;"></a>

Istio 的控制平面部署在 Kubernetes 中的部署架构如下图所示。

![Istio &#x5728; Kubernetes &#x4E2D;&#x7684;&#x90E8;&#x7F72;&#x67B6;&#x6784;&#x56FE;](https://jimmysong.io/kubernetes-handbook/images/istio-deployment-architecture-diagram.png)

我们可以清楚的看到 Istio 控制平面的几个组件的部署运行的命令与开发的端口，以及端口与服务之间的映射的关系。

### 安装环境 <a id="&#x5B89;&#x88C5;&#x73AF;&#x5883;"></a>

* Docker 17.12.0-ce
* Kubernetes 1.10.5
* istio 1.0

gcr.io和quay.io相关的镜像下载不了的话可以替换为我做好的墙内的镜像：

* daocloud.io/liukuan73/proxy\_init:1.0.0
* daocloud.io/liukuan73/galley:1.0.0
* daocloud.io/liukuan73/mixer:1.0.0
* daocloud.io/liukuan73/proxyv2:1.0.0
* daocloud.io/liukuan73/pilot:1.0.0
* daocloud.io/liukuan73/citadel:1.0.0
* daocloud.io/liukuan73/servicegraph:1.0.0
* daocloud.io/liukuan73/sidecar\_injector:1.0.0
* daocloud.io/liukuan73/istio-grafana:1.0.0

### 安装 <a id="&#x5B89;&#x88C5;"></a>

安装前说明：从0.2版本开始，Istio安装在它自己的istio-system命名空间中，并且可以管理来自所有其他命名空间的服务。

**1.下载安装包**

下载地址：[https://github.com/istio/istio/releases](https://github.com/istio/istio/releases)

到istio的[release地址](https://github.com/istio/istio/releases)下载安装包,我目前使用1.0.0版：

```text
wget https://github.com/istio/istio/releases/download/1.0.0/istio-1.0.0-linux.tar.gz    
```

或者用命令进行下载和自动解压缩：

```text
curl -L https://git.io/getLatestIstio | sh -
```

**2.解压**

解压后，得到的目录结构如下：

```text
├── bin
│   └── istioctl
├── install
│   ├── ansible
│   ├── consul
│   ├── eureka
│   ├── gcp
│   ├── kubernetes
│   ├── README.md
│   └── tools
├── istio.VERSION
├── LICENSE
├── README.md
├── samples
│   ├── bookinfo
│   ├── CONFIG-MIGRATION.md
│   ├── helloworld
│   ├── httpbin
│   ├── kubernetes-blog
│   ├── rawvm
│   ├── README.md
│   └── sleep
└── tools
    ├── cache_buster.yaml
    ├── deb
    ├── githubContrib
    ├── minikube.md
    ├── perf_istio_rules.yaml
    ├── perf_k8svcs.yaml
    ├── README.md
    ├── rules.yml
    ├── setup_perf_cluster.sh
    ├── setup_run
    ├── update_all
    └── vagrant
```

从文件里表中可以看到，安装包中包括了kubernetes的yaml文件，示例应用和安装模板

安装目录中包含：

* 在 `install/` 目录中包含了 Kubernetes 安装所需的 `.yaml` 文件
* `samples/` 目录中是示例应用
* `istioctl` 客户端文件保存在 `bin/` 目录之中。`istioctl` 的功能是手工进行 Envoy Sidecar 的注入，以及对路由规则、策略的管理
* `istio.VERSION` 配置文件

3.把 `istioctl` 客户端加入 PATH 环境变量，如果是 macOS 或者 Linux，可以这样实现：

```text
$ export PATH=$PWD/bin:$PATH
```

 或者也可以执行 cp bin/istioctl /usr/local/bin/，但是coreos系统是/usr是只读的，所以我选择上面做环境变量

#### 3.部署Istio核心服务

两种方式（选择其一执行）

* 禁止Auth：kubectl apply -f install/kubernetes/istio-demo.yaml
* 启用Auth：kubectl apply -f install/kubernetes/istio--demo-auth.yaml

注：**istio.yaml 中的 Ingress 服务是 LoadBalancer 类型的，如果测试集群不具备这样的条件，还请自行修改成其他合适内容。例如改为NodePort**

**文件内容在2218行   命令：vi进去 执行 2218+shift+g 可快速定位到此行进行更改（不然状态会一直处于pending状态）**

* istio-demo.yaml - 使用此生成的文件进行安装而不启用身份验证
* istio-demo-auth.yaml - 使用此生成的文件进行安装并启用身份验证
* templates - 目录包含用于生成istio.yaml和istio-auth.yaml的模板
* addons - 目录包含可选组件（Prometheus，Grafana，Service Graph，Zipkin，Zipkin到Stackdriver）
* helm - 目录包含Istio头盔发布配置文件。该目录还需要运行`updateVersion.sh`以生成一些配置文件

安装 Istio 的核心部分。从以下**四种**\_**非手动**\_部署方式中选择一种方式安装。然而，我们推荐您在生产环境时使用 [Helm Chart](https://istio.io/zh/docs/setup/kubernetes/helm-install/) 来安装 Istio，这样可以按需定制配置选项。

* 安装 Istio 而不启用 sidecar 之间的[双向 TLS 验证](https://istio.io/zh/docs/concepts/security/#双向-tls-认证)。对于现有应用程序的集群，使用 Istio sidecar 的服务需要能够与其他非 Istio Kubernetes 服务以及使用[存活和就绪探针](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)、headless 服务或 `StatefulSets` 的应用程序通信的应用程序选择此选项。

```text
$ kubectl apply -f install/kubernetes/istio-demo.yaml
```

或者

* 默认情况下安装 Istio，并强制在 sidecar 之间进行双向 TLS 身份验证。仅在保证新部署的工作负载安装了 Istio sidecar 的新建的 Kubernetes 集群上使用此选项。

```text
$ kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```

或者

* [使用 Helm 渲染出 Kubernetes 配置清单然后使用 `kubectl` 部署](https://istio.io/zh/docs/setup/kubernetes/helm-install/#选项1-通过-helm-的-helm-template-安装-istio)

或者

* [使用 Helm 和 Tiller 管理 Istio 部署](https://istio.io/zh/docs/setup/kubernetes/helm-install/#选项2-通过-helm-和-tiller-的-helm-install-安装-istio)

## 确认安装

 1.确认下列 Kubernetes 服务已经部署：`istio-pilot`、 `istio-ingressgateway`、`istio-policy`、`istio-telemetry`、`prometheus` 、`istio-sidecar-injector`（可选）。

```text
$ kubectl get svc -n istio-system
NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                                                               AGE
istio-citadel              ClusterIP      30.0.0.119   <none>          8060/TCP,9093/TCP                                                     7h
istio-egressgateway        ClusterIP      30.0.0.11    <none>          80/TCP,443/TCP                                                        7h
istio-ingressgateway       LoadBalancer   30.0.0.39    9.111.255.245   80:31380/TCP,443:31390/TCP,31400:31400/TCP                            7h
istio-pilot                ClusterIP      30.0.0.136   <none>          15003/TCP,15005/TCP,15007/TCP,15010/TCP,15011/TCP,8080/TCP,9093/TCP   7h
istio-policy               ClusterIP      30.0.0.242   <none>          9091/TCP,15004/TCP,9093/TCP                                           7h
istio-statsd-prom-bridge   ClusterIP      30.0.0.111   <none>          9102/TCP,9125/UDP                                                     7h
istio-telemetry            ClusterIP      30.0.0.246   <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                 7h
prometheus                 ClusterIP      30.0.0.253   <none>          9090/TC
```

如果您的集群在不支持外部负载均衡器的环境中运行（例如 minikube），`istio-ingressgateway`的 `EXTERNAL-IP` 将会显示为 `<pending>` 状态。您将需要使用服务的 NodePort 来访问，或者使用 port-forwarding。

2.确保所有相应的Kubernetes pod都已被部署且所有的容器都已启动并正在运行：`istio-pilot-*`、`istio-ingressgateway-*`、`istio-egressgateway-*`、`istio-policy-*`、`istio-telemetry-*`、`istio-citadel-*`、`prometheus-*`、`istio-sidecar-injector-*`（可选）。

```text
$ kubectl get pods -n istio-system
NAME                                       READY     STATUS      RESTARTS   AGE
istio-citadel-dcb7955f6-vdcjk              1/1       Running     0          11h
istio-egressgateway-56b7758b44-l5fm5       1/1       Running     0          11h
istio-ingressgateway-56cfddbd5b-xbdcx      1/1       Running     0          11h
istio-pilot-cbd6bfd97-wgw9b                2/2       Running     0          11h
istio-policy-699fbb45cf-bc44r              2/2       Running     0          11h
istio-statsd-prom-bridge-949999c4c-nws5j   1/1       Running     0          11h
istio-telemetry-55b675d8c-kfvvj            2/2       Running     0          11h
prometheus-86cb6dd77c-5j48h                1/1       Running     0          11h
```

### 注意： 缺省情况下，Istio 服务网格内的 Pod，由于其 iptables 将所有外发流量都透明的转发给了 Sidecar，所以这些集群内的服务无法访问集群之外的 URL，而只能处理集群内部的目标（可[参考egress](https://istio.io/zh/docs/tasks/traffic-management/egress/)） <a id="&#x5378;&#x8F7D;"></a>

#### 或者把ser ip加入到这里 

![](../../.gitbook/assets/image%20%2823%29.png)

### 卸载 <a id="&#x5378;&#x8F7D;"></a>

* 卸载 Istio 核心组件。对于该版本，卸载时将删除 RBAC 权限、`istio-system` 命名空间和该命名空间的下的各层级资源。

不必理会在层级删除过程中的各种报错，因为这些资源可能已经被删除的。

如果您使用 `istio.yaml` 安装的 Istio：

```text
$ kubectl delete -f install/kubernetes/istio.yaml
```

否则使用 [Helm 卸载 Istio](https://istio.io/zh/docs/setup/kubernetes/helm-install/#卸载)

说明：

 每个pod内都会有一个Envoy容器，其具备对流入和流出pod的流量进行管理，认证，控制的能力。Mixer则主要负责访问控制和遥测信息收集

 对控制面各组件的作用作用及故障影响进行了汇总，结果如下：[链接](http://dockone.io/article/8506)

![](../../.gitbook/assets/image%20%28156%29.png)



