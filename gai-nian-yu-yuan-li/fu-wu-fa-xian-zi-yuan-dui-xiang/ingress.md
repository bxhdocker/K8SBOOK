# Ingress

**Ingress 介绍**

> 我们知道，到目前为止 Kubernetes 暴露服务的有三种方式，分别为 LoadBlancer Service、NodePort Service、Ingress。官网对 Ingress 的定义为管理对外服务到集群内服务之间规则的集合，通俗点讲就是它定义规则来允许进入集群的请求被转发到集群中对应服务上，从来实现服务暴漏。 Ingress 能把集群内 Service 配置成外网能够访问的 URL，流量负载均衡，终止SSL，提供基于域名访问的虚拟主机等等。

**LoadBlancer Service**

LoadBlancer Service 是 Kubernetes 结合云平台的组件，如国外 GCE、AWS、国内阿里云等等，使用它向使用的底层云平台申请创建负载均衡器来实现，有局限性，对于使用云平台的集群比较方便。

**NodePort Service**

NodePort Service 是通过在节点上暴漏端口，然后通过将端口映射到具体某个服务上来实现服务暴漏，比较直观方便，但是对于集群来说，随着 Service 的不断增加，需要的端口越来越多，很容易出现端口冲突，而且不容易管理。当然对于小规模的集群服务，还是比较不错的。

**Ingress**

Ingress 使用开源的反向代理负载均衡器来实现对外暴漏服务，比如 Nginx、Apache、Haproxy等。Nginx Ingress 一般有三个组件组成：

* Nginx 反向代理负载均衡器
* Ingress Controller

  Ingress Controller 可以理解为控制器，它通过与 Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化实时获取后端 Service、Pod 等的变化，比如新增、删除等，然后读取它，按照自定义的规则，规则就是写明了哪个域名对应哪个service，生成一段 Nginx 配置，再写到 Nginx-ingress-control的 Pod 里，这个 Ingress Contronler 的pod里面运行着一个nginx服务，控制器会把生成的nginx配置写入/etc/nginx.conf文件中，然后 reload 一下 使用配置生效。以此来达到域名分配置及动态更新的问题。来达到服务自动发现的作用。

* Ingress

  Ingress 则是定义规则，通过它定义某个域名的请求过来之后转发到集群中指定的 Service。它可以通过 Yaml 文件定义，可以给一个或多个 Service 定义一个或多个 Ingress 规则。

![](../../.gitbook/assets/image%20%2899%29.png)

以上三者有机的协调配合起来，就可以完成 Kubernetes 集群服务的暴漏

### ingress-nginx部署 <a id="&#x4E8C;ingress-nginx&#x90E8;&#x7F72;"></a>

**ingress-nginx组件有几个部分组成：**

* `configmap.yaml`:提供configmap可以在线更行nginx的配置
* `default-backend.yaml`:提供一个缺省的后台错误页面 404
* `namespace.yaml`:创建一个独立的命名空间 ingress-nginx
* `rbac.yaml`:创建对应的role rolebinding 用于rbac
* `tcp-services-configmap.yaml`:修改L4负载均衡配置的configmap
* `udp-services-configmap.yaml`:修改L4负载均衡配置的configmap
* `with-rbac.yaml`:有应用rbac的nginx-ingress-controller组件

在本篇文章中你将会看到一些在其他地方被交叉使用的术语，为了防止产生歧义，我们首先来澄清下。

* 节点：Kubernetes 集群中的服务器；
* 集群：Kubernetes 管理的一组服务器集合；
* 边界路由器：为局域网和 Internet 路由数据包的路由器，执行防火墙保护局域网络；
* 集群网络：遵循 Kubernetes[网络模型](https://kubernetes.io/docs/admin/networking/) 实现集群内的通信的具体实现，比如 [flannel](https://github.com/coreos/flannel#flannel) 和 [OVS](https://github.com/openvswitch/ovn-kubernetes)。
* 服务：Kubernetes 的服务 \(Service\) 是使用标签选择器标识的一组 pod [Service](https://kubernetes.io/docs/user-guide/services/)。 除非另有说明，否则服务的虚拟 IP 仅可在集群内部访问。

## 什么是 Ingress？ <a id="shen-me-shi-ingress"></a>

通常情况下，service 和 pod 的 IP 仅可在集群内部访问。集群外部的请求需要通过负载均衡转发到 service 在 Node 上暴露的 NodePort 上，然后再由 kube-proxy 通过边缘路由器 \(edge router\) 将其转发给相关的 Pod 或者丢弃。如下图所示

```text
   internet        |  ------------  [Services]
```

而 Ingress 就是为进入集群的请求提供路由规则的集合，如下图所示

```text
    internet        |   [Ingress]   --|-----|--   [Services]
```

Ingress 可以给 service 提供集群外部访问的 URL、负载均衡、SSL 终止、HTTP 路由等。为了配置这些 Ingress 规则，集群管理员需要部署一个 [Ingress controller](https://kubernetes.feisky.xyz/cha-jian-kuo-zhan/ingress)，它监听 Ingress 和 service 的变化，并根据规则配置负载均衡并提供访问入口。

Nginx-ingress 是 Kubernetes 生态中的重要成员，主要负责向外暴露服务，同时提供负载均衡等附加功能；

截至目前，nginx-ingress 已经能够完成 7/4 层的代理功能（4 层代理基于 ConfigMap，感觉还有改进的空间）；

 Nginx 的 7 层反向代理模式，可以简单用下图表示：

![](../../.gitbook/assets/image%20%2860%29.png)

nginx-ingress 工作流程分析

       首先，上一张整体工作模式架构图（只关注配置同步更新）

![](https://img-blog.csdnimg.cn/20181114131809244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoaWRhX2NzZG4=,size_16,color_FFFFFF,t_70)

不考虑 nginx 状态收集等附件功能，nginx-ingress 模块在运行时主要包括三个主体：NginxController、Store、SyncQueue。其中，Store 主要负责从 kubernetes APIServer 收集运行时信息，感知各类资源（如 ingress、service等）的变化，并及时将更新事件消息（event）写入一个环形管道；SyncQueue 协程定期扫描 syncQueue 队列，发现有任务就执行更新操作，即借助 Store 完成最新运行数据的拉取，然后根据一定的规则产生新的 nginx 配置，（有些更新必须 reload，就本地写入新配置，执行 reload），然后执行动态更新操作，即构造 POST 数据，向本地 Nginx Lua 服务模块发送 post 请求，实现配置更新；NginxController 作为中间的联系者，监听 updateChannel，一旦收到配置更新事件，就向同步队列 syncQueue 里写入一个更新请求

CSDN 原文：[https://blog.csdn.net/shida\_csdn/article/details/84032019](https://blog.csdn.net/shida_csdn/article/details/84032019) 

## Ingress 格式 <a id="ingress-ge-shi"></a>

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        backend:
          serviceName: test
          servicePort: 80
```

每个 Ingress 都需要配置 `rules`，目前 Kubernetes 仅支持 http 规则。上面的示例表示请求 `/testpath` 时转发到服务 `test` 的 80 端口。

## API 版本对照表 <a id="api-ban-ben-dui-zhao-biao"></a>

| Kubernetes 版本 | Extension 版本 |
| :--- | :--- |
| v1.5-v1.9 | extensions/v1beta1 |

## Ingress 类型 <a id="ingress-lei-xing"></a>

根据 Ingress Spec 配置的不同，Ingress 可以分为以下几种类型：

### 单服务 Ingress <a id="dan-fu-wu-ingress"></a>

单服务 Ingress 即该 Ingress 仅指定一个没有任何规则的后端服务。

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80
```



> 注：单个服务还可以通过设置 `Service.Type=NodePort` 或者 `Service.Type=LoadBalancer` 来对外暴露。

### 多服务的 Ingress <a id="duo-fu-wu-de-ingress"></a>

路由到多服务的 Ingress 即根据请求路径的不同转发到不同的后端服务上，比如

```text
foo.bar.com -> 178.91.123.132 -> / foo    s1:80
                                 / bar    s2:80
```

可以通过下面的 Ingress 来定义：

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80
```

 使用 `kubectl create -f` 创建完 ingress 后

```text
$ kubectl get ing
NAME      RULE          BACKEND   ADDRESS
test      -
          foo.bar.com
          /foo          s1:80
          /bar          s2:80
```

### 虚拟主机 Ingress <a id="xu-ni-zhu-ji-ingress"></a>

虚拟主机 Ingress 即根据名字的不同转发到不同的后端服务上，而他们共用同一个的 IP 地址，如下所示

```text
foo.bar.com --|                 |-> foo.bar.com s1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com s2:80
```

 下面是一个基于 [Host header](https://tools.ietf.org/html/rfc7230#section-5.4) 路由请求的 Ingress：

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```



> 注：没有定义规则的后端服务称为默认后端服务，可以用来方便的处理 404 页面。

### TLS Ingress <a id="tls-ingress"></a>

TLS Ingress 通过 Secret 获取 TLS 私钥和证书 \(名为 `tls.crt` 和 `tls.key`\)，来执行 TLS 终止。如果 Ingress 中的 TLS 配置部分指定了不同的主机，则它们将根据通过 SNI TLS 扩展指定的主机名（假如 Ingress controller 支持 SNI）在多个相同端口上进行复用。

定义一个包含 `tls.crt` 和 `tls.key` 的 secret：

```text
apiVersion: v1
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
kind: Secret
metadata:
  name: testsecret
  namespace: default
type: Opaque
```

 Ingress 中引用 secret：

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: no-rules-map
spec:
  tls:
    - secretName: testsecret
  backend:
    serviceName: s1
    servicePort: 80
```

注意，不同 Ingress controller 支持的 TLS 功能不尽相同。 请参阅有关 [nginx](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/README.md#https)，[GCE](https://github.com/kubernetes/ingress/blob/master/controllers/gce/README.md#tls) 或任何其他 Ingress controller 的文档，以了解 TLS 的支持情况。

## 更新 Ingress <a id="geng-xin-ingress"></a>

可以通过 `kubectl edit ing name` 的方法来更新 ingress：

```text
$ kubectl get ing
NAME      RULE          BACKEND   ADDRESS
test      -                       178.91.123.132
          foo.bar.com
          /foo          s1:80
$ kubectl edit ing test
```

 这会弹出一个包含已有 IngressSpec yaml 文件的编辑器，修改并保存就会将其更新到 kubernetes API server，进而触发 Ingress Controller 重新配置负载均衡：

```text
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
        path: /foo
  - host: bar.baz.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
        path: /foo
..
```

 更新后：

```text
$ kubectl get ing
NAME      RULE          BACKEND   ADDRESS
test      -                       178.91.123.132
          foo.bar.com
          /foo          s1:80
          bar.baz.com
          /foo          s2:80
```



当然，也可以通过 `kubectl replace -f new-ingress.yaml` 命令来更新，其中 new-ingress.yaml 是修改过的 Ingress yaml。

## Ingress Controller <a id="ingress-controller"></a>

Ingress 正常工作需要集群中运行 Ingress Controller。Ingress Controller 与其他作为 kube-controller-manager 中的在集群创建时自动启动的 controller 成员不同，需要用户选择最适合自己集群的 Ingress Controller，或者自己实现一个。

Ingress Controller 以 Kubernetes Pod 的方式部署，以 daemon 方式运行，保持 watch Apiserver 的 /ingress 接口以更新 Ingress 资源，以满足 Ingress 的请求。比如可以使用 [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx)：

```text
helm install stable/nginx-ingress --name nginx-ingress --set rbac.create=true
```

其他 Ingress Controller 还有：

* ​[traefik ingress](https://kubernetes.feisky.xyz/cha-jian-kuo-zhan/ingress/service-discovery-and-load-balancing) 提供了一个 Traefik Ingress Controller 的实践案例
* ​[kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx) 提供了一个详细的 Nginx Ingress Controller 示例
* ​[kubernetes/ingress-gce](https://github.com/kubernetes/ingress-gce) 提供了一个用于 GCE 的 Ingress Controller 示例

## 参考文档 <a id="can-kao-wen-dang"></a>

* ​[Kubernetes Ingress Resource](https://kubernetes.io/docs/concepts/services-networking/ingress/)​
* ​[Kubernetes Ingress Controller](https://github.com/kubernetes/ingress/tree/master)​
* ​[使用 NGINX Plus 负载均衡 Kubernetes 服务](http://dockone.io/article/957)​
* ​[使用 NGINX 和 NGINX Plus 的 Ingress Controller 进行 Kubernetes 的负载均衡](http://www.cnblogs.com/276815076/p/6407101.html)​
* ​[Kubernetes : Ingress Controller with Træfɪk and Let's Encrypt](https://blog.osones.com/en/kubernetes-ingress-controller-with-traefik-and-lets-encrypt.html)​
* ​[Kubernetes : Træfɪk and Let's Encrypt at scale](https://blog.osones.com/en/kubernetes-traefik-and-lets-encrypt-at-scale.html)​
* ​[Kubernetes Ingress Controller-Træfɪk](https://docs.traefik.io/user-guide/kubernetes/)​
* ​[Kubernetes 1.2 and simplifying advanced networking with Ingress](http://blog.kubernetes.io/2016/03/Kubernetes-1.2-and-simplifying-advanced-networking-with-Ingress.html)​
* [https://kubernetes.feisky.xyz/he-xin-yuan-li/index-2/ingress](https://kubernetes.feisky.xyz/he-xin-yuan-li/index-2/ingress)

