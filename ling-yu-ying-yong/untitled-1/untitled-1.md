# 示例应用部署

### 部署前准备 <a id="&#x90E8;&#x7F72;&#x5E94;&#x7528;"></a>

### sidecar自动注入配置 <a id="21-sidecar&#x81EA;&#x52A8;&#x6CE8;&#x5165;&#x914D;&#x7F6E;"></a>

Istio装好后，如果想sidecar在应用启动时自动注入到pod中，还需要配置如下4步：

* **如上安装istio-sidecar-injector**  安装了istio-sidecar-injector后，kubectl create起应用的时候sidecar容器会直接自动注入到pod中，而不用手动注入。
* **启用**[**mutating webhook admission controller**](https://kubernetes.io/docs/admin/admission-controllers/)**（k8s 1.9以上支持）**   
  在kube-apiserver的启动参数的admission controller中按正确顺序加入如下两个controller:

  * MutatingAdmissionWebhook
  * ValidatingAdmissionWebhook

  **例如：**

  ```text
  --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota1
  ```

* **启用admissionregistration api**   
  查看admissionregistration api是否已启用：

  ```text
  kubectl api-versions | grep admissionregistration
  admissionregistration.k8s.io/v1alpha1
  admissionregistration.k8s.io/v1beta1123
  ```

* **为需要自动注入sidecar的namespace打label。**   
  istio-sidecar-injector会为打了`istio-injection=enabled` label的namespace中的pod自动注入sidecar：

  ```text
  kubectl label namespace istio-test istio-injection=enabled
  kubectl get namespace -L istio-injection
  ```

![](../../.gitbook/assets/image%20%2851%29.png)

## Bookinfo 应用 <a id="title"></a>

部署一个样例应用，它由四个单独的微服务构成，用来演示多种 Istio 特性。这个应用模仿在线书店的一个分类，显示一本书的信息。页面上会显示一本书的描述，书籍的细节（ISBN、页数等），以及关于这本书的一些评论。

Bookinfo 应用分为四个单独的微服务：

* `productpage` ：`productpage` 微服务会调用 `details` 和 `reviews` 两个微服务，用来生成页面。
* `details` ：这个微服务包含了书籍的信息。
* `reviews` ：这个微服务包含了书籍相关的评论。它还会调用 `ratings` 微服务。
* `ratings` ：`ratings` 微服务中包含了由书籍评价组成的评级信息。

`reviews` 微服务有 3 个版本：

* v1 版本不会调用 `ratings` 服务。
* v2 版本会调用 `ratings` 服务，并使用 1 到 5 个黑色星形图标来显示评分信息。
* v3 版本会调用 `ratings` 服务，并使用 1 到 5 个红色星形图标来显示评分信息。

下图展示了这个应用的端到端架构

![](../../.gitbook/assets/image%20%28124%29.png)

Istio 注入之前的 Bookinfo 应用

Bookinfo 是一个异构应用，几个微服务是由不同的语言编写的。这些服务对 Istio 并无依赖，但是构成了一个有代表性的服务网格的例子：它由多个服务、多个语言构成，并且 `reviews` 服务具有多个版本。

### 2.2 启动示例应用 <a id="22-&#x542F;&#x52A8;&#x793A;&#x4F8B;&#x5E94;&#x7528;"></a>

以上都配置好后，就可以[参照官方例子](https://istio.io/docs/guides/bookinfo/)启动samples下的bookinfo示例应用\(自动注入sidecar方式\)：

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/platform/kube/bookinfo.yaml1
```

可以看到每个pod中的容器都是2个，自动注入了一个sidecar容器：

 kubectl get pod -n istio-test  
   
**注意：** 应用的http流量必须使用HTTP/1.1或HTTP/2.0协议，HTTP/1.0协议不支持。

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180802162817142?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 2.3 Istio Gateway创建 <a id="23-istio-gateway&#x521B;&#x5EFA;"></a>

[Istio Gateway](https://istio.io/docs/concepts/traffic-management/#gateways)用来实现bookinfo应用从k8s集群外部可访问。为bookinfo应用创建一个gateway和virtualservice对象：

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/bookinfo-gateway.yaml
```

![&#x8DEF;&#x7531;](https://img-blog.csdn.net/20180802164924217?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

kubectl get gateway -n istio-test

### 2.4 应用默认rules <a id="24-&#x5E94;&#x7528;&#x9ED8;&#x8BA4;rules"></a>

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/destination-rule-all.yaml
```

可以用 `curl` 命令来确认 Bookinfo 应用的运行情况：

```text
$ curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
200
```

还可以用浏览器打开网址 `http://$GATEWAY_URL/productpage`，来浏览应用的 Web 页面。如果刷新几次应用的页面，就会看到页面中会随机展示 `reviews` 服务的不同版本的效果（红色、黑色的星形或者没有显示）。`reviews` 服务出现这种情况是因为我们还没有使用 Istio 来控制版本的路由。





文章参考：

 [https://blog.csdn.net/liukuan73/article/details/81165716](https://blog.csdn.net/liukuan73/article/details/81165716)

