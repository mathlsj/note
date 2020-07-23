# Persistent Volume

PersistentVolume(PV) 和 PersistentVolumeClaim(PVC) 提供了方便的持久化卷： PV提 供存储资源，而 PVC 请求存储资源。这样，设置持久化的工作流包括配置底层文件 系统或者云数据卷、创建持久性数据卷、最后创建 claim 来将 pod 跟数据卷关联起来。

## 生命周期

Volume的生命周期包括5个阶段

1. Provisioning，即 PV 的创建，可以直接创建PV（静态方式），也可以使用 StorageClass动态创建 
2. Binding，将 PV 分配给 PVC 
3. Using，Pod 通过 PVC 使用该 Volume
4. Releasing，Pod 释放 Volume 并删除 PVC 
5. Reclaiming，回收 PV，可以保留 PV 以便下次使用，也可以直接从云存储中删除

根据这5个阶段，Volume的状态有以下4种

+ Available：可用 
+ Bound：已经分配给 PVC 
+ Released：PVC 解绑但还未执行回收策略 
+ Failed：发生错误

## PV

PersistentVolume（PV）是集群之中的一块存储。跟 Node 一样，也是集群的资源。PV 跟 Volume (卷) 类似，不过会有独立于 Pod 的生命周期。

PV的访问模式（accessModes）有三种：

+ ReadWriteOnce（RWO）：是最基本的方式，可读可写，但只支持被单个 Pod 挂 载。
+ ReadOnlyMany（ROX）：可以以只读的方式被多个 Pod 挂载。 
+ ReadWriteMany（RWX）：这种存储可以以读写的方式被多个 Pod 共享。

不是每一 种存储都支持这三种方式，像共享方式，目前支持的还比较少，比较常用的是 NFS。在 PVC 绑定 PV 时通常根据两个条件来绑定，一个是存储的大小，另一个就是访问模式。

PV的回收策略（persistentVolumeReclaimPolicy）也有三种

+ Retain，不清理保留 Volume（需要手动清理） 
+ Recycle，删除数据，即 rm -rf /thevolume/* （只有NFS和HostPath支持）
+ Delete，删除存储资源，比如删除 AWS EBS 卷（只有 AWS EBS,GCE PD, Azure Disk 和 Cinder 支持）

## StorageClass

通过手动的方式创建 Volume，当 Volume 太多时管理会很不方便。为此，Kubernetes 还提供了 StorageClass 来动态创建 PV。

在使用 PVC 时，可以通过 DefaultStorageClass 准入控制给未指定 storageClassName 的 PVC 自动添加默认的 StorageClass。默认的 StorageClass 带有 annotation       storageclass.kubernetes.io/is-default-class=true。如下：加了 annotation 的默认 storargClass。

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: testStorageClassDefault
provisioner: kubernetes.io/vsphere-volume
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

## PVC

PV 是存储资源，而 PersistentVolumeClaim(PVC) 是对 PV 的请求。PVC 跟 Pod 类似： Pod 消费 Node 的资源，而 PVC 消费 PV 资源；Pod 能够请求 CPU 和内存资源，而 PVC 请求特 定大小和访问模式的数据卷。如下：

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
    - {key: environment, operator: In, values: [dev]}
```

PVC可以直接挂载到Pod中：

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: myfrontend
    image: dockerfile/nginx
    volumeMounts:
    - mountPath: "/var/www/html"
      name: mypd
  volumes:
  - name: mypd
    persistentVolumeClaim:
      claimName:  myclaim
```