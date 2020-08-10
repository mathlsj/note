## Node Controller

kubelet 进程在启动时通过 API Server 注册自身的节点信息，并定时向 API Server 汇报状态信息，API Server 在接收到这些信息后，会将这些 信息更新到 etcd 中。在 etcd 中存储的节点信息包括节点健康状况、节点资源、节点名称、节点地址信息、操作系统版本、Docker 版本、kubelet 版本等。节点健康状况包含“就绪”（True）“未就绪”（False）和“未 知”（Unknown）三种。 

Node Controller 通过 API Server 实时获取 Node 的相关信息，实现管理 和监控集群中的各个 Node 的相关控制功能。

Node Controller 的核心工作流程

+ Controller Manager 在启动时如果设置了 --cluster-cidr 参数，那么为每个没有设置S pec.PodCIDR 的 Node 都生成一个 CIDR 地址，并用该 CIDR 地址设置节点的 Spec.PodCIDR 属性，这样做的目的是防止不同节点的CIDR地址发生冲突。
+ 逐个读取 Node 信息，多次尝试修改 nodeStatusMap 中的节点状态信息，将该节点信息和 Node Controller 的 nodeStatusMap 中保存的节点信息做比较。如果判断出没有收到 kubelet 发送的节点信息、第1次收到节点 kubelet 发送的节点信息，或在该处理过程中节点状态变成非“健
康”状态，则在 nodeStatusMap 中保存该节点的状态信息，并用 Node Controller 所在节点的系统时间作为探测时间和节点状态变化时间。如果判断出在指定时间内收到新的节点信息，且节点状态发生变化，则在 nodeStatusMap 中保存该节点的状态信息，并用 Node Controller 所在节点的系统时间作为探测时间和节点状态变化时间。如果判断出在指定时间内收到新的节点信息，但节点状态没发生变化，则在 nodeStatusMap 中保存该节点的状态信息，并用 Node Controller 所在节点的系统时间作为探测时间，将上次节点信息中的节点状态变化时间作为该节点的状态变化时间。如果判断出在某段时间（gracePeriod）内没有收到节点状态信息，则设置节点状态为“未知”，并且通过 API Server 保存节点状态。逐个读取节点信息，如果节点状态变为非“就绪”状态，则将 节点加入待删除队列，否则将节点从该队列中删除。如果节点状态为非“就绪”状态，且系统指定了 Cloud Provider，则 Node Controller 调用 Cloud Provider 查看节点，若发现节点故障，则删除 etcd 中的节点信息，并删除和该节点相关的 Pod 等资源的信息。
