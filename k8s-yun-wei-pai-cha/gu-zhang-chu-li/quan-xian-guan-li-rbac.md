# 权限管理RBAC

Kubernetes集群的所有操作基本上都是通过kube-apiserver这个组件进行的，它提供HTTP [RESTful](https://www.centos.bz/tag/restful/)形式的API供集群内外客户端调用。需要注意的是：认证授权过程只存在[HTTPS](https://www.centos.bz/tag/https/)形式的API中。也就是说，如果客户端使用HTTP连接到kube-apiserver，那么是不会进行认证授权的。所以说，可以这么设置，在集群内部组件间通信使用HTTP，集群外部就使用HTTPS，这样既增加了安全性，也不至于太复杂。

![](../../.gitbook/assets/image%20%28158%29.png)

![](../../.gitbook/assets/image%20%28148%29.png)

![](../../.gitbook/assets/image%20%28121%29.png)

基本结构主要就是三个:

* 权限: 即对系统中指定资源的增删改查权限
* 角色: 将一定的权限组合在一起产生权限组，如管理员角色
* 用户: 具体的使用者，具有唯一身份标识\(ID\)，其后与角色绑定便拥有角色的对应权限

**Kubernetes 是不负责维护存储用户数据的；对于 Kubernetes 来说，它识别或者说认识一个用户主要就几种方式**

* X509 Client Certs: 使用由 k8s 根 CA 签发的证书，提取 O 字段
* Static Token File: 预先在 API Server 放置 Token 文件\(bootstrap 阶段使用过\)
* Bootstrap Tokens: 一种在集群内创建的 Bootstrap 专用 Token\(新的 Bootstarp 推荐\)
* Static Password File: 跟静态 Token 类似
* Service Account Tokens: 使用 Service Account 的 Token

其他不再一一列举，具体请看文档 [Authenticating](https://kubernetes.io/docs/admin/authentication/)；了解了这些，后面我们使用 RBAC 控制 kubectl 权限的时候就要使用如上几种方法创建对应用户

RBAC 权限定义部分主要有三个层级

* apiGroups: 指定那个 API 组下的权限
* resources: 该组下具体资源，如 pod 等
* verbs: 指对该资源具体执行哪些动作

定义一组权限\(角色\)时要根据其所需的真正需求做最细粒度的划分

下图是 API 访问要经过的三个步骤，前面两个是认证和授权，第三个是 Admission Control

client发起apiserver调用时，apiserver先**认证\(Authentication\)用户，再给用户**授权\(Authorization\)，最后执行**准入控制\(Admission Control\)**

* 认证：鉴别发起请求的**用户**是否合法
* 授权：授予用户**api请求**权限，并鉴别用户访问的**api请求**是否合法
* 准入控制：一些额外的检查机制，用于做最后的拦截

![](../../.gitbook/assets/image%20%28112%29.png)



#### TLS原理 <a id="h3_4"></a>

基于CA根证书签名的双向数字证书认证

* client和server向CA机构申请证书
* client-&gt;server，server下发服务端证书，client利用证书验证server是否合法
* client发送客户端证书给server，server利用证书校验client是否合法
* client和server利用随机密钥加密消息，然后通信

{% embed url="https://my.oschina.net/u/1378920/blog/1807839" %}



## 授权 <a id="h1_24"></a>

授权就是授予用户请求权限，并鉴别用户的api请求是否合法的过程

目前支持的几种授权策略：

* AlwaysDeny：拒绝所有请求，用于测试
* AlwaysAllow：允许所有请求，k8s默认策略
* ABAC：基于属性的访问控制。表示使用用户配置的规则对用户请求进行控制
* RBAC：基于角色的访问控制
* Webhook：通过调用外部rest服务对用户授权



RBAC的四个资源对象：

1. Role：一个角色就是一组权限的集合，拥有某个namespace下的权限
2. ClusterRole：集群角色，拥有整个cluster下的权限
3. RoleBinding：将Role绑定目标（user、group、service account）
4. ClusterRoleBinding：将ClusterRoleBinding绑定目标

一个典型的映射关系

![image\_1cc5o0qih1t61goh1ful9surla9.png-93.4kB](http://static.zybuluo.com/mjaow/zhkj02uys1gmycznw8tqs9gc/image_1cc5o0qih1t61goh1ful9surla9.png)

##  <a id="h1_28"></a>

为kubernetes 访问用户添加权限控制

了方便增加了一个`admin`用户，然后授予了`cluster-admin`的角色绑定，而该角色绑定是系统内置的一个超级管理员权限，也就是用该用户的`token`登录`Dashboard`后会很**强势**，什么权限都有，想干嘛干嘛，这样的操作显然是非常危险的。接下来我们来为一个新的用户添加访问权限控制。

### Role <a id="role"></a>

`Role`表示是一组规则权限，只能累加，`Role`可以定义在一个`namespace`中，只能用于授予对单个命名空间中的资源访问的权限。比如我们新建一个对默认命名空间中`Pods`具有访问权限的角色：

```text
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### ClusterRole <a id="clusterrole"></a>

`ClusterRole`具有与`Role`相同的权限角色控制能力，不同的是`ClusterRole`是集群级别的，可以用于:

* 集群级别的资源控制\(例如 node 访问权限\)
* 非资源型 endpoints\(例如 /healthz 访问\)
* 所有命名空间资源控制\(例如 pods\)

比如我们要创建一个授权某个特定命名空间或全部命名空间\(取决于绑定方式\)访问**secrets**的集群角色：

```text
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding和ClusterRoleBinding <a id="rolebinding&#x548C;clusterrolebinding"></a>

`RoloBinding`可以将角色中定义的权限授予用户或用户组，`RoleBinding`包含一组权限列表\(`subjects`\)，权限列表中包含有不同形式的待授予权限资源类型\(users、groups、service accounts\)，`RoleBinding`适用于某个命名空间内授权，而 `ClusterRoleBinding`适用于集群范围内的授权。

比如我们将默认命名空间的`pod-reader`角色授予用户jane，这样以后该用户在默认命名空间中将具有`pod-reader`的权限：

```text
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

`RoleBinding`同样可以引用`ClusterRole`来对当前 namespace 内用户、用户组或 ServiceAccount 进行授权，这种操作允许集群管理员在整个集群内定义一些通用的 ClusterRole，然后在不同的 namespace 中使用 RoleBinding 来引用

例如，以下 RoleBinding 引用了一个 ClusterRole，这个 ClusterRole 具有整个集群内对 secrets 的访问权限；但是其授权用户 dave 只能访问 development 空间中的 secrets\(因为 RoleBinding 定义在 development 命名空间\)

```text
# This role binding allows "dave" to read secrets in the "development" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: read-secrets
  namespace: development # This only grants permissions within the "development" namespace.
subjects:
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

最后，使用 ClusterRoleBinding 可以对整个集群中的所有命名空间资源权限进行授权；以下 ClusterRoleBinding 样例展示了授权 manager 组内所有用户在全部命名空间中对 secrets 进行访问

```text
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```







案例实践;[https://juejin.im/entry/5a615bbb6fb9a01cb42c6d9a](https://juejin.im/entry/5a615bbb6fb9a01cb42c6d9a)

