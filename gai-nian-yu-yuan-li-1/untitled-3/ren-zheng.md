# 认证

开启 TLS 时，所有的请求都需要首先认证。Kubernetes 支持多种认证机制，并支持同时开启多个认证插件（只要有一个认证通过即可）。如果认证成功，则用户的 `username` 会传入授权模块做进一步授权验证；而对于认证失败的请求则返回 HTTP 401。

> **Kubernetes 不直接管理用户**
>
> 虽然 Kubernetes 认证和授权用到了 username，但 Kubernetes 并不直接管理用户，不能创建 `user` 对象， 也不存储 username。但是 Kubernetes 提供了 Service Account，用来与 API 交互。

目前，Kubernetes 支持以下认证插件：

* X509 证书
* 静态 Token 文件
* 引导 Token
* 静态密码文件
* Service Account
* OpenID
* Webhook
* 认证代理
* OpenStack Keystone 密码

## Service Account <a id="service-account"></a>

ServiceAccount 是 Kubernetes 自动生成的，并会自动挂载到容器的 `/var/run/secrets/kubernetes.io/serviceaccount` 目录中。

在认证时，ServiceAccount 的用户名格式为 `system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT)`，并从属于两个 group：`system:serviceaccounts` 和 `system:serviceaccounts:(NAMESPACE)`。

##  认证代理

API Server 需要配置

```text
--requestheader-username-headers=X-Remote-User
--requestheader-group-headers=X-Remote-Group
--requestheader-extra-headers-prefix=X-Remote-Extra-
# 为了防止头部欺骗，证书是必选项
--requestheader-client-ca-file
# 设置允许的 CN 列表。可选。
--requestheader-allowed-names
```

## Credential Plugin <a id="credential-plugin"></a>

从 v1.11 开始支持 Credential Plugin（Beta），通过调用外部插件来获取用户的访问凭证。这是一种客户端认证插件，用来支持不在 Kubernetes 中内置的认证协议，如 LDAP、OAuth2、SAML 等。它通常与 [Webhook](https://kubernetes.feisky.xyz/cha-jian-kuo-zhan/auth/authentication#webhook) 配合使用。

 Credential Plugin 可以在 kubectl 的配置文件中设置，比如

```text
apiVersion: v1
kind: Config
users:
- name: my-user
  user:
    exec:
      # Command to execute. Required.
      command: "example-client-go-exec-plugin"
​
      # API version to use when decoding the ExecCredentials resource. Required.
      #
      # The API version returned by the plugin MUST match the version listed here.
      #
      # To integrate with tools that support multiple versions (such as client.authentication.k8s.io/v1alpha1),
      # set an environment variable or pass an argument to the tool that indicates which version the exec plugin expects.
      apiVersion: "client.authentication.k8s.io/v1beta1"
​
      # Environment variables to set when executing the plugin. Optional.
      env:
      - name: "FOO"
        value: "bar"
​
      # Arguments to pass when executing the plugin. Optional.
      args:
      - "arg1"
      - "arg2"
clusters:
- name: my-cluster
  cluster:
    server: "https://172.17.4.100:6443"
    certificate-authority: "/etc/kubernetes/ca.pem"
contexts:
- name: my-cluster
  context:
    cluster: my-cluster
    user: my-user
current-context: my-cluster
```

 具体的插件开发及使用方法请参考 [kubernetes/client-go](https://github.com/kubernetes/client-go/tree/master/plugin/pkg/client/auth)。

原文链接：[认证](https://kubernetes.feisky.xyz/cha-jian-kuo-zhan/auth/authentication)

