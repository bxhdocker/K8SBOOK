# Istio流量管理实现机制深度解析

### **Pilot高层架构**

### 

Istio控制面中负责流量管理的组件为Pilot，Pilot的高层架构如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/qsueicWSCrPDVZFMvW5l0ibe9fKPckU1K1USP0jfa37ZwiaCOGp173QxDiafuMERo2z4yFcoGrT1TzaBnJtLLicbhAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Pilot Architecture（来自Isio官网文档\[1\]\)

根据上图,Pilot主要实现了下述功能：

#### **统一的服务模型**

Pilot定义了网格中服务的标准模型，这个标准模型独立于各种底层平台。由于有了该标准模型，各个不同的平台可以通过适配器和Pilot对接，将自己特有的服务数据格式转换为标准格式，填充到Pilot的标准模型中。

例如Pilot中的Kubernetes适配器通过Kubernetes API服务器得到kubernetes中service和pod的相关信息，然后翻译为标准模型提供给Pilot使用。通过适配器模式，Pilot还可以从Mesos, Cloud Foundry, Consul等平台中获取服务信息，还可以开发适配器将其他提供服务发现的组件集成到Pilot中。

#### **标准数据面API**

Pilo使用了一套起源于Envoy项目的标准数据面API\[2\]来将服务信息和流量规则下发到数据面的sidecar中。

通过采用该标准API，Istio将控制面和数据面进行了解耦，为多种数据面sidecar实现提供了可能性。事实上基于该标准API已经实现了多种Sidecar代理和Istio的集成，除Istio目前集成的Envoy外，还可以和Linkerd, Nginmesh等第三方通信代理进行集成，也可以基于该API自己编写Sidecar实现。

控制面和数据面解耦是Istio后来居上，风头超过Service mesh鼻祖Linkerd的一招妙棋。Istio站在了控制面的高度上，而Linkerd则成为了可选的一种sidecar实现，可谓降维打击的一个典型成功案例！

数据面标准API也有利于生态圈的建立，开源，商业的各种sidecar以后可能百花齐放，用户也可以根据自己的业务场景选择不同的sidecar和控制面集成，如高吞吐量的，低延迟的，高安全性的等等。有实力的大厂商可以根据该API定制自己的sidecar，例如蚂蚁金服开源的Golang版本的Sidecar MOSN\(Modular Observable Smart Netstub\)（SOFAMesh中Golang版本的Sidecar\)；小厂商则可以考虑采用成熟的开源项目或者提供服务的商业sidecar实现。

备注：Istio和Envoy项目联合制定了Envoy V2 API,并采用该API作为Istio控制面和数据面流量管理的标准接口。

#### **业务DSL语言**

Pilot还定义了一套DSL（Domain Specific Language）语言，DSL语言提供了面向业务的高层抽象，可以被运维人员理解和使用。运维人员使用该DSL定义流量规则并下发到Pilot，这些规则被Pilot翻译成数据面的配置，再通过标准API分发到Envoy实例，可以在运行期对微服务的流量进行控制和调整。

Pilot的规则DSL是采用K8S API Server中的Custom Resource \(CRD\)\[3\]实现的，因此和其他资源类型如Service Pod Deployment的创建和使用方法类似，都可以用Kubectl进行创建。

通过运用不同的流量规则，可以对网格中微服务进行精细化的流量控制，如按版本分流，断路器，故障注入，灰度发布等。

### **Istio流量管理相关组件**

### 

我们可以通过下图了解Istio流量管理涉及到的相关组件。虽然该图来自Istio Github old pilot repo, 但图中描述的组件及流程和目前Pilot的最新代码的架构基本是一致的。

![](https://mmbiz.qpic.cn/mmbiz_png/qsueicWSCrPDVZFMvW5l0ibe9fKPckU1K1p9CzIiaibgObFE6LLyVuJB4UezqnCodgES1SiacveXbulsQIXiaR2JCwtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
Pilot Design Overview \(来自Istio old\_pilot\_repo\[4\]\)

图例说明：图中红色的线表示控制流，黑色的线表示数据流。蓝色部分为和Pilot相关的组件。  


从上图可以看到，Istio中和流量管理相关的有以下组件：

#### **控制面组件**

#### Discovery Services

对应的docker为gcr.io/istio-release/pilot,进程为pilot-discovery，该组件的功能包括：

* 从Service provider（如kubernetes或者consul）中获取服务信息
* 从K8S API Server中获取流量规则\(K8S CRD Resource\)
* 将服务信息和流量规则转化为数据面可以理解的格式，通过标准的数据面API下发到网格中的各个sidecar中。

#### K8S API Server

提供Pilot相关的CRD Resource的增、删、改、查。和Pilot相关的CRD有以下几种:

* Virtualservice：用于定义路由规则，如根据来源或 Header 制定规则，或在不同服务版本之间分拆流量。
* DestinationRule：定义目的服务的配置策略以及可路由子集。策略包括断路器、负载均衡以及 TLS 等。
* ServiceEntry：用 ServiceEntry 可以向Istio中加入附加的服务条目，以使网格内可以向istio 服务网格之外的服务发出请求。
* Gateway：为网格配置网关，以允许一个服务可以被网格外部访问。
* EnvoyFilter：可以为Envoy配置过滤器。由于Envoy已经支持Lua过滤器，因此可以通过EnvoyFilter启用Lua过滤器，动态改变Envoy的过滤链行为。我之前一直在考虑如何才能动态扩展Envoy的能力，EnvoyFilter提供了很灵活的扩展性。

####  ****

#### **数据面组件**

在数据面有两个进程Pilot-agent和envoy，这两个进程被放在一个docker容器gcr.io/istio-release/proxyv2中。

#### Pilot-agent

该进程根据K8S API Server中的配置信息生成Envoy的配置文件，并负责启动Envoy进程。注意Envoy的大部分配置信息都是通过xDS接口从Pilot中动态获取的，因此Agent生成的只是用于初始化Envoy的少量静态配置。在后面的章节中，本文将对Agent生成的Envoy配置文件进行进一步分析。

#### Envoy

Envoy由Pilot-agent进程启动，启动后，Envoy读取Pilot-agent为它生成的配置文件，然后根据该文件的配置获取到Pilot的地址，通过数据面标准API的xDS接口从pilot拉取动态配置信息，包括路由（route），监听器（listener），服务集群（cluster）和服务端点（endpoint）。Envoy初始化完成后，就根据这些配置信息对微服务间的通信进行寻址和路由。

#### **命令行工具**

kubectl和Istioctl，由于Istio的配置是基于K8S的CRD，因此可以直接采用kubectl对这些资源进行操作。Istioctl则针对Istio对CRD的操作进行了一些封装。Istioctl支持的功能参见该表格。

