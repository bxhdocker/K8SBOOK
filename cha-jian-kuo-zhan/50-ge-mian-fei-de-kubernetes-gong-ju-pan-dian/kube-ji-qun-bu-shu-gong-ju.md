# Kube集群部署工具

**1. Kubespray**

Kubespray面向Kubernetes的部署与配置场景提供一系列与Ansible类似的作用，且还可部署于AWS、GCE、Azure、OpenStack或裸机基础设施即服务（IaaS）平台之上。此外，Kubespray还是一个提供开放式开发模式的开源项目。对于已经熟悉Ansible的开发人员而言，因为Kubespray不再需要使用其他工具即可实现服务配置与编排，故而其无疑是个不错的选择。更值得一提的是，Kubespray的底层实现机制为Kubeadm。

链接： [https://github.com/kubernetes-incubator/kubespray](https://github.com/kubernetes-incubator/kubespray)

使用成本：免费。

**2. Minikube**

Minikube为Kubernetes提供一套本地实验环境，允许用户在本地安装并试用Kubernetes。该工具可为您提供试用体验以决定是否选用Kubernetes，且能够通过简单易操作的方式在笔记本电脑的虚拟机（VM）内启动一个单节点Kubernetes集群。此外，Minikube亦适用于Windows、Linux以及OSX，并且只需短短5分钟，就能够让您对Kubernetes的主要功能有所了解。最后，仅需一行命令即可启动Minikube仪表盘。

链接： [https://github.com/kubernetes/minikube](https://github.com/kubernetes/minikube)

使用成本：免费

**3. Kubeadm**

Kubeadm是Kubernetes自版本1.4以来就默认使用的分发工具，该工具可帮助用户在现有的基础架构上体验Kubernetes的最佳实践。尽管如此，Kubeadm无法为开发人员配置基础设施。该工具的主要优势在于其可在任何环境下启动最小的可行Kubernetes集群。需要注意的是，Kubeadm内不含任何附加组件与网络设置，因此您需要手动或使用其他工具完成相关工具的安装。

链接： [https://github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm)

使用成本：免费

**4. Kops**

Kops可帮助开发人员通过命令行创建、销毁、升级并维护生产级别与高可用性Kubernetes集群。该工具目前已得到了亚马逊网络服务（AWS）的官方支持，GCE与VMwarevSphere也分别以beta与alpha测试形式为其提供相应支持。此外，其他平台对于该工具的支持也正在按计划推进。Kops允许用户控制Kubernetes集群的完整生命周期——从基础设施配置到删除集群皆在其中。

链接： [https://github.com/kubernetes/kops](https://github.com/kubernetes/kops)

使用成本：免费

**5. Bootkube**

随着版本1.4的发布，CoreOS提出了自托管Kubernetes集群的概念。这一自托管集群方法的核心在于Bootkube，其可帮助用户建立一套临时的Kubernetes控制层。Bootkube所创建的控制层可持续运行，直到自托管控制层有能力处理相关请求为止。

链接： [https://github.com/kubernetes-incubator/bootkube](https://github.com/kubernetes-incubator/bootkube)

使用成本：免费

**6. Kubernetes on AWS \(Kube-AWS\)**

Kube-AWS是由CoreOS提供的一套控制台工具，其可使用AWSCloudFormation部署一套全功能Kubernetes集群。Kube-AWS允许用户部署传统的Kubernetes集群，也可使用原生AWS功能（例如ELB、S3与自动扩展等）为每个Kubernetes服务提供配置。

链接： [https://github.com/kubernetes-incubator/kube-aws](https://github.com/kubernetes-incubator/kube-aws)

使用成本：免费

**7. SimpleKube**

SimpleKube是一种bash脚本，该脚本允许用户在Linux服务器上部署单节点Kubernetes集群。同样是部署单节点集群，Minikube需要运行虚拟机管理程序（VirtualBox、KVM），而SimpleKube则把所有Kubernetes二进制文件安装到服务器当中。SimpleKube已经在Debian 8/9与Ubuntu 16.x/17.x上完成了测试，并且对于首次尝试使用Kubernetes的用户而言，SimpleKube绝对是一款不容错过的出色工具。

链接： [https://github.com/valentin2105/Simplekube](https://github.com/valentin2105/Simplekube)

使用成本：免费

**8. Juju**

Juju是由Canonical公司提供的一款管理程序。用户通过该管理程序可远程操作云供应商提供的解决方案。相较于Puppet/Ansible/Chef，Juju的抽象层级更高，并且其管理的对象为服务——而非机器/虚拟机。Canonical致力于提供适用于生产过程的“Kubernetes核心捆绑包”。由于Juju带有独立的控制台/用户界面，故而其也可作为专用工具使用。最后，Juju将在测试期间免费提供即服务（JaaS）版本。

链接： [https://jujucharms.com/](https://jujucharms.com/)

使用成本：

* 免费社区版
* 商业版——每年200美元起

**9. Conjure-up**

Conjure-up是来自于Canonical的另一款产品，该产品可通过一些简单的命令“在Ubuntu上部署Kubernetes的Canonical发行版”。该工具支持AWS、GCE、Azure、Joyent、OpenStack、VMware、裸机与本地主机等部署场景。此外，Juju、MAAS以及LXD均作为Conjure-up的底层技术存在。

链接： [https://conjure-up.io/](https://conjure-up.io/)

使用成本：免费

