# 流量转移

### 关于这个任务 <a id="&#x5173;&#x4E8E;&#x8FD9;&#x4E2A;&#x4EFB;&#x52A1;"></a>

一个常见的用例是将流量从一个版本的微服务逐渐迁移到另一个版本。 在Istio中，您可以通过配置一系列规则来实现此目标， 这些规则将一定百分比的流量路由到一个或另一个服务。 在此任务中，您将先分别向 `reviews:v1` 和 `reviews:v3` 各发送50%流量。 然后，您将通过向 `reviews:v3` 发送100％的流量来完成迁移。

## 基于权重的路由

这个实例将展示如何将逐渐地将流量从微服务的一个版本切换到另一个版本。

在istio中可以设置一系列的规则将流量按百分比从在服务间进行路由。下面这个例子首先各发50%的流量到reviews:v1和reviews:v3，然后通过设置将流量全部发到reviews:v3。

**&lt;1&gt;首先设置将流量全部发送到各微服务的v1版本：**

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/virtual-service-all-v1.yaml1
```

打开/product页面，由于所有流量都发送到v1版本，所以，页面一直展示无星版：   
![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/2018080312561214?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**&lt;2&gt;接下来将流量各导50%到reviews:v1和reviews:v3:**

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml1
```

验证规则有更新：

```text
kubectl get virtualservice reviews -n istio-test -o yaml1
```

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803130834941?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)   
此时不断刷新页面，会在无星版（v1）和红星版（v3）中等概率切换。

**&lt;3&gt;最后将流量全部导到reviews:v3**

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/virtual-service-reviews-v3.yaml1
```

验证规则有更新：

```text
kubectl get virtualservice reviews -n istio-test -o yaml1
```

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803131304165?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)   
此时打开页面，不断刷新，始终是红星版（v3）：   
![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803131428220?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**&lt;4&gt;原理**

这个实验使用Istio的加权路由功能将流量从旧版本的 reviews 服务逐渐切换到新版本。

**注意，这和使用容器编排平台的部署功能来进行版本迁移完全不同，后者使用了实例扩缩来对流量进行切换。**

使用Istio，两个版本的 reviews 服务可以独立地进行扩容和缩容，并不会影响这两个版本服务之间的流量分发。

如果想了解自动伸缩的版本路由，请查看这篇博客[使用istio进行金丝雀部署](https://istio.io/blog/2017/0.1-canary/)

