# ConfigMap

ConfigMap 用于保存配置数据的键值对，可以用来保存单个属性，也可以用来保存配置文件。 ConfigMap 跟 secret 很类似，但它可以更方便地处理不包含敏感信息的字符串。

## 创建

configmap 有多种创建方式，可以从 k-v，环境文件，目录，yaml/json 文件创建。

### keyvalue

```
[root@master ~]# kubectl create cm test-cm --from-literal=test.how=what
configmap/test-cm created
[root@master ~]# kubectl  get cm test-cm -o     go-template='{{.data}}' 
map[test.how:what]
```

### 环境文件变量

```
[root@master ~]# echo -e "a=b\nc=d" > config.env
[root@master ~]# cat config.env 
a=b
c=d
[root@master ~]# kubectl create configmap env-file --from-env-file=config.env
configmap/env-file created
[root@master ~]# kubectl get configmap env-file -o go-template='{{.data}}' 
map[a:b c:d]
```

### 目录

```
[root@master ~]# mkdir test-file
[root@master ~]# echo "a" > test-file/a
[root@master ~]# echo "b" > test-file/b
[root@master ~]# kubectl create configmap cm-file --from-file=test-file
configmap/cm-file created
[root@master ~]# kubectl get configmap cm-file -o go-template='{{.data}}' 
map[a:a
 b:b
]
```

### yaml/json 文件

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm-file
  namespace: default
data:
  special.how: very
  special.type: much
```

```
[root@master ~]# kubectl apply -f cm.yaml 
configmap/test-cm-file created
```

## 使用

ConfigMap 可以通过三种方式在 Pod 中使用，三种分别方式为：设置环境变量、设置容 器命令行参数以及在 Volume 中直接挂载文件或目录。

`注意`

+ ConfigMap 必须在 Pod 引用它之前创建
+ 使用 envFrom 时，将会自动忽略无效的键
+ Pod 只能使用同一个命名空间内的 ConfigMap

### 用作环境变量

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
  namespace: test
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command: ["/bin/bash", "-c", "env"]
        env:
        - name: TEST_KEY
          valueFrom:
            configMapKeyRef:
              name: test-cm
              key: special.how
        - name: TEST_TYPE
          valueFrom:
            configMapKeyRef:
              name: test-cm
              key: special.type
```

### 容器命令行

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
  namespace: test
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command: ["/bin/bash", "-c", "echo $(TEST_KEY)$(TEST_TYPE)"]
        env:
        - name: TEST_KEY
          valueFrom:
            configMapKeyRef:
              name: test-cm
              key: special.how
        - name: TEST_TYPE
          valueFrom:
            configMapKeyRef:
              name: test-cm
              key: special.type
```

### volume 挂载

```
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
  namespace: test
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command: ["/bin/bash", "-c", "cat /etc/special.how"]
        volumeMounts:
        - name: config
          mountPath: /etc
      volumes:
      - name: config
        configMap:
          name: test-cm
```