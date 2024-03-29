# Kubernetes 中如何保证优雅地停止 Pod

一直以来我对优雅地停止 Pod 这件事理解得很单纯：不就利用是 [PreStop hook](https://link.juejin.im?target=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fcontainers%2Fcontainer-lifecycle-hooks%2F%23container-hooks) 做优雅退出吗？但最近发现很多场景下 PreStop Hook 并不能很好地完成需求，这篇文章就简单分析一下“优雅地停止 Pod”这回事儿。

### 何谓优雅停止？

优雅停止（Graceful shutdown）这个说法来自于操作系统，我们执行关机之后都得 OS 先完成一些清理操作，而与之相对的就是硬中止（Hard shutdown），比如拔电源。

到了分布式系统中，优雅停止就不仅仅是单机上进程自己的事了，往往还要与系统中的其它组件打交道。比如说我们起一个微服务，网关把一部分流量分给我们，这时：

* 假如我们一声不吭直接把进程杀了，那这部分流量就无法得到正确处理，部分用户受到影响。不过还好，通常来说网关或者服务注册中心会和我们的服务保持一个心跳，过了心跳超时之后系统会自动摘除我们的服务，问题也就解决了；这是硬中止，虽然我们整个系统写得不错能够自愈，但还是会产生一些抖动甚至错误。
* 假如我们先告诉网关或服务注册中心我们要下线，等对方完成服务摘除操作再中止进程，那不会有任何流量受到影响；这是优雅停止，将单个组件的启停对整个系统影响最小化。

按照惯例，SIGKILL 是硬终止的信号，而 SIGTERM 是通知进程优雅退出的信号，因此很多微服务框架会监听 SIGTERM 信号，收到之后去做反注册等清理操作，实现优雅退出。

### PreStop Hook

回到 Kubernetes（下称 K8s），当我们想干掉一个 Pod 的时候，理想状况当然是 K8s 从对应的 Service（假如有的话）把这个 Pod 摘掉，同时给 Pod 发 SIGTERM 信号让 Pod 中的各个容器优雅退出就行了。但实际上 Pod 有可能犯各种幺蛾子：

* 已经卡死了，处理不了优雅退出的代码逻辑或需要很久才能处理完成。
* 优雅退出的逻辑有 BUG，自己死循环了。
* 代码写得野，根本不理会 SIGTERM。

因此，K8s 的 Pod 终止流程中还有一个“最多可以容忍的时间”，即 grace period（在 Pod 的 `.spec.terminationGracePeriodSeconds` 字段中定义），这个值默认是 30 秒，我们在执行 `kubectl delete` 的时候也可通过 `--grace-period` 参数显式指定一个优雅退出时间来覆盖 Pod 中的配置。而当 grace period 超出之后，K8s 就只能选择 SIGKILL 强制干掉 Pod 了。

很多场景下，除了把 Pod 从 K8s 的 Service 上摘下来以及进程内部的优雅退出之外，我们还必须做一些额外的事情，比如说从 K8s 外部的服务注册中心上反注册。这时就要用到 PreStop Hook 了，K8s 目前提供了 `Exec` 和 `HTTP` 两种 PreStop Hook，实际用的时候，需要通过 Pod 的 `.spec.containers[].lifecycle.preStop` 字段为 Pod 中的每个容器单独配置，比如：

```text
spec:
  contaienrs:
  - name: my-awesome-container
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh"，"-c"，"/pre-stop.sh"]
复制代码
```

`/pre-stop.sh` 脚本里就可以写我们自己的清理逻辑。

最后我们串起来再整个表述一下 Pod 退出的流程（[官方文档里更严谨哦](https://link.juejin.im?target=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fworkloads%2Fpods%2Fpod%2F%23termination-of-pods)）：

1. 用户删除 Pod。
2. * 2.1. Pod 进入 Terminating 状态。
   * 2.2. 与此同时，K8s 会将 Pod 从对应的 service 上摘除。
   * 2.3. 与此同时，针对有 PreStop Hook 的容器，kubelet 会调用每个容器的 PreStop Hook，假如 PreStop Hook 的运行时间超出了 grace period，kubelet 会发送 SIGTERM 并再等 2 秒。
   * 2.4. 与此同时，针对没有 PreStop Hook 的容器，kubelet 发送 SIGTERM。
3. grace period 超出之后，kubelet 发送 SIGKILL 干掉尚未退出的容器。

  
作者：PingCAP  
链接：https://juejin.im/post/5ca2caf6e51d450b007a6f72  
来源：掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

