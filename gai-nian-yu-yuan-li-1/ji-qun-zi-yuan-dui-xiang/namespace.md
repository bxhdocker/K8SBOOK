# Namespace

## 介绍

在一个Kubernetes集群中可以使用namespace创建多个“虚拟集群”，这些namespace之间可以完全隔离，也可以通过某种方式，让一个namespace中的service可以访问到其他的namespace中的服务，我们[在kubernetes1.6集群](https://jimmysong.io/kubernetes-handbook/practice/install-kubernetes-on-centos.html)的时候就用到了好几个跨越namespace的服务，比如Traefik ingress和`kube-system`namespace下的service就可以为整个集群提供服务，这些都需要通过RBAC定义集群级别的角色来实现。

#### 哪些情况下适合使用多个namespace

因为namespace可以提供独立的命名空间，因此可以实现部分的环境隔离。当你的项目和人员众多的时候可以考虑根据项目属性，例如生产、测试、开发划分不同的namespace。

## Namespace使用

 `kubectl` 可以通过 `--namespace` 或者 `-n` 选项指定 namespace。如果不指定，默认为 default。查看操作下, 也可以通过设置 --all-namespace=true 来查看所有 namespace 下的资源。

###  查询

**获取集群中有哪些namespace**

```text
kubectl get ns
```

 注意：namespace 包含两种状态 "**Active**" 和 "**Terminating**"。在 namespace 删除过程中，namespace 状态被设置成 "Terminating"。

集群中默认会有`default`和`kube-system`这两个namespace。

在执行`kubectl`命令时可以使用`-n`指定操作的namespace。

用户的普通应用默认是在`default`下，与集群管理相关的为整个集群提供服务的应用一般部署在`kube-system`的namespace下，例如我们在安装kubernetes集群时部署的`kubedns`、`heapseter`、`EFK`等都是在这个namespace下面。

另外，并不是所有的资源对象都会对应namespace，`node`和`persistentVolume`就不属于任何namespace。

###  创建

```text
(1) 命令行直接创建
$ kubectl create namespace new-namespace
​
(2) 通过文件创建
$ cat my-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: new-namespace
​
$ kubectl create -f ./my-namespace.yaml
```

 注意：命名空间名称满足正则表达式 `[a-z0-9]([-a-z0-9]*[a-z0-9])?`, 最大长度为 63 位

###  删除

```text
$ kubectl delete namespaces new-namespace
```

注意：

1. 删除一个 namespace 会自动删除所有属于该 namespace 的资源。
2. `default` 和 `kube-system` 命名空间不可删除。
3. PersistentVolume 是不属于任何 namespace 的，但 PersistentVolumeClaim 是属于某个特定 namespace 的。
4. Event 是否属于 namespace 取决于产生 event 的对象。
5. v1.7 版本增加了 `kube-public` 命名空间，该命名空间用来存放公共的信息，一般以 ConfigMap 的形式存放。

```text
$ kubectl get configmap  -n=kube-public
NAME           DATA      AGE
cluster-info   2         29d
```



## 参考文档 <a id="can-kao-wen-dang"></a>

* ​[Kubernetes Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)​
* ​[Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)​
* ~~~~[namespace说明](https://kubernetes.feisky.xyz/he-xin-yuan-li/index-2/namespace)

