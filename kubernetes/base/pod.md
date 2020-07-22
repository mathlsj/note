# pod

Pod 是 Kubernetes 应用程序的基本执行单元，即它是 Kubernetes 对象模型中创建或部署的最小和最简单的单元。Pod 表示在 集群 上运行的进程。

Pod是一组紧密关联的容器集合，它们共享IPC、Network和UTC	namespace，是 Kubernetes调度的基本单位。Pod的设计理念是支持多个容器在一个Pod中共享网络和 文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

## pod 的特征

+ 包含多个共享IPC、Network和UTC   namespace的容器，可直接通过localhost通信
+ 所有Pod内容器都可以访问共享的Volume，可以访问共享数据
+ Pod一旦调度后就跟Node绑定，即使Node挂掉也不会重新调度，推荐使用 Deployments、Daemonsets等控制器来容错
+ 优雅终止：Pod删除的时候先给其内的进程发送SIGTERM，等待一段时间（grace period）后才强制停止依然还在运行的进程
+ 特权容器（通过SecurityContext配置）具有改变系统配置的权限（在网络插件中大 量应用）

## pod 的用途

+ 运行单个容器的 Pod。"每个 Pod 一个容器"模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
+ 运行多个协同工作的容器的 Pod。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众，而另一个单独的“挂斗”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。 Kubernetes 博客 上有一些其他的 Pod 用例信息。

## 网络

每个 Pod 分配一个唯一的 IP 地址。 Pod 中的每个容器共享网络命名空间，包括 IP 地址和网络端口。 Pod 内的容器 可以使用 localhost 互相通信。 当 Pod 中的容器与 Pod 之外 的实体通信时，它们必须协调如何使用共享的网络资源（例如端口）。

## 存储

一个 Pod 可以指定一组共享存储卷。 Pod 中的所有容器都可以访问共享卷，允许这些容器共享数据。 卷还允许 Pod 中的持久数据保留下来，以防其中的容器需要重新启动。

## 定义

通过yaml或json描述Pod和其内Container的运行环境以及期望状态，比如一个最简单的 nginx  pod可以定义为：

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
```

### volume

Volume可以为容器提供持久化存储.

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

### RestartPoliy

支持三种RestartPolicy

+ Always：只要退出就重启 
+ OnFailure：失败退出（exit  code不等于0）时重启 
+ Never：只要退出就不再重启

注意，这里的重启是指在Pod所在Node上面本地重启，并不会调度到其他Node上去

### ImagePullPolicy

支持三种ImagePullPolicy

+ Always：不管镜像是否存在都会进行一次拉取。 
+ Never：不管镜像是否存在都不会进行拉取
+ IfNotPresent：只有镜像不存在时，才会进行镜像拉取。

注意：

+ 默认为 IfNotPresent，但 :latest 标签的镜像默认为 Always。
+ 拉取镜像时docker会进行校验，如果镜像中的MD5码没有变，则不会拉取镜像数 据。
+ 生产环境中应该尽量避免使用 :latest 标签，而开发环境中可以借助 :latest 标 签自动拉取最新的镜像。

### 使用主机的网络命名空间

通过设置hostNetwork参数True，使用主机的网络命名空间，默认为False。

### 设置Pod的子域名

通过subdomain参数设置Pod的子域名，默认为空

指定hostname为busybox-2和subdomain为default-subdomain，完整域名 为 nginx-2.default-subdomain.default.svc.cluster.local ：

```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  hostname: nginx-2 
  subdomain: default-subdomain
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort:  80
```

### 资源限制

Kubernetes通过cgroups限制容器的CPU和内存等计算资源，包括requests（请求，调 度器保证调度到资源充足的Node上）和limits（上限）等：

+ limits.cpu： CPU上限，可以短暂超过，容器 也不会被停止。
+ limits.memory：内存上限，不可以超过；如果超过，容器可能会被停止或调度到其他资源充足的机器上。
+ requests.cpu：CPU请求，可以超过。
+ requests.memory： 内存请求，可以超过；但如果超过，容器可能会在Node内存不足时清理。

例如： 下面容器请求 1 核 CPU 和 1G 的内存，限制 2 核 CPU 和 2G 的内存

```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  hostname: nginx-2 
  subdomain: default-subdomain
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort:  80
    resources:
      requests:
        cpu: 1
        memory: 1G
      limits:
        cpu: 2
        memory: 2G
```