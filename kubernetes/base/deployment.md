# Deployment

## 什么是 Deployment

Deployment 为 Pod 和 Replica Set 提供声明式更新。你只需要在 Deployment 中描述你想要的目标状态是什么，Deployment controller 就会帮你将 Pod 和 Replica Set 的实际状态改变到你的目标状态。你可以定义一个全新的Deployment，也可以创建一个新的替换旧的 Deployment。

## 创建 Deployment

示例 yaml 文件如下：

```
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

创建 deployment

```
[root@master ~]# kubectl apply -f nginx.yaml 
deployment.apps/nginx-deployment created
```

获取 deployment

```
[root@master ~]# kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           9m6s

```

输出结果表明我们希望的 repalica 数是 1（根据 deployment 中的 .spec.replicas 配置）当前 replica 数是1， 最新的 replica 数是1，可用的 replica 数是1。

## 扩缩容

直接在 yaml 文件上将 replicas 更改为 4。重新 apply 。结果如下：

```
[root@master ~]# kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   4/4     4            4           29m
```

### 比例扩容

RollingUpdate Deployment 支持同时运行一个应用的多个版本。当你或者 autoscaler 扩容 一个正在 rollout 中（进行中或者已经暂停）的 RollingUpdate  Deployment的时候，为了 降低风险，Deployment controller将会平衡已存在的 active 的 ReplicaSets（有Pod的 ReplicaSets）和新加入的 replicas 。这被称为比例扩容。

maxSurge:滚动更新时最多可以多启动多少个pod

maxUnavailable:滚动更新时最大可以删除多少个pod

修改 yaml 文件如下，有10个 replica 的 Deployment。maxSurge=3，maxUnavailable=2：

```
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort:  80
```

查看的 Deployment 信息如下（rs 后面会有个哈杀值，由不同的版本生成）：

```
[root@master ~]# kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   10/10   10           10          61m
[root@master ~]# kubectl get rs | grep nginx-deployment
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-76f4cbc5d   10        10        10      61m
```

现在更新一个不存在的镜像，image: nginx:2222。查看 rs 结果如下：

```
[root@master ~]# kubectl get rs | grep nginx-deployment
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-76f4cbc5d   8         8         8       64m
nginx-deployment-89c989fbb   5         5         0       19s
```

镜像更新启动了一个包含ReplicaSet nginx-deployment-89c989fbb的新的rollout，但 是它被阻塞了，因为我们上面提到的 maxUnavailable。

由上面设置可知，最多可多启动 3 (maxSurge)个 pod，总的个数： 10 + 3 ＝ 13。又由 maxUnavailable 可知最大可以删除 2 个 pod，所有可用个数 10 - 2 = 8。因此，新的 rollout 的期望是 5 个。

## Deployment状态

Deployment在生命周期中有多种状态。

### Progressing Deployment

Kubernetes 将执行过下列任务之一的 Deploymen t标记为 progressing 状态：

+ Deployment 正在创建新的 ReplicaSet 过程中。 
+ Deployment 正在扩容一个已有的 ReplicaSet。
+ Deployment 正在缩容一个已有的 ReplicaSet。 有新的可用的 pod 出现。

你可以使用 kubectl roullout status 命令监控 Deployment 的进度。

### Complete Deployment

Kubernetes 将包括以下特性的 Deployment 标记为 complete 状态：

+ Deployment 最小可用。最小可用意味着 Deployment 的可用 replica 个数等于或者超 过 Deployment 策略中的期望个数。
+ 所有与该 Deployment 相关的 replica 都被更新到了你指定版本，也就说更新完成。
+ 该 Deployment 中没有旧的 Pod 存在。

你可以用 kubectl rollout status 命令查看 Deployment 是否完成

### Failed Deployment

你的 Deployment 在尝试部署新的 ReplicaSet 的时候可能卡住，永远也不会完成。这可能 是因为以下几个因素引起的：

+ 无效的引用 
+ 不可读的probe failure 
+ 镜像拉取错误 
+ 权限不够 
+ 范围限制
+ 程序运行时配置错误

## 删除 Deployment

```
[root@master ~]# kubectl delete -f nginx.yaml 
deployment.apps "nginx-deployment" deleted
[root@master ~]# kubectl get deployment nginx-deployment
Error from server (NotFound): deployments.apps "nginx-deployment" not found
```
