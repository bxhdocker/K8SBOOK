# 应用日志收集

### 前言 <a id="&#x524D;&#x8A00;"></a>

在进行日志收集的过程中，我们首先想到的是使用Logstash，因为它是ELK stack中的重要成员，但是在测试过程中发现，Logstash是基于JDK的，在没有产生日志的情况单纯启动Logstash就大概要消耗**500M**内存，在每个Pod中都启动一个日志收集组件的情况下，使用logstash有点浪费系统资源.

日志采集的工具有很多种，如fluentd, flume, logstash,betas等等。我们选择使用**Filebeat**替代，经测试单独启动Filebeat容器大约会消耗**12M**内存，比起logstash相当轻量级。常用的ELK日志采集方案中，大部分的做法就是将所有节点的日志内容通过filebeat送到kafka消息队列，然后使用logstash集群读取消息队列内容，根据配置文件进行过滤。然后将过滤之后的文件输送到elasticsearch中，通过kibana去展示。

### 方案选择 <a id="&#x65B9;&#x6848;&#x9009;&#x62E9;"></a>

Kubernetes官方提供了EFK\(es+fluentd+kibana\)的日志收集解决方案，但是这种方案并不适合所有的业务场景，它本身就有一些局限性，例如：

* 所有日志都必须是out前台输出，真实业务场景中无法保证所有日志都在前台输出
* 只能有一个日志输出文件，而真实业务场景中往往有多个日志输出文件
* Fluentd并不是常用的日志收集工具，我们更习惯用logstash，现使用filebeat替代
* 我们已经有自己的ELK集群且有专人维护，没有必要再在kubernetes上做一个日志收集服务

基于以上几个原因，我们决定使用自己的ELK集群。

**Kubernetes集群中的日志收集解决方案**

| **编号** | **方案** | **优点** | **缺点** |
| :--- | :--- | :--- | :--- |
| **1** | 每个app的镜像中都集成日志收集组件 | 部署方便，kubernetes的yaml文件无须特别配置，可以为每个app自定义日志收集配置 | 强耦合，不方便应用和日志收集组件升级和维护且会导致镜像过大 |
| **2** | 单独创建一个日志收集组件跟app的容器一起运行在同一个pod中 | 低耦合，扩展性强，方便维护和升级 | 需要对kubernetes的yaml文件进行单独配置，略显繁琐 |
| **3** | 将所有的Pod的日志都挂载到宿主机上，每台主机上单独起一个日志收集Pod | 完全解耦，性能最高，管理起来最方便 | 需要统一日志收集规则，目录和输出方式 |

综合以上优缺点，我们选择使用方案二。

该方案在扩展性、个性化、部署和后期维护方面都能做到均衡，因此选择该方案

![&#x56FE;&#x7247; - filebeat&#x65E5;&#x5FD7;&#x6536;&#x96C6;&#x67B6;&#x6784;&#x56FE;](../../.gitbook/assets/image.png)

我们创建了自己的filebeat镜像。创建过程和使用方式见[https://github.com/rootsongjc/docker-images](https://github.com/rootsongjc/docker-images)

镜像地址：`index.tenxcloud.com/jimmy/filebeat:5.4.0`

### 测试 <a id="&#x6D4B;&#x8BD5;"></a>

我们部署一个应用filebeat来收集日志的功能测试。

创建应用yaml文件`filebeat-test.yaml`。

```text
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: filebeat-test
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
      labels:
        k8s-app: filebeat-test
    spec:
      containers:
      - image: harbor-001.jimmysong.io/library/filebeat:5.4.0
        name: filebeat
        volumeMounts:
        - name: app-logs
          mountPath: /log
        - name: filebeat-config
          mountPath: /etc/filebeat/
      - image: harbor-001.jimmysong.io/library/analytics-docker-test:Build_8
        name : app
        ports:
        - containerPort: 80
        volumeMounts:
        - name: app-logs
          mountPath: /usr/local/TalkingData/logs
      volumes:
      - name: app-logs
        emptyDir: {}
      - name: filebeat-config
        configMap:
          name: filebeat-config
---
apiVersion: v1
kind: Service
metadata:
  name: filebeat-test
  labels:
    app: filebeat-test
spec:
  ports:
  - port: 80
    protocol: TCP
    name: http
  selector:
    run: filebeat-test
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
data:
  filebeat.yml: |
    filebeat.prospectors:
    - input_type: log
      paths:
        - "/log/*"
        - "/log/usermange/common/*"
    output.elasticsearch:
      hosts: ["172.23.5.255:9200"]
      username: "elastic"
      password: "changeme"
      index: "filebeat-docker-test"
```

**说明**

该文件中包含了配置文件filebeat的配置文件的[ConfigMap](https://jimmysong.io/posts/kubernetes-configmap-introduction/)，因此不需要再定义环境变量。

当然你也可以不同ConfigMap，通过传统的传递环境变量的方式来配置filebeat。

例如对filebeat的容器进行如下配置：

```text
      containers:
      - image: harbor-001.jimmysong.io/library/filebeat:5.4.0
        name: filebeat
        volumeMounts:
        - name: app-logs
          mountPath: /log
        env: 
        - name: PATHS
          value: "/log/*"
        - name: ES_SERVER
          value: 172.23.5.255:9200
        - name: INDEX
          value: logstash-docker
        - name: INPUT_TYPE
          value: log
```

目前使用这种方式会有个问题，及时`PATHS`只能传递单个目录，如果想传递多个目录需要修改filebeat镜像的`docker-entrypoint.sh`脚本，对该环境变量进行解析增加filebeat.yml文件中的PATHS列表。

推荐使用ConfigMap，这样filebeat的配置就能够更灵活。

**注意事项**

* 将app的`/usr/local/TalkingData/logs`目录挂载到filebeat的`/log`目录下。
* 该文件可以在`manifests/test/filebeat-test.yaml`找到。
* 我使用了自己的私有镜像仓库，测试时请换成自己的应用镜像。
* Filebeat的环境变量的值配置请参考[https://github.com/rootsongjc/docker-images](https://github.com/rootsongjc/docker-images)

**创建应用**

部署Deployment

```text
kubectl create -f filebeat-test.yaml
```

查看`http://172.23.5.255:9200/_cat/indices`将可以看到列表有这样的indices：

```text
green open filebeat-docker-test            7xPEwEbUQRirk8oDX36gAA 5 1   2151     0   1.6mb 841.8kb
```

访问Kibana的web页面，查看`filebeat-2017.05.17`的索引，可以看到filebeat收集到了app日志

![&#x56FE;&#x7247; - Kibana&#x9875;&#x9762;](../../.gitbook/assets/image%20%28125%29.png)

点开每个日志条目，可以看到

以下详细字段

![&#x56FE;&#x7247; - filebeat&#x6536;&#x96C6;&#x7684;&#x65E5;&#x5FD7;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;](../../.gitbook/assets/image%20%28146%29.png)

* `_index`值即我们在YAML文件的`configMap`中配置的index值
* `beat.hostname`和`beat.name`即pod的名称
* source表示filebeat容器中的日志目录

我们可以通过人为得使`index` = `service name`，这样就可以方便的收集和查看每个service的日志。

