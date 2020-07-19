# namespace

Namespace是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不 同的项目组或用户组。常见的pods, services, replication controllers和deployments等都 是属于某一个namespace的（默认是default)。

## namespace 操作

### 查询

```
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   20d
istio-system      Active   20d
kube-node-lease   Active   20d
kube-public       Active   20d
kube-system       Active   20d
```

### 创建

命名空间名称满足正则表达式：[a-z0-9]([-a-z0-9]*[a-z0-9])? ,最大长度为 63位

```
[root@master ~]# kubectl create ns test
namespace/test created
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   20d
istio-system      Active   20d
kube-node-lease   Active   20d
kube-public       Active   20d
kube-system       Active   20d
test              Active   20s
```

### 删除

删除 `namespaces` 时需要注意以下几点：

1. 删除一个 namespace 会自动删除所有属于该 namespace 的资源。 
2. default 和 kube-system 命名空间不可删除。
3. PersistentVolumes 是不属于任何 namespace 的，但 PersistentVolumeClaim 是属于 某个特定 namespace 的。
4. Events 是否属于 namespace 取决于产生 events 的对象。

```
[root@master ~]# kubectl delete ns test
namespace "test" deleted
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   20d
istio-system      Active   20d
kube-node-lease   Active   20d
kube-public       Active   20d
kube-system       Active   20d
```
