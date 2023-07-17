# golang

centos 安装

去官网下载[golang](https://go.dev/dl/)安装包。

执行命令：

```shell
tar -C /usr/local/ -xzf go1.20.6.linux-amd64.tar.gz 
```

安装完成后：

```shell
[root@bogon ~]# go version
go version go1.20.6 linux/amd64
```

设置环境变量：

```shell
vim /etc/profile

# 文件末尾加入以下环境变量
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/home/go
export GOPROXY=https://goproxy.cn
```