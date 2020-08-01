# CronJob

CronJob 即定时任务，就类似于 Linux 系统的 crontab，在指定的时间周期运行指定的任务。

## CronJob Spec

+ .spec.schedule 指定任务运行周期，格式同Cron
+ .spec.jobTemplate 指定需要运行的任务，格式同Job
+ .spec.startingDeadlineSeconds 指定任务开始的截止期限 
+ .spec.concurrencyPolicy 指定任务的并发策略，支持 Allow、Forbid 和 Replace 三个选项

具体例子如下：

```
apiVersion: batch/v1beta1 
kind: CronJob 
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - "/bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done" 
          restartPolicy:  OnFailure
```