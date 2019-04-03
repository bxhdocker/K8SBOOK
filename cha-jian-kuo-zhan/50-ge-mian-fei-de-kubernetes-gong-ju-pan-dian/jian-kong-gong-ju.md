# 监控工具

**10. Kubebox**

Kubebox是一套用于Kubernetes集群的终端控制台，其能够让用户通过美观且经典的界面对集群实时状态进行管理与监控。Kubebox能够显示容器资源的使用情况、集群监控以及容器日志等。除此之外，用户还可借助Kubebox轻松导航到目标名称空间，并在目标容器中执行相关操作，借此以快速排除故障/恢复。

链接： [https://github.com/astefanutti/kubebox](https://github.com/astefanutti/kubebox)

使用成本：免费

**11. Kubedash**

Kubedash针对Kubernetes提供了一套性能分析UI。Kubedash汇集并总结不同来源的指标，并为管理员提供高级分析数据。Kubedash使用Heapster作为数据源，在默认情况下，该数据源会在所有Kubernetes集群中以服务形式运行，从而收集各个容器的量化指标。

链接： [https://github.com/kubernetes-retired/kubedash](https://github.com/kubernetes-retired/kubedash)

使用成本：免费

**12. Kubernetes Operational View \(Kube-ops-view\)**

Kube-ops-view是一款面向多个Kubernetes集群的只读系统仪表板。用户可以通过Kube-ops-view在集群、监控节点以及pod健康状况之间轻松导航，且其还能够为部分进程提供动画效果——例如pod的创建与终止。此外，类似于Kubedash，Kube-ops-view也将Heapster作为其数据源。

链接： [https://github.com/hjacobs/kube-ops-view](https://github.com/hjacobs/kube-ops-view)

使用成本：免费

**13. Kubetail**

Kubetail是一个小型bash脚本，其能够将来自于多个pod的日志聚合到同一数据流中。Kubetail的初始版本不提供过滤或高亮功能，但其目前已经在GitHub上添加了一款Kubetail分叉，该产品可使用multitail工具以构建日志并进行色彩填充。

链接：

* [https://github.com/johanhaleby/kubetail](https://github.com/johanhaleby/kubetail)
* [https://github.com/aks/kubetail](https://github.com/aks/kubetail)

使用成本：免费

**14. Kubewatch**

Kubewatch是一款Kubernetes监控工具，该产品可将Kubernetes事件发布到团队通信应用程序，即Slack。Kubewatch以Kubernetes集群内部pod的形式运行，借此监视相关系统中所发生的各种变化。另外，您可以通过编辑配置文件来指定需要接收的通知。

链接： [https://github.com/bitnami-labs/kubewatch](https://github.com/bitnami-labs/kubewatch)

使用成本：免费

**15. Weave Scope**

WeaveScope是一款面向Docker与Kubernetes集群的故障排除与监控工具，该工具可自动生成应用程序与基础架构拓扑，借此帮助用户轻松识别应用程序的性能瓶颈。用户可在本地服务器/笔记本电脑上将Weave Scope部署为独立应用程序，或者选用WeaveCloud上的Weave Scope软件即服务（SaaS）解决方案。在WeaveScope的帮助下，用户可通过名称、标签与/或资源消耗量对容器执行分组、筛选或搜索。

链接： [https://www.weave.works/oss/scope/](https://www.weave.works/oss/scope/)

使用成本：

* 独立模式——免费
* 标准模式——每月30美元（免费试用期为30天）
* 企业模式——每节点/每月150美元

**16. Searchlight**

AppsCode推出的Searchlight是一款面向lcinga的Kubernetes运营工具。Searchlight会定期对Kubernetes集群执行各种检查，并会在发现问题后，通过电子邮件、短信或对话框发送警告信息。Searchlight包含专为Kubernetes编写的默认检查套件。此外，其还能够通过联合外部黑盒子监控功能来增强Prometheus的监测性能，并在内部系统完全失效的情况下充当后备选项。

链接： [https://github.com/appscode/searchlight](https://github.com/appscode/searchlight)

使用成本：免费

**17. Heapster**

Heapster能够为Kubernetes提供容器集群监控与性能分析功能。Heapster设计之初即支持Kubernetes并能够作为pod运行于所有Kubernetes配置之上。此外，Heapster的数据还可被推送到配置后端以实现存储与可视化。

链接： [https://github.com/kubernetes/heapster](https://github.com/kubernetes/heapster)

使用成本：免费

