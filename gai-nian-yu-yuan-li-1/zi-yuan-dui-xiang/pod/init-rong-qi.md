# Init 容器

该特性在 1.6 版本已经推出 beta 版本。Init 容器可以在 PodSpec 中同应用程序的 `containers` 数组一起来指定。 beta 注解的值将仍需保留，并覆盖 PodSpec 字段值。

本文讲解 Init 容器的基本概念，它是一种专用的容器，在应用程序容器启动之前运行，并包括一些应用镜像中不存在的实用工具和安装脚本。

### 概念

在K8S的POD中，有两类容器，一类是系统容器（POD Container），一类是用户容器（User Container），在用户容器中，现在又分成两类容器，一类是初始化容器（Init Container），一类是应用容器（App Container）。

Init Container就是做初始化工作的容器。可以有一个或多个，如果多个按照定义的顺序依次执行，只有所有的执行完后，主容器才启动。由于一个Pod里的存储卷是共享的，所以Init Container里产生的数据可以被主容器使用到。

Init Container可以在多种K8S资源里被使用到如Deployment、DaemonSet, StatefulSet、Job等，但都是在Pod启动时，在主容器启动前执行，做初始化工作。

### 理解 Init 容器 <a id="&#x7406;&#x89E3;-init-&#x5BB9;&#x5668;"></a>

[Pod](https://kubernetes.io/docs/concepts/abstractions/pod/) 能够具有多个容器，应用运行在容器里面，但是它也可能有一个或多个先于应用容器启动的 Init 容器。

Init 容器与普通的容器非常像，除了如下两点：

* Init 容器总是运行到成功完成为止。
* 每个 Init 容器都必须在下一个 Init 容器启动之前成功完成。

如果 Pod 的 Init 容器失败，Kubernetes 会不断地重启该 Pod，直到 Init 容器成功为止。然而，如果 Pod 对应的 `restartPolicy` 为 Never，它不会重新启动。

指定容器为 Init 容器，在 PodSpec 中添加 `initContainers` 字段，以 [v1.Container](https://kubernetes.io/docs/api-reference/v1.6/#container-v1-core) 类型对象的 JSON 数组的形式，还有 app 的 `containers` 数组。 Init 容器的状态在 `status.initContainerStatuses` 字段中以容器状态数组的格式返回（类似 `status.containerStatuses` 字段）。

#### 与普通容器的不同之处 <a id="&#x4E0E;&#x666E;&#x901A;&#x5BB9;&#x5668;&#x7684;&#x4E0D;&#x540C;&#x4E4B;&#x5904;"></a>

Init 容器支持应用容器的全部字段和特性，包括资源限制、数据卷和安全设置。 然而，Init 容器对资源请求和限制的处理稍有不同，在下面 [资源](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#resources) 处有说明。 而且 Init 容器不支持 Readiness Probe，因为它们必须在 Pod 就绪之前运行完成。

如果为一个 Pod 指定了多个 Init 容器，那些容器会按顺序一次运行一个。 每个 Init 容器必须运行成功，下一个才能够运行。 当所有的 Init 容器运行完成时，Kubernetes 初始化 Pod 并像平常一样运行应用容器

### Init 容器能做什么？ <a id="init-&#x5BB9;&#x5668;&#x80FD;&#x505A;&#x4EC0;&#x4E48;&#xFF1F;"></a>

因为 Init 容器具有与应用程序容器分离的单独镜像，所以它们的启动相关代码具有如下优势：

* 它们可以包含并运行实用工具，但是出于安全考虑，是不建议在应用程序容器镜像中包含这些实用工具的。
* 它们可以包含使用工具和定制化代码来安装，但是不能出现在应用程序镜像中。例如，创建镜像没必要 `FROM` 另一个镜像，只需要在安装过程中使用类似 `sed`、 `awk`、 `python` 或 `dig` 这样的工具。
* 应用程序镜像可以分离出创建和部署的角色，而没有必要联合它们构建一个单独的镜像。
* Init 容器使用 Linux Namespace，所以相对应用程序容器来说具有不同的文件系统视图。因此，它们能够具有访问 Secret 的权限，而应用程序容器则不能。
* 它们必须在应用程序容器启动之前运行完成，而应用程序容器是并行运行的，所以 Init 容器能够提供了一种简单的阻塞或延迟应用容器的启动的方法，直到满足了一组先决条件。

### 应用场景

#### 等待其它模块Ready

比如使用apache部署web服务，需要做一些准备工作（例如从git服务器上拉取代码、检查运行环境是否到位等），可以在运行Web服务的Pod里使用一个Init Container，去执行准备工作，完成后Init Container结束退出，然后启动正在的apache容器。

#### 做初始化配置

比如集群里检测所有已经存在的成员节点，为主容器准备好集群的配置信息，这样主容器起来后就能用这个配置信息加入集群。

### 例子1：

下面是一个 Init 容器的示例：

```text
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

因为 Init 容器具有与应用容器分离的单独镜像，使用 init 容器启动相关代码具有如下优势：

* 它们可以包含并运行实用工具，出于安全考虑，是不建议在应用容器镜像中包含这些实用工具的。
* 它们可以包含使用工具和定制化代码来安装，但是不能出现在应用镜像中。例如，创建镜像没必要 FROM 另一个镜像，只需要在安装过程中使用类似 sed、 awk、 python 或 dig 这样的工具。
* 应用镜像可以分离出创建和部署的角色，而没有必要联合它们构建一个单独的镜像。
* 它们使用 Linux Namespace，所以对应用容器具有不同的文件系统视图。因此，它们能够具有访问 Secret 的权限，而应用容器不能够访问。
* 它们在应用容器启动之前运行完成，然而应用容器并行运行，所以 Init 容器提供了一种简单的方式来阻塞或延迟应用容器的启动，直到满足了一组先决条件。

Init 容器的资源计算，选择一下两者的较大值：

* 所有 Init 容器中的资源使用的最大值
* Pod 中所有容器资源使用的总和

Init 容器的重启策略：

* 如果 Init 容器执行失败，Pod 设置的 restartPolicy 为 Never，则 pod 将处于 fail 状态。否则 Pod 将一直重新执行每一个 Init 容器直到所有的 Init 容器都成功。
* 如果 Pod 异常退出，重新拉取 Pod 后，Init 容器也会被重新执行。所以在 Init 容器中执行的任务，需要保证是幂等的。

### 例子2 此为1.5的语法

```text
cat << EOF > lykops-deploy-init-container.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: lykops-deploy-init-container
  labels:
    project: lykops
    app: init-container
    version: v1         
  annotations:
    pod.beta.K8S.io/init-containers: 
    - name: apache-web,
      image: web:apache,
      command: ["sh", "httpd -t"]
spec:
  replicas: 1
  minReadySeconds: 30
  selector:
    matchLabels:
      name: lykops-deploy-init-container
      project: lykops
      app: init-container
      version: v1
  template:
    metadata:
      labels:
        name: lykops-deploy-init-container
        project: lykops
        app: init-container
        version: v1
    spec:
      containers:
      - name: webapache
        image: web:apache
        command: [ "sh", "/etc/run.sh" ]
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
EOF
kubectl create -f lykops-deploy-init-container.yaml

```

#### 示例 <a id="&#x793A;&#x4F8B;"></a>

下面是一些如何使用 Init 容器的想法：

* 等待一个 Service 创建完成，通过类似如下 shell 命令：

  ```text
    for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; exit 1
  ```

* 将 Pod 注册到远程服务器，通过在命令中调用 API，类似如下：

  ```text
    curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```

* 在启动应用容器之前等一段时间，使用类似 `sleep 60` 的命令。
* 克隆 Git 仓库到数据卷。
* 将配置值放到配置文件中，运行模板工具为主应用容器动态地生成配置文件。例如，在配置文件中存放 POD\_IP 值，并使用 Jinja 生成主应用配置文件。

更多详细用法示例，可以在 [StatefulSet 文档](https://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/) 和 [生产环境 Pod 指南](https://kubernetes.io/docs/user-guide/production-pods.md#handling-initialization) 中找到。

#### 使用 Init 容器 <a id="&#x4F7F;&#x7528;-init-&#x5BB9;&#x5668;"></a>

这是 Kubernetes 1.6 版本的新语法，尽管老的 annotation 语法仍然可以使用。我们已经把 Init 容器的声明移到 `spec` 中：

```text
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

在 Kubernetes 1.6 版本中，Init 容器在 API 中新建了一个字段。 虽然期望使用 beta 版本的 annotation，但在未来发行版将会被废弃掉。

下面的 yaml 文件展示了 `mydb` 和 `myservice` 两个 Service：

```text
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9377
```

这个 Pod 可以使用下面的命令进行启动和调试：

```text
$ kubectl create -f myapp.yaml
pod "myapp-pod" created
$ kubectl get -f myapp.yaml
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
$ kubectl describe -f myapp.yaml 
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
$ kubectl logs myapp-pod -c init-myservice # Inspect the first init container
$ kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
```

一旦我们启动了 `mydb` 和 `myservice` 这两个 Service，我们能够看到 Init 容器完成，并且 `myapp-pod` 被创建：

```text
$ kubectl create -f services.yaml
service "myservice" created
service "mydb" created
$ kubectl get -f myapp.yaml
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

这个例子非常简单，但是应该能够为我们创建自己的 Init 容器提供一些启发。

### 具体行为 <a id="&#x5177;&#x4F53;&#x884C;&#x4E3A;"></a>

在 Pod 启动过程中，Init 容器会按顺序在网络和数据卷初始化之后启动。 每个容器必须在下一个容器启动之前成功退出。 如果由于运行时或失败退出，导致容器启动失败，它会根据 Pod 的 `restartPolicy` 指定的策略进行重试。 然而，如果 Pod 的 `restartPolicy` 设置为 Always，Init 容器失败时会使用 `RestartPolicy`策略。

在所有的 Init 容器没有成功之前，Pod 将不会变成 `Ready` 状态。 Init 容器的端口将不会在 Service 中进行聚集。 正在初始化中的 Pod 处于 `Pending` 状态，但应该会将条件 `Initializing` 设置为 true。

如果 Pod [重启](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#pod-restart-reasons)，所有 Init 容器必须重新执行。

对 Init 容器 spec 的修改，被限制在容器 image 字段中。 更改 Init 容器的 image 字段，等价于重启该 Pod。

因为 Init 容器可能会被重启、重试或者重新执行，所以 Init 容器的代码应该是幂等的。 特别地，被写到 `EmptyDirs` 中文件的代码，应该对输出文件可能已经存在做好准备。

Init 容器具有应用容器的所有字段。 除了 `readinessProbe`，因为 Init 容器无法定义不同于完成（completion）的就绪（readiness）的之外的其他状态。 这会在验证过程中强制执行。

在 Pod 上使用 `activeDeadlineSeconds`，在容器上使用 `livenessProbe`，这样能够避免 Init 容器一直失败。 这就为 Init 容器活跃设置了一个期限。

在 Pod 中的每个 app 和 Init 容器的名称必须唯一；与任何其它容器共享同一个名称，会在验证时抛出错误。

#### 资源 <a id="&#x8D44;&#x6E90;"></a>

为 Init 容器指定顺序和执行逻辑，下面对资源使用的规则将被应用：

* 在所有 Init 容器上定义的，任何特殊资源请求或限制的最大值，是 _有效初始请求/限制_
* Pod 对资源的有效请求/限制要高于：
  * 所有应用容器对某个资源的请求/限制之和
  * 对某个资源的有效初始请求/限制
* 基于有效请求/限制完成调度，这意味着 Init 容器能够为初始化预留资源，这些资源在 Pod 生命周期过程中并没有被使用。
* Pod 的 _有效 QoS 层_，是 Init 容器和应用容器相同的 QoS 层。

基于有效 Pod 请求和限制来应用配额和限制。 Pod 级别的 cgroups 是基于有效 Pod 请求和限制，和调度器相同。

#### Pod 重启的原因 <a id="pod-&#x91CD;&#x542F;&#x7684;&#x539F;&#x56E0;"></a>

Pod 能够重启，会导致 Init 容器重新执行，主要有如下几个原因：

* 用户更新 PodSpec 导致 Init 容器镜像发生改变。应用容器镜像的变更只会重启应用容器。
* Pod 基础设施容器被重启。这不多见，但某些具有 root 权限可访问 Node 的人可能会这样做。
* 当 `restartPolicy` 设置为 Always，Pod 中所有容器会终止，强制重启，由于垃圾收集导致 Init 容器完整的记录丢失。

### 支持与兼容性 <a id="&#x652F;&#x6301;&#x4E0E;&#x517C;&#x5BB9;&#x6027;"></a>

Apiserver 版本为 1.6 或更高版本的集群，通过使用 `spec.initContainers` 字段来支持 Init 容器。 之前的版本可以使用 alpha 和 beta 注解支持 Init 容器。 `spec.initContainers` 字段也被加入到 alpha 和 beta 注解中，所以 Kubernetes 1.3.0 版本或更高版本可以执行 Init 容器，并且 1.6 版本的 apiserver 能够安全地回退到 1.5.x 版本，而不会使存在的已创建 Pod 失去 Init 容器的功能。

原文地址：[https://k8smeetup.github.io/docs/concepts/workloads/pods/init-containers/](https://k8smeetup.github.io/docs/concepts/workloads/pods/init-containers/)





