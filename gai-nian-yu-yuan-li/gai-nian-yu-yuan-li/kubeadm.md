---
description: kubeadm工作原理
---

# kubeadm



kubeadm是Kubernetes主推的部署工具之一，正在快速迭代开发中。

### 初始化系统 <a id="&#x521D;&#x59CB;&#x5316;&#x7CFB;&#x7EDF;"></a>

所有机器都需要初始化容器执行引擎（如docker或frakti等）和kubelet。这是因为kubeadm依赖kubelet来启动Master组件，比如kube-apiserver、kube-manager-controller、kube-scheduler、kube-proxy等。

### 安装master <a id="&#x5B89;&#x88C5;master"></a>

在初始化master时，只需要执行kubeadm init命令即可，比如

```text
kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version stable
```

这个命令会自动

* 系统状态检查
* 生成token
* 生成自签名CA和client端证书
* 生成kubeconfig用于kubelet连接API server
* 为Master组件生成Static Pod manifests，并放到`/etc/kubernetes/manifests`目录中
* 配置RBAC并设置Master node只运行控制平面组件
* 创建附加服务，比如kube-proxy和kube-dns

### 配置Network plugin <a id="&#x914D;&#x7F6E;network-plugin"></a>

kubeadm在初始化时并不关心网络插件，默认情况下，kubelet配置使用CNI插件，这样就需要用户来额外初始化网络插件。

#### CNI bridge <a id="cni-bridge"></a>

```text
mkdir -p /etc/cni/net.d
cat >/etc/cni/net.d/10-mynet.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.244.1.0/24",
        "routes": [
            { "dst": "0.0.0.0/0"  }
        ]
    }
}
EOF
cat >/etc/cni/net.d/99-loopback.conf <<-EOF
{
    "cniVersion": "0.3.0",
    "type": "loopback"
}
EOF
```

#### flannel <a id="flannel"></a>

```text
kubectl create -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel-rbac.yml
kubectl create -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```

#### weave <a id="weave"></a>

```text
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

#### calico <a id="calico"></a>

```text
kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
```

### 添加Node <a id="&#x6DFB;&#x52A0;node"></a>

```text
token=$(kubeadm token list | grep authentication,signing | awk '{print $1}')
kubeadm join --token $token ${master_ip}
```

这包括以下几个步骤

* 从API server下载CA
* 创建本地证书，并请求API Server签名
* 最后配置kubelet连接到API Server

### 删除安装 <a id="&#x5220;&#x9664;&#x5B89;&#x88C5;"></a>

```text
kubeadm reset
```

### 参考文档 <a id="&#x53C2;&#x8003;&#x6587;&#x6863;"></a>

* [kubeadm Setup Tool](https://kubernetes.io/docs/admin/kubeadm/)

