# Helm安装

## 使用Helm管理kubernetes应用 <a id="&#x4F7F;&#x7528;helm&#x7BA1;&#x7406;kubernetes&#x5E94;&#x7528;"></a>

读完本文后您应该可以自己创建chart，并创建自己的私有chart仓库。

[Helm](http://helm.sh/)是一个kubernetes应用的包管理工具，用来管理[charts](https://github.com/kubernetes/charts)——预先配置好的安装包资源，有点类似于Ubuntu的APT和CentOS中的yum。

Helm chart是用来封装kubernetes原生应用程序的yaml文件，可以在你部署应用的时候自定义应用程序的一些metadata，便与应用程序的分发。

Helm和charts的主要作用：

* 应用程序封装
* 版本管理
* 依赖检查
* 便于应用程序分发

### 前提条件 <a id="&#x524D;&#x63D0;&#x6761;&#x4EF6;"></a>

需要准备以下前提条件才能成功且安全地使用 Helm。

1. 一个 Kubernetes 集群
2. 确定使用哪种安装安全配置（如果有的话）
3. 安装和配置 Helm 和集群端服务 Tiller。

#### 安装 Kubernetes 或有权访问群集 <a id="&#x5B89;&#x88C5;-kubernetes-&#x6216;&#x6709;&#x6743;&#x8BBF;&#x95EE;&#x7FA4;&#x96C6;"></a>

* 必须已安装 Kubernetes。对于 Helm 的最新版本，我们推荐最新的 Kubernetes 稳定版本，在大多数情况下它是次新版本。
* 应该有一个本地配置好的 `kubectl`。

**注意：**1.6 之前的 Kubernetes 版本对于基于角色的访问控制（RBAC），要么有限制，或者不支持。

Helm 将通过 Kubernetes 配置文件（通常是 `$HOME/.kube/config`）来确定在哪里安装 Tiller 。这个配置文件也是 kubectl 使用的文件。

要找出 Tiller 将安装到哪个集群，可以运行 `kubectl config current-context` 或 `kubectl cluster-info`。

```text
$ kubectl config current-context 
my-cluster
```

### 安装Helm <a id="&#x5B89;&#x88C5;helm"></a>

下载 Helm 客户端的二进制版本。可以使用类似工具如 `homebrew`，或查看 [官方发布页面](https://github.com/kubernetes/helm/releases)。

有关更多详细信息或其他选项，请参阅 [安装指南](https://whmzsu.github.io/helm-doc-zh-cn/quickstart/install-zh_cn.html)。

Helm 有两个部分：Helm 客户端（helm）和 Helm 服务端（Tiller）。本指南介绍如何安装客户端，然后继续演示两种安装服务端的方法。

**重要提示** ：如果你负责的群集是在受控的环境，尤其是在共享资源时，强烈建议使用安全配置安装 Tiller。有关指导，请参阅 [安全 Helm 安装](https://whmzsu.github.io/helm-doc-zh-cn/quickstart/securing_installation-zh_cn.html)。

 coreos 需要有 socat 安装

**安装步骤**

### 安装 Helm 客户端 <a id="&#x5B89;&#x88C5;-helm-&#x5BA2;&#x6237;&#x7AEF;"></a>

Helm 客户端可以从源代码安装，也可以从预构建的二进制版本安装。

#### 方法一、从二进制版本 <a id="&#x4ECE;&#x4E8C;&#x8FDB;&#x5236;&#x7248;&#x672C;"></a>

每一个版本 [release](https://github.com/kubernetes/helm/releases)Helm 提供多种操作系统的二进制版本。这些二进制版本可以手动下载和安装。

1. 下载你 [想要的版本](https://github.com/kubernetes/helm/releases)

```text
# 下载 Helm
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
# 解压 Helm
$ tar -zxvf helm-v2.9.1-linux-amd64.tar.gz
# 复制客户端执行文件到 bin 目录下
$ cp linux-amd64/helm /usr/local/bin/
```

到这里，你应该可以运行客户端了：`helm help`。

#### 方法二、从脚本

 Helm官方提供的脚本一键安装 shell 脚本，将自动获取最新版本的 Helm 客户端并在本地安装。

可以获取该脚本，然后在本地执行它。这种方法也有文档指导，以便可以在运行之前仔细阅读并理解它在做什么。

```text
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

 注：storage.googleapis.com 默认是不能访问的，该问题请自行解决

```text
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

 也可以做到这一点。

### 安装 Helm 服务器端**-** Tiller <a id="&#x5B89;&#x88C5;-tiller"></a>

Helm 的服务器端部分 Tiller 通常运行在 Kubernetes 集群内部。但是对于开发，它也可以在本地运行，并配置为与远程 Kubernetes 群集通信。

#### 快捷群集内安装 <a id="&#x5FEB;&#x6377;&#x7FA4;&#x96C6;&#x5185;&#x5B89;&#x88C5;"></a>

安装 `tiller` 到群集中最简单的方法就是运行 `helm init`。这将验证 `helm` 本地环境设置是否正确（并在必要时进行设置）。然后它会连接到 `kubectl` 默认连接的任何集群（`kubectl config view`）。一旦连接，它将安装 `tiller` 到 `kube-system` 命名空间中。

`helm init` 以后，可以运行 `kubectl get pods --namespace kube-system` 并看到 Tiller 正在运行。

你可以通过参数运行 `helm init`:

* `--canary-image` 参数安装金丝雀版本
* `--tiller-image` 安装特定的镜像（版本）
* `--kube-context` 使用安装到特定群集
* `--tiller-namespace` 用一个特定的命名空间 \(namespace\) 安装

一旦安装了 Tiller，运行 helm version 会显示客户端和服务器版本。（如果它仅显示客户端版本， helm 则无法连接到服务器, 使用 `kubectl` 查看是否有任何 tiller Pod 正在运行。）

除非设置 `--tiller-namespace` 或 `TILLER_NAMESPACE` 参数，否则 Helm 将在命名空间 `kube-system` 中查找 Tiller 。

  helm init  在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "[https://kubernetes-charts.storage.googleapis.com](https://kubernetes-charts.storage.googleapis.com/)" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。如果你当前执行的机器不能访问该域名的话可以使用以下命令来安装：

```text
# 使用阿里云镜像安装并把默认仓库设置为阿里云上的镜像仓库
$ helm init --upgrade --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.1 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

\(目前最新版v2.9.1，可以使用阿里云镜像，如： helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google\_containers/tiller:v2.9.1 --stable-repo-url [https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts）](https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts%EF%BC%89)

 我们使用`-i`指定自己的镜像，因为官方的镜像因为某些原因无法拉取，官方镜像地址是：`gcr.io/kubernetes-helm/tiller:v2.8.1`，我在DockerHub上放了一个备份`jimmysong/kubernetes-helm-tiller:v2.8.1`，该镜像的版本与helm客户端的版本相同，使用`helm version`可查看helm客户端版本。

**给 Tiller 授权**

因为 Helm 的服务端 Tiller 是一个部署在 Kubernetes 中 Kube-System Namespace 下 的 Deployment，它会去连接 Kube-Api 在 Kubernetes 里创建和删除应用。

而从 Kubernetes 1.6 版本开始，API Server 启用了 RBAC 授权。目前的 Tiller 部署时默认没有定义授权的 ServiceAccount，这会导致访问 API Server 时被拒绝。所以我们需要明确为 Tiller 部署添加授权。

* 创建 Kubernetes 的服务帐号和绑定角色

```text
$ kubectl get deployment --all-namespaces
NAMESPACE     NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   tiller-deploy          1         1         1            1           1h
$ kubectl create serviceaccount --namespace kube-system tiller
$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```

* 为 Tiller 设置帐号

### 总结 <a id="&#x603B;&#x7ED3;"></a>

在大多数情况下，安装和获取预先构建的 helm 二进制代码和 `helm init` 一样简单。这个文档提供而了一些用例给那些想要用 Helm 做更复杂的事情的人。

一旦成功安装了Helm Client和Tiller，可以继续下一步使用Helm来管理charts。

文章出处：helm文章 [https://rootsongjc.gitbooks.io/kubernetes-handbook/practice/helm.html](https://rootsongjc.gitbooks.io/kubernetes-handbook/practice/helm.html)

[https://whmzsu.github.io/helm-doc-zh-cn/quickstart/install-zh\_cn.html](https://whmzsu.github.io/helm-doc-zh-cn/quickstart/install-zh_cn.html)

[http://www.10tiao.com/html/357/201808/2247486154/1.html](http://www.10tiao.com/html/357/201808/2247486154/1.html)

[https://www.soasme.com/techshack.weekly/verses/384bfb0c-e6f8-4f40-8b05-43944c5fe232.html](https://www.soasme.com/techshack.weekly/verses/384bfb0c-e6f8-4f40-8b05-43944c5fe232.html)



