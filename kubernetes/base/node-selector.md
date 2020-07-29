# node selector

Pod 运行在指定的 Node 节点上有多种方式：

+ nodeSelector： 只调度到匹配指定 label 的 Node 上 
+ nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作 
+ podAffinity：调度到满足条件的 Pod 所在的 Node 上

## nodeSelector

首先给 Node 打标签

```
[root@master ~]# kubectl label nodes node1 pool=all
node/node1 labeled
```

然后指定 nodeSelector 为 pool=all

```
spec:
  nodeSelector:
    pool: all
```

## nodeAffinity

nodeAffinity 目前支持两种：requiredDuringSchedulingIgnoredDuringExecution 和 preferredDuringSchedulingIgnoredDuringExecution，分别代表必须满足条件和优选条 件。比如下面的例子代表调度到包含标签 pool 并且值为 poo11 或 pool2 的 Node 上，并且优选还带有标签 another-node-label-key=anothernode-label-value 的 Node。

```
apiVersion: v1
kind: Pod
metadata:
  name: test-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: pool
            operator: In
            values:
            - pool1
            - pool2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: test-node-affinity
    image:  gcr.io/google_containers/pause:2.0
```

## podAffinity

podAffinity 基于 Pod 的标签来选择 Node，仅调度到满足条件 Pod 所在的 Node 上，支持 podAffinity 和 podAntiAffinity。以下面的例子为例：

+ 如果一个 “ Node 所在 Zone 中包含至少一个带有 pool=test 标签且运行中的 Pod”，那么可以调度到该 Node 
+ 不调度到 “包含至少一个带有 pool=test2 标签且运行中Pod” 的 Node 上

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
          matchExpressions:
          - key: pool 
            operator: In 
            values:
            - test
          topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: test2
              operator: In
              values:
              - S2
          topologyKey:
          kubernetes.io/hostname
  containers:
  - name: test-pod-affinity
    image:  gcr.io/google_containers/pause:2.0
```