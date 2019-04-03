# CI/CD

持续集成与发布，简称CI/CD，是微服务构建的重要环节，也是DevOps中推崇的方法论。如何在kubernetes中使用持续构建与发布工具？可以既可以与企业内部原有的持续构建集成，例如Jenkins，也可以在kubernetes中部署一套新的持续构建与发布工具，例如Drone。

众所周知Kubernetes并不提供代码构建、发布和部署，所有的这些工作都是由CI/CD工作流完成的，最近TheNewStack又出了本小册子（117页）介绍了Kubernetes中CI/CD的现状。下载本书的PDF请访问：[https://thenewstack.io/ebooks/kubernetes/ci-cd-with-kubernetes/](https://thenewstack.io/ebooks/kubernetes/ci-cd-with-kubernetes/)

![](../../.gitbook/assets/image%20%2869%29.png)



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

## 其他 <a id="qi-ta"></a>

* ​[Kompose](https://kubernetes.feisky.xyz/fu-wu-zhi-li/devops/kompose)​

![](../../.gitbook/assets/image%20%2879%29.png)

