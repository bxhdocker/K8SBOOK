# Bookinfo 应用-

## Bookinfo 应用 <a id="title"></a>

部署一个样例应用，它由四个单独的微服务构成，用来演示多种 Istio 特性。这个应用模仿在线书店的一个分类，显示一本书的信息。页面上会显示一本书的描述，书籍的细节（ISBN、页数等），以及关于这本书的一些评论。

Bookinfo 应用分为四个单独的微服务：

* `productpage` ：`productpage` 微服务会调用 `details` 和 `reviews` 两个微服务，用来生成页面。
* `details` ：这个微服务包含了书籍的信息。
* `reviews` ：这个微服务包含了书籍相关的评论。它还会调用 `ratings` 微服务。
* `ratings` ：`ratings` 微服务中包含了由书籍评价组成的评级信息。

`reviews` 微服务有 3 个版本：

* v1 版本不会调用 `ratings` 服务。
* v2 版本会调用 `ratings` 服务，并使用 1 到 5 个黑色星形图标来显示评分信息。
* v3 版本会调用 `ratings` 服务，并使用 1 到 5 个红色星形图标来显示评分信息。

下图展示了这个应用的端到端架构

![](../../.gitbook/assets/image%20%2871%29.png)

Istio 注入之前的 Bookinfo 应用

Bookinfo 是一个异构应用，几个微服务是由不同的语言编写的。这些服务对 Istio 并无依赖，但是构成了一个有代表性的服务网格的例子：它由多个服务、多个语言构成，并且 `reviews` 服务具有多个版本。

### 部署应用 <a id="&#x90E8;&#x7F72;&#x5E94;&#x7528;"></a>

要在 Istio 中运行这一应用，无需对应用自身做出任何改变。我们只要简单的在 Istio 环境中对服务进行配置和运行，具体一点说就是把 Envoy sidecar 注入到每个服务之中。这个过程所需的具体命令和配置方法由运行时环境决定，而部署结果较为一致，如下图所示

![Bookinfo &#x5E94;&#x7528;](../../.gitbook/assets/image%20%2898%29.png)

所有的微服务都和 Envoy sidecar 集成在一起，被集成服务所有的出入流量都被 sidecar 所劫持，这样就为外部控制准备了所需的 Hook，然后就可以利用 Istio 控制平面为应用提供服务路由、遥测数据收集以及策略实施等功能。

### 任务 <a id="&#x4EFB;&#x52A1;"></a>

1. [请求路由](https://istio.io/zh/docs/tasks/traffic-management/request-routing/)任务首先会把 Bookinfo 应用的进入流量导向 `reviews` 服务的 `v1` 版本。接下来会把特定用户的请求发送给 `v2` 版本，其他用户则不受影响。
2. [故障注入](https://istio.io/zh/docs/tasks/traffic-management/fault-injection/)任务会使用 Istio 测试 Bookinfo 应用的弹性，具体方式就是在 `reviews:v2` 和 `ratings` 之间进行延迟注入。接下来以测试用户的角度观察后续行为，我们会注意到 `reviews` 服务的 `v2` 版本有一个 Bug。注意所有其他用户都不会感知到正在进行的测试过程。
3. [流量迁移](https://istio.io/zh/docs/tasks/traffic-management/traffic-shifting/)，最后，会使用 Istio 将所有用户的流量转移到 `reviews` 的 `v3` 版本之中，以此来规避 `v2` 版本中 Bug 造成的影响。



