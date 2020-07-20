# Node

Node 是 Pod 真正运行的主机，可以物理机，也可以是虚拟机。为了管理 Pod，每个 Node 节点上至少要运行container  runtime（比如docker或者rkt) 和 管理 Pod 的主件（如 kubelet 或 kubeedge）。

## Node 管理

不像其他的资源（如 Pod），Node 本质上不是 Kubernetes 来创建的，Kubernetes 只是管理 Node 上的资源。虽然可以通过Manifest创建一个Node对象（如下 json所示），但 Kubernetes 也只是去检查是否真的是有这么一个 Node，如果检查失败，也不会往上调度Pod。

```
{
    "kind": "Node",
    "apiVersion": "v1",
    "metadata": {
        "name": "node2",
        "labels": {
            "name": "node2",
            "type": "test"
        }
    }
}
```

这个检查是由 Node Controller 来完成的。Node Controller负责
+ 维护Node状态。
+ 与 Cloud Provider 同步 Node。
+ 给Node分配容器CIDR。
+ 删除带有 NoExecute taint的 Node 上的 Pods。

默认情况下，kubelet 在启动时会向 master 注册自己，并创建 Node 资源。

## Node 状态

每个Node都包括以下状态信息
+ 地址：包括hostname、外网IP和内网IP。
+ 条件（Condition）：包括OutOfDisk、Ready、MemoryPressure和DiskPressure。
+ 容量（Capacity）：Node上的可用资源，包括CPU、内存和Pod总数。
+ 基本信息（Info）：包括内核版本、容器引擎版本、OS类型等

## 污点和容忍

Taints（污点） 和 tolerations （容忍）用于保证 Pod 不被调度到不合适的 Node 上，Taint 应用于 Node 上，而 toleration 则应用于 Pod 上（Toleration 是可选的）。

例如： 使用 taint 给 node1 添加污点

```
## 增加 key 为 key1, value 为 value1 的不可调度污点。
kubectl taint nodes node1 key1=value1:NoSchedule
```

## Node维护模式

标志Node不可调度但不影响其上正在运行的Pod，这种维护Node时是非常有用的

```
kubectl cordon node1
```
