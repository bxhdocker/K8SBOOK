# 故障注入

### 前提条件 <a id="&#x524D;&#x63D0;&#x6761;&#x4EF6;"></a>

* 按照[安装指南](https://istio.io/zh/docs/setup/)中的说明设置 Istio 。
* 部署示例应用程序 [Bookinfo](https://istio.io/zh/docs/examples/bookinfo/) 。
* 通过首先执行[请求路由](https://istio.io/zh/docs/tasks/traffic-management/request-routing/)任务或运行以下命令来初始化应用程序版本路由：

  ```text
  $ istioctl create -f samples/bookinfo/networking/virtual-service-all-v1.yaml
  $ istioctl replace -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
  ```

注：如果安装前面的步骤做，这在上一节配置请求的路由规则中做过可以忽略

**注入HTTP延迟实验**

**&lt;1&gt;注入HTTP延迟故障**

为了测试我们的微服务应用程序 Bookinfo 的弹性，为jason用户在reviews:v2和ratings两个微服务的访问路径之间注入一个7s延迟的人工“bug”,由于 _reviews:v2_ 服务对其 ratings 服务的调用具有 10 秒的硬编码连接超时，因此我们期望端到端流程是正常的（没有任何错误）。

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml1
```

查看延迟是否注入成功：

```text
$ kubectl get virtualservice ratings -n istio-test -o yaml
```

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803103149415?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

观察应用程序行为

以 “jason” 用户身份登录。如果应用程序的首页设置为正确处理延迟，我们预计它将在大约 7 秒内加载。 要查看网页响应时间，请在IE，Chrome 或 Firefox 中打开 _Developer Tools_ 菜单（通常，组合键 _Ctrl+Shift+I_ 或 _Alt+Cmd+I_ ）， 选项卡 Network，然后重新加载 `productpage` 网页 。

您将看到网页加载大约 6 秒钟。评论部分将显示 _对不起，此书的产品评论目前不可用_ 。

**&lt;2&gt;测试**

reviews:v2微服务在连接ratings的代码里硬编码了一个10s的连接超时机制，所以尽管引入了一个7s的延迟bug，两个服务之前的端到端流程理论上依然应该是正常的。

以jason用户登录/productpage，我们原本以为bookinfo的页面会在7s后正常加载，但是实际上Reviews区域有报错：   
![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803105252266?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这样，通过注入延迟我们发现了一个bug。

**&lt;3&gt;怎么回事？**

虽然我们引入的7s的延迟在reviews:v2和ratings之间10s的超时机制允许范围内，但是productpage和reviews:v2两个服务的超时时间仅有6s\(单次3s加一次重试机制\)，这导致了最终productpage报错。

以上是企业内部不同微服务独立开发时经常会遇到的一个bug。istio的故障注入机制可以使你在不影响终端用户的情况下发现此类异常。

**修复错误：** 此时我们通常会通过增加产品页面超时或减少评级服务超时的评论来解决问题， 终止并重启固定的微服务，然后确认 `productpage` 返回其响应, 没有任何错误。

但是，我们已经在评论服务的第 3 版中运行此修复程序， 因此我们可以通过将所有流量迁移到 `reviews:v3` 来解决问题， 如[流量转移](https://istio.io/zh/docs/tasks/traffic-management/traffic-shifting/)中所述任务。

（作为读者的练习 - 将延迟规则更改为使用 2.8 秒延迟，然后针对 v3 版本的评论运行它。）



## 使用 HTTP Abort 进行故障注入

作为弹性的另一个测试，我们将在 ratings 服务中，给用户 jason 的调用加上一个 HTTP 中断 。 我们希望页面能够立即加载，而不像延迟示例那样显示”产品评级不可用”消息。

1.为用户 “jason” 创建故障注入规则发送 HTTP 中止

```text
kubectl apply -n istio-test -f istio-1.0.0/samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
```

观察应用程序行为

以 “jason” 用户名登录, 如果规则成功传播到所有的 pod ，您应该能立即看到页面加载”产品评级不可用”消息。 从用户 “jason” 注销，您应该会在产品页面网页上看到评级星标的评论成功显示。

查看延迟是否注入成功：

```text
kubectl get virtualservice ratings -n istio-test -o yaml1
```

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803112731207?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**&lt;2&gt;测试**

以jason用户名登录, 看到页面显示：“Ratings service is currently unavailable”：   
![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803113218349?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)   
登出jason，页面显示恢复正常：   
![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20180803113306243?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdWt1YW43Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



