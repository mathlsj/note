# Network Policy

Network Policy 提供了基于策略的网络控制，用于隔离应用并减少攻击面。它使用标签 选择器模拟传统的分段网络，并通过策略控制它们之间的流量以及来自外部的流量。

## Namespaces 隔离

默认情况下，所有Pod之间是全通的。每个Namespace可以配置独立的网络策略，来隔 离Pod之间的流量。

通过创建匹配所有 Pod 的 Network Policy 来作为默认的网络策略，比如默认拒 绝所有Pod之间通信

```
apiVersion: networking.k8s.io/v1 
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}

```

默认允许所有Pod通信的策略为

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec: 
  podSelector: {}
  ingress:
  -  {}
```

## Pod 隔离

通过使用标签选择器（包括 namespaceSelector 和 podSelector）来控制 Pod 之间的流 量。比如下面的 Network Policy

+ 允许 default namespace 中带有 role=frontend 标签的 Pod 访问 default namespace 中带有 role=db 标签 Pod 的6379端口 
+ 允许带有 project=myprojects 标签的 namespace 中所有 Pod 访问 default namespace 中带有 role=db 标签 Pod 的6379端口

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - namespaceSelector: 
        matchLabels:
          project: myproject
    - podSelector: 
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
```
