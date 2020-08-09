# Namespace Controller

用户通过 API Server 可以创建新的 Namespace 并将其保存在 etcd 中， Namespace Controller 定时通过 API Server 读取这些 Namespace 的信息。如果 Namespace 被API标识为优雅删除（通过设置删除期限实现，即设置 DeletionTimestamp 属性），则将该 NameSpace 的状态设置成 Terminating 并保存到etcd中。同时Namespace Controller 删除该 Namespace 下的ServiceAccount、RC、Pod、Secret、PersistentVolume、ListRange、 ResourceQuota和Event等资源对象。

在 Namespace 的状态被设置成Terminating后，由Admission Controller 的 NamespaceLifecycle 插件来阻止为该 Namespace 创建新的资 源。同时，在Namespace Controller 删除该 Namespace 中的所有资源对象后，Namespace Controller 对该 Namespace 执行 finalize 操作，删除Namespace 的 spec.finalizers 域中的信息。 

如果 Namespace Controller 观察到 Namespace 设置了删除期限，同时 Namespace 的 spec.finalizers 域值是空的，那么 Namespace Controller 将通过 API Server 删除该 Namespace 资源。