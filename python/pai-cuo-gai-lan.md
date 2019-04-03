# 排错概览

Kubernetes 集群以及应用排错的一般方法，主要包括

* ​[集群状态异常排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/cluster)​
* ​[Pod运行异常排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/pod)​
* ​[网络异常排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/network)​
* ​[持久化存储异常排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/pv)​
  * ​[AzureDisk 排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/pv/azuredisk)​
  * ​[AzureFile 排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/pv/azurefile)​
* ​[Windows容器排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/windows)​
* ​[云平台异常排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/cloud)​
  * ​[Azure 排错](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/cloud/azure)​
* ​[常用排错工具](https://kubernetes.feisky.xyz/pai-cuo-zhi-nan/tools)​

在排错过程中，`kubectl` 是最重要的工具，通常也是定位错误的起点。这里也列出一些常用的命令，在后续的各种排错过程中都会经常用到。

## 查看 Pod 状态以及运行节点 <a id="cha-kan-pod-zhuang-tai-yi-ji-yun-hang-jie-dian"></a>

```text
kubectl get pods -o widekubectl -n kube-system get pods -o wide
```

## 查看 Pod 事件 <a id="cha-kan-pod-shi-jian"></a>

```text
kubectl describe pod <pod-name>
```

## 查看 Node 状态 <a id="cha-kan-node-zhuang-tai"></a>

```text
kubectl get nodeskubectl describe node <node-name>
```

## kube-apiserver 日志 <a id="kubeapiserver-ri-zhi"></a>

```text
PODNAME=$(kubectl -n kube-system get pod -l component=kube-apiserver -o jsonpath='{.items[0].metadata.name}')kubectl -n kube-system logs $PODNAME --tail 100
```

以上命令操作假设控制平面以 Kubernetes 静态 Pod 的形式来运行。如果 kube-apiserver 是用 systemd 管理的，则需要登录到 master 节点上，然后使用 journalctl -u kube-apiserver 查看其日志。

## kube-controller-manager 日志 <a id="kubecontrollermanager-ri-zhi"></a>

```text
PODNAME=$(kubectl -n kube-system get pod -l component=kube-controller-manager -o jsonpath='{.items[0].metadata.name}')kubectl -n kube-system logs $PODNAME --tail 100
```

以上命令操作假设控制平面以 Kubernetes 静态 Pod 的形式来运行。如果 kube-controller-manager 是用 systemd 管理的，则需要登录到 master 节点上，然后使用 journalctl -u kube-controller-manager 查看其日志。

## kube-scheduler 日志 <a id="kubescheduler-ri-zhi"></a>

```text
PODNAME=$(kubectl -n kube-system get pod -l component=kube-scheduler -o jsonpath='{.items[0].metadata.name}')kubectl -n kube-system logs $PODNAME --tail 100
```

以上命令操作假设控制平面以 Kubernetes 静态 Pod 的形式来运行。如果 kube-scheduler 是用 systemd 管理的，则需要登录到 master 节点上，然后使用 journalctl -u kube-scheduler 查看其日志。

## kube-dns 日志 <a id="kubedns-ri-zhi"></a>

kube-dns 通常以 Addon 的方式部署，每个 Pod 包含三个容器，最关键的是 kubedns 容器的日志：

```text
PODNAME=$(kubectl -n kube-system get pod -l k8s-app=kube-dns -o jsonpath='{.items[0].metadata.name}')kubectl -n kube-system logs $PODNAME -c kubedns
```

## Kubelet 日志 <a id="kubelet-ri-zhi"></a>

Kubelet 通常以 systemd 管理。查看 Kubelet 日志需要首先 SSH 登录到 Node 上。

```text
journalctl -l -u kubelet
```

## Kube-proxy 日志 <a id="kubeproxy-ri-zhi"></a>

Kube-proxy 通常以 DaemonSet 的方式部署，可以直接用 kubectl 查询其日志

```text
$ kubectl -n kube-system get pod -l component=kube-proxyNAME               READY     STATUS    RESTARTS   AGEkube-proxy-42zpn   1/1       Running   0          1dkube-proxy-7gd4p   1/1       Running   0          3dkube-proxy-87dbs   1/1       Running   0          4d$ kubectl -n kube-system logs kube-proxy-42zpn
```

