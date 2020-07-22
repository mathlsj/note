# 服务发现和负载均衡

kubernetes 中的负载均衡大致可以分为以下几种机制，每种机制都有其特定的应用 场景：

+ Service：直接用Service提供cluster内部的负载均衡，并借助cloud provider提供的 LB提供外部访问。
+ Ingress Controller：还是用Service提供cluster内部的负载均衡，但是通过自定义LB 提供外部访问 Service。
+ Load Balancer：把 load balancer 直接跑在容器中，实现 Bare Metal 的 Service Load Balancer。
+ Custom Load Balancer：自定义负载均衡，并替代 kube-proxy，一般在物理部署 Kubernetes时使用，方便接入公司已有的外部服务。

## Service

Service 是对一组提供相同功能的 Pods 的抽象，并为它们提供一个统一的入口。借助 Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。 Service 通过标签来选取服务后端。这些匹配标签的 Pod IP 和端口列表组成 endpoints，由 kubeproxy 负责将服务IP负载均衡到这些 endpoints 上。

Service 有四种类型：

+ ClusterIP：默认类型，自动分配一个仅 cluster 内部可以访问的虚拟 IP。
+ NodePort：在 ClusterIP 基础上为 Service 在每台机器上绑定一个端口，这样就可以通过 <NodeIP>:NodePort 来访问该服务。
+ LoadBalancer：在 NodePort 的基础上，借助 cloud provider 创建一个外部的负载均 衡器，并将请求转发到 <NodeIP>:NodePort。
+ ExternalName：将服务通过 DNS CNAME 记录方式转发到指定的域名（通 过 spec.externlName 设定）。需要 kube-dns 版本在 1.7 以上。

另外，也可以将已有的服务以 Service 的形式加入到 Kubernetes 集群中来，只需要在创建 Service 的时候不指定 Label selector，而是在 Service 创建好后手动为其添加 endpoint。

### 定义

定义一个 nginx 服务 ，将服务的 80 端口转发到 default 命名空间中带有标签 app=nginx 的 Pod 的 80 端口

```
# service.yaml
apiVersion: v1 
kind: Service 
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
```

### 查看

```
# 创建服务
[root@master ~]# kubectl apply -f service.yaml 
service/nginx created

# 获取服务，其自动分配的 cluster ip: 10.105.149.161
[root@master ~]# kubectl get service nginx
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.105.149.161   <none>        80/TCP    31s

# 查看 service
[root@master ~]# kubectl describe service nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       Selector:  app=nginx
Type:              ClusterIP
IP:                10.105.149.161
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

### 不指定 selectors

在创建 Service 的时候，也可以不指定 Selectors，用来将 service 转发到 kubernetes 集群 外部的服务（而不是 Pod ）。目前支持两种方法

1. 自定义 endpoint，即创建同名的 service 和 endpoint，在 endpoint 中设置外部服务的 IP 和端口。

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8088
---
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 1.2.3.4
  ports:
  - port: 8088
```

2. 通过 DNS 转发，在 service 定义中指定 externalName。此时 DNS 服务会 给 <service-name>.<namespace>.svc.cluster.local 创建一个 CNAME 记录，其值为 my.database.example.com。并且，该服务不会自动分配 Cluster IP，需要通过 service 的 DNS 来访问（这种服务也称为 Headless Service）。

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  type: ExternalName
  externalName: my.database.example.com
```

### 保留源 IP

各种类型的Service对源IP的处理方法不同：

+ ClusterIP Service：使用 iptables 模式，集群内部的源 IP 会保留（不做SNAT）。如 果 client 和 server pod在同一个 Node 上，那源 IP 就是 client pod的IP地址；如果在不 同的 Node 上，源 I P则取决于网络插件是如何处理的，比如使用 flannel 时，源IP是
Service node flannel IP地址。
+ NodePort Service：源 IP 会做 SNAT，server pod看到的源 IP 是Node IP。为了避免 这种情况，可以给 service 加上 annotation service.beta.kubernetes.io/external-traffic=OnlyLocal，让 service 只代理本地 endpoint 的请求（如果没有本地 endpoint 则直接丢包），从而保留源IP。
+ LoadBalancer Service：源 IP 会做SNAT，server pod看到的源 IP 是Node IP。在 GKE/GCE中，添加 annotation service.beta.kubernetes.io/externaltraffic=OnlyLocal    后可以自动从负载均衡器中删除没有本地 endpoint 的 Node。

### 删除

```
# 删除 service
[root@master ~]# kubectl delete service nginx
service "nginx" deleted

# 请求 service 后找不到资源
[root@master ~]# kubectl get service nginx
Error from server (NotFound): services "nginx" not found
```

## Ingress Controller

Service虽然解决了服务发现和负载均衡的问题，但它在使用上还是有一些限制，比如

+ 只支持4层负载均衡，没有7层功能。
+ 对外访问的时候，NodePort类型需要在外部搭 建额外的负载均衡。

想要通过负载均衡器实现不同子域名到不同服务的 访问：

```
foo.bar.com --|                   |-> foo.bar.com s1:80
              |   178.91.123.132  | 
bar.foo.com --|                   |-> bar.foo.com s2:80
```

可以这样来定义Ingress：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata: 
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend: 
        serviceName: s1
        servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
        serviceName: s2
        servicePort: 80
```


## Service Load Balanc

在 Ingress 出现以前，Service Load Balancer是推荐的解决 Service 局限性的方式。 Service Load Balancer 将 haproxy 跑在容器中，并监控 service 和 endpoint 的变化，通过 容器IP对外提供4层和7层负载均衡服务。

## Custom Load Balancer

虽然 Kubernetes 提供了丰富的负载均衡机制，但在实际使用的时候，还是会碰到一些复 杂的场景是它不能支持的，比如

+ 接入已有的负载均衡设备 
+ 多租户网络情况下，容器网络和主机网络是隔离的，这样 kube-proxy 就不能正常工作

这个时候就可以自定义组件，并代替 kube-proxy 来做负载均衡。基本的思路是监控 kubernetes 中 service 和 endpoints 的变化，并根据这些变化来配置负载均衡器。比如 weave  flux、nginx  plus、kube2haproxy等
