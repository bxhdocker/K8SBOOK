# 安全工具

**23. Trireme**

Trireme是一项灵活且直接的Kubernetes网络策略实现方案，其适用于任何Kubernetes集群，并允许用户管理不同集群内pod之间的流量。Tririme的主要优势在于其无需任何集中式策略管理，能够轻松实现Kubernetes中所部署的两种资源的彼此交互，并且无需配合任何复杂的SDN、VLAN标签以及子网（Trireme使用常规的L3-网络）。

链接： [https://github.com/aporeto-inc/trireme-kubernetes](https://github.com/aporeto-inc/trireme-kubernetes)

使用成本：免费

**24. Aquasec**

Aquasec能够在Kubernetes整个部署生命周期内提供安全保障。AquaSecurity会在每个容器实例上部署一个专用代理，该代理可充当防火墙并屏蔽容器中所存在的安全漏洞，此外，该代理会与中央Aqua Security控制台——负责执行已定义的安全限制——进行通信。另外，AquaSecurity还可面向云与本地环境提供灵活的安全交付通道。Kube-Bench是一款由AquaSec发布的开源工具，其可根据CISKubernetes Benchmark中所提供的条目清单对Kubernetes环境进行检查。

链接： [https://www.aquasec.com/](https://www.aquasec.com/)

使用成本：每次扫描0.29美元。

**25. Twistlock**

Twistlock是另一种可用于“云原生应用程序防火墙”的工具，且能够分析容器与服务之间的网络流量。Twistlock能够分析标准容器行为并据此生成适当的规则，这样一来，管理者将无需以手动方式生成策略规则。此外，Twistlock还支持Kubernete 2.2版本中的CISBenchmark。

链接： [https://www.twistlock.com/](https://www.twistlock.com/)

使用成本：每份许可证每年1700美元起（试用版免费）。

**26. Sysdig Falco**

SysdigFalco是一款行为活动监视器，旨在检测应用程序中的异常活动。Falco的基础为Sysdig项目——Sysdig是一款开源工具（现已转化为商业服务），可通过追踪内核系统调用来监控容器性能。Falco允许用户通过一套规则以持续监控并检测容器、应用程序、主机以及网络活动。

链接： [https://sysdig.com/opensource/falco/](https://sysdig.com/opensource/falco/)

使用成本：

* 独立工具——免费
* 基础云——每月20美元（可免费试用）
* 专业云——每月30美元
* 专业版软件——自订价格

**27. Sysdig Secure**

SysdigSecure作为Sysdig容器智能平台的一部分，除了具有无与伦比的容器可见性之外，还可与容器编排工具实现深度集成。其中集成的编排工具具体包括：Kubernetes、Docker、AWS ECS以及Apache Mesos。借助SysdigSecure，用户可以实现服务感知策略、阻止攻击、分析历史记录并监控集群性能。最后，SysdigSecure的定位为云与内部部署软件产品。

链接： [https://sysdig.com/product/secure/](https://sysdig.com/product/secure/)

使用成本：

* 独立工具——免费
* 专业云——自订价格
* 专业版软件——自订价格

**28. Kubesec.io**

Kubesec.io是一项能够针对安全功能使用情况对Kubernetes资源进行评分的服务。Kubesec.io可根据Kubernetes安全最佳实践验证资源配置。因此，对于如何改进系统整体安全性，用户将拥有完全的控制权，并能够据此提供额外的建议。另外，该网站还包括大量与容器与Kubernetes安全相关的外部链接。

链接： [https://kubesec.io](https://kubesec.io/)

使用成本：免费

