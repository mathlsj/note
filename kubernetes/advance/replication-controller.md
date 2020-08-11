## Replication controller

Replication Controller 的核心作用是确保在任何时候集群中某个 RC 关联的 Pod 副本数量都保持预设值。如果发现 Pod 的副本数量超过预期值，则 Replication Controller 会销毁一些 Pod 副本；反之，Replication Controller 会自动创建新的 Pod 副本，直到符合条件的 Pod 副本数量达到 预设值。需要注意：只有当 Pod 的重启策略是 Always 时（RestartPolicy=Always），Replication Controller 才会管理该 Pod 的操作
（例如创建、销毁、重启等）。在通常情况下，Pod 对象被成功创建后 不会消失，唯一的例外是当 Pod 处于 succeeded 或 failed 状态的时间过长 （超时参数由系统设定）时，该 Pod 会被系统自动回收，管理该 Pod 的副本控制器将在其他工作节点上重新创建、运行该 Pod 副本。

最好不要越过 RC 直接创建 Pod，因为 Replication Controller 会通过 RC 管理 Pod 副本，实现自动创建、补足、替换、删除 Pod 副本，这样能提高系统的容灾能力，减少由于节点崩溃等意外状况造成的损失。即使你的 应用程序只用到一个 Pod 副本，我们也强烈建议你使用 RC 来定义 Pod。 

Replication Controller 的职责，如下所述。 

+ 确保在当前集群中有且仅有 N 个 Pod 实例，N是在 RC 中定义的 Pod 副本数量。
+ 通过调整 RC 的 spec. replicas 属性值来实现系统扩容或者缩容。
+ 通过改变 RC 中的 Pod 模板（主要是镜像版本）来实现系统的滚动升级。

Replication Controller 的典型使用场景，如下所述。 

+ 重新调度（Rescheduling）。如前面所述，不管想运行1个副 本还是1000个副本，副本控制器都能确保指定数量的副本存在于集群
中，即使发生节点故障或 Pod 副本被终止运行等意外状况。
+ 弹性伸缩（Scaling）。手动或者通过自动扩容代理修改副本 控制器的spec.replicas属性值，非常容易实现增加或减少副本的数量。 
+ 滚动更新（Rolling Updates）。副本控制器被设计成通过逐个 替换 Pod 的方式来辅助服务的滚动更新。推荐的方式是创建一个只有一 个副本的新RC，若新RC副本数量加1，则旧RC的副本数量减1，直到这 个旧RC的副本数量为0，然后删除该旧 RC。通过上述模式，即使在滚动 更新的过程中发生了不可预料的错误，Pod 集合的更新也都在可控范围 内。在理想情况下，滚动更新控制器需要将准备就绪的应用考虑在内， 并保证在集群中任何时刻都有足够数量的可用 Pod。



