# yaml文件例子--pod

```text
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中  通过kubectl get apiversion可以查看
kind: Pod #指定创建资源的角色/类型  #详解资源对象
metadata: #资源的元数据/属性  
  name: web04-pod #资源的名字，在同一个namespace中必须唯一  
  namespace: web   #默认不写就在default下
  labels: #设定资源的标签，详情请见http://blog.csdn.net/liyingke112/article/details/77482384
    k8s-app: apache  #标签名字 
    version: v1  
    kubernetes.io/cluster-service: "true"  
  annotations:            #自定义注解列表  
    #- name: String        #自定义注解名字
    scheduler.alpha.kubernetes.io/critical-pod: '' #Rescheduler 最后被驱逐
    scheduler.alpha.kubernetes.io/tolerations: |   #在1.10后已经废弃，改用优先级priorityClass
       [{"key": "dedicated", "value": "master", "effect": "NoSchedule" },
        {"key":"CriticalAddonsOnly", "operator":"Exists"}]
spec:#specification of the resource content 指定该资源的内容  
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器  
  nodeSelector:     #节点选择，需要先给主机打标签kubectl label nodes kube-node1 zone=node1   
    zone: node1     #然后这个pod会调度到打标签zone=node1 的node节点上去
  containers:  
  - name: web04-pod #容器的名字  
    image: web:apache #容器使用的镜像地址  
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT  
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数  
    env: #指定容器中的环境变量  
    - name: str #变量的名字  
      value: "/etc/run.sh" #变量的值  
      #默认的情况下, k8s不会限制pod的cpu和内存的
    resources: #资源管理，请求请见http://blog.csdn.net/liyingke112/article/details/77452630
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行 会预先占用node节点的cpu  
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量  
      limits: #资源限制  如果不写- max:  min:则按max算
       #- max:
        cpu: 0.5  
        memory: 32Mi
       #min: 
    ports:  
    - containerPort: 80 #容器开放对外的端口
      name: httpd  #名称
      protocol: TCP  
    livenessProbe: #pod内容器健康检查的设置，详情请见http://blog.csdn.net/liyingke112/article/details/77531584
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常  
        path: / #URI地址  
        port: 80  
        #host: 127.0.0.1 #主机地址  
        scheme: HTTP  
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始  
      timeoutSeconds: 5 #检测的超时时间  
      periodSeconds: 15  #检查间隔时间  
      #也可以用这种方法  
      #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常  
      #  command:  
      #    - cat  
      #    - /tmp/health  
      #也可以用这种方法  
      #tcpSocket: //通过tcpSocket检查健康   
      #  port: number   
    lifecycle: #生命周期管理  
      postStart: #容器运行之前运行的任务  
        exec:  
          command:  
            - 'sh'  
            - 'yum upgrade -y'  
      preStop:#容器关闭之前运行的任务  
        exec:  
          command: ['service httpd stop']  
    volumeMounts:  #详情请见http://blog.csdn.net/liyingke112/article/details/76577520
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应    
      mountPath: /data #挂载到容器的某个路径下  
      readOnly: True  
  volumes: #定义一组挂载设备  
  - name: volume #定义一个挂载设备的名字  
    #meptyDir: {}  
    hostPath:  
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种  

```

