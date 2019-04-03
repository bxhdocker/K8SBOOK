# Kubernetes简介

Kubernetes是谷歌开源的容器集群管理系统，是Google多年大规模容器管理技术Borg的开源版本，主要功能包括：

* 基于容器的应用部署、维护和滚动升级
* 负载均衡和服务发现
* 跨机器和跨地区的集群调度
* 自动伸缩
* 无状态服务和有状态服务
* 广泛的Volume支持
* 插件机制保证扩展性

Kubernetes发展非常迅速，已经成为容器编排领域的领导者。

#### Kubernetes 特点 <a id="kubernetes-is"></a>

* **可移植**: 支持公有云，私有云，混合云，多重云（multi-cloud）
* **可扩展**: 模块化, 插件化, 可挂载, 可组合
* **自动化**: 自动部署，自动重启，自动复制，自动伸缩/扩展

Kubernetes是Google 2014年创建管理的，是Google 10多年大规模容器管理技术Borg的开源版本。

### Kubernetes是一个平台 <a id="kubernetes&#x662F;&#x4E00;&#x4E2A;&#x5E73;&#x53F0;"></a>

Kubernetes 提供了很多的功能，它可以简化应用程序的工作流，加快开发速度。通常，一个成功的应用编排系统需要有较强的自动化能力，这也是为什么 Kubernetes 被设计作为构建组件和工具的生态系统平台，以便更轻松地部署、扩展和管理应用程序。

用户可以使用Label以自己的方式组织管理资源，还可以使用Annotation来自定义资源的描述信息，比如为管理工具提供状态检查等。

此外，Kubernetes控制器也是构建在跟开发人员和用户使用的相同的API之上。用户还可以编写自己的控制器和调度器，也可以通过各种插件机制扩展系统的功能。

这种设计使得可以方便地在Kubernetes之上构建各种应用系统。

容器优势总结：

* **快速创建/部署应用：**与VM虚拟机相比，容器镜像的创建更加容易。
* **持续开发、集成和部署：**提供可靠且频繁的容器镜像构建/部署，并使用快速和简单的回滚\(由于镜像不可变性\)。
* **开发和运行相分离：**在build或者release阶段创建容器镜像，使得应用和基础设施解耦。
* **开发，测试和生产环境一致性：**在本地或外网（生产环境）运行的一致性。
* **云平台或其他操作系统：**可以在 Ubuntu、RHEL、 CoreOS、on-prem、Google Container Engine或其它任何环境中运行。
* **Loosely coupled，分布式，弹性，微服务化：**应用程序分为更小的、独立的部件，可以动态部署和管理。
* **资源隔离**
* **资源利用：**更高效

### Kubernetes不是什么 <a id="kubernetes&#x4E0D;&#x662F;&#x4EC0;&#x4E48;"></a>

Kubernetes 不是一个传统意义上，包罗万象的 PaaS \(平台即服务\) 系统。它给用户预留了选择的自由。

* 不限制支持的应用程序类型，它不插手应用程序框架, 也不限制支持的语言 \(如Java, Python, Ruby等\)，只要应用符合 [12因素](http://12factor.net/) 即可。Kubernetes 旨在支持极其多样化的工作负载，包括无状态、有状态和数据处理类型的工作负载。只要应用可以在容器中运行，那么它就可以很好的在 Kubernetes 上运行。
* Kubernetes不提供内置的中间件 \(如消息中间件\)、数据处理框架 \(如Spark\)、数据库 \(如 mysql\)或集群存储系统 \(如Ceph\)等。但这些应用都可以运行在Kubernetes上面。
* 不提供点击即部署的服务市场。
* 不直接部署代码，Kubernetes不部署源码不编译应用。也不会构建您的应用程序，持续集成的 \(CI\)工作流方面，不同的用户有不同的需求和偏好的区域，因此，我们提供分层的 CI工作流，但并不定义它应该如何工作。
* 允许用户选择自己的日志、监控和告警系统。
* 不提供或授权一个全面的应用程序配置 语言/系统 \(如 [jsonnet](https://github.com/google/jsonnet)\)。
* 不提供任何机器配置、维护、管理或者自修复系统。

另外，已经有很多 PaaS 系统运行在 Kubernetes 之上，如 [Openshift](https://github.com/openshift/origin), [Deis](http://deis.io/) 和 [Eldarion](http://eldarion.cloud/) 等。 您也可以构建自己的PaaS系统，或者只使用Kubernetes管理您的容器应用。

由于Kubernetes运行在应用级别而不是硬件级，因此提供了普通的Paas平台提供的一些通用功能，比如部署，扩展，负载均衡，日志，监控等。这些默认功能是可选的。

另外，Kuberenets不仅仅是一个“编排系统”，它消除了编排的需要。“编排”的定义是指执行一个预定的工作流：先执行A，之B，然C。相反，Kubernetes由声明式的API和一系列独立、可组合的控制器组成。怎么样从A到C并不重要，达到目的就好。保证应用总是在期望的状态，而用户并不需要关心中间状态是如何转换的。这使得整个系统更容易使用，而且更强大、更可靠、更具弹性和可扩展性

### 使用Kubernetes能做什么？

可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能提供一个以“**容器为中心的基础架构**”，满足在生产环境中运行应用的一些常见需求，如：

* [多个进程（作为容器运行）协同工作。](http://docs.kubernetes.org.cn/312.html)（Pod）
* 存储系统挂载
* Distributing secrets
* 应用健康检测
* [应用实例的复制](http://docs.kubernetes.org.cn/314.html)
* Pod自动伸缩/扩展
* Naming and discovering
* 负载均衡
* 滚动更新
* 资源监控
* 日志访问
* 调试应用程序
* [提供认证和授权](http://docs.kubernetes.org.cn/51.html)

#### Kubernetes没有做什么 <a id="kubernetes&#x6CA1;&#x6709;&#x505A;&#x4EC0;&#x4E48;"></a>

看下这张图中的两个服务，它们使用的是kube-proxy里基于iptables的原生的负载均衡，并且服务间的流量也没有任何控制。

![](https://ws1.sinaimg.cn/large/00704eQkgy1frr5dsurx6j320i140tpf.jpg)

Kubernetes缺少的最重要的一个功能就是微服务的治理，微服务比起单体服务来说使得部署和运维起来更加复杂，对于微服务的可观测性也有更高的要求，同时CI/CD流程Kubernetes本身也没有提供

**Kubernetes**的名字来自希腊语，意思是“_舵手”_ 或 “领航员”。_K8s_是将8个字母“ubernete”替换为“8”的缩写。

