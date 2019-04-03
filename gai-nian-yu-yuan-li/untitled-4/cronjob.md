# CronJob

CronJob 即定时任务，就类似于 Linux 系统的 crontab，在指定的时间周期运行指定的任务。

## API 版本对照表 <a id="api-ban-ben-dui-zhao-biao"></a>

| Kubernetes 版本 | Batch API 版本 | 默认开启 |
| :--- | :--- | :--- |
| v1.5-v1.7 | batch/v2alpha1 | 否 |
| v1.8-v1.9 | batch/v1beta1 | 是 |

注意：使用默认未开启的 API 时需要在 kube-apiserver 中配置 `--runtime-config=batch/v2alpha1`。

## CronJob Spec <a id="cronjob-spec"></a>

* `.spec.schedule` 指定任务运行周期，格式同 [Cron](https://en.wikipedia.org/wiki/Cron)​
* `.spec.jobTemplate` 指定需要运行的任务，格式同 [Job](https://kubernetes.feisky.xyz/he-xin-yuan-li/index-2/job)​
* `.spec.startingDeadlineSeconds` 指定任务开始的截止期限
* `.spec.concurrencyPolicy` 指定任务的并发策略，支持 Allow、Forbid 和 Replace 三个选项

```text
apiVersion: batch/v1beta1kind: CronJobmetadata:  name: hellospec:  schedule: "*/1 * * * *"  jobTemplate:    spec:      template:        spec:          containers:          - name: hello            image: busybox            args:            - /bin/sh            - -c            - date; echo Hello from the Kubernetes cluster          restartPolicy: OnFailure
```

```text
$ kubectl create -f cronjob.yamlcronjob "hello" created
```

当然，也可以用 `kubectl run` 来创建一个 CronJob：

```text
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```

```text
$ kubectl get cronjobNAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULEhello     */1 * * * *   False     0         <none>$ kubectl get jobsNAME               DESIRED   SUCCESSFUL   AGEhello-1202039034   1         1            49s$ pods=$(kubectl get pods --selector=job-name=hello-1202039034 --output=jsonpath={.items..metadata.name} -a)$ kubectl logs $podsMon Aug 29 21:34:09 UTC 2016Hello from the Kubernetes cluster​# 注意，删除 cronjob 的时候不会自动删除 job，这些 job 可以用 kubectl delete job 来删除$ kubectl delete cronjob hellocronjob "hello" deleted
```

## 参考文档 <a id="can-kao-wen-dang"></a>

* ​[Cron Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)​

