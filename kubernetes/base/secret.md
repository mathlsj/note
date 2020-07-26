# Secret

Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露 到镜像或者 Pod Spec 中。

## 类型

Secret 有三种类型：

+ Service Account：用来访问 Kubernetes API，由 Kubernetes 自动创建，并且会自动 挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中
+ Opaque：base64 编码格式的 Secret，用来存储密码、密钥等
+ kubernetes.io/dockerconfigjson： 用来存储私有docker registry的认证信息

### Service Account

查看 pod 名称

```
[root@master ~]# kubectl get pods | grep nginx
nginx-deployment-76f4cbc5d-ql9fx   2/2     Running   0          11m
```

查看 /run/secrets/kubernetes.io/serviceaccount 目录

```
[root@master ~]# kubectl exec  nginx-deployment-76f4cbc5d-ql9fx -c nginx -- ls /run/secrets/kubernetes.io/serviceaccount
ca.crt
namespace
token
```

### Opaque  Secret

Opaque类型的数据是一个map类型，要求value是base64编码格式。

创建一个 secret

```
[root@master ~]# kubectl apply -f secret.yaml 
secret/mysecret created
```

secret.yml 文件如下

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: cGFzc3dvcmQNCg== 
  username: dGVzdA==
```

#### 引用

创建好 secret 后，有两种方式使用它：

+ 以 volume 的方式
+ 以环境变量的方式

将 secret 挂到 volume 中

```
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort:  80
        volumeMounts:
        - name: secrets
          mountPath: /etc
      volumes:
      - name: secrets
        secret:
          secretName: mysecret
```

导出到环境变量

```
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort:  80
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
```

### kubernetes.io/dockerconfigjson

可以直接读取文件的内容来创建 secret

```
kubectl create secret docker-registry myregistrykey --from-file="~/.dockercfg"
```

在创建Pod的时候，通过 imagePullSecrets 来引用刚创建的 myregistrykey :

```
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
      imagePullSecrets:
      - name: myregistrykey
```
