# istio：监控能力介绍

 我们知道每个pod内都会有一个Envoy容器，其具备对流入和流出pod的流量进行管理，认证，控制的能力。Mixer则主要负责访问控制和遥测信息收集。  
  
如拓扑图所示，当某个服务被请求时，首先会请求istio-policy服务，来判定是否具备访问资格，若具备资格则放行反之则请求不会被下发到服务。这一切的访问信息，都会被记录在Envoy中，之后会上报给mixer作为原始数据。遥测数据的收集及其他功能完全是灵活可控的，你既可以配置新的收集指标和日志，也可以完全禁用这些功能。

![](../../.gitbook/assets/image%20%283%29.png)

 1.Prometheus的应用和指标介绍  
  
Prometheus是一款开源的监控和告警系统，2016年加入CNCF，以其灵活的检索语言，高效的数据存储方式以及多维度的数据模型使得越来越多的人使用。Istio自0.8开始就默认的将Prometheus包含在内，我们可以通过查询service或者pod看到普罗的运行状态和地址。点开Prometheus界面，UI十分简洁明了。

![](../../.gitbook/assets/image%20%28157%29.png)

 用户在Expression内输入想要查询数据的表达式，并且再输入的过程中，普罗还会在已有的指标中做出提示方便用户查找。我们输入一个简单的查询表达式istio\_requests\_total,点击Execute，在图形界面中，将鼠标放到图中的折线可以看到请求的详细信息。

![](../../.gitbook/assets/image%20%28138%29.png)

 详细信息中的每一项都可以作为选定参考指标的特性，例如我们需要查询返回值为200的productpage请求总数，就可以在之前的表达式中添加大括号和限定条件。

![](../../.gitbook/assets/image%20%2857%29.png)

 Istio支持和允许用户自己定义新的遥测数据，并且在官网-&gt;任务-&gt;遥测-&gt;收集指标和日志中有详细的描述。用户可以自定义需要的监控指标进而可以再普罗查看监控数据结果。  
  
2.Jaeger UI的使用和介绍  
  
Istio配合jaeger可以解决端到端的分布式追踪问题。Jaeger于2017年9月成为CNCF的成员。Jaeger是一款开源的分布式追踪系统，由Google Dapper和OpenZipkin社区联合推动。

![](../../.gitbook/assets/image%20%28149%29.png)

 Jager主要可以使用在微服务的架构上来完成分布式上下文广播，分布式事务监控，根因分析，服务依赖关系分析，性能/延迟优化等功能。

![](../../.gitbook/assets/image%20%2897%29.png)

 Jaeger的界面极其简洁，在首页面选择你想了解的服务（productpage）以及选择你想观测的时间范围（过去两天），而后点击find trace按钮，页面就会显示过去两天内访问productpage的所有trace。点击trace的名字，则会跳转到详情界面。

![](../../.gitbook/assets/image%20%28135%29.png)

 这个界面中你可以看到每个请求可能会分为不止一个的子请求，以及这个请求的处理时间。例如我们访问productpage，productpage会请求details和reviews这两个服务，那么初始的请求就会分为两个子请求，一个请求details的内容另一个请求reviews的内容。Details部分的请求总耗时4.99ms，reviews部分的请求耗时5.61ms。内容返回并处理后，整个productpage的请求耗时21.32ms。这个详情界面不仅会体现每个请求的耗时也反映服务之间的调用关系。根据istio官方给出的解释，我们知道istio proxy根据http部分headers来归纳和合并请求的。

![](../../.gitbook/assets/image%20%2811%29.png)

 对于一个结构复杂，流量庞大的服务网格，追踪所有的调用不但不利于收集有效数据，还会造成冗余，浪费资源等问题，所以在制定监控服务的时候也需要去设定其采样频率。对于Bookinfo这种示例型应用，我们的采样率可以设的高一点，对于大型应用就要进行适当的降低。  
  
调整采样率一共有两种方式。  
  
•在创建服务网格之前，我们可以提前设定好采样频率，在Helm模板的values.yaml文件中，pilot内的traceSampling属性可以对采样频率进行修改。

![](../../.gitbook/assets/image%20%28107%29.png)

 打开istio/chart/pilot/templates/deployment.yaml可以看到一个简单的赋值过程

![](../../.gitbook/assets/image%20%284%29.png)

 •正在运行的服务网格，对deployment istio-pilot进行编辑。首先查看所有的deployment：  


![](../../.gitbook/assets/image%20%28124%29.png)

 然后对其进行编辑，搜索PILOT\_TRACE\_SAMPLING这个属性，并对其值进行修改：

![](../../.gitbook/assets/image%20%28130%29.png)

 我们先打开jaeger UI确定过去一个小时没有任何对productpage的访问。

![](../../.gitbook/assets/image%20%28155%29.png)

 而后将PILOT\_TRACE\_SAMPLING的值从原有的100改为50。修改并保存后会有提示信息显示istio-pilot已经被修改。 

![](../../.gitbook/assets/image%20%28113%29.png)

 稍等片刻后，我们使用脚本curl productpage10次。再次在jaeger UI上选择productpage选择过去一小时，点击Find Trace，会发现这次只检测到4个trace。我们在用相同的脚本再运行一次，发现检测到10个trace。至此我们一共curl product page 20次总共获得10次 trace，符合总次数的50%。

![](../../.gitbook/assets/image%20%28101%29.png)

 现在我们用相同的方法，将PILOT\_TRACE\_SAMPLING改为100%并且稍等片刻。使用相同的脚本curl10次product page，再点击Find trace，现在总共有20个Trace，也就是先前的10个trace加上后来curl的10次，证明 PILOT\_TRACE\_SAMPLING修改完毕会采集所有的请求。

[原文链接](http://dockone.io/article/8497)

