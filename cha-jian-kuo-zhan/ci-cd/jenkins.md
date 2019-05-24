# Jenkins



## 使用Jenkins进行持续集成与发布 <a id="&#x4F7F;&#x7528;jenkins&#x8FDB;&#x884C;&#x6301;&#x7EED;&#x96C6;&#x6210;&#x4E0E;&#x53D1;&#x5E03;"></a>

我们基于Jenkins的CI/CD流程如下所示

![](../../.gitbook/assets/image%20%2896%29.png)

图片 - 基于Jenkins的持续集成与发布

### 流程说明 <a id="&#x6D41;&#x7A0B;&#x8BF4;&#x660E;"></a>

应用构建和发布流程说明。

1. 用户向Gitlab提交代码，代码中必须包含`Dockerfile`
2. 将代码提交到远程仓库
3. 用户在发布应用时需要填写git仓库地址和分支、服务类型、服务名称、资源数量、实例个数，确定后触发Jenkins自动构建
4. Jenkins的CI流水线自动编译代码并打包成docker镜像推送到Harbor镜像仓库
5. Jenkins的CI流水线中包括了自定义脚本，根据我们已准备好的kubernetes的YAML模板，将其中的变量替换成用户输入的选项
6. 生成应用的kubernetes YAML配置文件
7. 更新Ingress的配置，根据新部署的应用的名称，在ingress的配置文件中增加一条路由信息
8. 更新PowerDNS，向其中插入一条DNS记录，IP地址是边缘节点的IP地址。关于边缘节点，请查看[边缘节点配置](https://rootsongjc.gitbooks.io/kubernetes-handbook/practice/edge-node-configuration.html)
9. Jenkins调用kubernetes的API，部署应用

