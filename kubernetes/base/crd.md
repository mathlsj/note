# CustomResourceDefinition

CustomResourceDefinition（CRD）无需改变代码就可以扩展 Kubernetes    API 的机制，用来管理自定义对象。

## 例子：

下面的例子会创建一 个 /apis/stable.example.com/v1/namespaces/<namespace>/crontabs/... 的 API

```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plura l>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # version name to use for REST API: /apis/<group>/<version>
  version: v1
  # either  Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plu ral>
    plural: crontabs 
    # singular name to b use as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased ingular type.  You resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```

API 创建好后，就可以创建具体的 CronTab 对象

```
apiVersion: "stable.example.com/v1" 
kind: CronTab 
metadata:       
 name: my-new-cron-object 
spec:       
  cronSpec: "* * * * /5"        
  image: my-awesome-cron-image
```

```
$ kubectl create -f my-crontab.yaml crontab "my-new-cron-object" created

```

## Finalize

Finalizer 用于实现控制器的异步预删除钩子，可以通过 metadata.finalizers 来指定 Finalizer。

```
apiVersion: "stable.example.com/v1" 
kind: CronTab 
metadata: 
  finalizers:       
  - finalizer.stable.example.com 
spec:       
  group: stable.example.com     
  version: v1       
  scope: Namespaced     
  names:                
    plural: crontabs                
    singular: crontab               
    kind: CronTab               
    shortNames:             
    - ct
```

Finalizer 指定后，客户端删除对象的操作只会设置 metadata.deletionTimestamp  而 不是直接删除。这会触发正在监听 CRD 的控制器，控制器执行一些删除前的清理操作， 从列表中删除自己的 finalizer，然后再重新发起一个删除操作。此时，被删除的对象才会 真正删除。