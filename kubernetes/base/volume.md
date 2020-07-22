# volume

Kubernetes Volume 的生命周期与 Pod 绑定

+ 容器挂掉后，再次重启容器时，Volume 的数据依然还在。 
+ 而 Pod 删除时，Volume 才会清理。数据是否丢失取决于具体的 Volume 类型，比如 emptyDir 的数据会丢失，而 PV 的数据则不会丢。

## emptyDir

emptyDir 类型 Volume，Pod 被分配到 Node 上时候，会创建 emptyDir， 只要 Pod 运行在 Node 上，emptyDir 都会存在（容器挂掉不会导致 emptyDir 丢失数据）， 但是如果 Pod 从 Node 上被删除（Pod被删除，或者Pod发生迁移），emptyDir 也会被删 除，并且永久丢失。

```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort:  80
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    emptyDir: {}
```

## hostPath

hostPath 允许挂载 Node 上的文件系统到 Pod 里面去。 由于是在 Node 上的文件，Pod 删除，文件仍然存在。

```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort:  80
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    hostPath:
      path: /data
```