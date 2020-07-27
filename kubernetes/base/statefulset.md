# StatefulSet

StatefulSet 是为了解决有状态服务的问题（对应 Deployments 和 ReplicaSets 是为无状态 服务而设计），其应用场景包括

+ 稳定的持久化存储
+ 稳定的网络标志
+ 有序收缩，有序删除（即从N-1到0）部署，有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺 序依次依序进行（即从0到N-1，在下一个Pod运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于init containers来实现 
+ 有序收缩，有序删除（即从N-1到0）

StatefulSet 中每个 Pod 的 DNS 格式为    statefulSetName-{0..N1}.serviceName.namespace.svc.cluster.local ，其中
+ serviceName 为 Headless Service的名字  
+ 0..N-1 为 Pod 所在的序号，从0开始到N-1
+ statefulSetName 为 StatefulSet 的名字 
+ namespace 为服务所在的 namespace，Headless Servic和StatefulSet必须在相同 的namespace  
+ .cluster.local  为Cluster Domain

## statefulset 操作

以 nginx.yaml 为例子：

```
---
apiVersion: apps/v1 
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```


```
## 创建 statefulset
[root@master ~]# kubectl apply -f nginx.yaml 
statefulset.apps/nginx created
service/nginx created

## 查看 statefulset
[root@master ~]# kubectl get service nginx
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   None         <none>        80/TCP    8m3s
[root@master ~]# kubectl get statefulset nginx
NAME    READY   AGE
nginx   3/3     8m12s

## 查看创建的 POD
[root@master ~]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
nginx-0                           2/2     Running   0          9m14s
nginx-1                           2/2     Running   0          8m28s
nginx-2                           2/2     Running   0          7m43s

```

## 更新策略

StatefulSet 的自动更新，通过 spec.updateStrategy 设置更新策略。目前支 持两种策略

+ OnDelete：当 .spec.template 更新时，并不立即删除旧的 Pod，而是等待用户手 动删除这些旧 Pod 后自动创建新 Pod。这是默认的更新策略。 
+ RollingUpdate：当 .spec.template  更新时，自动删除旧的 Pod 并创建新 Pod 替 换。在更新时，这些 Pod 是按逆序的方式进行，依次删除、创建并等待 Pod 变成
 StatefulSet Ready 状态才进行下一个 Pod 的更新。

## 管理策略

可以通过 .spec.podManagementPolicy 设置 Pod 管理策略，支持两种方式

+ OrderedReady：默认的策略，按照 Pod 的次序依次创建每个 Pod 并等待 Ready 之后 才创建后面的 Pod 
+ Parallel：并行创建或删除 Pod（不等待前面的 Pod Ready 就开始创建所有的 Pod）
