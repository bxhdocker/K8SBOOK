# CKA认证

关于CKA考试CKA（Certified Kubernetes Administrator）是CNCF基金会（Cloud Native Computing Foundation）官方推出的Kubernetes管理员认证计划，用于证明持有人有履行Kubernetes管理的知识，技能等相关的能力。除了对于个人技能提供认证以外，如果企业想要加入CNCF基金会的KCSP计划（Kubernetes认证服务提供商）也需要企业至少有3名及以上工程师的通过CKA认证考试。考试注册地址：[https://www.cncf.io/certification/expert/](https://www.cncf.io/certification/expert/)

CKA考试费用参与人员需要支付300$的考试费用，因此一张双币信用卡是必须的。

### 1.考前准备

#### 1.1 报考相关

报考地址。[https://www.cncf.io/certification/cka](https://www.cncf.io/certification/cka)

购买了CKA考试，完成后CNFC会给出Exam Preparation Checklist![](//upload-images.jianshu.io/upload_images/6553110-56519377689b1398?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)cka1

![](../.gitbook/assets/image%20%28134%29.png)

**1、有效期一年**。在一年内需要定好考试的时间。

2、**提前15分钟进入考试系统，** 提前进入考试系统后并不是立马开始考试，而是预留给考官时间考察你的考试环境

3、**考试时间** ，注意报考的TimeZone。**默认是UTC时间，请改成China Time。**

4、**修改考试时间**，考试前24小时且工作日内可更改。  


1.1 CKA/CKAD考纲

考纲隔几个月会更新，考前请留意GitHub上Latest commit 时间，一般变动不大，留意考点分值比重，以及考点中的细项![](//upload-images.jianshu.io/upload_images/6553110-524c379de92bf7b3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)github

#### 1.2 考生手册

这部分基本变动不大，除了分值，考纲同步更新。文档中几点需要留意

**Candidate Requirements（考试环境要求） ：**

1、谷歌浏览器、稳定的网络、摄像头、麦克风；

> 避坑提示：
>
> 重点提一下摄像头。考试前请充分测试摄像头，考官会要求你在摄像头前出示护照，摄像头要能够对焦并清晰地拍摄到上面的文字。
>
> 实践证明**罗技 C270i**不能自动对焦。

2、干净整洁的桌面。桌面、桌底、键盘下上不得放任何东西。

3、摄像头的线要够长，允许移动。（笔记本就不用担心了）

**Retake Policy**

重考策略，有一次免费重考的机会。一年后过期

**Exam Rules and Policies**

考试规则，这个熟读一下，环境要求必须安静、考生背后不能有玻璃或者强光，考试期间不能大声朗读，不能吃、喝。除非要求暂停考试（暂停考试不停止时间）、考试期间脸不能离开摄像头、不允许任何电子设备，不允许在考试系统外记录任何笔记。等等

**Policy on Warnings and Exam Terminations**

**Exam Misconduct**

这两块也大致浏览一下。避坑，否则考官会无条件终止你的考试。

注意，只允许打开多一个Chrome的 Tab，Tab只允许[https://kubernetes.io/docs/](https://kubernetes.io/docs/)和[https://kubernetes.io/blog/](https://kubernetes.io/blog/)。使用官方文档内的搜索功能可能会搜出除上述两个地址之外的链接，要注意甄别！

**考生有义务甄别不是上述两个地址的网址，千万不要点开！**

#### 1.3 考试Tip手册

主要看两部分：**Technical Instructions** 、**CKA & CKAD Environment** ；这份文档中，Technical Instructions与考生手册重复了。主要留意一下Ctrl+W 和 Ctrl+C 、Ctrl+V的替代键。以及考试中的k8s环境，要用到

kubectl config use-context &lt;环境名称&gt;进行替换，题目中会有提醒。

#### 1.4 官方文档

除了HOME CONTRIBUTE之外，其余栏目都要仔细看过一遍。SETUP CONCEPTS TASKS TUTORIALS REFERENCE里面的与考纲相关的均不能放过，我就因为大意漏了**kubectl Cheat Sheet**，超简单的第一题加上考试系统故障，浪费了快10min（涉及GCE等云服务的、集群联邦除外，不在考点内，可不看），英文文档难啃，难免有遗漏，**建议结合JimmySong的**[**Kubernetes-handbook**](https://jimmysong.io/kubernetes-handbook/)**加深印象**。最好使用二进制部署一遍，尤其是kubelet等组件的配置，再回到英文doc定位考点。

### 2、考中避坑经验

#### 2.1 与考官交流

​ 考试全程与考官在Live Chat上英文交流（文字）,考前考官会跟你提示注意事项。考验平时英语作文词汇量的时候到了。国外的考官都很人性化的，考试中有意外状况，询问考官，并向考官争取你应有的权益。举例一下我遇到的 场景（基本上是大意，英文我就不写了，渣渣）：

1、考试正式开始前，罗技摄像头无法清晰看到护照，向考官申请使用笔记本电脑摄像头验证。验证完成后请求考官使用回台式机重新考试（屏幕大有好处）

2、考试过程中，考试系统崩溃，提示，lost connection，询问考官什么情况，是我电脑的问题还是系统问题。考试的中断是否会影响我的计时。我能延长多少时间等等。

3、考试中如果有半边脸不小心离开摄像头，考官会在Live Chat会弹出窗口提示你，这时候请注意。并向考官表达歉意。

#### 2.2 考试中的技巧

**1、审题：**

审题一定要仔细再仔细。题目有中文翻译的版本，但标点符号特别少，断句可能会出现问题，建议切换成英文版本再看一遍。

**2、答题策略**

整场考试总计24道题，分值高的题目花费的时间较多，我遇见的是集群故障排查、和TLS Bootstraping，以及节点失联、一般较为靠后。但不是所有分值高的题目都在最后，（我的最后一题是创建PV，2%）。**合理规划答题策略。**

**3、网络问题**

根据同行交流经验所得，CKA考试普遍网络较慢，表现为考试系统Lost Connection，终端反应较慢、无法进入终端的情况:

**Lost Connection**：在考试计时器停止后，考官会给你相应的额外时间。

**无法进入终端**：这种情况使用考试系统上的reflash exam window按钮。刷新后重新进入终端，此时会退出特权模式，需要重新使用sudo -i 进入。且bash环境需要重新配置。（bash这块不确定，反正kubectl的自动补全我是执行了好几次）

**终端反应慢**：这块无解，只能重其他技巧节省时间。

> **不要刷新浏览器！不要刷新浏览器！不要刷新浏览器！**刷新浏览器会导致考试被终止。

**4、节省时间的技巧**

* 考试题目刷出来较慢，建议开考时，将考题列记录考试系统notepad里面，过一遍（后悔没这么做）。
* kubectl bash自动补全的命令一定要用，最好记录在notepad上。切换环境，关键时刻可以复制粘贴到终端内。

  ```text
  source <(kubectl completion bash)
  ```

* 考试中的终端是可以复制黏贴的、但是鼠标姿势要对，多尝试几次（我一开始以为复制不了，白敲了很久），无论如何请多使用终端内复制粘贴的功能（Windows下是Ctrl+Insect复制，Shift+Insect）。终端内复制粘贴一个yaml是毫无压力的，不要被Important Tip和考生手册给误导了。

  > Important Tip和考生手册里面的Technical Instructions有这么一句话，容易被误导
  >
  > 1. Ctrl+C & and Ctrl+V are not supported in your exam terminal, nor is copy and pasting large amounts of text. To copy and paste **limited amounts of text \(1−2 lines\)** please use;

* kubectl explain 查定义，太慢了，查个pod.spce.containers.livenessProbe.initialDelaySeconds只能一层层的查，可能是我不熟悉grep + 正则（脑子转的不够快）乖乖在doc上找案例Shift+Insert（粘贴）更快
* 善用官方文档的搜索功能，记住一些考点的关键字。有些知识点在CONCEPTS、TASKS、TUTORIALS、REFERENCE 连番出现，不好查找，搜索关键字较长时，搜索效率会低下。

  > 比如，查找Pod和Service的解析记录，搜索组合关键字Service DNS效率太低，不如直接搜索nslookup

* 与考点相关的文档可以提前加入收藏夹，利用地址栏自动补全功能跳转到对应的文档 @Kevin Wang
* 避免手敲yaml，能不手敲yaml的，就别手敲，尽量从文档上复制，手敲的效率太低了，碰上终端响应慢，那是时间的杀手！！
* 尽量使用命令创建Pod、deployment、service

  ```text
  #创建Pod
  kubectl run <podname> --image=<imagename> --restart=Never -n <namespace>
  #创建Deployment
  kubectl run <deploymentname> --image=<imagename> -n <namespace>
  #暴露Service
  kubectl expose <deploymentname> --port=<portNo.> --name=<svcname>
  ```

* 养成使用--dry-run、kubectl apply -f、kubectl delete -f的习惯，同时将答案和yaml写入到文件里，方便根据命令模板修改yaml，以及后面检查答案时重做题目。举个例子，使用run命令忘记增加namespace了

  ```text
  #初次生成
  kubectl run <podname> --image=<imagename> --restart=Never --dry-run -o yaml > <题目名称>.yaml
  #应用yaml
  kubectl apply -f  <题目名称>.yaml
  #审题错了，删除之前做的结果
  kubectl delete -f  <题目名称>.yaml
  #修改命令或修改yaml重新执行kubectl apply -f
  ```

* 见到不懂或忘记的命令参数，甭管什么kubectl，etcdctl的命令，~~盘他~~ -h，-h是万能的。里面有示例
* 最后，考试环境中一定要留意k8s环境和尤其是主机名和用户名，默认可以使用ssh 进入 node， 进入node做完题目后记得退出，不然节点内是无法ssh到下一个题目的node当中去的，节点内终端一般显示student@&lt;worknodename&gt;

### 3、题目与解答

重点来了。由于考试不允许将考题记录在考试系统外，凭着自己的记忆，加上浏览器历史纪录，记录下22题（总共24题），内容有部分差异，考题中的pod之类的镜像、命名肯定都记不清了，凭空造的，请见谅。

1. **列出环境内所有的pv 并以 name字段排序（使用kubectl自带排序功能）**

   ```text
    kubectl get pv --sort-by=.metadata.name
   ```

考点：kubectl命令熟悉程度

1. **列出指定pod的日志中状态为Error的行，并记录在指定的文件上**

   ```text
   kubectl logs <podname> | grep bash > /opt/KUCC000xxx/KUCC000xxx.txt
   ```

   考点：Monitor, Log, and Debug

2. **列出k8s可用的节点，不包含不可调度的 和 NoReachable的节点，并把数字写入到文件里**

   ```text
   #笨方法，人工数
   kubectl get nodes

   #CheatSheet方法，应该还能优化JSONPATH
   JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
    && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"
   ```

   考点：kubectl命令熟悉程度

   参考：[kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

3. **创建一个pod名称为nginx，并将其调度到节点为 disk=stat上**

   ```text
   #我的操作,实际上从文档复制更快
   kubectl run nginx --image=nginx --restart=Never --dry-run > 4.yaml
   #增加对应参数
   vi 4.yaml
   kubectl apply -f 4.yaml
   ```

```text
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

考点：pod的调度。

参考：[assign-pod-node](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)

1. **提供一个pod的yaml，要求添加Init Container，Init Container的作用是创建一个空文件，pod的Containers判断文件是否存在，不存在则退出**

   ```text
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
   spec:
     containers:
     - name: apline
       image: nginx
       command: ['sh', '-c', 'if [目录下有work文件];then sleep 3600; else exit; fi;']
   ###增加init Container####
   initContainers:
    - name: init
       image: busybox
       command: ['sh', '-c', 'touch /目录/work;']
    
   ```

   考点：init Container。一开始审题不仔细，以为要用到livenessProbes

   参考：[init-containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

2. **指定在命名空间内创建一个pod名称为test，内含四个指定的镜像nginx、redis、memcached、busybox**

   ```text
   kubectl run test --image=nginx --image=redis --image=memcached --image=buxybox --restart=Never -n <namespace>
   ```

   考点：kubectl命令熟悉程度、多个容器的pod的创建

   参考：[kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

3. **创建一个pod名称为test，镜像为nginx，Volume名称cache-volume为挂在在/data目录下，且Volume是non-Persistent的**

   ```text
   apiVersion: v1
   kind: Pod
   metadata:
     name: test
   spec:
     containers:
     - image: nginx
       name: test-container
       volumeMounts:
       - mountPath: /cache
         name: cache-volume
     volumes:
     - name: cache-volume
       emptyDir: {}
   ```

   考点：Volume、emptdir

   参考：[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

4. **列出Service名为test下的pod 并找出使用CPU使用率最高的一个，将pod名称写入文件中**

   ```text
   #使用-o wide 获取service test的SELECTOR
   kubectl get svc test -o wide
   ##获取结果我就随便造了
   NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE       SELECTOR
   test   ClusterIP   None         <none>        3306/TCP   50d       app=wordpress,tier=mysql

   #获取对应SELECTOR的pod使用率，找到最大那个写入文件中
   kubectl top test -l 'app=wordpress,tier=mysql'
   ```

   考点：获取service selector，kubectl top监控pod资源

   参考：[Tools for Monitoring Resources](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring)

5. **创建一个Pod名称为nginx-app，镜像为nginx，并根据pod创建名为nginx-app的Service，type为NodePort**

   ```text
   kubectl run nginx-app --image=nginx --restart=Never --port=80
   kubectl create svc nodeport nginx-app --tcp=80:80 --dry-run -o yaml > 9.yaml
   #修改yaml，保证selector name=nginx-app
   vi 9.yaml
   ```

   ```text
   apiVersion: v1
   kind: Service
   metadata:
     creationTimestamp: null
     labels:
       app: nginx-app
     name: nginx-app
   spec:
     ports:
     - name: 80-80
       port: 80
       protocol: TCP
       targetPort: 80
     selector:
   #注意要和pod对应  
       name: nginx-app
     type: NodePort
   ```

   考点：Service

   参考：[publishing-services-service-types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

6. **创建一个nginx的Workload，保证其在每个节点上运行，注意不要覆盖节点原有的Tolerations**

   这道题直接复制文档的yaml太长了，由于damonSet的格式和Deployment格式差不多，我用旁门左道的方法 先创建Deploy，再修改，这样速度会快一点

   ```text
   #先创建一个deployment的yaml模板
   kubectl run nginx --image=nginx --dry-run -o yaml > 10.yaml
   #将yaml改成DaemonSet
   vi 10.yaml
   ```

   ```text
   #修改apiVersion和kind
   #apiVersion: extensions/v1beta1
   #kind: Deployment
   apiVersion:apps/v1
   kind: DaemonSet
   metadata:
     creationTimestamp: null
     labels:
       run: nginx
     name: nginx
   spec:
   #去掉replicas
   # replicas: 1
     selector:
       matchLabels:
         run: nginx
     strategy: {}
     template:
       metadata:
         creationTimestamp: null
         labels:
           run: nginx
       spec:
         containers:
         - image: nginx
           name: nginx
           resources: {}
   status: {}

   ```

   考点：DaemonSet

   参考：[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

7. **将deployment为nginx-app的副本数从1变成4。**

   ```text
   #方法1
   kubectl scale  --replicas=4 deployment nginx-app
   #方法2，使用edit命令将replicas改成4
   kubectl edit deploy nginx-app
   ```

   考点：deployment的Scaling，搜索Scaling

   参考：[Scaling the application by increasing the replica count](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/#scaling-the-application-by-increasing-the-replica-count)

8. **创建nginx-app的deployment ，使用镜像为nginx:1.11.0-alpine ,修改镜像为1.11.3-alpine，并记录升级，再使用回滚，将镜像回滚至nginx:1.11.0-alpine**

   ```text
   kubectl run nginx-app --image=nginx:1.11.0-alpine
   kubectl set image deployment nginx-app --image=nginx:1.11.3-alpine
   kubectl rollout undo deployment nginx-app 
   kubectl rollout status -w deployment nginx-app 
   ```

   考点：资源的更新

   参考：[Kubectl Cheat Sheet:Updating Resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#updating-resources)

9. **根据已有的一个nginx的pod、创建名为nginx的svc、并使用nslookup查找出service dns记录，pod的dns记录并分别写入到指定的文件中**

   ```text
   #创建一个服务
   kubectl create svc nodeport nginx --tcp=80:80
   #创建一个指定版本的busybox，用于执行nslookup
   kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
   #将svc的dns记录写入文件中
   kubectl exec -ti busybox -- nslookup nginx > 指定文件
   #获取pod的ip地址
   kubectl get pod nginx -o yaml
   #将获取的pod ip地址使用nslookup查找dns记录
   kubectl exec -ti busybox -- nslookup <Pod ip>
   ```

   考点：网络相关，DNS解析

   参考：[Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

10. **创建Secret 名为mysecret，内含有password字段，值为bob，然后 在pod1里 使用ENV进行调用，Pod2里使用Volume挂载在/data 下**

    ```text
    #将密码值使用base64加密,记录在Notepad里
    echo -n 'bob' | base64
    ```

    14.secret.yaml

    ```text
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysecret
    type: Opaque
    data:
      password: Ym9i
    ```

    14.pod1.yaml

    ```text
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod1
    spec:
      containers:
      - name: mypod
        image: nginx
        volumeMounts:
        - name: mysecret
          mountPath: "/data"
          readOnly: true
      volumes:
      - name: mysecret
        secret:
          secretName: mysecret
    ```

    14.pod2.yaml

    ```text
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod2
    spec:
      containers:
      - name: mycontainer
        image: redis
        env:
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: password
    ```

    考点 Secret

    参考：[Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

11. **使node1节点不可调度，并重新分配该节点上的pod**

    ```text
    #直接drain会出错，需要添加--ignore-daemonsets --delete-local-data参数
    kubectl drain node node1  --ignore-daemonsets --delete-local-data
    ```

    考点：节点调度、维护

    参考：\[Safely Drain a Node while Respecting Application SLOs\]: （[https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)\)

12. **使用etcd 备份功能备份etcd（提供enpoints，ca、cert、key）**

    ```text
    ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 \
    --cacert=ca.pem --cert=cert.pem --key=key.pem \
    snapshot save snapshotdb
    ```

    考点：etcd的集群的备份与恢复

    参考：[backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

13. **给出一个失联节点的集群，排查节点故障，要保证改动是永久的。**

    ```text
    #查看集群状态
    kubectl get nodes
    #查看故障节点信息
    kubectl describe node node1

    #Message显示kubelet无法访问（记不清了）
    #进入故障节点
    ssh node1

    #查看节点中的kubelet进程
    ps -aux | grep kubelete
    #没找到kubelet进程，查看kubelet服务状态
    systemctl status kubelet.service 
    #kubelet服务没启动，启动服务并观察
    systemctl start kubelet.service 
    #启动正常，enable服务
    systemctl enable kubelet.service 

    #回到考试节点并查看状态
    exit

    kubectl get nodes #正常

    ```

    考点：故障排查

    参考：[Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

14. **给出一个集群，排查出集群的故障**

    这道题没空做完。kubectl get node显示connection refuse，估计是apiserver的故障。

    考点：故障排查

    参考：[Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

15. **给出一个节点，完善kubelet配置文件，要求使用systemd配置kubelet**

    这道题没空做完，

    考点我知道，doc没找到··在哪 逃~ε=ε=ε=┏\(゜ロ゜;\)┛

16. **给出一个集群，将节点node1添加到集群中，并使用TLS bootstrapping**

    这道题没空做完，花费时间比较长，可惜了。

    考点：TLS Bootstrapping

    参考：[TLS Bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)

17. **创建一个pv，类型是hostPath，位于/data中，大小1G，模式ReadOnlyMany**

```text
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 1Gi  
  accessModes:
    - ReadOnlyMany
  hostPath:
    path: /data 

```

考点：创建PV  
 参考：[persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

