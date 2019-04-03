# StatefulSet

## StatefulSet <a id="statefulset"></a>

StatefulSet 作为 Controller 为 Pod 提供唯一的标识。它可以保证部署和 scale 的顺序。

使用案例参考：[kubernetes contrib - statefulsets](https://github.com/kubernetes/contrib/tree/master/statefulsets)，其中包含zookeeper和kakfa的statefulset设置和使用说明。

StatefulSet是为了解决有状态服务的问题（对应Deployments和ReplicaSets是为无状态服务而设计），其应用场景包括：

* 稳定的持久化存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
* 稳定的网络标志，即Pod重新调度后其PodName和HostName不变，基于Headless Service（即没有Cluster IP的Service）来实现
* 有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从0到N-1，在下一个Pod运行之前所有之前的Pod必须都是Running和Ready状态），基于init containers来实现
* 有序收缩，有序删除（即从N-1到0）

从上面的应用场景可以发现，StatefulSet由以下几个部分组成：

* 用于定义网络标志（DNS domain）的Headless Service
* 用于创建PersistentVolumes的volumeClaimTemplates
* 定义具体应用的StatefulSet

StatefulSet中每个Pod的DNS格式为`statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local`，其中

* `serviceName`为Headless Service的名字
* `0..N-1`为Pod所在的序号，从0开始到N-1
* `statefulSetName`为StatefulSet的名字
* `namespace`为服务所在的namespace，Headless Servic和StatefulSet必须在相同的namespace
* `.cluster.local`为Cluster Domain

### 使用 StatefulSet <a id="&#x4F7F;&#x7528;-statefulset"></a>

StatefulSet 适用于有以下某个或多个需求的应用：

* 稳定，唯一的网络标志。
* 稳定，持久化存储。
* 有序，优雅地部署和 scale。
* 有序，优雅地删除和终止。
* 有序，自动的滚动升级。

在上文中，稳定是 Pod （重新）调度中持久性的代名词。 如果应用程序不需要任何稳定的标识符、有序部署、删除和 scale，则应该使用提供一组无状态副本的 controller 来部署应用程序，例如 [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment) 或 [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset) 可能更适合您的无状态需求

### 限制 <a id="&#x9650;&#x5236;"></a>

* StatefulSet 是 beta 资源，Kubernetes 1.5 以前版本不支持。
* 对于所有的 alpha/beta 的资源，您都可以通过在 apiserver 中设置 `--runtime-config` 选项来禁用。
* 给定 Pod 的存储必须由 PersistentVolume Provisioner 根据请求的 `storage class` 进行配置，或由管理员预先配置。
* 删除或 scale StatefulSet 将_不会_删除与 StatefulSet 相关联的 volume。 这样做是为了确保数据安全性，这通常比自动清除所有相关 StatefulSet 资源更有价值。
* StatefulSets 目前要求 [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) 负责 Pod 的网络身份。 您有责任创建此服务。

### 组件 <a id="&#x7EC4;&#x4EF6;"></a>

下面的示例中描述了 StatefulSet 中的组件。

* 一个名为 nginx 的 headless service，用于控制网络域。
* 一个名为 web 的 StatefulSet，它的 Spec 中指定在有 3 个运行 nginx 容器的 Pod。
* volumeClaimTemplates 使用 PersistentVolume Provisioner 提供的 [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/volumes) 作为稳定存储。

```text
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
      annotations:
        volume.beta.kubernetes.io/storage-class: anything
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

```text
$ kubectl create -f web.yaml
service "nginx" created
statefulset "web" created
​
# 查看创建的 headless service 和 statefulset
$ kubectl get service nginx
NAME      CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     None         <none>        80/TCP    1m
$ kubectl get statefulset web
NAME      DESIRED   CURRENT   AGE
web       2         2         2m
​
# 根据 volumeClaimTemplates 自动创建 PVC（在 GCE 中会自动创建 kubernetes.io/gce-pd 类型的 volume）
$ kubectl get pvc
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-d064a004-d8d4-11e6-b521-42010a800002   1Gi        RWO           16s
www-web-1   Bound     pvc-d06a3946-d8d4-11e6-b521-42010a800002   1Gi        RWO           16s
​
# 查看创建的 Pod，他们都是有序的
$ kubectl get pods -l app=nginx
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          5m
web-1     1/1       Running   0          4m
​
# 使用 nslookup 查看这些 Pod 的 DNS
$ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh
/ # nslookup web-0.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
​
Name:      web-0.nginx
Address 1: 10.244.2.10
/ # nslookup web-1.nginx
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
​
Name:      web-1.nginx
Address 1: 10.244.3.12
/ # nslookup web-0.nginx.default.svc.cluster.local
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local
​
Name:      web-0.nginx.default.svc.cluster.local
Address 1: 10.244.2.10
```

 还可以进行其他的操作

```text
# 扩容
$ kubectl scale statefulset web --replicas=5
​
# 缩容
$ kubectl patch statefulset web -p '{"spec":{"replicas":3}}'
​
# 镜像更新（目前还不支持直接更新 image，需要 patch 来间接实现）
$ kubectl patch statefulset web --type='json' -p='[{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"gcr.io/google_containers/nginx-slim:0.7"}]'
​
# 删除 StatefulSet 和 Headless Service
$ kubectl delete statefulset web
$ kubectl delete service nginx
​
# StatefulSet 删除后 PVC 还会保留着，数据不再使用的话也需要删除
$ kubectl delete pvc www-web-0 www-web-1
```

###  <a id="pod-&#x8EAB;&#x4EFD;"></a>

### Pod 身份 <a id="pod-&#x8EAB;&#x4EFD;"></a>

StatefulSet Pod 具有唯一的身份，包括序数，稳定的网络身份和稳定的存储。 身份绑定到 Pod 上，不管它（重新）调度到哪个节点上。

#### 序数 <a id="&#x5E8F;&#x6570;"></a>

对于一个有 N 个副本的 StatefulSet，每个副本都会被指定一个整数序数，在 \[0,N\)之间，且唯一。

### 稳定的网络 ID <a id="&#x7A33;&#x5B9A;&#x7684;&#x7F51;&#x7EDC;-id"></a>

StatefulSet 中的每个 Pod 从 StatefulSet 的名称和 Pod 的序数派生其主机名。构造的主机名的模式是`$（statefulset名称)-$(序数)`。 上面的例子将创建三个名为`web-0，web-1，web-2`的 Pod。

StatefulSet 可以使用 [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) 来控制其 Pod 的域。此服务管理的域的格式为：`$(服务名称).$(namespace).svc.cluster.local`，其中 “cluster.local” 是集群域。

在创建每个Pod时，它将获取一个匹配的 DNS 子域，采用以下形式：`$(pod 名称).$(管理服务域)`，其中管理服务由 StatefulSet 上的 `serviceName` 字段定义。

以下是 Cluster Domain，服务名称，StatefulSet 名称以及如何影响 StatefulSet 的 Pod 的 DNS 名称的一些示例。

| Cluster Domain | Service \(ns/name\) | StatefulSet \(ns/name\) | StatefulSet Domain | Pod DNS | Pod Hostname |
| :--- | :--- | :--- | :--- | :--- | :--- |
| cluster.local | default/nginx | default/web | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local | foo/nginx | foo/web | nginx.foo.svc.cluster.local | web-{0..N-1}.nginx.foo.svc.cluster.local | web-{0..N-1} |
| kube.local | foo/nginx | foo/web | nginx.foo.svc.kube.local | web-{0..N-1}.nginx.foo.svc.kube.local | web-{0..N-1} |

注意 Cluster Domain 将被设置成 `cluster.local` 除非进行了其他配置。

#### 稳定存储 <a id="&#x7A33;&#x5B9A;&#x5B58;&#x50A8;"></a>

Kubernetes 为每个 VolumeClaimTemplate 创建一个 [PersistentVolume](https://kubernetes.io/docs/concepts/storage/volumes)。上面的 nginx 的例子中，每个 Pod 将具有一个由 `anything` 存储类创建的 1 GB 存储的 PersistentVolume。当该 Pod （重新）调度到节点上，`volumeMounts` 将挂载与 PersistentVolume Claim 相关联的 PersistentVolume。请注意，与 PersistentVolume Claim 相关联的 PersistentVolume 在 产出 Pod 或 StatefulSet 的时候不会被删除。这必须手动完成。

### 部署和 Scale 保证 <a id="&#x90E8;&#x7F72;&#x548C;-scale-&#x4FDD;&#x8BC1;"></a>

* 对于有 N 个副本的 StatefulSet，Pod 将按照 {0..N-1} 的顺序被创建和部署。
* 当 删除 Pod 的时候，将按照逆序来终结，从{N-1..0}
* 对 Pod 执行 scale 操作之前，它所有的前任必须处于 Running 和 Ready 状态。
* 在终止 Pod 前，它所有的继任者必须处于完全关闭状态。

不应该将 StatefulSet 的 `pod.Spec.TerminationGracePeriodSeconds` 设置为 0。这样是不安全的且强烈不建议您这样做。进一步解释，请参阅 [强制删除 StatefulSet Pod](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod)。

上面的 nginx 示例创建后，3 个 Pod 将按照如下顺序创建 web-0，web-1，web-2。在 web-0 处于 [运行并就绪](https://kubernetes.io/docs/user-guide/pod-states) 状态之前，web-1 将不会被部署，同样当 web-1 处于运行并就绪状态之前 web-2也不会被部署。如果在 web-1 运行并就绪后，web-2 启动之前， web-0 失败了，web-2 将不会启动，直到 web-0 成功重启并处于运行并就绪状态。

如果用户通过修补 StatefulSet 来 scale 部署的示例，以使 `replicas=1`，则 web-2 将首先被终止。 在 web-2 完全关闭和删除之前，web-1 不会被终止。 如果 web-0 在 web-2 终止并且完全关闭之后，但是在 web-1 终止之前失败，则 web-1 将不会终止，除非 web-0 正在运行并准备就绪。



## 更新 StatefulSet <a id="geng-xin-statefulset"></a>

v1.7 + 支持 StatefulSet 的自动更新，通过 `spec.updateStrategy` 设置更新策略。目前支持两种策略

* OnDelete：当 `.spec.template` 更新时，并不立即删除旧的 Pod，而是等待用户手动删除这些旧 Pod 后自动创建新 Pod。这是默认的更新策略，兼容 v1.6 版本的行为
* RollingUpdate：当 `.spec.template` 更新时，自动删除旧的 Pod 并创建新 Pod 替换。在更新时，这些 Pod 是按逆序的方式进行，依次删除、创建并等待 Pod 变成 Ready 状态才进行下一个 Pod 的更新。

### Partitions <a id="partitions"></a>

RollingUpdate 还支持 Partitions，通过 `.spec.updateStrategy.rollingUpdate.partition` 来设置。当 partition 设置后，只有序号大于或等于 partition 的 Pod 会在 `.spec.template` 更新的时候滚动更新，而其余的 Pod 则保持不变（即便是删除后也是用以前的版本重新创建）。

```text
# 设置 partition 为 3$ kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":3}}}}'statefulset "web" patched​# 更新 StatefulSet$ kubectl patch statefulset web --type='json' -p='[{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"gcr.io/google_containers/nginx-slim:0.7"}]'statefulset "web" patched​# 验证更新$ kubectl delete po web-2pod "web-2" deleted$ kubectl get po -lapp=nginx -wNAME      READY     STATUS              RESTARTS   AGEweb-0     1/1       Running             0          4mweb-1     1/1       Running             0          4mweb-2     0/1       ContainerCreating   0          11sweb-2     1/1       Running             0          18s
```

## Pod 管理策略 <a id="pod-guan-li-ce-lve"></a>

v1.7 + 可以通过 `.spec.podManagementPolicy` 设置 Pod 管理策略，支持两种方式

* OrderedReady：默认的策略，按照 Pod 的次序依次创建每个 Pod 并等待 Ready 之后才创建后面的 Pod
* Parallel：并行创建或删除 Pod（不等待前面的 Pod Ready 就开始创建所有的 Pod）

### Parallel 示例 <a id="parallel-shi-li"></a>

```text
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  podManagementPolicy: "Parallel"
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

 可以看到，所有 Pod 是并行创建的

```text
$ kubectl create -f webp.yaml
service "nginx" created
statefulset "web" created
​
$ kubectl get po -lapp=nginx -w
NAME      READY     STATUS              RESTARTS  AGE
web-0     0/1       Pending             0         0s
web-0     0/1       Pending             0         0s
web-1     0/1       Pending             0         0s
web-1     0/1       Pending             0         0s
web-0     0/1       ContainerCreating   0         0s
web-1     0/1       ContainerCreating   0         0s
web-0     1/1       Running             0         10s
web-1     1/1       Running             0         10s
```



参考：

{% embed url="https://rootsongjc.gitbooks.io/kubernetes-handbook/concepts/statefulset.html" %}

