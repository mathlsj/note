# Service Account

Service account 是为了方便Pod里面的进程调用 Kubernetes API 或其他外部服务而设计 的。它与 User account 不同

+ User account 是为用户设计的，而 service account 则是为 Pod 中的进程调用 Kubernetes API 而设计；
+ User account 是跨 namespace 的，而 service account 则是仅局限它所在的 namespace； 
+ 每个 namespace 都会自动创建一个 default service account 
+ Token controller 检测 service account 的创建，并为它们创建 secret 
+ 开启 ServiceAccount Admission Controller后 
    + 每个 Pod 在创建后都会自动设置 spec.serviceAccount 为 default（除非指定 了其他 ServiceAccout） 
    + 验证 Pod 引用的 service account 已经存在，否则拒绝创建
    + 如果 Pod 没有指定 ImagePullSecrets，则把 service account 的 ImagePullSecrets 加到 Pod 中 
    + 每个container 启动后都会挂载该 service account 的 token 和 ca.crt 到  /var/run/secrets/kubernetes.io/serviceaccount/

## 创建

```
## 创建一个 serviceaccount 名为 admin
[root@master ~]# kubectl create sa admin
serviceaccount/admin created
[root@master ~]# kubectl get sa
NAME                   SECRETS   AGE
admin                  1         17s
## 查看 serviceaccount 为 admin 的信息，系统自动创建一个 token 信息
[root@master ~]# kubectl describe sa admin
Name:                admin
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-token-442gg
Tokens:              admin-token-442gg
Events:              <none>

## 查看自动创建的 secret
[root@master ~]# kubectl get  secrets admin-token-442gg -o yaml
apiVersion: v1
data:
  ca.crt: (base64)
  token: (base64)
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: admin
    kubernetes.io/service-account.uid: c44efba1-ccfc-469f-9242-be9ed5c93bb5
  creationTimestamp: "2020-07-28T13:52:17Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:ca.crt: {}
        f:namespace: {}
        f:token: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubernetes.io/service-account.name: {}
          f:kubernetes.io/service-account.uid: {}
      f:type: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-07-28T13:52:17Z"
  name: admin-token-442gg
  namespace: default
  resourceVersion: "219206"
  selfLink: /api/v1/namespaces/default/secrets/admin-token-442gg
  uid: ce23f8e2-86ae-4c95-9137-91742dbc3c13
type: kubernetes.io/service-account-token
```

## 授权

Service Account 为服务提供了一种方便的认证机制，但它不关心授权的问题。可以配 合 RBAC 来为 Service Account鉴权：

+ 配置 --authorization-mode=RBAC 和 --runtimeconfig=rbac.authorization.k8s.io/v1alpha1  
+ 配置  --authorization-rbac-super-user=admin 
+ 定义Role、ClusterRole、RoleBinding或ClusterRoleBinding
