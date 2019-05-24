# Label

Label机制是K8S中一个重要设计，通过Label进行对象弱关联，灵活地分类和选择不同服务或业务，让用户根据自己特定的组织结构以松耦合方式进行服务部署

## 命令操作

### 1.创建label

```text
kubectl label nodes <node-name> <label-key>=<label-value>
例如： kubectl label no 10.130.14.67 tester=chenqiang   #打标签 tester=chenqiang
```

### 2.查看labels

```text
kubectl get nodes -Lsystem/build-node  列表

kubectl get nodes --show-labels  //# 查看所有标签
```

### 3.删除

```text
例如：kubectl label no 10.130.14.67 tester-  #删除之前打过的标签 tester=chenqiang
```

删除一个Label，只需在命令行最后指定Label的key名并与一个减号相连即可

\# 通过特定标签进行过滤节点

```text
kubectl get no -l tester=chenqiang
```

修改一个Label的值，需要加上--overwrite参数： kubectl label pod redis-master-bobr0 role=master –overwrite

Label是附着到object上（例如Pod）的键值对。可以在创建object的时候指定，也可以在object创建后随时指定。Labels的值对系统本身并没有什么含义，只是对用户才有意义。

```text
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

Kubernetes最终将对labels最终索引和反向索引用来优化查询和watch，在UI和命令行中会对它们排序。不要在label中使用大型、非标识的结构化数据，记录这样的数据应该用annotation。

## 动机

Label能够将组织架构映射到系统架构上（就像是康威定律），这样能够更便于微服务的管理，你可以给object打上如下类型的label：

* `"release" : "stable"`, `"release" : "canary"`
* `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
* `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
* `"partition" : "customerA"`, `"partition" : "customerB"`
* `"track" : "daily"`, `"track" : "weekly"`
* `"team" : "teamA"`,`"team:" : "teamB"`

## 语法和字符集

Label key的组成：

* 不得超过63个字符
* 可以使用前缀，使用/分隔，前缀必须是DNS子域，不得超过253个字符，系统中的自动化组件创建的label必须指定前缀，`kubernetes.io/`由kubernetes保留
* 起始必须是字母（大小写都可以）或数字，中间可以有连字符、下划线和点

Label value的组成：

* 不得超过63个字符
* 起始必须是字母（大小写都可以）或数字，中间可以有连字符、下划线和点

## Label selector

Label不是唯一的，很多object可能有相同的label。

通过label selector，客户端／用户可以指定一个object集合，通过label selector对object的集合进行操作。

 标签选择器可以由逗号分隔的多个requirements 组成。在多重需求的情况下，必须满足所有要求，因此逗号分隔符作为AND逻辑运算符。

Label selector有两种类型：

* _equality-based_ ：可以使用`=`、`==`、`!=`操作符，可以使用逗号分隔多个表达式
* _set-based_ ：可以使用`in`、`notin`、`!`操作符，另外还可以没有操作符，直接写出某个label的key，表示过滤有某个key的object而不管该key的value是何值，`!`表示没有该label的object

 equality-based（基于平等）和set-based（基于集合）的。前者采用等式的表达式，后者采用集合操作的表达式匹配标签。

### 示例 <a id="&#x793A;&#x4F8B;"></a>

```text
$ kubectl get pods -l environment=production,tier=frontend
$ kubectl get pods -l 'environment in (production),tier in (frontend)'
$ kubectl get pods -l 'environment in (production, qa)'
$ kubectl get pods -l 'environment,environment notin (frontend)'
```

 RC只支持基于等式的Selector，而RS两种Selector都支持

Label Selector set-based筛选功能仅在以下新出现的管理对象中得到了支持：

 Deployment

 ReplicaSet

 DaemonSet

 Job 

Label Selector的几个重要使用场景： 

kube-controller进程，通过资源对象RC上定义的Label Selector来筛选要监控和管理的Pod副本的数量。

 kube-proxy进程，通过Service的Lable Selector来选择对应的Pod，建立起每个Service到对应Pod的请求转发路由表，进而实现Service的智能负载均衡机制。

kube-scheduler进程，通过为某些Node定义特定的Label，然后在Pod定义文件中使用NodeSelector进行筛选，进而实现Pod的定向调度功能！

## 在API object中设置label selector

在`service`、`replicationcontroller`等object中有对pod的label selector，使用方法只能使用等于操作，例如：

```text
selector:
    component: redis
```

在`Job`、`Deployment`、`ReplicaSet`和`DaemonSet`这些object中，支持_set-based_的过滤，例如：

```text
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

如Service通过label selector将同一类型的pod作为一个服务expose出来

![](../../.gitbook/assets/image%20%28100%29.png)

图片 - label示意图

另外在node affinity和pod affinity中的label selector的语法又有些许不同，示例如下：

```text
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
```



Label的最常见的用法便是通过**spec.selector**来引用对象。下面是文章：[Kubernetes对象之ReplicaSet](https://www.jianshu.com/p/fd8d8d51741e) 中新建一个RC的例子：

```text
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

关于Label的用法重点在于这两步：

* 通过**template.metadata.labels**字段为即将新建的Pod附加Label。在上面的例子中，新建了一个名称为nginx的Pod，它拥有一个键值对为`app:nginx`的Label。
* 通过**spec.selector**字段来指定这个RC管理哪些Pod。在上面的例子中，新建的RC会管理所有拥有`app:nginx`Label的Pod。这样的**spec.selector**在Kubernetes中被称作**Label Selector**。

  
  
文章出处：  
链接：https://www.jianshu.com/p/cd6b4b4caaab  


