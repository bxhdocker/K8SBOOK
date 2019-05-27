# CI/CD

持续集成与发布，简称CI/CD，是微服务构建的重要环节，也是DevOps中推崇的方法论。如何在kubernetes中使用持续构建与发布工具？可以既可以与企业内部原有的持续构建集成，例如Jenkins，也可以在kubernetes中部署一套新的持续构建与发布工具，例如Drone。

众所周知Kubernetes并不提供代码构建、发布和部署，所有的这些工作都是由CI/CD工作流完成的，最近TheNewStack又出了本小册子（117页）介绍了Kubernetes中CI/CD的现状。下载本书的PDF请访问：[https://thenewstack.io/ebooks/kubernetes/ci-cd-with-kubernetes/](https://thenewstack.io/ebooks/kubernetes/ci-cd-with-kubernetes/)

![](../../.gitbook/assets/image%20%2874%29.png)



Kubernetes 生态中的 Devops 工具实践。

## 源码部署（Source to Deployment） <a id="yuan-ma-bu-shu-source-to-deployment"></a>

* ​[Draft](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/draft)： 提供了一个用于简化容器构建和部署（基于 Helm）的工具，使用方法见[这里](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/draft)​
* ​[Skaffold](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/skaffold)：同 Draft 类似，但不支持 Helm，使用方法见[这里](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/skaffold)​
* Metaparticle：提供了一套用于开发云原生应用的标准库，使用方法见 [https://metaparticle.io](https://metaparticle.io/)​

## CI/CD <a id="ci-cd"></a>

* ​[Jenkins X](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/jenkinsx)​
* ​[Spinnaker](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/spinnaker)​
* ​[Argo](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/argo)​
* ​[Flux GitOps](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/flux)​
* TeamCity

## 其他 <a id="qi-ta"></a>

* ​[Kompose](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/kompose)​

![](../../.gitbook/assets/image%20%2884%29.png)



## TeamCity

TeamCity是一款成熟的CI服务器，来自JetBrains公司。JetBrains已经在软件开发世界中建立了权威，他们的工具如WebStorm和ReSharper正被全球的开发者所使用。

TeamCity在它的免费版本中提供了所有功能，但仅限于20个配置和3个构建代理。额外的构建代理和构建配置需要购买，你可以在这里找到价格。

TeamCity安装后即可使用，可以在多种不同的平台上工作，并支持各种各样的工具和框架。能够支持JetBrains和第三方公司开发的公开的插件。尽管是基于Java的解决方案，TeamCity在众多的持续集成工具中提供了最好的.NET支持。TeamCity也有多种企业软件包，可以按所需代理的数量进行扩展。

总结：整体而言是TeamCity是非常好的持续集成解决方案，但由于其复杂性和价格，更适合企业需求。

官方网站：TeamCity

可用性：3个代理和20个构建配置是免费的，额外的代理和配置需要付费

平台：Servlet容器（本地）

## Codeship

Codeship是一个本地的持续集成解决方案。它有两种不同的版本：基本版和专业版。在基本版中提供了安装即用的持续集成服务但是不能够支持Docker，它的主要用途就是通过UI来进行应用的构建等操作。专业版本提供了更灵活的功能以及Docker支持。

在基本版中有几个可选的付费包，越贵的付费包并行能力越好。在专业版本中你可以选择你的实例类型和并行级别（最高的级别为20x），价格稍微有点贵，但是大多数的团队应该会需要这种并行化构建的功能。

总结：Codeship是一个强大的带有Docker支持的本地持续集成解决方案。

官方网站：Codeship

可用性：每个月的前100次构建免费，后续的构建需要付费

平台：托管

## Codefresh

上面所提到的很多工具都能够支持Docker，但Codefrsh从设计到开发都将容器的理念贯彻其中。

Codefresh的开发者们从一开始就意识到Docker会广受欢迎。Codefresh除了能够在现有的Docker文件中工作外，你也可以选择几个不同的模板来轻松地的将你的项目迁移到docker容器中。它的UI非常的干净和容易理解，你可以很容易地上手。

之所以将Codefresh介绍给你们的原因在于它有一个让人非常惊喜的功能。这个功能就是将你的镜像发布到一个临时的环境中。当项目被建立时，它的镜像也被建立了，你可以发布这个镜像并观察它是如何工作的。那意味着你可以得到一个临时的工作环境，而不需要一个额外的虚拟机，这就是它非常棒的地方。

Codefresh是一款比较新的工具，有很多能够改进的地方和新的特性可以增加。但是它把容器作为它的重要组成部分使得它对任何一个打算使用Docker容器的团队来说都将是一个非常好的持续集成解决方案。

总结：Codefresh是一个支持Docker的持续集成工具，它可以发布和建立本地环境的Docker镜像。

官方网站：Codefresh

可用性：每个月的前200次构建，5个并发的构建和一个本地环境是免费的，额外的服务需要付费。

平台：本地

