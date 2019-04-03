# 发展历史

“ 如果有一个 workload 到了亚马逊那儿，你们就完了！” 2013 年，VMware 公司首席执行官 Pat Gelsinger 在公司的合作伙伴交流大会（Partner Exchange Conference）如是说。“现在到永远，我们都要拥有公司的 workload”。作为前英特尔资深经理， Gelsinger 的这些话对云计算五年前的情景作了很好的还原：**彼时，VMware 还认为自己是宇宙的中心，并试图捍卫和巩固自己的地位。亚马逊正努力将数据中心迁移到自己的公有云领域。**

那时的 Gelsinger 肯定没想到，四年后，他确实再也不用担心失去公司的 workload 了。一场让他意想不到的“江湖纷争”即将袭来，他要对付的可不仅仅是老对手 Amazon 了。

**重新洗牌**

**VMware 一统江湖**

“VMware” 中的 “VM” 代表虚拟机，它成为了组织在云平台上部署应用程序的第一个工具。**虚拟机的出现引发了大众对“可移植性” （即在任何地方都能部署应用程序）的渴望。**但是 VMware 聪明地通过部署和管理虚拟机的方式来保护自己的利益，使得 vSphere 成为所有入站道路的必须监督者。

![](https://mmbiz.qpic.cn/mmbiz_png/5icZkHQug27ibgplM7EkgF4X9DopAeqoaofocgQ59UxibhRxmpQn5Z4ZPjQwiaILGoxGtXDTicvpOiaotZGcQhXibjRlw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

**理想的践行者 Docker** 

然后，Docker 出现了。它是第一个将“可移植性”赋予数据中心的工具。2013 年，Docker 开发了一个容器机制，将 workload 从笨重的操作系统中解救出来，来维护虚拟机。这样，他们就可以使用 Linux kernel 而不是虚拟机监视器来进行管理。这是当时众多组织触不可及的 “革命理想”，但只有 Docker 践行并实现了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/5icZkHQug27ibgplM7EkgF4X9DopAeqoaoLvalZunjbJZBmqVTSBSDnB6R1BldHtrSW1NibxcvAOX2YRssfiaH83dQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

从出现伊始， Docker 就遭遇了来自企业的质疑目光。同时出现的还有一个广为流传的论断：**一个新的市场即将围绕着 Docker 容器形成**。套用 PC 世界的旧模板，软件市场当时也是围绕格式和兼容性形成的。**推断下去，接下来将出现各大容器公司相互竞争的局面，然后是优胜劣汰，紧接着 Docker 或者其他更强大的竞争对手，会从对手的灰烬中取得最终胜利。**

只有当“可移植性”无处不在时，才实现了真正的可移植。2015 年，Docker 放大胆子赌了一把，将整个容器 format 捐赠给了 Linux 基金会赞助的一个开源项目。这个项目现在被称为 Open Container Initiative（OCI）。彼时的 Docker 计划先行一步，放弃人人想要的“可移植性”，让容器 format 增值。Docker 捐赠了容器 format，让竞争对手再也找不到与之竞争的明确方向，Docker 笃定它的未来就在部署和维护容器 workload 的机制上。

**Kubernetes 初出茅庐**

就在同年，谷歌推动建立了一个独立基金会：Cloud Native Computing Foundation \(CNCF\)。 CNCF 与 OCI 有很多相同的成员，但 CNCF 专注于容器 workload 的成员们，试图在“部署和管理”方面寻找竞争优势。紧接着，CNCF 就推出了 Kubernetes，这个编排工具将谷歌工作workload staging 概念与 Docker 容器相结合（最初名为“Borg”）。

Red Hat 公司围绕 Kubernetes 重建了 OpenShift PaaS 平台，而 CoreOS 将其业务重点转向 Tectonic，即 Kubernetes 的商业实施。

**Kubernetes 的野蛮生长**   


就像推倒了一块多米诺骨牌，容器领域的众多利益相关者在 2017 年一个接一个地将部署和管理战略倒向了 Kubernetes，包括 Docker 公司本身 ：  


![](https://mmbiz.qpic.cn/mmbiz_jpg/5icZkHQug27ibgplM7EkgF4X9DopAeqoaoNNjV9YJB5LKiaX0AOXQOJ3wMtL4vKyjuibGfAB4Fll46lQxDWGTCEKFQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

* **4 月**，托管的 OpenStack 云管理服务提供商 Mirantis，宣布将 Kubernetes 集成到其云平台 1.0 中，并承诺将其 Staging 和配置机制的重点从以安装人员为中心模式，转向以 workload 为中心模式。
* **同月**，微软收购了基于 Kubernetes 的容器部署平台制造商 Deis，以前所未有的姿态向 Azure 开放平台迁移。紧接着 Deis 技术在 Azure 上出现。 （ 2016 年 6 月微软就已经从 Google 挖走了 Kubernetes 的创始人 Brendan Burns。）
* **5 月**，在波士顿举行的 OpenStack 峰会上，OpenStack 许多社区领导齐聚一堂，承认 Kubernetes 是其私有云模型中，容器 workload 的事实 staging 环境。当时还有一个亟待解决的问题：Kubernetes 或 OpenStack 究竟应该留在虚拟基础架构的最底层，还是应该针对每个客户的情况合理地解决这个问题？
* 几乎在同时，IBM 推出了 Kubernetes 支持的云容器服务，向客户提供了一种无缝、即刻启动 Docker 容器的方法。
* **6 月初**，Oracle 在开源会议上承认 Kubernetes 是其新容器 Staging 策略的核心。作为 Docker 的竞争对手，CoreOS 自从平台成立以来就被称为 “Tectonic 的商用 Kubernetes 环境的生产者”。 CoreOS 把最小的 Linux kernel 贡献给 Oracle，Oracle 把自己的 Linux 踢到了一边。
* **6 月末**，在 Cloud Foundry 峰会上，Cloud Foundry 基金会发布了 Kubo，这是一种在传统机器内部 staging Kubernetes 的多个负载均衡实例的工具。这为 8 月份的VMware 和姊妹公司 Pivotal 与 Google 建立合作关系打下基础，并推出 Pivotal Container Service（PKS）。这个消息发布于 Amazon 与 VMware 建立合作伙伴关系一天后，当时 Kubo 已经为 Google 云平台定制进行了商业实施。值得注意的是，在整个 PKS 首次发布会期间，“Docker” 这个词一次也没被提过。（10 月份，Kubo 项目更名为 Cloud Foundry Container Runtime。）
* **8 月初**，亚马逊终于下了“赌注”，加入了 CNCF，并承诺做出实质性贡献。
* **9 月中旬,** Oracle ****也正式加入了 CNCF。
* **9 月中旬**，Mesosphere 做出了一个相当惊人的举动，在 beta 版本中，加入了一种整合这两个平台的方法。其用户可以安装、扩展和升级多个生产级 Kubernetes 集群。
* **10 月中旬**，这场容器竞争战似乎发出了即将结束的信号。Docker 宣布拥抱 Kubernetes，Docker 的创始人 Solomon Hykes 在大会上宣布：下一个版本的 Docker 将支持两种编排平台—— Swarm 和 Kubernetes！
* **10 月下旬**，微软推出了一个专门 Azure 容器服务（AKS）预览版，这一次 Kubernetes 不仅站在了“舞台” 中央上，而且 “K” 还是最中间的字母。不久之后，该公司的市场营销和网站给出了它的 Kubernetes 平台的第一个版本，推出了基于 DC / OS 的容器 Staging 平台作为备选方案。
* 在同一个月，思科在基于 ACI 的数据中心架构和谷歌云之间，通过 Kubernetes 宣布了一个名为 Goodzilla 的 Bridge。
* **11 月底**，亚马逊正式推出了自己的 AKS 容器服务，并把 AWS 云与 Google 和 Azure 相提并论。

**此消彼长**   


从市场角度来看，所有这些发展都表明：谷歌已经成功地控制了所有通向数据中心容器化道路。众所周知，没有可移植性的容器是毫无意义的，对于数据中心的客户们来说，一个 Staging 和编排环境是远远不够的。

**微软第一个证明，通过开源核心优势来迅速占领市场是个可行的办法。**当年，它让 Internet Explorer 变得免费，逼得 Netscape 只能通过增加大量不必要的新功能慌张回击，最终还是落得失败下场。Docker 显然也知道这个道理，开源了容器 format，让同类产品不能与之争锋。

然而，Docker 还是没有如当日所愿，建立自己的增值路线——就是那个可以通过开源手段来占领市场的“大杀器”。有人可能觉得，Docker 的基于集群的编排平台 Swarm 还不“成熟”。可是 Kubernetes 在向 Docker 发起进攻的时候可能更“幼稚”，但它有自己的“杀手锏”：相关的容器可以分组到 pod 中。

Red Hat 第一个出来，利用 OpenShift 来展示这一“杀手锏”, 这使得众多开源贡献者都对 Kubernetes 魂牵梦绕。在 Docker 移动创建 OCI 之后, Google 几乎马上就允许 Linux 基金会建立 CNCF，他们的成员几乎是同一拨人，这确保 OCI 部分的设计不会让 CNCF 感到意外，同时也保证了 Kubernetes 在社区讨论桌上的好位置。  


![](https://mmbiz.qpic.cn/mmbiz_jpg/5icZkHQug27ibgplM7EkgF4X9DopAeqoaoVHC0qFIgzsqVBVpKiaWUEiaDuRxPLc4lrRbb4yZCib0ibFe4zpbiaU1nPgQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

**Kubernetes 正在进行时**

**以下是 Kubernetes 正在进行时，在 2018 年肯定会让 Kubernetes 受益：**

* **CSI 项目：**这个项目最近得到了戴尔技术公司（戴尔 EMC 的母公司）的支持，这个项目承诺，将通过与数据库和 Storage Volume 保持持久链接，提供微服务 。使用 CSI 接口，任何打开这种持久链接的 API 都可以通过三个主要容器编排平台：Kubernetes，Mesos（DC / OS）和 Swarm 进行同样寻址。它目前由 Moby 管理，而 Moby 却是由 Docker 原始的开放源代码产生的，但现在 Moby 有一个独立的方向。这对于 Kubernetes 而言非常有利，开源数据 Plugins 的主题再也不是 Docker 领地中的本地特性了。
* **CRI-O 项目：**它利用 Kubernetes 的原生 Container Runtime Interface \(CRI\)，使容器编排平台能够通过本地 API（一个称为“runtime”的可寻址组件）实例化一个容器。目前在许多数据中心，生产环境把 Docker Engine 作为 Orchestrator 和 Runtime 之间的中介; CRI-O 专门为 Kubernetes 提供让 Docker 完全隔离在生产环境的方法，让它只能出现在开发人员的工作台上。
* **Kata 项目**：Kata 与 Docker 的竞争对手 Hyper 一同加入英特尔的 Clear Containers 项目中。Kata 可以为数据中心提供构建和部署 workload 的方法，Docker 将会完全被淘汰，同时还能够在基于容器的 workload 和第一代虚拟机之间共存。OpenStack 基金支持这个项目，它将利用 Kubernetes 作为其主要容器编排平台。

然而即便迎来了如此全胜局面，Kubernetes 仍然面临着巨大的挑战。具体来说，Kubernetes 可能变得如此无处不在，如此标准化，以至于任何供应商或开源贡献者都难以围绕它再创造竞争优势了。

**路漫漫其修远兮**   


对于数据中心来说，Kubernetes 的突然出现意味着：过去基于公有云的 PaaS 平台，如 Heroku 和原来的 Windows Azure，只有使用它支持的资源和语言才可用。而现在，只要使用 Kubernetes，每个人的平台都支持容器内部，而不是外部。这有助于在一定程度上平衡服务提供商，因为他们现在都可以提供相同的界面来获取和托管客户的 workload。这也缩小了这些提供者在服务上彼此竞争的空间。规律如此，每当市场商品化，幸存者都是那些赢得价格战的人。

Kubernetes 可能已经在 2017 年完全超过了 Docker。**但是市场瞬息万变，谁又能保证在 2018 年 Google 或其他任何人不会超过它呢，江湖路远，让我们拭目以待！**

原文链接  [k8sKubernetes 版图扩张全记录](https://mp.weixin.qq.com/s/cEGhA442ixMpWUP4jjHK7A)

