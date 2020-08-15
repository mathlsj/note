# kubelet

在 Kubernetes 集群中，在每个Node（又称Minion）上都会启动一个 kubelet 服务进程。该进程用于处理 Master 下发到本节点的任务，管理 Pod 及 Pod 中的容器。每个 kubelet 进程都会在 API Server 上注册节点自身的信息，定期向 Master 汇报节点资源的使用情况，并通过 cAdvisor 监控容器和节点资源。

## 节点管理

在某些情况下，Kubernetes 集群中的某些 kubelet 没有选择自注册模
式，用户需要自己去配置 Node 的资源信息，同时告知 Node 上 Kubelet API Server 的位置。集群管理者能够创建和修改节点信息。如果管理者希望手动创建节点信息，则通过设置 kubelet 的启动参数“--registernode=false”即可完成。 

kubelet 在启动时通过 API Server 注册节点信息，并定时向 API Server 发送节点的新消息，API Server 在接收到这些信息后，将这些信息写入 etcd。通过 kubelet 的启动参数 “--node-status-update-frequency”设置 kubelet每隔多长时间向API Server报告节点状态，默认为10s。

## Pod 管理

kubelet 通过以下几种方式获取自身 Node 上要运行的 Pod 清单。 

+ 文件：kubelet 启动参数“--config”指定的配置文件目录下的文件（默认目录为“/etc/kubernetes/manifests/”）。通过 --file-checkfrequency 设置检查该文件目录的时间间隔，默认为20s。 
+ HTTP端点（URL）：通过“--manifest-url”参数设置。通过 -http-check-frequency 设置检查该 HTTP 端点数据的时间间隔，默认为 20s。 
+ API Server：kubelet通过API Server监听etcd目录，同步Pod列表。

所有以非 API Server 方式创建的 Pod 都叫作 Static Pod。kubelet 将 Static Pod 的状态汇报给 API Server，API Server 为该 Static Pod 创建一个 Mirror Pod 和其相匹配。 Mirror Pod 的状态将真实反映 Static Pod 的状态。 当Static Pod 被删除时，与之相对应的 Mirror Pod 也会被删除。

kubelet 通过 API Server Client 使用 Watch 加 List 的方式监听“/registry/nodes/$”当前节点的名称和“/registry/pods”目录，将获取的信息同步到本地缓存中。 

kubelet 监听 etcd，所有针对 Pod 的操作都会被 kubelet 监听。如果发现有新的绑定到本节点的 Pod，则按照 Pod 清单的要求创建该 Pod。

如果发现本地的 Pod 被修改，则 kubelet 会做出相应的修改，比如在删除 Pod 中的某个容器时，会通过 Docker Client 删除该容器。 

如果发现删除本节点的 Pod，则删除相应的 Pod，并通过 Docker Client 删除 Pod 中的容器。 

kubelet读取监听到的信息，如果是创建和修改Pod任务，则做如下处理。

1. 为该 Pod 创建一个数据目录。 
2. 从 API Server 读取该 Pod 清单。 
3. 为该 Pod 挂载外部卷（External Volume）。 
4. 下载 Pod 用到的Secret。 
5. 检查已经运行在节点上的 Pod，如果该 Pod 没有容器或 Pause 容器（“kubernetes/pause”镜像创建的容器）没有启动，则先停止 Pod 里所有容器的进程。如果在 Pod 中有需要删除的容器，则删除这些容器。 
6. 用“kubernetes/pause”镜像为每个 Pod 都创建一个容器。该 Pause 容器用于接管 Pod 中所有其他容器的网络。每创建一个新的 Pod，kubelet 都会先创建一个 Pause 容器，然后创建其他容器。
7. 为Pod中的每个容器做如下处理。 
    + 为容器计算一个 Hash 值，然后用容器的名称去查询对应 Docker 容器的 Hash 值。若查找到容器，且二者的 Hash 值不同，则停止 Docker 中容器的进程，并停止与之关联的 Pause 容器的进程；若二者相同，则不做任何处理。
    + 如果容器被终止了，且容器没有指定的 restartPolicy（重启策 略），则不做任何处理。 
    + 调用 Docker Client 下载容器镜像，调用 Docker Client 运行容器。

## 健康检查

Pod 通过两类探针来检查容器的健康状态。一类是 LivenessProbe 探针，用于判断容器是否健康并反馈给 kubelet。如果 LivenessProbe 探针探测到容器不健康，则 kubelet 将删除该容器，并根据容器的重启策略做相应的处理。如果一个容器不包含 LivenessProbe 探针，那么 kubelet 认为该容器的 LivenessProbe 探针返回的值永远是 Success；另一类是 ReadinessProbe 探针，用于判断容器是否启动完成，且准备接收请求。如果 ReadinessProbe 探针检测到容器启动失败，则 Pod 的状态将被修改， Endpoint Controller 将从 Service 的 Endpoint 中删除包含该容器所在 Pod 的 IP 地址的 Endpoint 条目。

kubelet定期调用容器中的 LivenessProbe 探针来诊断容器的健康状况。 LivenessProbe 包含以下3种实现方式。

1. ExecAction：在容器内部执行一个命令，如果该命令的退出 状态码为0，则表明容器健康。 
2. TCPSocketAction：通过容器的 IP 地址和端口号执行 TCP 检查，如果端口能被访问，则表明容器健康。 
3. HTTPGetAction：通过容器的IP地址和端口号及路径调用 HTTP Get 方法，如果响应的状态码大于等于200且小于等于400，则认为容器状态健康。

LivenessProbe探针被包含在Pod定义的spec.containers.{某个容器} 中。




