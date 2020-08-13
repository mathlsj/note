# scheduler

Kubernetes Scheduler 的作用是将待调度的 Pod（API新创 建的Pod、Controller Manager 为补足副本而创建的 Pod 等）按照特定的调度算法和调度策略绑定（Binding）到集群中某个合适的 Node 上，并将绑定信息写入 etcd 中。在整个调度过程中涉及三个对象，分别是待调度 Pod 列表、可用 Node 列表，以及调度算法和策略。简单地说，就是通过调度算法调度为待调度 Pod 列表中的每个 Pod 从 Node 列表中选择一个最适合的Node。 

Kubernetes Scheduler当前提供的默认调度流程分为以下两步。

1. 预选调度过程，即遍历所有目标 Node，筛选出符合要求的候选节点。为此，Kubernetes 内置了多种预选策略（xxx Predicates）供用户选择。 
2. 优选，在第1步的基础上，采用优选策略（xxx Priority）计算出每个候选节点的积分，积分最高者胜出。 

Scheduler 的调度流程是通过插件方式加载的“调度算法 提供者”（AlgorithmProvider）具体实现的。一个AlgorithmProvider其实 就是包括了一组预选策略与一组优先选择策略的结构体，注册 AlgorithmProvider的函数如下：

```
func RegisterAlgorithmProvider(name string, predicateKeys, priorityKeys util.StringSet)
```

它包含三个参数：name string为算法名；predicateKeys 为算法用到的预选策略集合；priorityKeys 为算法用到的优选策略集合。 

### 预选

Scheduler 中可用的预选策略包含：NoDiskConflict、 PodFitsResources、PodSelectorMatches、PodFitsHost、 CheckNodeLabelPresence、CheckServiceAffinity 和 PodFitsPorts 策略等。 其默认的 AlgorithmProvide r加载的预选策略Predicates包括： PodFitsPorts（PodFitsPorts）、PodFitsResources（PodFitsResources）、 NoDiskConflict（NoDiskConflict）、 MatchNodeSelector（PodSelectorMatches）和 HostName（PodFitsHost），即每个节点只有通过前面提及的5个默认预 选策略后，才能初步被选中，进入下一个流程。

下面列出的是对所有预选策略的详细说明。 

1. NoDiskConflict

判断备选 Pod 的 gcePersistentDisk 或 AWSElasticBlockStore 和备选的节 点中已存在的 Pod 是否存在冲突。检测过程如下。 

+  首先，读取备选 Pod 的所有 Volume 的信息（即 pod.Spec.Volumes），对每个 Volume 执行以下步骤进行冲突检测。
+  如果该 Volume 是gcePersistentDisk，则将 Volume 和备选节点上的所有 Pod 的每个 Volume 都进行比较，如果发现相同的 gcePersistentDisk，则返回 false，表明存在磁盘冲突，检查结束，反馈给调度器该备选节点不适合作为备选 Pod；如果该 Volume是 AWSElasticBlockStore，则将 Volume 和备选节点上的所有 Pod 的每个 Volume 都进行比较，如果发现相同的 AWSElasticBlockStore，则返回 false，表明存在磁盘冲突，检查结束，反馈给调度器该备选节点不适合备选 Pod。
+  如果检查完备选 Pod 的所有 Volume 均未发现冲突，则返回
true，表明不存在磁盘冲突，反馈给调度器该备选节点适合备选 Pod 

2. PodFitsResources

判断备选节点的资源是否满足备选 Pod 的需求，检测过程如下。 

+  计算备选 Pod 和节点中已存在 Pod 的所有容器的需求资源（内存和CPU）的总和。
+  获得备选节点的状态信息，其中包含节点的资源信息。 
+  如果在备选 Pod和节点中已存在 Pod 的所有容器的需求资源 （内存和CPU）的总和，超出了备选节点拥有的资源，则返回false，表明备选节点不适合备选Pod，否则返回true，表明备选节点适合备选 Pod。 

3. PodSelectorMatches

判断备选节点是否包含备选Pod的标签选择器指定的标签。 

+  如果 Pod 没有指定 spec.nodeSelector 标签选择器，则返回true。 
+  否则，获得备选节点的标签信息，判断节点是否包含备选Pod 的标签选择器（spec.nodeSelector）所指定的标签，如果包含，则返回true，否则返回false。 

4. PodFitsHost

判断备选 Pod 的spec.nodeName域所指定的节点名称和备选节点的名 称是否一致，如果一致，则返回true，否则返回false。 

5. CheckNodeLabelPresence

如果用户在配置文件中指定了该策略，则 Scheduler 会通过 RegisterCustomFitPredicate 方法注册该策略。该策略用于判断策略列出的标签在备选节点中存在时，是否选择该备选节点。 

+  读取备选节点的标签列表信息。
+  如果策略配置的标签列表存在于备选节点的标签列表中，且策略配置的 presence值为 false，则返回 false，否则返回true；如果策略配置的标签列表不存在于备选节点的标签列表中，且策略配置的 presence 值为 true，则返回 false，否则返回 true。 

6. CheckServiceAffinity

如果用户在配置文件中指定了该策略，则 Scheduler 会通过 RegisterCustomFitPredicate 方法注册该策略。该策略用于判断备选节点是否包含策略指定的标签，或包含和备选 Pod 在相同 Service 和 Namespace 下的 Pod 所在节点的标签列表。如果存在，则返回true，否则返回false。 

7. PodFitsPorts

判断备选 Pod 所用的端口列表中的端口是否在备选节点中已被占用，如果被占用，则返回 false，否则返回 true。

### 优选

Scheduler 中的优选策略包含：LeastRequestedPriority、 CalculateNodeLabelPriority 和 BalancedResourceAllocation 等。每个节点通过优先选择策略时都会算出一个得分，计算各项得分，最终选出得分值最大的节点作为优选的结果（也是调度算法的结果）。

下面是对所有优选策略的详细说明

1. LeastRequestedPriority

该优选策略用于从备选节点列表中选出资源消耗最小的节点。 

+  计算出在所有备选节点上运行的 Pod 和备选 Pod 的CPU占用量 totalMilliCPU。
+  计算出在所有备选节点上运行的 Pod 和备选 Pod 的内存占用量 totalMemory。
+  计算每个节点的得分，计算规则大致如下，其中， NodeCpuCapacity 为节点 CPU 计算能力，NodeMemoryCapacity 为节点内存大小：

```
score=int((((nodeCpuCapacity - totalMillCPU) * 10 ) / nodeCpuCapacity + ((nodeMemoryCapacity - totalMemory) * 10) / nodeCpuMemory) / 2)
```

2. CalculateNodeLabelPriority

如果用户在配置文件中指定了该策略，则 scheduler 会通过 RegisterCustomPriorityFunction 方法注册该策略。该策略用于判断策略列出的标签在备选节点中存在时，是否选择该备选节点。如果备选节点的标签在优选策略的标签列表中且优选策略的 presence 值为 true，或者备选节点的标签不在优选策略的标签列表中且优选策略的 presence 值为 false，则备选节点 score=10，否则备选节点 score=0。 

3. BalancedResourceAllocation

该优选策略用于从备选节点列表中选出各项资源使用率最均衡的节
点。

+  计算出在所有备选节点上运行的Pod和备选Pod的CPU占用量 totalMilliCPU。
+  计算出在所有备选节点上运行的Pod和备选Pod的内存占用量 totalMemory。
+  计算每个节点的得分，计算规则大致如下，其中， NodeCpuCapacity 为节点的 CPU 计算能力，NodeMemoryCapacity 为节点的内存大小：

```
score = int(10- math.Abs(totalMillCPU / nodeCpuCapacity - totalMemory / nodeMemoryCapacity) * 10)
```
