# 配置请求的路由规则

在上一节[安装istio](https://jimmysong.io/kubernetes-handbook/usecases/istio-installation.html)中我们创建[BookInfo](https://istio.io/docs/samples/bookinfo.html)的示例，熟悉了Istio的基本功能，现在我们再来看一下istio的高级特性——配置请求的路由规则。

使用istio我们可以根据**权重**和**HTTP headers**来动态配置请求路由

### 基于内容的路由 <a id="&#x57FA;&#x4E8E;&#x5185;&#x5BB9;&#x7684;&#x8DEF;&#x7531;"></a>

因为BookInfo示例部署了3个版本的评论微服务，我们需要设置一个默认路由。 否则，当你多次访问应用程序时，会注意到有时输出包含星级，有时候又没有。 这是因为没有明确的默认版本集，Istio将以随机方式将请求路由到服务的所有可用版本。

**注意**：假定您尚未设置任何路由。如果您已经为示例创建了冲突的路由规则，则需要在以下命令中使用replace而不是create。

下面这个例子能够根据网站的不同登陆用户，将流量划分到服务的不同版本和实例。跟kubernetes中的应用一样，所有的路由规则都是通过声明式的yaml配置。关于`reviews:v1`和`reviews:v2`的唯一区别是，v1没有调用评分服务，productpage页面上不会显示评分星标。

1. **将微服务的默认版本设置成v1。**

   ```text
   istioctl create -f samples/apps/bookinfo/route-rule-all-v1.yaml
   ```

   使用以下命令查看定义的路由规则。

   ```text
   istioctl get route-rules -o yaml
   ```

   ```text
   type: route-rule
   name: details-default
   namespace: default
   spec:
   destination: details.default.svc.cluster.local
   precedence: 1
   route:
   - tags:
       version: v1
   ---
   type: route-rule
   name: productpage-default
   namespace: default
   spec:
   destination: productpage.default.svc.cluster.local
   precedence: 1
   route:
   - tags:
       version: v1
   ---
   type: route-rule
   name: reviews-default
   namespace: default
   spec:
   destination: reviews.default.svc.cluster.local
   precedence: 1
   route:
   - tags:
       version: v1
   ---
   type: route-rule
   name: ratings-default
   namespace: default
   spec:
   destination: ratings.default.svc.cluster.local
   precedence: 1
   route:
   - tags:
       version: v1
   ---
   ```

   由于对代理的规则传播是异步的，因此在尝试访问应用程序之前，需要等待几秒钟才能将规则传播到所有pod。

2. 在浏览器中打开BookInfo URL（[http://$GATEWAY\_URL/productpage](http://$gateway_url/productpage) ，我们在上一节中使用的是 [http://ingress.istio.io/productpage](http://ingress.istio.io/productpage) ）您应该会看到BookInfo应用程序的产品页面显示。 注意，产品页面上没有评分星，因为`reviews:v1`不访问评级服务。
3. **将特定用户路由到`reviews:v2`。**

   为测试用户jason启用评分服务，将productpage的流量路由到`reviews:v2`实例上。

   ```text
   istioctl create -f samples/apps/bookinfo/route-rule-reviews-test-v2.yaml
   ```

   确认规则生效：

   ```text
   istioctl get route-rule reviews-test-v2
   ```

   ```text
   destination: reviews.default.svc.cluster.local
   match:
     httpHeaders:
       cookie:
         regex: ^(.*?;)?(user=jason)(;.*)?$
   precedence: 2
   route:
   - tags:
       version: v2
   ```

4. 使用jason用户登陆productpage页面。

   你可以看到每个刷新页面时，页面上都有一个1到5颗星的评级。如果你使用其他用户登陆的话，将因继续使用`reviews:v1`而看不到星标评分。



