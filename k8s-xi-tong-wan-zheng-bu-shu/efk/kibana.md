# kibana

## 简介

kibana使用指南：[**https://www.elastic.co/guide/cn/kibana/current/setup.html**](https://www.elastic.co/guide/cn/kibana/current/setup.html)  
镜像仓库 

```text
docker pull docker.elastic.co/kibana/kibana:6.0.0
```

Elasticsearch版本

**Kibana的版本需要和Elasticsearch的版本一致。这是官方支持的配置**。

运行不同主版本号的Kibana和Elasticsearch是不支持的（例如Kibana 5.x和Elasticsearch 2.x），若主版本号相同，运行Kibana子版本号比Elasticsearch子版本号新的版本也是不支持的（例如Kibana 5.1和Elasticsearch 5.0

运行一个Elasticsearch子版本号大于Kibana的版本基本不会有问题，这种情况一般是便于先将Elasticsearch升级（例如Kibana 5.0和Elasticsearch 5.1）。在这种配置下，Kibana启动日志中会出现一个警告，所以一般只是使用于Kibana即将要升级到和Elasticsearch相同版本的场景。

运行不同的Kibana和Elasticsearch补丁版本一般是支持的（例如：Kibana 5.0.0和Elasticsearch 5.0.1），尽管我们鼓励用户去运行最新的补丁更新版本。

## 安装kibana



从6.0.0开始，Kibana只支持64位操作系统。

Kibana提供以下格式的安装包：

<table>
  <thead>
    <tr>
      <th style="text-align:left"><code>tar.gz</code>/<code>zip</code>
      </th>
      <th style="text-align:left">
        <p><code>tar.gz</code> &#x5305;&#x7528;&#x6765;&#x5728;Linux&#x548C;Darwin&#x7CFB;&#x7EDF;&#x4E0B;&#x5B89;&#x88C5;&#xFF0C;&#x4E5F;&#x662F;&#x6700;&#x65B9;&#x4FBF;&#x7684;&#x4E00;&#x79CD;&#x9009;&#x62E9;&#x3002;</p>
        <p><a href="https://www.elastic.co/guide/cn/kibana/current/targz.html">&#x4F7F;&#x7528;<code>.tar.gz</code>&#x5B89;&#x88C5;Kibana</a>&#x6216;&#x8005;
          <a
          href="https://www.elastic.co/guide/cn/kibana/current/windows.html">&#x5728;Windows&#x4E0A;&#x5B89;&#x88C5;Kibana</a>
        </p>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>deb</code>
      </td>
      <td style="text-align:left">
        <p><code>deb</code> &#x5305;&#x7528;&#x6765;&#x5728;Debian&#xFF0C;Ubuntu&#x548C;&#x5176;&#x4ED6;&#x57FA;&#x4E8E;Debian&#x7684;&#x7CFB;&#x7EDF;&#x4E0B;&#x5B89;&#x88C5;&#xFF0C;Debian&#x5305;&#x53EF;&#x4EE5;&#x4ECE;Elastic&#x5B98;&#x7F51;&#x6216;&#x8005;&#x6211;&#x4EEC;&#x7684;Debian&#x4ED3;&#x5E93;&#x4E2D;&#x4E0B;&#x8F7D;&#x3002;</p>
        <p><a href="https://www.elastic.co/guide/cn/kibana/current/deb.html">&#x4F7F;&#x7528;Debian&#x5305;&#x5B89;&#x88C5;Kibana</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>rpm</code>
      </td>
      <td style="text-align:left">
        <p><code>rpm</code> &#x5305;&#x7528;&#x6765;&#x5728;Red Hat&#xFF0C;Centos&#xFF0C;SLES&#xFF0C;OpenSuSe&#x4EE5;&#x53CA;&#x5176;&#x4ED6;&#x57FA;&#x4E8E;RPM&#x7684;&#x7CFB;&#x7EDF;&#x4E0B;&#x5B89;&#x88C5;.RPM&#x5305;&#x53EF;&#x4EE5;&#x4ECE;Elastic&#x5B98;&#x7F51;&#x6216;&#x8005;&#x6211;&#x4EEC;&#x7684;RPM&#x4ED3;&#x5E93;&#x4E0B;&#x8F7D;&#x3002;</p>
        <p><a href="https://www.elastic.co/guide/cn/kibana/current/rpm.html">&#x4F7F;&#x7528;RPM&#x5305;&#x5B89;&#x88C5;Kibana</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>docker</code>
      </td>
      <td style="text-align:left">
        <p>Elastic Docker&#x4ED3;&#x5E93;&#x4E2D;&#x6709;&#x73B0;&#x6709;&#x7684;&#x53EF;&#x4EE5;&#x8FD0;&#x884C;Kibana&#x7684;Docker&#x955C;&#x50CF;&#xFF0C;&#x5E76;&#x9884;&#x88C5;&#x4E86;
          <a
          href="https://www.elastic.co/products/x-pack">X-Pack</a>&#x3002;</p>
        <p><a href="https://www.elastic.co/guide/cn/kibana/current/docker.html"><em>Docker&#x5BB9;&#x5668;&#x4E2D;&#x8FD0;&#x884C;Kibana</em></a>
        </p>
      </td>
    </tr>
  </tbody>
</table>为 了方便本文以rpm安装演示

### rpm安装kibana

下载并安装签名公钥：

```text
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```



Kibana v6.0.0 的 RPM 包可以使用如下命令从网站下载安装 ：

```text
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.0.0-x86_64.rpm
sha1sum kibana-6.0.0-x86_64.rpm 
sudo rpm --install kibana-6.0.0-x86_64.rpm
```

| [![](https://www.elastic.co/guide/cn/kibana/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/cn/kibana/current/rpm.html#CO5-1) | 比较 `sha1sum` 或 `shasum` 产生的 SHA 跟 [发布的 SHA](https://artifacts.elastic.co/downloads/kibana/kibana-6.0.0-x86_64.rpm.sha1)。 |
| :--- | :--- |


Kibana 启动和停止命令如下：

```text
sudo systemctl start kibana.service
sudo systemctl stop kibana.service
```

这些命令不会提供任何关于 Kibana 是否成功启动的反馈信息。而是将这些信息写入日志文件中，日志文件的位置在 `/var/log/kibana/` 。

更多安装详情见：[rpm安装](https://www.elastic.co/guide/cn/kibana/current/rpm.html)

## 配置kibana

 Kibana server 启动时从 `kibana.yml` 文件中读取配置属性。Kibana 默认配置 `localhost:5601` 。改变主机和端口号，或者连接其他机器上的 Elasticsearch，需要更新 `kibana.yml` 文件。也可以启用 SSL 和设置其他选项

 **Kibana 配置项**

```text
server.port:
默认值: 5601 Kibana 由后端服务器提供服务，该配置指定使用的端口号。

server.host:
默认值: "localhost" 指定后端服务器的主机地址。

server.basePath:
如果启用了代理，指定 Kibana 的路径，该配置项只影响 Kibana 生成的 URLs，转发请求到 Kibana 时代理会移除基础路径值，该配置项不能以斜杠 (/)结尾。
server.maxPayloadBytes:
默认值: 1048576 服务器请求的最大负载，单位字节。
server.name:
默认值: "您的主机名" Kibana 实例对外展示的名称。
server.defaultRoute:
默认值: "/app/kibana" Kibana 的默认路径，该配置项可改变 Kibana 的登录页面。
elasticsearch.url:
默认值: "http://localhost:9200" 用来处理所有查询的 Elasticsearch 实例的 URL 。
elasticsearch.preserveHost:
默认值: true 该设置项的值为 true 时，Kibana 使用 server.host 设定的主机名，该设置项的值为 false 时，Kibana 使用主机的主机名来连接 Kibana 实例。
kibana.index:
默认值: ".kibana" Kibana 使用 Elasticsearch 中的索引来存储保存的检索，可视化控件以及仪表板。如果没有索引，Kibana 会创建一个新的索引。
kibana.defaultAppId:
默认值: "discover" 默认加载的应用。
tilemap.url:
Kibana 用来在 tile 地图可视化组件中展示地图服务的 URL。默认时，Kibana 从外部的元数据服务读取 url，用户也可以覆盖该参数，使用自己的 tile 地图服务。例如："https://tiles.elastic.co/v2/default/{z}/{x}/{y}.png?elastic_tile_service_tos=agree&my_app_name=kibana"
tilemap.options.minZoom:
默认值: 1 最小缩放级别。
tilemap.options.maxZoom:
默认值: 10 最大缩放级别。
tilemap.options.attribution:
默认值: "© [Elastic Tile Service](https://www.elastic.co/elastic-tile-service)" 地图属性字符串。
tilemap.options.subdomains:
服务使用的二级域名列表，用 {s} 指定二级域名的 URL 地址。
elasticsearch.username: 和 elasticsearch.password:
Elasticsearch 设置了基本的权限认证，该配置项提供了用户名和密码，用于 Kibana 启动时维护索引。Kibana 用户仍需要 Elasticsearch 由 Kibana 服务端代理的认证。
server.ssl.enabled
默认值: "false" 对到浏览器端的请求启用 SSL，设为 true 时， server.ssl.certificate 和 server.ssl.key 也要设置。
server.ssl.certificate: 和 server.ssl.key:
PEM 格式 SSL 证书和 SSL 密钥文件的路径。
server.ssl.keyPassphrase
解密私钥的口令，该设置项可选，因为密钥可能没有加密。
server.ssl.certificateAuthorities
可信任 PEM 编码的证书文件路径列表。
server.ssl.supportedProtocols
默认值: TLSv1、TLSv1.1、TLSv1.2 版本支持的协议，有效的协议类型: TLSv1 、 TLSv1.1 、 TLSv1.2 。
server.ssl.cipherSuites
默认值: ECDHE-RSA-AES128-GCM-SHA256, ECDHE-ECDSA-AES128-GCM-SHA256, ECDHE-RSA-AES256-GCM-SHA384, ECDHE-ECDSA-AES256-GCM-SHA384, DHE-RSA-AES128-GCM-SHA256, ECDHE-RSA-AES128-SHA256, DHE-RSA-AES128-SHA256, ECDHE-RSA-AES256-SHA384, DHE-RSA-AES256-SHA384, ECDHE-RSA-AES256-SHA256, DHE-RSA-AES256-SHA256, HIGH,!aNULL, !eNULL, !EXPORT, !DES, !RC4, !MD5, !PSK, !SRP, !CAMELLIA. 具体格式和有效参数可通过[OpenSSL cipher list format documentation](https://www.openssl.org/docs/man1.0.2/apps/ciphers.html#CIPHER-LIST-FORMAT) 获得。
elasticsearch.ssl.certificate: 和 elasticsearch.ssl.key:
可选配置项，提供 PEM格式 SSL 证书和密钥文件的路径。这些文件确保 Elasticsearch 后端使用同样的密钥文件。
elasticsearch.ssl.keyPassphrase
解密私钥的口令，该设置项可选，因为密钥可能没有加密。
elasticsearch.ssl.certificateAuthorities:
指定用于 Elasticsearch 实例的 PEM 证书文件路径。
elasticsearch.ssl.verificationMode:
默认值: full 控制证书的认证，可用的值有 none 、 certificate 、 full 。 full 执行主机名验证，certificate 不执行主机名验证。
elasticsearch.pingTimeout:
默认值: elasticsearch.requestTimeout setting 的值，等待 Elasticsearch 的响应时间。
elasticsearch.requestTimeout:
默认值: 30000 等待后端或 Elasticsearch 的响应时间，单位微秒，该值必须为正整数。
elasticsearch.requestHeadersWhitelist:
默认值: [ 'authorization' ] Kibana 客户端发送到 Elasticsearch 头体，发送 no 头体，设置该值为[]。
elasticsearch.customHeaders:
默认值: {} 发往 Elasticsearch的头体和值， 不管 elasticsearch.requestHeadersWhitelist 如何配置，任何自定义的头体不会被客户端头体覆盖。
elasticsearch.shardTimeout:
默认值: 0 Elasticsearch 等待分片响应时间，单位微秒，0即禁用。
elasticsearch.startupTimeout:
默认值: 5000 Kibana 启动时等待 Elasticsearch 的时间，单位微秒。
pid.file:
指定 Kibana 的进程 ID 文件的路径。
logging.dest:
默认值: stdout 指定 Kibana 日志输出的文件。
logging.silent:
默认值: false 该值设为 true 时，禁止所有日志输出。
logging.quiet:
默认值: false 该值设为 true 时，禁止除错误信息除外的所有日志输出。
logging.verbose
默认值: false 该值设为 true 时，记下所有事件包括系统使用信息和所有请求的日志。
ops.interval
默认值: 5000 设置系统和进程取样间隔，单位微妙，最小值100。
status.allowAnonymous
默认值: false 如果启用了权限，该项设置为 true 即允许所有非授权用户访问 Kibana 服务端 API 和状态页面。
cpu.cgroup.path.override
如果挂载点跟 /proc/self/cgroup 不一致，覆盖 cgroup cpu 路径。
cpuacct.cgroup.path.override
如果挂载点跟 /proc/self/cgroup 不一致，覆盖 cgroup cpuacct 路径。
console.enabled
默认值: true 设为 false 来禁用控制台，切换该值后服务端下次启动时会重新生成资源文件，因此会导致页面服务有点延迟。
elasticsearch.tribe.url:
Elasticsearch tribe 实例的 URL，用于所有查询。
elasticsearch.tribe.username: 和 elasticsearch.tribe.password:
Elasticsearch 设置了基本的权限认证，该配置项提供了用户名和密码，用于 Kibana 启动时维护索引。Kibana 用户仍需要 Elasticsearch 由 Kibana 服务端代理的认证。
elasticsearch.tribe.ssl.certificate: 和 elasticsearch.tribe.ssl.key:
可选配置项，提供 PEM 格式 SSL 证书和密钥文件的路径。这些文件确保 Elasticsearch 后端使用同样的密钥文件。
elasticsearch.tribe.ssl.keyPassphrase
解密私钥的口令，该设置项可选，因为密钥可能没有加密。
elasticsearch.tribe.ssl.certificateAuthorities:
指定用于 Elasticsearch tribe 实例的 PEM 证书文件路径。
elasticsearch.tribe.ssl.verificationMode:
默认值: full 控制证书的认证，可用的值有 none 、 certificate 、 full 。 full 执行主机名验证， certificate 不执行主机名验证。
elasticsearch.tribe.pingTimeout:
默认值: elasticsearch.tribe.requestTimeout setting 的值，等待 Elasticsearch 的响应时间。
elasticsearch.tribe.requestTimeout:
Default: 30000 等待后端或 Elasticsearch 的响应时间，单位微秒，该值必须为正整数。
elasticsearch.tribe.requestHeadersWhitelist:
默认值: [ 'authorization' ] Kibana 发往 Elasticsearch 的客户端头体，发送 no 头体，设置该值为[]。
elasticsearch.tribe.customHeaders:
默认值: {} 发往 Elasticsearch的头体和值，不管 elasticsearch.tribe.requestHeadersWhitelist 如何配置，任何自定义的头体不会被客户端头体覆盖。
```

访问kibana

Kibana是一个web应用，可以通过5601端口访问。只需要在浏览器中指定Kibana运行的机器，然后指定端口号即可。例如，`localhost:5601`或者`http://YOURDOMAIN.com:5601`。

当访问Kibana时，[发现页面](https://www.elastic.co/guide/cn/kibana/current/discover.html)默认会加载默认的索引模式。时间过滤器设置的时间为过去15分钟，查询设置为匹配所有（\ \*）。

如果看不到任何文档，试着把时间过滤器的范围调大。如果还是看不到任何结果，可能很根本的英文就**没有**任何文档。

#### 检查Kibana状态[编辑](https://github.com/elasticsearch-cn/kibana/edit/cn/docs/setup/access.asciidoc)

您可以通过`localhost:5601/status`来访问Kibana的服务器状态页，状态页展示了服务器资源使用情况和已安装插件列表。

![&#x56FE;&#x7247;/ kibana&#x72B6;&#x6001;&#xFF0C;page.png](https://www.elastic.co/guide/cn/kibana/current/images/kibana-status-page.png)

##  连接 Kibana 和 Elasticsearch



开始使用 Kibana 前，需要告诉 Kibana 您想要探索的 Elasticsearch 索引。第一次访问 Kibana 时，会提示您定义一个 _index pattern\(索引模式\)_ 匹配一个或多个索引。这就是初次使用 Kibana 时所有需要配置的。任何时候都可以在 [Management 页面](https://www.elastic.co/guide/cn/kibana/current/index-patterns.html#settings-create-pattern)增加索引模式。

{% hint style="info" %}
默认情况下，Kibana 会连接运行在 `localhost` 上的 Elasticsearch 实例。如果需要连接不同的 Elasticsearch实例，可以修改 `kibana.yml` 配置文件中的 Elasticsearch URL 配置项并重启 Kibana。如果在生产环境节点上使用 Kibana，请参考 [_生产环境中使用 Kibana_](https://www.elastic.co/guide/cn/kibana/current/production.html) 。
{% endhint %}

设置您想通过 Kibana 访问的 Elasticsearch 索引：

1. 浏览器中指定端口号5601来访问 Kibana UI 页面。例如， `localhost:5601` 或者 `http://YOURDOMAIN.com:5601` 。

![](../../.gitbook/assets/image%20%2856%29.png)



  2.指定一个索引模式来匹配一个或多个 Elasticsearch 索引名称。默认情况下，Kibana 会认为数据是通过 Logstash 解析送进 Elasticsearch 的。这种情况可以使用默认的 `logstash-*` 作为索引模式。星号 \(\*\) 匹配0或多个索引名称中的字符。如果 Elasticsearch 索引遵循其他命名约定，请输入一个恰当的模式。该模式也可以直接用单个索引的名称。

 3.如果您想做一些基于时间序列的数据比较，可以选择索引中包含时间戳的字段。Kibana 会读取索引映射，列出包含时间戳的所有字段。如果索引中没有基于时间序列的数据，则禁用 **Index contains time-based events** 选项。

{% hint style="info" %}
使用事件时间来创建索引名称在这个版本的 Kibana 中会被标记成 **deprecated** 。这个功能将在下一个 Kibana 的主要版本中被彻移除。Elasticsearch 2.1 拥有强大的日期解析 API 用于分割日期信息，没必要在索引模式名称中指定日期。
{% endhint %}

 4.点击 **Create** 增加索引模式。默认情况下，第一个模式被自动配置为默认的。当索引模式不止一个时，可以通过点击 **Management &gt; Index Patterns** 索引模式题目上的星星图标来指定默认的索引模式。

全部设置完毕！Kibana 连接了 Elasticsearch 的数据。展示了一个匹配到的索引的字段只读列表。

{% hint style="info" %}
Kibana 可视化控件中的字段以及 `.kibana` 索引管理依赖动态映射（dynamic mapping）功能，如果动态映射被关闭，您需要手动提供字段的映射以便于 Kibana 用于正确的创建可视化控件。更多信息，请参考 [Kibana 和 Elasticsearch 动态映射](https://www.elastic.co/guide/cn/kibana/current/connect-to-elasticsearch.html#kibana-dynamic-mapping) 。
{% endhint %}

#### 开始探索您的数据！

准备挖掘您的数据：

* 在 [Discover](https://www.elastic.co/guide/cn/kibana/current/discover.html) 页搜索和浏览数据。
* 在 [Visualize](https://www.elastic.co/guide/cn/kibana/current/visualize.html) 页做数据图表映射。
* 在 [Dashboard](https://www.elastic.co/guide/cn/kibana/current/dashboard.html) 页创建并查看自定义仪表板。

您可以通过查看 [基础入门](https://www.elastic.co/guide/cn/kibana/current/getting-started.html)来获取 Kibana 的主要内容介绍。

#### Kibana 和 Elasticsearch 动态映射

默认情况下，Elasticsearch 启用了字段的 [动态映射](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/dynamic-mapping.html)。Kibana 需要利用动态映射在可视化控件中正确使用字段，同时管理 `.kibana` 索引，这些索引存储了已保存的搜索、可视化图表和仪表板。

如果 Elasticsearch 使用场景需要禁止动态映射，这就需要手动提供创建可视化控件所需要字段的映射，并手动启用 `.kibana` 索引的动态映射。

下面的步骤假定 Elasticsearch 中不存在索引 `.kibana` ，且 `elasticsearch.yml` 中 `index.mapper.dynamic`配置为 `false` ：

1. 启动 Elasticsearch。
2. 创建 `.kibana` 索引并只针对该索引打开动态映射：

   ```text
   PUT .kibana
   {
     "index.mapper.dynamic": true
   }
   ```

3. 启动 Kibana ，转到 web 页面，确认没有动态映射相关的错误。

##  生产环境中使用Kibana

### 在kibana中使用X-Pack

使用[X-Pack安全模块](https://www.elastic.co/guide/en/x-pack/6.0/xpack-security.html)控制用户通过Kibana可以访问哪些Elasticsearch数据。

当安装X-Pack时，Kibana用户必须登陆。这些用户需要用`kibana_user`角色来访问通过Kibana管理的索引库。

如果一个用户要通过Kibana仪表板访问某个索引库中的数据，但没有权限查看，他们会收到一个错误提示：索引不存在.X-Pack的安全机制目前还不能提供一种方法来控制不同的用户加载不同的仪表板。

关于设置Kibana用户和X-Pack信息，请参见[X-Pack安全插件](https://www.elastic.co/guide/en/x-pack/6.0/xpack-security.html)

### 多个Elasticsearch节点间的负载均衡[编辑](https://github.com/elasticsearch-cn/kibana/edit/cn/docs/setup/production.asciidoc)

如果Elasticsearch集群有多个节点，分发Kibana节点之间请求的最简单的方法就是在Kibana机器上运行一个Elasticsearch _协调（仅协调节点）_的节点.Elasticsearch协调节点本质上是智能负载均衡器，也是集群的一部分，如果有需要，这些节点会处理传入HTTP请求，重定向操作给集群中其它节点，收集并返回结果。更多详细信息，请参考[节点](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/modules-node.html)部分。

使用本地客户端节点负载均衡Kibana请求：

1. 在安装Kibana的机器上安装Elasticsearch。
2.  配置节点为协调节点。在配置文件 `elasticsearch.yml` 中，设置 `node.data` 、 `node.master` 和 `node.ingest` 为 `false` ：

   ```text
   ＃3。您希望此节点既不是主节点，也不是数据节点，也不是摄取节点，但是
   ＃充当“搜索负载均衡器”（从节点获取数据，
   ＃聚合结果等）
   ＃
   node.master：false
   node.data：false
   node.ingest：false
   ```

3. 设置客户端节点接入Elasticsearch集群。在配置文件`elasticsearch.yml`中，通过`cluster.name`选项设定集群名字。

   ```text
   cluster.name：“my_cluster”
   ```

4.  检查 `elasticsearch.yml` 中的 transport 和 HTTP 主机配置： `network.host` 和 `transport.host`。`transport.host` 需要为集群中其它成员网络可达， `network.host` 是访问 Kibana 的 HTTP 网络连接\(默认为 localhost:9200 \)。

   ```text
   network.host：localhost
   http：运行：9200

   ＃默认情况下transport.host是指network.host
   transport.host：<external ip>
   transport.tcp.port:9300-9400
   ```

5. 请确认Kibana设置为指向本地客户端节点。在配置文件`kibana.yml`中，`elasticsearch.url`应设为`localhost:9200`。

   ```text
   ＃用于所有查询的Elasticsearch实例。
   elasticsearch.url：“http：// localhost：9200”
   ```





