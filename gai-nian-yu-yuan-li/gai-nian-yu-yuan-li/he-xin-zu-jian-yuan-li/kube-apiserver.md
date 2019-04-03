# kube-apiserver

## API Server <a id="api-server"></a>

kube-apiserver 是 Kubernetes 最重要的核心组件之一，主要提供以下的功能

* 提供集群管理的 REST API 接口，包括认证授权、数据校验以及集群状态变更等
* 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才直接操作 etcd）

### APIService详解 <a id="apiservice&#x8BE6;&#x89E3;"></a>

使用`apiregistration.k8s.io/v1beta1` 版本的APIService，在metadata.name中定义该API的名字。

使用上面的yaml的创建`v1alpha1.custom-metrics.metrics.k8s.io` APIService。

* `insecureSkipTLSVerify`：当与该服务通信时，禁用TLS证书认证。强加建议不要设置这个参数，默认为 false。应该使用CABundle代替。
* `service`：与该APIService通信时引用的service，其中要注明service的名字和所属的namespace，如果为空的话，则所有的服务都会该API groupversion将在本地443端口处理所有通信。
* `groupPriorityMinimum`：该组API的处理优先级，主要排序是基于`groupPriorityMinimum`，该数字越大表明优先级越高，客户端就会与其通信处理请求。次要排序是基于字母表顺序，例如v1.bar比v1.foo的优先级更高。
* `versionPriority`：VersionPriority控制其组内的API版本的顺序。必须大于零。主要排序基于VersionPriority，从最高到最低（20大于10）排序。次要排序是基于对象名称的字母比较。 （v1.foo在v1.bar之前）由于它们都是在一个组内，因此数字可能很小，一般都小于10。

### REST API <a id="rest-api"></a>

kube-apiserver 支持同时提供 https（默认监听在 6443 端口）和 http API（默认监听在 127.0.0.1 的 8080 端口），其中 http API 是非安全接口，不做任何认证授权机制，不建议生产环境启用。两个接口提供的 REST API 格式相同，参考 [Kubernetes API Reference](https://kubernetes.io/docs/api-reference/v1.8/) 查看所有 API 的调用格式。

在实际使用中，通常通过 [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 来访问 apiserver，也可以通过 Kubernetes 各个语言的 client 库来访问 apiserver。在使用 kubectl 时，打开调试日志也可以看到每个 API 调用的格式，比如

```text
$ kubectl --v=8 get pods
```

### OpenAPI 和 Swagger <a id="openapi-&#x548C;-swagger"></a>

通过 `/swaggerapi` 可以查看 Swagger API，`/swagger.json` 查看 OpenAPI。

开启 `--enable-swagger-ui=true` 后还可以通过 `/swagger-ui` 访问 Swagger UI。

### 访问控制 <a id="&#x8BBF;&#x95EE;&#x63A7;&#x5236;"></a>

Kubernetes API 的每个请求都会经过多阶段的访问控制之后才会被接受，这包括认证、授权以及准入控制（Admission Control）等。

![](https://kubernetes.feisky.xyz/zh/components/images/access_control.png)

#### 认证 <a id="&#x8BA4;&#x8BC1;"></a>

开启 TLS 时，所有的请求都需要首先认证。Kubernetes 支持多种认证机制，并支持同时开启多个认证插件（只要有一个认证通过即可）。如果认证成功，则用户的 `username` 会传入授权模块做进一步授权验证；而对于认证失败的请求则返回 HTTP 401。

> **\[warning\] Kubernetes 不管理用户**
>
> 虽然 Kubernetes 认证和授权用到了 username，但 Kubernetes 并不直接管理用户，不能创建 `user`对象，也不存储 username。

更多认证模块的使用方法可以参考 [Kubernetes 认证插件](https://kubernetes.feisky.xyz/zh/plugins/auth.html#%20%E8%AE%A4%E8%AF%81)。

#### 授权 <a id="&#x6388;&#x6743;"></a>

认证之后的请求就到了授权模块。跟认证类似，Kubernetes 也支持多种授权机制，并支持同时开启多个授权插件（只要有一个验证通过即可）。如果授权成功，则用户的请求会发送到准入控制模块做进一步的请求验证；而对于授权失败的请求则返回 HTTP 403.

更多授权模块的使用方法可以参考 [Kubernetes 授权插件](https://kubernetes.feisky.xyz/zh/plugins/auth.html#%20%E6%8E%88%E6%9D%83)。

#### 准入控制 <a id="&#x51C6;&#x5165;&#x63A7;&#x5236;"></a>

准入控制（Admission Control）用来对请求做进一步的验证或添加默认参数。不同于授权和认证只关心请求的用户和操作，准入控制还处理请求的内容，并且仅对创建、更新、删除或连接（如代理）等有效，而对读操作无效。准入控制也支持同时开启多个插件，它们依次调用，只有全部插件都通过的请求才可以放过进入系统。

更多准入控制模块的使用方法可以参考 [Kubernetes 准入控制](https://kubernetes.feisky.xyz/zh/plugins/admission.html)。

### API Aggregation <a id="api-aggregation"></a>

API Aggregation 允许在不修改 Kubernetes 核心代码的同时扩展 Kubernetes API。

> 备注：另外一种扩展 Kubernetes API 的方法是使用 [CustomResourceDefinition \(CRD\)](https://kubernetes.feisky.xyz/zh/concepts/customresourcedefinition.html)。

#### 开启 API Aggregation <a id="&#x5F00;&#x542F;-api-aggregation"></a>

kube-apiserver 增加以下配置

```text
--requestheader-client-ca-file=<path to aggregator CA cert>
--requestheader-allowed-names=aggregator
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--proxy-client-cert-file=<path to aggregator proxy cert>
--proxy-client-key-file=<path to aggregator proxy key>
```

如果 `kube-proxy` 没有在 Master 上面运行，还需要配置

```text
--enable-aggregator-routing=true
```

#### 创建扩展 API <a id="&#x521B;&#x5EFA;&#x6269;&#x5C55;-api"></a>

1. 确保开启 APIService API（默认开启，可用 `kubectl get apiservice` 命令验证）
2. 创建 RBAC 规则
3. 创建一个 namespace，用来运行扩展的 API 服务
4. 创建 CA 和证书，用于 https
5. 创建一个存储证书的 secret
6. 创建一个部署扩展 API 服务的 deployment，并使用上一步的 secret 配置证书，开启 https 服务
7. 创建一个 ClusterRole 和 ClusterRoleBinding
8. 创建一个非 namespace 的 apiservice，注意设置 `spec.caBundle`
9. 运行 `kubectl get <resource-name>`，正常应该返回 `No resources found.`

可以使用 [apiserver-builder](https://github.com/kubernetes-incubator/apiserver-builder) 工具自动化上面的步骤。

```text
# 初始化项目
$ cd GOPATH/src/github.com/my-org/my-project
$ apiserver-boot init repo --domain <your-domain>
$ apiserver-boot init glide

# 创建资源
$ apiserver-boot create group version resource --group <group> --version <version> --kind <Kind>

# 编译
$ apiserver-boot build executables
$ apiserver-boot build docs

# 本地运行
$ apiserver-boot run local

# 集群运行
$ apiserver-boot run in-cluster --name nameofservicetorun --namespace default --image gcr.io/myrepo/myimage:mytag
$ kubectl create -f sample/<type>.yaml
```

#### 示例 <a id="&#x793A;&#x4F8B;"></a>

见 [sample-apiserver](https://github.com/kubernetes/sample-apiserver) 和 [apiserver-builder/example](https://github.com/kubernetes-incubator/apiserver-builder/tree/master/example)。

### 启动 apiserver 示例 <a id="&#x542F;&#x52A8;-apiserver-&#x793A;&#x4F8B;"></a>

```text
kube-apiserver --feature-gates=AllAlpha=true --runtime-config=api/all=true \
    --requestheader-allowed-names=front-proxy-client \
    --client-ca-file=/etc/kubernetes/pki/ca.crt \
    --allow-privileged=true \
    --experimental-bootstrap-token-auth=true \
    --storage-backend=etcd3 \
    --requestheader-username-headers=X-Remote-User \
    --requestheader-extra-headers-prefix=X-Remote-Extra- \
    --service-account-key-file=/etc/kubernetes/pki/sa.pub \
    --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
    --tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
    --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt \
    --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \
    --insecure-port=8080 \
    --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds \
    --requestheader-group-headers=X-Remote-Group \
    --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key \
    --secure-port=6443 \
    --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \
    --service-cluster-ip-range=10.96.0.0/12 \
    --authorization-mode=RBAC \
    --advertise-address=192.168.0.20 --etcd-servers=http://127.0.0.1:2379
```

### 工作原理 <a id="&#x5DE5;&#x4F5C;&#x539F;&#x7406;"></a>

kube-apiserver 提供了 Kubernetes 的 REST API，实现了认证、授权、准入控制等安全校验功能，同时也负责集群状态的存储操作（通过 etcd）。

![](https://kubernetes.feisky.xyz/zh/components/images/kube-apiserver.png)

### API 访问 <a id="api-&#x8BBF;&#x95EE;"></a>

有多种方式可以访问 Kubernetes 提供的 REST API：

* [kubectl](https://kubernetes.feisky.xyz/zh/components/kubectl.html) 命令行工具
* SDK，支持多种语言
  * [Go](https://github.com/kubernetes/client-go)
  * [Python](https://github.com/kubernetes-incubator/client-python)
  * [Javascript](https://github.com/kubernetes-client/javascript)
  * [Java](https://github.com/kubernetes-client/java)
  * [CSharp](https://github.com/kubernetes-client/csharp)
  * 其他 [OpenAPI](https://www.openapis.org/) 支持的语言，可以通过 [gen](https://github.com/kubernetes-client/gen) 工具生成相应的 client

#### kubectl <a id="kubectl"></a>

```text
kubectl get --raw /api/v1/namespaces
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

#### kubectl proxy <a id="kubectl-proxy"></a>

```text
$ kubectl proxy --port=8080 &

$ curl http://localhost:8080/api/
{
  "versions": [
    "v1"
  ]
}
```

#### curl <a id="curl"></a>

```text
$ APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")
$ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d '') | grep -E'^token'| cut -f2 -d':'| tr -d'\t')
$ curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

### 查看集群支持的APISerivce

作为Kubernetes中的一种资源对象，可以使用`kubectl get apiservice`来查看。

例如查看集群中所有的APIService：

```text
$ kubectl get apiservice
NAME                                     AGE
v1.                                      2d
v1.authentication.k8s.io                 2d
v1.authorization.k8s.io                  2d
v1.autoscaling                           2d
v1.batch                                 2d
v1.monitoring.coreos.com                 1d
v1.networking.k8s.io                     2d
v1.rbac.authorization.k8s.io             2d
v1.storage.k8s.io                        2d
v1alpha1.custom-metrics.metrics.k8s.io   2h
v1beta1.apiextensions.k8s.io             2d
v1beta1.apps                             2d
v1beta1.authentication.k8s.io            2d
v1beta1.authorization.k8s.io             2d
v1beta1.batch                            2d
v1beta1.certificates.k8s.io              2d
v1beta1.extensions                       2d
v1beta1.policy                           2d
v1beta1.rbac.authorization.k8s.io        2d
v1beta1.storage.k8s.io                   2d
v1beta2.apps                             2d
v2beta1.autoscaling                      2d
```

另外查看当前kubernetes集群支持的API版本还可以使用`kubectl api-version`：

```text
$ kubectl api-versions
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1beta1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
```

### API 参考文档 <a id="api-&#x53C2;&#x8003;&#x6587;&#x6863;"></a>

* [v1.6 API Reference](https://kubernetes.io/docs/api-reference/v1.6)
* [v1.7 API Reference](https://kubernetes.io/docs/api-reference/v1.7/)
* [v1.8 API Reference](https://kubernetes.io/docs/api-reference/v1.8/)
* [v1.9 API Reference](https://kubernetes.io/docs/api-reference/v1.9/)
* [v1.10 API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/)
*  [API Conventions](https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md#resources)

