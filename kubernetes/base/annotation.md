# annotation

Annotation（注解）与 Label 类似，也使用 key/value 键值对的形式进 行定义。不同的是 Label 具有严格的命名规则，它定义的是 Kubernetes 对 象的元数据（Metadata），并且用于 Label Selector。Annotation 则是用户任意定义的附加信息，以便于外部工具查找。在很多时候，Kubernetes 的模块自身会通过 Annotation 标记资源对象的一些特殊信息。

通常来说，用 Annotation 来记录的信息如下。 

+ build 信息、release 信息、Docker 镜像信息等，例如时间戳、 release id号、PR 号、镜像 Hash 值、Docker Registry 地址等。
+ 日志库、监控库、分析库等资源库的地址信息。
+ 程序调试工具信息，例如工具名称、版本号等。
+ 团队的联系信息，例如电话号码、负责人名称、网址等。
