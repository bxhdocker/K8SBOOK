# Istio 04：Istio性能及扩展性介绍

 Istio的性能问题一直是国内外相关厂商关注的重点，Istio对于数据面应用请求时延的影响更是备受关注，而以现在Istio官方与相关厂商的性能测试结果来看，四位数的qps显然远远不能满足应用于生产的要求。从发布以来，Istio官方也在不断的对其性能进行优化增强。同时，Istio控制面的可靠性是Istio用于生产的另一项重要考量标准，自动伸缩扩容，自然是可靠性保证的重要手段。下面我们先从性能测试的角度入手，了解下Istio官方提供的性能测试方法与基准，主要分为以下四个方面展开。  
  
一、函数级别测试  
  
Istio提供了函数级别的性能测试用例，开发者可用更好的有针对性的进行性能优化，这里不做展开，感兴趣的朋友可用参考：  
  
[https://raw.githubusercontent. ... st.go](https://raw.githubusercontent.com/istio/istio/release-1.0/mixer/test/perf/singlecheck_test.go)   
  
二、端到端测试基准  
  
为了更好的跟踪Istio的性能问题，Istio提供了一个专门用于Isito性能测试的测试工具——Fortio。你可以通过kubernetes集群，轻而易举的将Forito部署起来，测试工具的安装链接如下：  
  
[https://github.com/istio/istio ... guide](https://github.com/istio/istio/tree/release-1.0/tools#istio-load-testing-user-guide)。  
  
下图是测试工具的组织结构图，其中上半部分为Istio服务网格管理的两个服务S1与S2，其中S1服务关掉了mixer上报功能（mixer遥测功能实现方案一直备受争议，业界普遍认为sidecar向mixer上报遥测数据一定程度上损害了sidecar的请求转发能力），请求通过ingressgateway流入系统，再经由sidecar分发到各个服务，是典型的Istio服务访问方式。下半部分为经典的kubernetes集群中的服务访问方式，请求经由k8s的proxy的转发，负载均衡到各个pod。两者的对比也就展示了Istio访问模式与经典模式相比，性能方面的损耗，下面介绍一下Fortio的几个功能点：  
  
[![0121\_12.jpg](http://dockone.io/uploads/article/20190121/d5e893a23253fa710a2ab7e36af05929.jpg)](http://dockone.io/uploads/article/20190121/d5e893a23253fa710a2ab7e36af05929.jpg)  
  
a\)Fortio是一个快速，小巧，可重复使用，可嵌入的go库以及命令行工具和服务器进程，该服务器包括一个简单的Web UI和结果的图形表示；  
  
b\)Fortio也是100％开源的，除了go和gRPC之外没有外部依赖，能够轻松地重现Istio官方性能测试场景也能适应自己想要探索的变体或场景；  
  
c\)Fortio以每秒指定的qps对服务进行压测，并记录影响时延的直方图并，计算百分比，平均值，tp99等统计数值。  
  
Fortio在kubernetes中的安装步骤：  
  
kubectl -n fortio run fortio --image=istio/fortio:latest\_release --port=8080  
  
kubectl -n fortio expose deployment fortio --target-port=8080 --type=LoadBalancer  
  
三、真实场景下测试基准  
  
Acmeair（又名BluePerf）是一个用Java实现的类似客户的微服务应用程序。此应用程序在WebSphere Liberty上运行，并模拟虚拟航空公司的运营。  
  
Acmeair由以下微服务组成：  
  
a\) Flight Service检索航班路线数据。预订服务会调用它来检查奖励操作的里程（Acmeair客户忠诚度计划）。  
  
b\) 客户服务存储，更新和检索客户数据。它由Auth服务用于登录和预订服务用于奖励操作。  
  
c\) 预订服务存储，更新和检索预订数据。  
  
d\) 如果用户/密码有效，Auth服务会生成JWT。  
  
e\) 主服务主要包括与其他服务交互的表示层（网页）。这允许用户通过浏览器直接与应用程序交互，但在负载测试期间不会执行此操作。  
  
[![0121\_13.jpg](http://dockone.io/uploads/article/20190121/53c5ef780787d0c8569ffc408300e32f.jpg)](http://dockone.io/uploads/article/20190121/53c5ef780787d0c8569ffc408300e32f.jpg)  
  
这个模拟航空公司的运营系统demo，能够更好的模拟在实际生产环境中的Istio应用，感兴趣的朋友可用到如下链接了解一下：[https://github.com/blueperf](https://github.com/blueperf)   
  
四、每日构建自动化测试结果  
  
Istio与IBM会对Istio的每日构建版本进行性能测试，并将测试结果公布出来，供大家参考。自动化测试包括端对端用例fortio以及应用级别bluePerf的性能测试结果。对Isito性能感兴趣，但没有时间精力进行性能测试的朋友，可以关注一下官方的每日性能测试结果，跟踪Istio性能优化的最新进展。  
  
● [https://fortio-daily.istio.io/](https://fortio-daily.istio.io/)  
  
● [https://ibmcloud-perf.istio.io/regpatrol/](https://ibmcloud-perf.istio.io/regpatrol/)  
  
这里，我们一起来看下IBM的性能测试结果，并进行一下分析。  
  
IBM的性能测试对比主要包括三部分，第一部分是各个Release版本之间的性能比较，其中列出了0.6.0,0.7.1,0.8.0版本的性能测试情况，这里的指标数据是qps，可以看到，前三个版本之间的数据十分相近，没有比较大的提升，且Istio与IngressOnly之间的对比可以看出，Istio造成了相当大的性能损耗，大约只能达到无Istio时百分之三十多的qps，可见，性能方面Istio还需要进一步优化。  
  
[![0121\_02.jpg](http://dockone.io/uploads/article/20190121/72fc4c020850ea5400b2f690bb3d5bd8.jpg)](http://dockone.io/uploads/article/20190121/72fc4c020850ea5400b2f690bb3d5bd8.jpg)  
  
第二部分是当前release每日构建版本的性能情况，可以对比出每天的修改对性能方面的影响，这里我们列出一部分，更详细的信息大家可以到相应链接中查看，可以看到近期每日构建版本相较于1.0基线版本，有了一定的提升。  
  
[![0121\_14.jpg](http://dockone.io/uploads/article/20190121/e0d5b62d3305b1972459f70f1c7dad2c.jpg)](http://dockone.io/uploads/article/20190121/e0d5b62d3305b1972459f70f1c7dad2c.jpg)  
  
最后一部分是master分支的每日构建性能测试结果。可以看到，最新的master分支的性能测试结果，相较基线版本已经有了较大的提升，但是QPS损耗严重的问题依然存在，同时，千级别的QPS也不能真正满足生产需求，我们期待Istio的发展与进步。  
  
[![0121\_15.jpg](http://dockone.io/uploads/article/20190121/cdedf988b2512c8b2d6030a5ce2d0e64.jpg)](http://dockone.io/uploads/article/20190121/cdedf988b2512c8b2d6030a5ce2d0e64.jpg)  
  
五、Isito的可靠性与可扩展性  
  
对控制面各组件的作用作用及故障影响进行了汇总，结果如下：  
  
[![0121\_03.jpg](http://dockone.io/uploads/article/20190121/f9aa42a7375decc336ca356868a219e4.jpg)](http://dockone.io/uploads/article/20190121/f9aa42a7375decc336ca356868a219e4.jpg)  
  
a\) 考量到Istio控制面的可靠性，以及对资源的有效利用，建议将重要组件设置为多实例，包括：  
  
● ingressgateway：作为外部流量入口，服务网格管理的所有服务的对外流量，都要经过gateway才能进入到服务网格内部，与业务逻辑强相关，建议配置为多实例；  
  
● mixer：遥测数据的搜集端，用于汇总服务网格内所有服务上报的日志、监控数据，处理数据量巨大，建议设置为多实例，并给予更多资源配置；  
  
● 其他控制面组件运行压力并不大，可根据需要自行调整实例数。  
  
b\) 容器自动扩缩容，Istio为主要组件gateway，pilot与mixer设置了自动扩缩容策略，且策略可以在安装时配置，我们以pilot为例看一下其自动扩缩容配置，以CPU占用率55%作为阈值，进行pod实例数的扩缩容侧率：  
  
[![0121\_16.jpg](http://dockone.io/uploads/article/20190121/0aa5ab6031b83c395f03fe0cdcb7a923.jpg)](http://dockone.io/uploads/article/20190121/0aa5ab6031b83c395f03fe0cdcb7a923.jpg)  
  
c\) 高可用（HA），可根据需要，为Istio控制面组件设置亲和性策略，使相同控制面组件的多个不同pod运行不同node上，确保Istio控制面的高可用状态，下面以pilot为例，Istio为其增加了节点反亲和策略，使pod尽可能不运行在相同架构操作系统的节点上，大家可根据需要，自行增加亲和与反亲和策略。  
  
[![0121\_17.jpg](http://dockone.io/uploads/article/20190121/6495d30605dcbd984d62f174c1445ac6.jpg)](http://dockone.io/uploads/article/20190121/6495d30605dcbd984d62f174c1445ac6.jpg)  
  
d\) 健康检查，Istio也为控制面组件配置了健康检查策略，以保证控制面组件异常时，k8s能够将其重新拉起，同样以sidecarinjector为例，设置了启动健康检查readinessProbe与运行时健康检查livenessProbe，以确保容器正常运行：  
  
[![0121\_18.jpg](http://dockone.io/uploads/article/20190121/8f167a0a777cca25a3e613badc9adc9e.jpg)](http://dockone.io/uploads/article/20190121/8f167a0a777cca25a3e613badc9adc9e.jpg)  
  
六、总结  
  
再次说明一下，性能与可靠性是Istio生产可用至关重要的一环，功能方面Istio已经做的十分强大，并在不断的完善发展中，但在性能与可靠性方面，Istio还有很长的路要走。其中微服务sidecar的路由转发与mixer遥测功能对请求时延的影响，是摆在Istio性能提升前面的两道障碍，我们共同努力，共同关注，望Istio早日成熟，发展壮大，扬帆起航。

时间：2019/9/21

