# 开发工具

**33. Telepresence**

Telepresence可将来自Kubernetes环境的数据转发至本地进程，进而在本地对Kubernetes集群进行调试。在本地代码被部署至目标集群之后，Telepresence可帮助其实现对Kubernetes服务与AWS/GCP资源的访问。在Telepresence的帮助下，Kubernetes能够将本地代码算作为集群中的常规pod进行计数。

链接： [https://www.telepresence.io/](https://www.telepresence.io/)

使用成本：免费

**34. Helm**

Helm是一款适用于Kubernetes的软件包管理器。其与APT/Yum/Homebrew类似，但作用对象为Kubernetes。Helm使用Char实现运行，而Char是一套用于为分布式应用程序构建Kubernetes资源清单的归档集。用户可通过创建Helm图表来实现应用程序共享。此外，Helm允许用户创建可重复的构建模式，并通过简单方式管理Kubernetes清单。

链接： [https://github.com/kubernetes/helm](https://github.com/kubernetes/helm)

使用成本：免费

**35. Keel**

Keel允许用户自动执行Kubernetes部署更新，并能够在专用命名空间内以Kubernetes服务的形式进行启动。通过这样的组织方式，Keel可尽可能降低环境中的额外负载水平，并显著提升鲁棒性。此外，Keel可通过标签、注释以及图表强化Kubernetes服务。因此，用户只需为每个部署或Helm版本指定更新策略，即可在存储库中出现新的应用程序版本时，由Keel自动为其更新相关环境。

链接： [https://keel.sh/](https://keel.sh/)

使用成本：免费

**36. Apollo**

Apollo是一款开源应用程序，旨在为团队提供可用于创建并将相关服务部署到Kubernetes的自助式UI。只需一次点击，操作人员即可通过Apollo查看日志并将部署进程恢复到任意时间点。Apollo具有灵活的部署许可模式，保证每个用户仅可部署其需要的内容。

链接： [https://github.com/logzio/apollo](https://github.com/logzio/apollo)

使用成本：免费

**37. Draft**

Draft是Azure团队推出的一款工具，可简化Kubernetes集群中的应用程序开发与部署过程。Draft可在代码部署与代码提交之间创建“内部循环”，从而极大地缩短变更验证过程。利用Draft，开发人员仅使用两行命令即可完成应用程序Dockerfiles与Helm图表的准备工作，同时将应用程序部署至远程或本地Kubernetes集群。

链接： [https://github.com/azure/draft](https://github.com/azure/draft)

使用成本：免费

**38. Deis Workflow**

DeisWorkflow是一款开源工具，这一平台即服务（PaaS）方案在Kubernetes集群上创建额外的抽象层。这些抽象层允许用户在缺少开发领域知识的情况下对Kubernetes应用程序进行部署与/或更新。基于Kubernetes概念创建而成的Workflow旨在提供简单且开发者友好的应用程序部署方式。DeisWorkflow现已作为一项Kubernetes微服务进行交付，操作人员可轻松完成该平台的安装。Workflow能够以零宕机方式部署新的应用程序版本。

链接： [https://deis.com/workflow/](https://deis.com/workflow/)

使用成本：免费

**39. Kel**

Kel是一项来自Eldarion公司的开源PaaS，其可在整个生命周期内对面向Kubernetes的应用程序加以管理。Kel在Kubernetes的基础上还添加了两个分别由Python与Go编写而成的附加层，其中Level 0 允许用户配置Kubernetes资源，而Level 1允许用户在Kubernetes上部署任何应用程序。

链接： [http://www.kelproject.com/](http://www.kelproject.com/)

使用成本：免费

