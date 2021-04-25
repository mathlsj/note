# GPU 调度

机子上的 GPU 是 `NVIDIA` 的，因此以 `NVIDIA` 为例。

## 部署 NVIDIA GPU

官方的 NVIDIA GPU 设备插件 有以下要求:

+ Kubernetes 的节点必须预先安装了 NVIDIA 驱动
+ Kubernetes 的节点必须预先安装 nvidia-docker 2.0.
+ Docker 的默认运行时必须设置为 nvidia-container-runtime，而不是 runc.
+ NVIDIA 驱动版本 ~= 384.81

### 安装 NVIDIA 驱动

到 [nvidia](https://www.nvidia.cn/Download/index.aspx?lang=cn) 官方下载驱动。安装驱动：
```
sh NVIDIA-Linux-x86_64-440.118.02.run -a -q -s
```

检查驱动是否安装成功：

```
nvidia-smi
```

### 安装 `nvidia-docker 2.0`

安装 [nvidia-docker2.0](https://github.com/NVIDIA/nvidia-docker) 这里以 `centos` 为例。

设置 `stable` 存储库和 `GPG` 密钥

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

安装 `nvidia-docker2` 包

```
sudo apt-get update
sudo apt-get install -y nvidia-docker2
```


重启 docker 

```
sudo systemctl restart docker
```

验证，注意自己安装的驱动是否支持 nvidia/cuda:11.0 的版本。

```
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

有以下显示信息说明安装成功

```
| NVIDIA-SMI 450.51.06    Driver Version: 450.51.06    CUDA Version: 11.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

### docker 默认运行时

需要修改 docker 的默认运行时。

```
cat /etc/docker/daemon.json 
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

## 使用

安装完成，可查看 work 结点上已经有 GPU 的信息

```
kubect describe nodes nodename

Capacity:
  cpu:                32
  ephemeral-storage:  205375464Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      4452Mi
  memory:             131991384Ki
  nvidia.com/gpu:     10
  pods:               110
Allocatable:
  cpu:                32
  ephemeral-storage:  189274027310
  hugepages-1Gi:      0
  hugepages-2Mi:      4452Mi
  memory:             127330136Ki
  nvidia.com/gpu:     10
  pods:               110
```

现在，k8s 可以通过像使用 CPU 和 memory 类似的方式来使用这些 GPU。但，在使用 GPU 时有一些限制。

1. 只能在 `limit` 部分中指定 GPU。
2. 容器不共享 GPU。没法超部。
3. 每个容器可以请求一个或多个 GPU。无法只请求一小部分。

下面是一个官方的例子：

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
```
