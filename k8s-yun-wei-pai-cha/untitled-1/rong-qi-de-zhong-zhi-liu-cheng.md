# 容器的终止流程

#### 容器的终止流程

一个示例流程：

1. 用户发送一个命令来删除Pod，默认的优雅退出时间是30秒

2. API服务器中的Pod更新时间，超过该时间Pod被认为死亡

3. 在客户端命令的的里面，Pod显示为”Terminating（退出中）”的状态

4. （与第3同时）当Kubelet看到Pod标记为退出中的时候，因为第2步中时间已经设置了，它开始pod关闭的流程

i. 如果该Pod定义了一个停止前的钩子，其会在pod内部被调用。如果钩子在优雅退出时间段超时仍然在运行，第二步会意一个很小的优雅时间断被调用

ii. 进程被发送TERM的信号

5. （与第三步同时进行）Pod从service的列表中被删除，不在被认为是运行着的pod的一部分。缓慢关闭的pod可以继续对外服务，当负载均衡器将他们轮流移除。

6. 当优雅退出时间超时了，任何pod中正在运行的进程会被发送SIGKILL信号被杀死。

7. Kubelet会完成pod的删除，将优雅退出的时间设置为0（表示立即删除）。pod从API中删除，不在对客户端可见。

默认情况下，所有的删除操作的优雅退出时间都在30秒以内。kubectl delete命令支持–graceperiod=的选项，以运行用户来修改默认值。0表示删除立即执行，并且立即从API中删除pod这样一个新的pod会在同时被创建。在节点上，被设置了立即结束的的pod，仍然会给一个很短的优雅退出时间段，才会开始被强制杀死
