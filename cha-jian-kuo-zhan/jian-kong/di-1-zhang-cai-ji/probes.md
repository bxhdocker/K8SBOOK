# Probes

### Kubernetes Probes具有定期监测集装箱或服务的健康状况的重要功能，并在发生不健康的事件时采取行动。Kubernetes监控探针允许你通过设定一个特定的命令来定义“Liveness”状态，这个命令应该在Pod中成功执行。你还可以设定Liveness探针执行的频率。以下是一个简单的示例，基于cat命令。

```text
#Example Liveness probe for the Sysdig Blog on "Monitoring Kubernetes"apiVersion: v1kind: Podmetadata:  labels:    test: liveness  name: liveness-execspec:  containers:  -name: liveness    args:    - /bin/sh    - -c    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600    image: gcr.io/google_containers/busybox    livenessProbe:      exec:        command:        - cat        - /tmp/healthy      initialDelaySeconds: 5      periodSeconds: 5      #source https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
--------------------- 
作者：Docker_ 
来源：CSDN 
原文：https://blog.csdn.net/M2l0ZgSsVc7r69eFdTj/article/details/79608015 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

