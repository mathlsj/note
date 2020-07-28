# DaemonSet

DaemonSet保证在每个Node上都运行一个容器副本，常用来部署一些集群的日志、监 控或者其他系统管理应用。典型的应用包括：

+ 日志收集，比如 fluentd，logstash 等 
+ 系统监控，比如Prometheus Node Exporter，collectd，New Relic agent，Ganglia gmond 等 
+ 系统程序，比如kube-proxy, kube-dns, glusterd, ceph

例子：

```
---
apiVersion: apps/v1 
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: logging
      id: fluentd
      name: fluentd
  template:
    metadata:                                               
      labels:                                                         
        app: logging                                                         
        id: fluentd                                         
        name: fluentd                         
    spec:                                           
      containers:                                             
      - name: fluentd-es                                                              
        image:  gcr.io/google_containers/fluentd-elasticsearch:1.3
        env:
        - name: FLUENTD_ARGS
          value:
          -qq
        volumeMounts:
        - name: containers
          mountPath: /var/lib/docker/containers
        - name: varlog
          mountPath: /varlog
      volumes:
      - hostPath:
          path: /var/lib/docker/containers
        name: containers
      - hostPath:
          path: /var/log
        name: varlog
```

## 滚动更新

DaemonSet 的滚动更新，可以通过 .spec.updateStrategy.type 设置更新策略。目前支持两种策略

+ OnDelete：默认策略，更新模板后，只有手动删除了旧的 Pod 后才会创建新的 Pod 
+ RollingUpdate：更新 DaemonSet 模版后，自动删除旧的 Pod 并创建新的 Pod

在使用RollingUpdate策略时，还可以设置

+ .spec.updateStrategy.rollingUpdate.maxUnavailable,默认1 
+ spec.minReadySeconds，默认0

## 静态 POD

除了 DaemonSet，还可以使用静态 Pod 来在每台机器上运行指定的 Pod，这需要 kubelet 在启动的时候指定 manifest目录：

```
kubelet --pod-manifest-path=/etc/kubernetes/manifests
```

然后将所需要的 Pod 定义文件放到指定的 manifest 目录中。

注意：静态 Pod 不能通过 API Server 来删除，但可以通过删除 manifest 文件来自动删除对 应的 Pod



