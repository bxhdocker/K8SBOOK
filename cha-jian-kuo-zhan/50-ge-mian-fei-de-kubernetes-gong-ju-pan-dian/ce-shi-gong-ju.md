# 测试工具

**18. Kube-monkey**

Kube-monkey是Netflix公司旗下ChaosMonkey项目的Kubernetes版本。Kube-monkey是一款遵循混沌工程原理的工具，其可以随机删除Kubernetes pod，检查服务是否具备抗失效能力并帮助维持相关系统的健康运转。Kube-monkey也可经由TOML文件完成配置，而TOML文件不仅能够终止指定的应用程序，还可以决定恢复策略的执行时间。

链接： [https://github.com/asobti/kube-monkey](https://github.com/asobti/kube-monkey)

使用成本：免费

**19. K8s-testsuite**

K8s-testsuite由两个Helm图表组合而成，适用于网络带宽测试与单个Kubernetes集群的负载测试。负载测试模拟了带有loadbots的简单网页服务器，这些服务器可在Vegeta基础上以Kubernetes微服务的形式运行。网络测试则是在内部连续对iperf3与netperf-2.7.0运行三次。这两项测试都会生成涵盖全部结果与指标的综合日志信息。

链接： [https://github.com/mrahbar/k8s-testsuite](https://github.com/mrahbar/k8s-testsuite)

使用成本：免费

**20. Test-infra**

Test-infra是一套用于Kubernetes测试与结果验证的工具集合。Test-infra包括多种仪表板，分别用于显示历史记录、汇总故障以及当前正在测试的内容。用户可通过创建自定义测试作业以增强Test-infra套件。此外，Test-infra可在使用Kubetest的不同供应商平台上，通过模拟完整的Kubernetes生命周期实现端到端Kubernetes测试。

链接： [https://github.com/kubernetes/test-infra](https://github.com/kubernetes/test-infra)

使用成本：免费

**21. Sonobuoy**

Sonobuoy允许用户以易于访问与非破坏性的方式运行一组测试，从而对当前Kubernetes集群状态进行评估。Sonobuoy可生成有关集群性能详细信息的信息性报告，并能够支持Kubernetes1.8及更高版本。SonobuoyScanner是一款基于浏览器的工具。在该工具的帮助下，用户只需点击数下即可完成对Kubernetes集群的测试。当然，其CLI版本能够应对规模更大的测试集群。

链接： [https://github.com/heptio/sonobuoy](https://github.com/heptio/sonobuoy)

使用成本：免费

**22. PowerfulSeal**

PowerfulSeal类似于Kube-monkey，同样遵循混沌工程原理。因此，PowerfulSeal不仅可终止pod，还能够在集群中添加或删除虚拟机。不同于Kube-monkey，PowerfulSeal具有交互模式，从而允许用户以手动方式中断特定的集群组件。另外，除了SSH以外，PowerfulSeal无需其它外部依赖。

链接： [https://github.com/bloomberg/powerfulseal](https://github.com/bloomberg/powerfulseal)

使用成本：免费

