# Resource Quotas

资源配额（Resource Quotas）是用来限制用户资源用量的一种机制。
它的工作原理为

+ 资源配额应用在 Namespace 上，并且每个 Namespace 最多只能有一 个 ResourceQuota 对象
+ 开启计算资源配额后，创建容器时必须配置计算资源请求或限制（也可以用 LimitRange 设置默认值） 
+ 用户超额后禁止创建新的资源

## 启用

在 API Serve r启动时配置 ResourceQuota adminssion control；然后在 namespace 中创建 ResourceQuota 对象即可。

## 类型

+ 计算资源，包括cpu和memory
    + cpu, limits.cpu, requests.cpu 
    + memory, limits.memory, requests.memory 
+ 存储资源，包括存储资源的总量以及指定storage   class的总量 
    + requests.storage：存储资源总量，如500Gi 
    + persistentvolumeclaims：pvc的个数 
    + .storageclass.storage.k8s.io/requests.storage
    + .storageclass.storage.k8s.io/persistentvolumeclaims 
+ 对象数，即可创建的对象的个数 
    + pods, replicationcontrollers, configmaps, secrets 
    + resourcequotas, persistentvolumeclaims 
    + services, services.loadbalancers, services.nodeports

计算资源例子：

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resources
spec:
  hard:
    pods: "4"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

## LimitRange

默认情况下，Kubernetes 中所有容器都没有任何 CPU 和内存限制。 LimitRange 用来给 Namespace 增加一个资源限制，包括最小、最大和默认资源。

例子：

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimits
spec:
  limits:
  - max:
      cpu: "2"
      memory: 1Gi
    min:
      cpu: 200m
      memory: 6Mi
    type: Pod
  - default:
      cpu: 300m
      memory: 200Mi
    defaultRequest:
      cpu: 200m
      memory: 100Mi
    max:
      cpu: "2"
      memory: 1Gi
    min:
      cpu: 100m
      memory: 3Mi
    type: Container
```

## 配额范围

每个配额在创建时可以指定一系列的范围

|范围 |说明 |
|---|---|
|Terminating |podSpec.ActiveDeadlineSeconds>=0的Pod|
|NotTerminating | podSpec.activeDeadlineSeconds=nil的Pod|
|BestEffort| 所有容器的requests和limits都没有设置的Pod（Best-Effort）| 
| NotBestEffort| 与BestEffort相反|
