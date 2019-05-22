# pod其他设置

###    使用 Volume

Volume 可以为容器提供持久化存储，比如

```text
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

更多挂载存储卷的方法参考 [Volume](https://github.com/feiskyer/kubernetes-handbook/blob/master/concepts/volume.md)。

### 私有镜像

在使用私有镜像时，需要创建一个 docker registry secret，并在容器中引用。

创建 docker registry secret：

```text
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

比如使用 Azure Container Registry（ACR）：

```text
ACR_NAME=dregistry
SERVICE_PRINCIPAL_NAME=acr-service-principal

# Populate the ACR login server and resource id.
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

# Create a contributor role assignment with a scope of the ACR resource.
SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --role Reader --scopes $ACR_REGISTRY_ID --query password --output tsv)

# Get the service principle client id.
CLIENT_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

# Create secret
kubectl create secret docker-registry acr-auth --docker-server $ACR_LOGIN_SERVER --docker-username $CLIENT_ID --docker-password $SP_PASSWD --docker-email local@local.domain
```

容器中引用该 secret：

```text
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
    - name: private-reg-container
      image: dregistry.azurecr.io/acr-auth-example
  imagePullSecrets:
    - name: acr-auth
```

### RestartPolicy

支持三种 RestartPolicy

* Always：只要退出就重启
* OnFailure：失败退出（exit code 不等于 0）时重启
* Never：只要退出就不再重启

注意，这里的重启是指在 Pod 所在 Node 上面本地重启，并不会调度到其他 Node 上去。

### 环境变量

环境变量为容器提供了一些重要的资源，包括容器和 Pod 的基本信息以及集群中服务的信息等：

\(1\) hostname

`HOSTNAME` 环境变量保存了该 Pod 的 hostname。

（2）容器和 Pod 的基本信息

Pod 的名字、命名空间、IP 以及容器的计算资源限制等可以以 [Downward API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/) 的方式获取并存储到环境变量中。

```text
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: ["sh", "-c"]
      args:
      - env
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
  restartPolicy: Never
```

\(3\) 集群中服务的信息

容器的环境变量中还可以引用容器运行前创建的所有服务的信息，比如默认的 kubernetes 服务对应以下环境变量：

```text
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
```

由于环境变量存在创建顺序的局限性（环境变量中不包含后来创建的服务），推荐使用 [DNS](https://github.com/feiskyer/kubernetes-handbook/blob/master/components/kube-dns.md) 来解析服务。

### 镜像拉取策略

支持三种 ImagePullPolicy

* Always：不管镜像是否存在都会进行一次拉取
* Never：不管镜像是否存在都不会进行拉取
* IfNotPresent：只有镜像不存在时，才会进行镜像拉取

注意：

* 默认为 `IfNotPresent`，但 `:latest` 标签的镜像默认为 `Always`。
* 拉取镜像时 docker 会进行校验，如果镜像中的 MD5 码没有变，则不会拉取镜像数据。
* 生产环境中应该尽量避免使用 `:latest` 标签，而开发环境中可以借助 `:latest` 标签自动拉取最新的镜像。

### 访问 DNS 的策略

通过设置 dnsPolicy 参数，设置 Pod 中容器访问 DNS 的策略

* ClusterFirst：优先基于 cluster domain （如 `default.svc.cluster.local`） 后缀，通过 kube-dns 查询 \(默认策略\)
* Default：优先从 Node 中配置的 DNS 查询

### 使用主机的 IPC 命名空间

通过设置 `spec.hostIPC` 参数为 true，使用主机的 IPC 命名空间，默认为 false。

### 使用主机的网络命名空间

通过设置 `spec.hostNetwork` 参数为 true，使用主机的网络命名空间，默认为 false。

### 使用主机的 PID 空间

通过设置 `spec.hostPID` 参数为 true，使用主机的 PID 命名空间，默认为 false。

```text
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostIPC: true
  hostPID: true
  hostNetwork: true
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```

### 设置 Pod 的 hostname

通过 `spec.hostname` 参数实现，如果未设置默认使用 `metadata.name` 参数的值作为 Pod 的 hostname。

### 设置 Pod 的子域名

通过 `spec.subdomain` 参数设置 Pod 的子域名，默认为空。

比如，指定 hostname 为 busybox-2 和 subdomain 为 default-subdomain，完整域名为 `busybox-2.default-subdomain.default.svc.cluster.local`，也可以简写为 `busybox-2.default-subdomain.default`：

```text
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```

注意：

* 默认情况下，DNS 为 Pod 生成的 A 记录格式为 `pod-ip-address.my-namespace.pod.cluster.local`，如 `1-2-3-4.default.pod.cluster.local`
* 上面的示例还需要在 default namespace 中创建一个名为 `default-subdomain`（即 subdomain）的 headless service，否则其他 Pod 无法通过完整域名访问到该 Pod（只能自己访问到自己）

```text
kind: Service
apiVersion: v1
metadata:
  name: default-subdomain
spec:
  clusterIP: None
  selector:
    name: busybox
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
```

注意，必须为 headless service 设置至少一个服务端口（`spec.ports`，即便它看起来并不需要），否则 Pod 与 Pod 之间依然无法通过完整域名来访问。

### 设置 Pod 的 DNS 选项

从 v1.9 开始，可以在 kubelet 和 kube-apiserver 中设置 `--feature-gates=CustomPodDNS=true` 开启设置每个 Pod DNS 地址的功能。

> 注意该功能在 v1.10 中为 Beta 版，v1.9 中为 Alpha 版。

```text
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

对于旧版本的集群，可以使用 ConfigMap 来自定义 Pod 的 `/etc/resolv.conf`，如

```text
kind: ConfigMap
apiVersion: v1
metadata:
  name: resolvconf
  namespace: default
data:
  resolv.conf: |
    search default.svc.cluster.local svc.cluster.local cluster.local
    nameserver 10.0.0.10

---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: dns-test
  namespace: default
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: dns-test
    spec:
      containers:
        - name: dns-test
          image: alpine
          stdin: true
          tty: true
          command: ["sh"]
          volumeMounts:
            - name: resolv-conf
              mountPath: /etc/resolv.conf
              subPath: resolv.conf
      volumes:
        - name: resolv-conf
          configMap:
            name: resolvconf
            items:
            - key: resolv.conf
              path: resolv.conf
```

### 资源限制

k8s中的Pod，除了显式写明pod.spec.Nodename的之外，一般是经历这样的过程  


```text
Pod(被创建，写入etcd)  --> 调度器watch到pod.spec.nodeName --> 节点上的kubelet watch到属于自己，创建容器并运行
```

调度时仅仅使用了requests，而没有使用limits。而在运行的时候两者都使用了。

![](../../../.gitbook/assets/image%20%2835%29.png)

而在运行时，这些参数的整体发挥作用的途径如下

```text
k8s ---> docker ---> linux cgroup
```

最后由操作系统内核来控制这些进程的资源限制

用一张表来描述里面的关系

| docker参数 | 处理过程 |
| :--- | :--- |
| cpuShares  | 如果requests.cpu为0且limits.cpu非0，以limits.cpu为转换输入值，否则从request.cpu为转换输入值。转换算法为：如果转换输入值为0，则设置为minShares == 2， 否则为\*1024/100 |
| oomScoreAdj | 使用了一个非常复杂的算法把容器分为三类，详见[参考文档3](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/resource-qos.md)，按优先级降序为：Guaranteed, Burstable, Best-Effort，和request.memory负相关，这个值越为负数越不容易被杀死 |
|  cpuQuota, cpuPeriod  | 由limits.cpu 转换而来，默认cpuQuota为100ms，而cpuPeriod为limits.cpu的核数 \* 100ms，这是一个硬限制 |
| memoryLimit | limits.memor |

\| docker参数 \| 处理过程 \|  
 \| ------------------- \| ---------------------------------------- \|  
 \| cpuShares   \| 如果requests.cpu为0且limits.cpu非0，以limits.cpu为转换输入值，否则从request.cpu为转换输入值。转换算法为：如果转换输入值为0，则设置为minShares == 2， 否则为\*1024/100 \|  
 \| oomScoreAdj \| 使用了一个非常复杂的算法把容器分为三类，详见[参考文档3](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/resource-qos.md)，按优先级降序为：Guaranteed, Burstable, Best-Effort，和request.memory负相关，这个值越为负数越不容易被杀死 \|  
 \| cpuQuota, cpuPeriod \| 由limits.cpu 转换而来，默认cpuQuota为100ms，而cpuPeriod为limits.cpu的核数 \* 100ms，这是一个硬限制 \|  
 \| memoryLimit \| == limits.memory \|

注意：配置了limit，没有配置request，默认会以limit的值来定义request

总结一下，说明4个参数在k8s里面的使用：

![](../../../.gitbook/assets/image%20%2885%29.png)

Kubernetes 通过 cgroups 限制容器的 CPU 和内存等计算资源，包括 requests（请求，**调度器保证调度到资源充足的 Node 上，如果无法满足会调度失败**）和 limits（上限）等：

* `spec.containers[].resources.limits.cpu`：CPU 上限，可以短暂超过，容器也不会被停止
* `spec.containers[].resources.limits.memory`：内存上限，不可以超过；如果超过，容器可能会被终止或调度到其他资源充足的机器上
* `spec.containers[].resources.limits.ephemeral-storage`：临时存储（容器可写层、日志以及 EmptyDir 等）的上限，超过后 Pod 会被驱逐
* `spec.containers[].resources.requests.cpu`：CPU 请求，也是调度 CPU 资源的依据，可以超过
* `spec.containers[].resources.requests.memory`：内存请求，也是调度内存资源的依据，可以超过；但如果超过，容器可能会在 Node 内存不足时清理
* `spec.containers[].resources.requests.ephemeral-storage`：临时存储（容器可写层、日志以及 EmptyDir 等）的请求，调度容器存储的依据

比如 nginx 容器请求 30% 的 CPU 和 56MB 的内存，但限制最多只用 50% 的 CPU 和 128MB 的内存：

```text
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "300m"
          memory: "56Mi"
        limits:
          cpu: "1"
          memory: "128Mi"
```

注意

* CPU 的单位是 CPU 个数，可以用 `millicpu (m)` 表示少于 1 个 CPU 的情况，如 `500m = 500millicpu = 0.5cpu`，而一个 CPU 相当于
  * AWS 上的一个 vCPU
  * GCP 上的一个 Core
  * Azure 上的一个 vCore
  * 物理机上开启超线程时的一个超线程
* 内存的单位则包括 `E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki` 等。
* 从 v1.10 开始，可以设置 `kubelet ----cpu-manager-policy=static` 为 Guaranteed（即 requests.cpu 与 limits.cpu 相等）Pod 绑定 CPU（通过 cpuset cgroups）。

### k8s资源限制

在K8S中可以对两类资源进行限制：cpu和内存。

CPU的单位有：

* `正实数`，代表分配几颗CPU，可以是小数点，比如`0.5`代表0.5颗CPU，意思是一颗CPU的一半时间。`2`代表两颗CPU。
* `正整数m`，也代表`1000m=1`，所以`500m`等价于`0.5`。

内存的单位：

* `正整数`，直接的数字代表Byte
* `k`、`K`、`Ki`，Kilobyte
* `m`、`M`、`Mi`，Megabyte
* `g`、`G`、`Gi`，Gigabyte
* `t`、`T`、`Ti`，Terabyte
* `p`、`P`、`Pi`，Petabyte

#### 方法一：在Pod Container Spec中设定资源限制

在K8S中，对于资源的设定是落在Pod里的Container上的，主要有两类，`limits`控制上限，`requests`控制下限。其位置在：

* `spec.containers[].resources.limits.cpu`
* `spec.containers[].resources.limits.memory`
* `spec.containers[].resources.requests.cpu`
* `spec.containers[].resources.requests.memory`

举例： 

```text
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: ...
    image: ...
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### 方法二：在Namespace中限定

方法一虽然很好，但是其不是强制性的，因此很容易出现因忘记设定`limits`/`request`，导致Host资源使用过度的情形，因此我们需要一种全局性的资源限制设定，以防止这种情况发生。K8S通过在`Namespace`设定`LimitRange`来达成这一目的。

#### 配置默认`request`/`limit`： <a id="articleHeader3"></a>

如果配置里默认的`request`/`limit`，那么当Pod Spec没有设定`request`/`limit`的时候，会使用这个配置，有效避免无限使用资源的情况。

配置位置在：

* `spec.limits[].default.cpu`，default limit
* `spec.limits[].default.memory`，同上
* `spec.limits[].defaultRequest.cpu`，default request
* `spec.limits[].defaultRequest.memory`，同上

例子： 

```text
apiVersion: v1
kind: LimitRange
metadata:
  name: <name>
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 0.5
    type: Container
```

#### 配置`request`/`limit`的约束 <a id="articleHeader4"></a>

我们还可以在K8S里对`request`/`limit`进行以下限定：

* 某资源的`request`必须`>=某值`
* 某资源的`limit`必须`<=某值`

这样的话就能有效避免Pod Spec中乱设`limit`导致资源耗尽的情况，或者乱设`request`导致Pod无法得到足够资源的情况。

配置位置在：

* `spec.limits[].max.cpu`，`limit`必须`<=某值`
* `spec.limits[].max.memory`，同上
* `spec.limits[].min.cpu`，`request`必须`>=某值`
* `spec.limits[].min.memory`，同上

例子： 

```text
apiVersion: v1
kind: LimitRange
metadata:
  name: <name>
spec:
  limits:
  - max:
      memory: 1Gi
      cpu: 800m
    min:
      memory: 500Mi
      cpu: 200m
    type: Container﻿​
```

### 健康检查

为了确保容器在部署后确实处在正常运行状态，Kubernetes 提供了两种探针（Probe）来探测容器的状态：

* LivenessProbe：探测应用是否处于健康状态，如果不健康则删除并重新创建容器
* ReadinessProbe：探测应用是否启动完成并且处于正常服务状态，如果不正常则不会接收来自 Kubernetes Service 的流量

Kubernetes 支持三种方式来执行探针：

* exec：在容器中执行一个命令，如果 [命令退出码](http://www.tldp.org/LDP/abs/html/exitcodes.html) 返回 `0` 则表示探测成功，否则表示失败
* tcpSocket：对指定的容器 IP 及端口执行一个 TCP 检查，如果端口是开放的则表示探测成功，否则表示失败
* httpGet：对指定的容器 IP、端口及路径执行一个 HTTP Get 请求，如果返回的 [状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) 在 `[200,400)` 之间则表示探测成功，否则表示失败

```text
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
spec:
    containers:
    - image: nginx
      imagePullPolicy: Always
      name: http
      livenessProbe:
        httpGet:
          path: /
          port: 80
          httpHeaders:
          - name: X-Custom-Header
            value: Awesome
        initialDelaySeconds: 15
        timeoutSeconds: 1
      readinessProbe:
        exec:
          command:
          - cat
          - /usr/share/nginx/html/index.html
        initialDelaySeconds: 5
        timeoutSeconds: 1
    - name: goproxy
      image: gcr.io/google_containers/goproxy:0.1
      ports:
      - containerPort: 8080
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
```



### 限制网络带宽

可以通过给 Pod 增加 `kubernetes.io/ingress-bandwidth` 和 `kubernetes.io/egress-bandwidth` 这两个 annotation 来限制 Pod 的网络带宽

```text
apiVersion: v1
kind: Pod
metadata:
  name: qos
  annotations:
    kubernetes.io/ingress-bandwidth: 3M
    kubernetes.io/egress-bandwidth: 4M
spec:
  containers:
  - name: iperf3
    image: networkstatic/iperf3
    command:
    - iperf3
    - -s
```

> **仅 kubenet 支持限制带宽**
>
> 目前只有 kubenet 网络插件支持限制网络带宽，其他 CNI 网络插件暂不支持这个功能。

kubenet 的网络带宽限制其实是通过 tc 来实现的

```text
# setup qdisc (only once)
tc qdisc add dev cbr0 root handle 1: htb default 30
# download rate
tc class add dev cbr0 parent 1: classid 1:2 htb rate 3Mbit
tc filter add dev cbr0 protocol ip parent 1:0 prio 1 u32 match ip dst 10.1.0.3/32 flowid 1:2
# upload rate
tc class add dev cbr0 parent 1: classid 1:3 htb rate 4Mbit
tc filter add dev cbr0 protocol ip parent 1:0 prio 1 u32 match ip src 10.1.0.3/32 flowid 1:3
```

### 调度到指定的 Node 上

**可以通过 nodeSelector、nodeAffinity、podAffinity 以及 Taints 和 tolerations 等来将 Pod 调度到需要的 Node 上。**

也可以通过设置 nodeName 参数，将 Pod 调度到指定 node 节点上。

比如，使用 nodeSelector，首先给 Node 加上标签：

```text
kubectl label nodes <your-node-name> disktype=ssd
```

接着，指定该 Pod 只想运行在带有 `disktype=ssd` 标签的 Node 上：

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

nodeAffinity、podAffinity 以及 Taints 和 tolerations 等的使用方法请参考 [调度器章节](https://github.com/feiskyer/kubernetes-handbook/blob/master/components/scheduler.md)。[kube-scheduler调度](https://darren.gitbook.io/project/~/edit/drafts/-LOpj1Qqx8l5t4yNpLB9/gai-nian-yu-yuan-li-1/gai-nian-yu-yuan-li/he-xin-zu-jian-yuan-li/kube-scheduler)



### 自定义 hosts

默认情况下，容器的 `/etc/hosts` 是 kubelet 自动生成的，并且仅包含 localhost 和 podName 等。不建议在容器内直接修改 `/etc/hosts` 文件，因为在 Pod 启动或重启时会被覆盖。

默认的 `/etc/hosts` 文件格式如下，其中 `nginx-4217019353-fb2c5` 是 podName：

```text
$ kubectl exec nginx-4217019353-fb2c5 -- cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.244.1.4	nginx-4217019353-fb2c5
```

从 v1.7 开始，可以通过 `pod.Spec.HostAliases` 来增加 hosts 内容，如

```text
apiVersion: v1
kind: Pod
metadata:
  name: hostaliases-pod
spec:
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
  containers:
  - name: cat-hosts
    image: busybox
    command:
    - cat
    args:
    - "/etc/hosts"
```

```text
$ kubectl logs hostaliases-pod
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.244.1.5	hostaliases-pod
127.0.0.1	foo.local
127.0.0.1	bar.local
10.1.2.3	foo.remote
10.1.2.3	bar.remote
```

### HugePages

v1.8 + 支持给容器分配 HugePages，资源格式为 `hugepages-<size>`（如 `hugepages-2Mi`）。使用前要配置

* 开启 `--feature-gates="HugePages=true"`
* 在所有 Node 上面预分配好 HugePage ，以便 Kubelet 统计所在 Node 的 HugePage 容量

使用示例

```text
apiVersion: v1
kind: Pod
metadata:
  generateName: hugepages-volume-
spec:
  containers:
  - image: fedora:latest
    command:
    - sleep
    - inf
    name: example
    volumeMounts:
    - mountPath: /hugepages
      name: hugepage
    resources:
      limits:
        hugepages-2Mi: 100Mi
  volumes:
  - name: hugepage
    emptyDir:
      medium: HugePages
```

注意事项

* HugePage 资源的请求和限制必须相同
* HugePage 以 Pod 级别隔离，未来可能会支持容器级的隔离
* 基于 HugePage 的 EmptyDir 存储卷最多只能使用请求的 HugePage 内存
* 使用 `shmget()` 的 `SHM_HUGETLB` 选项时，应用必须运行在匹配 `proc/sys/vm/hugetlb_shm_group` 的用户组（supplemental group）中

### 优先级

从 v1.8 开始，可以为 Pod 设置一个优先级，保证高优先级的 Pod 优先调度。

优先级调度功能目前为 Beta 版，在 v1.11 版本中默认开启。对 v1.8-1.10 版本中使用前需要开启：

* `--feature-gates=PodPriority=true`
* `--runtime-config=scheduling.k8s.io/v1alpha1=true --admission-control=Controller-Foo,Controller-Bar,...,Priority`

为 Pod 设置优先级前，先创建一个 PriorityClass，并设置优先级（数值越大优先级越高）：

```text
apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

> Kubernetes 自动创建了 `system-cluster-critical` 和 `system-node-critical` 等两个 PriorityClass，用于 Kubernetes 核心组件。

为 Pod 指定优先级

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

当调度队列有多个 Pod 需要调度时，优先调度高优先级的 Pod。而当高优先级的 Pod 无法调度时，Kubernetes 会尝试先删除低优先级的 Pod 再将其调度到对应 Node 上（Preemption）。

注意：**受限于 Kubernetes 的调度策略，抢占并不总是成功**。

### PodDisruptionBudget

[PodDisruptionBudget \(PDB\)](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) 用来保证一组 Pod 同时运行的数量，这些 Pod 需要使用 Deployment、ReplicationController、ReplicaSet 或者 StatefulSet 管理。

```text
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: zookeeper
```

### Sysctls

Sysctls 允许容器设置内核参数，分为安全 Sysctls 和非安全 Sysctls：

* 安全 Sysctls：即设置后不影响其他 Pod 的内核选项，只作用在容器 namespace 中，默认开启。包括以下几种
  * `kernel.shm_rmid_forced`
  * `net.ipv4.ip_local_port_range`
  * `net.ipv4.tcp_syncookies`
* 非安全 Sysctls：即设置好有可能影响其他 Pod 和 Node 上其他服务的内核选项，默认禁止。如果使用，需要管理员在配置 kubelet 时开启，如 `kubelet --experimental-allowed-unsafe-sysctls 'kernel.msg*,net.ipv4.route.min_pmtu'`

v1.6-v1.10 示例：

```text
apiVersion: v1
kind: Pod
metadata:
  name: sysctl-example
  annotations:
    security.alpha.kubernetes.io/sysctls: kernel.shm_rmid_forced=1
    security.alpha.kubernetes.io/unsafe-sysctls: net.ipv4.route.min_pmtu=1000,kernel.msgmax=1 2 3
spec:
  ...
```

从 v1.11 开始，Sysctls 升级为 Beta 版本，不再区分安全和非安全 sysctl，统一通过 podSpec.securityContext.sysctls 设置，如

```text
apiVersion: v1
kind: Pod
metadata:
  name: sysctl-example
spec:
  securityContext:
    sysctls:
    - name: kernel.shm_rmid_forced
      value: "0"
    - name: net.ipv4.route.min_pmtu
      value: "552"
    - name: kernel.msgmax
      value: "65536"
  ...
```

### Pod 时区

很多容器都是配置了 UTC 时区，与国内集群的 Node 所在时区有可能不一致，可以通过 HostPath 存储插件给容器配置与 Node 一样的时区：

```text
apiVersion: v1
kind: Pod
metadata:
  name: sh
  namespace: default
spec:
  containers:
  - image: alpine
    stdin: true
    tty: true
    volumeMounts:
    - mountPath: /etc/localtime
      name: time
      readOnly: true
  volumes:
  - hostPath:
      path: /etc/localtime
      type: ""
    name: time
```

### 参考文档

* [What is Pod?](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
* [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
* [DNS Pods and Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
* [Container capabilities](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container)
* [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)
* [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
* [Linux Capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html)
* [Manage HugePages](https://kubernetes.io/docs/tasks/manage-hugepages/scheduling-hugepages/)
* [Document supported docker image \(Dockerfile\) features](https://github.com/kubernetes/kubernetes/issues/30039)

