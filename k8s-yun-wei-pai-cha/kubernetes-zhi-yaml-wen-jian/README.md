# Kubernetes之YAML文件

### YAML 基础 <a id="yaml-&#x57FA;&#x7840;"></a>

YAML是专门用来写配置文件的语言，非常简洁和强大，使用比json更方便。它实质上是一种通用的数据串行化格式。后文会说明定义YAML文件创建Pod和创建Deployment。

YAML语法规则：

* 大小写敏感
* 使用缩进表示层级关系
* **缩进时不允许使用Tal键，只允许使用空格**
* 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
* ”\#” 表示注释，从这个字符一直到行尾，都会被解析器忽略

在Kubernetes中，只需要知道两种结构类型即可：

* Lists
* Maps

使用YAML用于K8s的定义带来的好处包括：

* 便捷性：不必添加大量的参数到命令行中执行命令
* 可维护性：YAML文件可以通过源头控制，跟踪每次操作
* 灵活性：YAML可以创建比命令行更加复杂的结构

#### YAML Maps <a id="yaml-maps"></a>

Map顾名思义指的是字典，即一个Key:Value 的键值对信息。例如：

```text
---
apiVersion: v1
kind: Pod123
```

注：`---` 为**可选的分隔符** ，当需要在一个文件中定义多个结构的时候需要使用。上述内容表示有两个键apiVersion和kind，分别对应的值为v1和Pod。

Maps的value既能够对应字符串**也能够对应一个Maps。**例如：

```text
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web1234567
```

注：上述的YAML文件中，metadata这个KEY对应的值为一个Maps，而嵌套的labels这个KEY的值又是一个Map。实际使用中可视情况进行多层嵌套。

​ YAML处理器根据行缩进来知道内容之间的关联。上述例子中，使用**两个空格作为缩进，但空格的数据量并不重要，只是至少要求一个空格并且所有缩进保持一致的空格数** 。例如，name和labels是相同缩进级别，因此YAML处理器知道他们属于同一map；它知道app是lables的值因为app的缩进更大。

**注意：在YAML文件中绝对不要使用tab键**



#### YAML Lists <a id="yaml-lists"></a>

List即列表，说白了就是数组，例如：

```text
args
 -beijing
 -shanghai
 -shenzhen
 -guangzhou12345
```

可以指定任何数量的项在列表中，每个项的定义以破折号（-）开头，并且与父元素之间存在缩进。在JSON格式中，表示如下：

```text
{
  "args": ["beijing", "shanghai", "shenzhen", "guangzhou"]
}123
```

当然Lists的子项也可以是Maps，Maps的子项也可以是List，例如：

```text
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: jcdemo/flaskapp
      ports: 808012345678910111213141516
```

如上述文件所示，定义一个containers的List对象，每个子项都由name、image、ports组成，每个ports都有一个KEY为containerPort的Map组成，转成JSON格式文件：

```text
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
        "name": "kube100-site",
        "labels": {
            "app": "web"
        },

  },
  "spec": {
        "containers": [{
            "name": "front-end",
            "image": "nginx",
            "ports": [{
                "containerPort": "80"
            }]
        }, {
            "name": "flaskapp-demo",
            "image": "jcdemo/flaskapp",
            "ports": [{
                "containerPort": "5000"
            }]
        }]
  }
}1234567891011121314151617181920212223242526
```

### 使用YAML创建Pod <a id="&#x4F7F;&#x7528;yaml&#x521B;&#x5EFA;pod"></a>

#### 创建Pod <a id="&#x521B;&#x5EFA;pod"></a>

```text
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: jcdemo/flaskapp
      ports:
        - containerPort: 50001234567891011121314151617
```

上面定义了一个普通的Pod文件，简单分析下文件内容：

* apiVersion：此处值是v1，这个版本号需要根据安装的Kubernetes版本和资源类型进行变化，**记住不是写死的**。
* kind：此处创建的是Pod，根据实际情况，此处资源类型可以是Deployment、Job、Ingress、Service等。
* metadata：包含Pod的一些meta信息，比如名称、namespace、标签等信息。
* spe：包括一些container，storage，volume以及其他Kubernetes需要的参数，以及诸如是否在容器失败时重新启动容器的属性。可在特定Kubernetes API找到完整的Kubernetes Pod的属性。

下面是一个典型的容器的定义：

```text
…
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
…12345678
```

* 上述例子只是一个简单的最小定义：一个名字（front-end）、基于nginx的镜像，以及容器将会监听的指定端口号（80）。
* 除了上述的基本属性外，还能够指定复杂的属性，包括**容器启动运行的命令、使用的参数、工作目录以及每次实例化是否拉取新的副本。** 还可以指定更深入的信息，例如容器的退出日志的位置。容器可选的设置属性包括：

  name、image、command、args、workingDir、ports、env、resource、volumeMounts、livenessProbe、readinessProbe、livecycle、terminationMessagePath、imagePullPolicy、securityContext、stdin、stdinOnce、tty

了解了Pod的定义后，将上面创建Pod的YAML文件保存成pod.yaml，然后使用`Kubectl`创建Pod：

```text
$ kubectl create -f pod.yaml
pod "kube100-site" created12
```

可以使用Kubectl命令查看Pod的状态

```text
$ kubectl get pods
NAME           READY     STATUS    RESTARTS   AGE
kube100-site   2/2       Running   0          1m123
```

注： Pod创建过程中如果出现错误，可以使用`kubectl describe` 进行排查。

### 创建Deployment <a id="&#x521B;&#x5EFA;deployment"></a>

上述介绍了如何使用YAML文件创建Pod实例，但是如果这个Pod出现了故障的话，对应的服务也就挂掉了，所以**Kubernetes提供了一个Deployment的概念** ，目的是**让Kubernetes去管理一组Pod的副本，也就是副本集** ，这样就能够保证一定数量的副本一直可用，不会因为某一个Pod挂掉导致整个服务挂掉。

```text
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kube100-site
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: flaskapp-demo
          image: jcdemo/flaskapp
          ports:
            - containerPort: 5000123456789101112131415161718192021
```

一个完整的Deployment的YAML文件如上所示，接下来解释部分内容：

* 注意这里apiVersion对应的值是extensions/v1beta1，同时也需要将kind的类型指定为Deployment。
* metadata指定一些meta信息，包括名字或标签之类的。
* **spec** 选项定义需要两个副本，此处可以设置很多属性，例如受此Deployment影响的Pod的选择器等
* **spec** 选项的template其实就是对Pod对象的定义
* 可以在Kubernetes v1beta1 API 参考中找到完整的Deployment可指定的参数列表

将上述的YAML文件保存为deployment.yaml，然后创建Deployment：

```text
$ kubectl create -f deployment.yaml
deployment "kube100-site" created12
```

可以使用如下命令检查Deployment的列表：

```text
$ kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube100-site   2         2         2            2           2m123
```

### 参考资料： <a id="&#x53C2;&#x8003;&#x8D44;&#x6599;"></a>

* [使用YAML文件创建Kubernetes Deployment](https://blog.qikqiak.com/post/use-yaml-create-kubernetes-deployment/)
* [使用YAML创建一个Kubernetes Deployment](https://www.kubernetes.org.cn/1414.html)

