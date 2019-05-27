# kibana简单使用



### 1.日志收集处理框架 <a id="1&#x65E5;&#x5FD7;&#x6536;&#x96C6;&#x5904;&#x7406;&#x6846;&#x67B6;"></a>

* Scribe\(facebook\)
* Flume\(apache\)
* Logstash\(Elastic\)

### 2.Logstash简介 <a id="2logstash&#x7B80;&#x4ECB;"></a>

* `Logstash 项目诞生于 2009 年 8 月 2 日。其作者是世界著名的运维工程师乔丹西塞(JordanSissel)。`
* `2013 年，Logstash 被 Elasticsearch 公司收购。Elasticsearch 本身 也是近两年最受关注的大数据项目之一，三次融资已经超过一亿美元。`
* `在 Elasticsearch 开发人员的共同努力下，Logstash 的发布机制，插件架构也愈发科学和合理`

### 3.Kibana简介 <a id="3kibana&#x7B80;&#x4ECB;"></a>

* `Logstash 早期曾经自带了一个特别简单的 logstash-web 用来查看 ES 中的数据。其功能太过简单，于是 Rashid Khan 用 PHP 写了一个更好用的 web，取名叫 Kibana。`
* `2014 年 4 月，Kibana3 停止开发，ES 公司集中人力开始 Kibana4 的重构`

### 4.Discover功能 <a id="4discover&#x529F;&#x80FD;"></a>

* 1.设置时间过滤器
* 2.搜索数据  在 Discover 页提交一个搜索，你就可以搜索匹配当前索引模式的索引数据了。你可以直接输入简单的请求字符串，也就是用 Lucene [query syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
* 3.按字段过滤   

![](../../.gitbook/assets/image%20%2859%29.png)

### 5.Visualize功能 <a id="5visualize&#x529F;&#x80FD;"></a>

* 1.创建一个新可视化   


  * 第 1 步: 选择可视化类型
  * 第 2 步: 选择数据源
  * 第 3 步: 可视化编辑器 

  

* 2.聚合构建器

![](../../.gitbook/assets/image%20%2884%29.png)

### 6.Dashboard功能 <a id="6dashboard&#x529F;&#x80FD;"></a>

* 1.创建一个新的仪表板  
  * 添加可视化到仪表板上
  * 保存仪表板
  * 加载已保存仪表板
  * 分享仪表板
  * 嵌入仪表板
* 2.自动刷新页面  

![](../../.gitbook/assets/image%20%28164%29.png)

