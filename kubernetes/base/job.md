# Job

Job 负责批量处理短暂的一次性任务 (short lived one-off tasks)，即仅执行一次的任务， 它保证批处理任务的一个或多个 Pod 成功结束。

Kubernetes 支持以下几种 Job：

+ 非并行 Job：通常创建一个 Pod 直至其成功结束 
+ 固定结束次数的 Job：设置 .spec.completions ，创建多个 Pod，直到 .spec.completions 个 Pod 成功结束 
+ 带有工作队列的并行Job：设置 .spec.Parallelism 但不设置 .spec.completions ，当所有 Pod 结束并且至少一个成功时，Job 就认为是成功

根据 .spec.completions 和 .spec.Parallelism 的设置，可以将 Job 划分为以下几种类型：

|Job类型|使用示例|行为|complections|parallelism|
|---|---|---|---|---|
|一次性 Job|数据库迁移|创建一个Pod直至其成 功结束| 1| 1|
|固定结束次数的 Job|处理工作队列的Pod|依次创建一个Pod运行直至completions个成 功结束|2+ |1|
|固定结束次数的并行Job|多个Pod同时处理工作队列|依次创建多个Pod运行直至completions个成 功结束|2+ | 2+ |
|并行Job|多个Pod同时处理工作队列|创建一个或多个Pod直至有一个成功结束| 1| 2+|

## Job Controller

Job Controller 负责根据 Job Spec 创建 Pod，并持续监控 Pod 的状态，直至其成功结束。 如果失败，则根据 restartPolicy（只支持 OnFailure 和 Never，不支持Always）决定是否 创建新的 Pod 再次重试任务。

Job Spec格式

+ spec.template 格式同Pod 
+ RestartPolicy 仅支持 Never 或 OnFailure 
+ 单个 Pod 时，默认 Pod 成功运行后 Job 即结束 
+ .spec.completions 标志 Job 结束需要成功运行的 Pod 个数，默认为1
+ .spec.parallelism 标志并行运行的 Pod 的个数，默认为1
+ spec.activeDeadlineSeconds 标志失败 Pod 的重试最大时间，超过这个时间不 会继续重试

job.yaml 例子如下：

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
  namespace: test
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command:
        - "sleep"
        - "10"
```


```
## 创建 job
[root@master ~]# kubectl apply -f job.yaml 
job.batch/job-demo created

## 查看 job
[root@master ~]# kubectl describe job job-demo -n test
Name:           job-demo
Namespace:      test
Selector:       controller-uid=9dbd418a-dea5-4fb8-98b1-2d5f912241d2
Labels:         controller-uid=9dbd418a-dea5-4fb8-98b1-2d5f912241d2
                job-name=job-demo
Annotations:    Parallelism:  1
Completions:    1
Start Time:     Tue, 28 Jul 2020 22:43:44 +0800
Completed At:   Tue, 28 Jul 2020 22:44:13 +0800
Duration:       29s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=9dbd418a-dea5-4fb8-98b1-2d5f912241d2
           job-name=job-demo
  Containers:
   counter:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sleep
      10
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  46s   job-controller  Created pod: job-demo-9wv2c
  Normal  Completed         17s   job-controller  Job completed
```