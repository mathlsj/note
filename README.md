# note

一些学习和使用过程的汇总。主要包括学习笔记，源码分析和平时使用的总结。

## linux

### cpu

* [cpu 使用率](./operating-system/cpu/cpu-used.md)
* [cpu 平均负载](./operating-system/cpu/cpu-load.md)
* [cpu 上下文切换](./operating-system/cpu/cpu-context.md)

### memory

### 网络

* [iptables 四表五链](./operating-system/network/iptables-base.md)
* [iptables 基本操作](./operating-system/network/iptables-operator.md)
* [dns 解析过程](./operating-system/network/dns-parse-progress.md)

### 硬盘

## 架构设计

* [容量设计](./architecture/capacity.md)
* [百万流量架构设计](./architecture/traffic.md)
* [为什么需要服务化](./architecture/micro-service-soltion.md)
* [微服务拆分粒度](./architecture/micro-service-resolution.md)
* [微服务高可用](./architecture/micro-service-ha.md)
* [微服务高并发](./architecture/micro-service-high.md)
* [微服务负载](./architecture/micro-service-balance.md)

## kubernetes

### 基础

* [k8s 架构](./kubernetes/base/kubernetes-arch.md)
* [pod](./kubernetes/base/pod.md)
* [namespace](./kubernetes/base/namespace.md)
* [master](./kubernetes/base/master.md)
* [node](./kubernetes/base/node.md)
* [service](./kubernetes/base/service.md)
* [volume](./kubernetes/base/volume.md)
* [pv](./kubernetes/base/pv.md)
* [deployment](./kubernetes/base/deployment.md)
* [secret](./kubernetes/base/secret.md)
* [statefulset](./kubernetes/base/statefulset.md)
* [daemonset](./kubernetes/base/daemonset.md)
* [node-selector](./kubernetes/base/node-selector.md)
* [service-account](./kubernetes/base/service-account.md)
* [job](./kubernetes/base/job.md)
* [cronjob](./kubernetes/base/cronjob.md)
* [resource-quota](./kubernetes/base/resource-quota.md)
* [network-policy](./kubernetes/base/network-policy.md)
* [configmap](./kubernetes/base/configmap.md)
* [annotation](./kubernetes/base/annotation.md)
* [crd](./kubernetes/base/crd.md)

### 原理

* [apiserver](./kubernetes/advance/apiserver.md)
* [namespace-controller](./kubernetes/advance/namespace-controller.md)
* [node-controller](./kubernetes/advance/node-controller.md)
* [replication-controller](./kubernetes/advance/replication-controller.md)
* [resource-quota-controller](./kubernetes/advance/resource-qouta-controller.md)
* [scheduler](./kubernetes/advance/scheduler.md)
* [kube-proxy](./kubernetes/advance/kube-proxy.md)

### 源码分析

## Envoy

### 源码分析

* [event事件](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_event.md)
* [buffer](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_buffer.md)
* [网络](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_network.md)
* [网络L4过滤管理](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_l4_filter_manager.md)
* [程序初始化](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_initialization.md)
* [LDS](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/envoy_lds.md)
* [热重启](https://github.com/mathlsj/envoy-sourcecode-analysis/blob/master/evnvoy_hot_restart.md)
