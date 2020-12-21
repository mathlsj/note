# 为什么我在容器中不能kill 1号进程？

[容器实战高手课
](https://time.geekbang.org/column/article/309423)

## 问题再现

需要修改容器镜像里的一个 bug，但因为网路配置的问题，又不想为了重建 pod 去改变 pod IP。由于 Kubernetes 上是没有 restart pod 这个命令的。这样看来，他似乎只能让 pod 做个原地重启。首先想到的，就是在容器中使用 kill pid 1 的方式重启容器。

### 如何理解 init 进程？

一个 Linux 操作系统，在系统打开电源，执行 BIOS/boot-loader 之后，就会由 boot-loader 负责加载 Linux 内核。

Linux 内核执行文件一般会放在 /boot 目录下，文件名类似 vmlinuz*。在内核完成了操作系统的各种初始化之后，这个程序需要执行的第一个用户态程就是 init 进程。

内核代码启动 1 号进程的时候，在没有外面参数指定程序路径的情况下，一般会从几个缺省路径尝试执行 1 号进程的代码。这几个路径都是 Unix 常用的可执行代码路径。

系统启动的时候先是执行内核态的代码，然后在内核中调用 1 号进程的代码，从内核态切换到用户态。

但无论是哪种 Linux init 进程，它最基本的功能都是创建出 Linux 系统中其他所有的进程，并且管理这些进程。

在 Linux 上有了容器的概念之后，一旦容器建立了自己的 Pid Namespace（进程命名空间），这个 Namespace 里的进程号也是从 1 开始标记的。所以，容器的 init 进程也被称为 1 号进程。

1 号进程是第一个用户态的进程，由它直接或者间接创建了 Namespace 中的其他进程。

### 如何理解 Linux 信号？

其实信号这个概念在很早期的 Unix 系统上就有了。它一般会从 1 开始编号，通常来说，信号编号是 1 到 31，这个编号在所有的 Unix 系统上都是一样的。

```
[root@master ~]# kill -l
 1) SIGHUP   2) SIGINT   3) SIGQUIT  4) SIGILL   5) SIGTRAP
 6) SIGABRT  7) SIGBUS   8) SIGFPE   9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG  24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF 28) SIGWINCH    29) SIGIO   30) SIGPWR
31) SIGSYS
```

用一句话来概括，信号（Signal）其实就是 Linux 进程收到的一个通知。

比如下面这几个典型的场景，：

+ 如果我们按下键盘“Ctrl+C”，当前运行的进程就会收到一个信号 SIGINT 而退出；
+ 如果我们的代码写得有问题，导致内存访问出错了，当前的进程就会收到另一个信号 SIGSEGV；
+ 我们也可以通过命令 kill <pid>，直接向一个进程发送一个信号，缺省情况下不指定信号的类型，那么这个信号就是 SIGTERM。也可以指定信号类型，比如命令 “kill -9 ”, 这里的 9，就是编号为 9 的信号，SIGKILL 信号。

进程在收到信号后，就会去做相应的处理。怎么处理呢？对于每一个信号，进程对它的处理都有下面三个选择。

第一个选择是忽略（Ignore），就是对这个信号不做任何处理，但是有两个信号例外，对于 SIGKILL 和 SIGSTOP 这个两个信号，进程是不能忽略的。这是因为它们的主要作用是为 Linux kernel 和超级用户提供删除任意进程的特权。

第二个选择，就是捕获（Catch），这个是指让用户进程可以注册自己针对这个信号的 handler。具体怎么做我们目前暂时涉及不到，你先知道就行，我们在后面课程会进行详细介绍。

还有一个选择是缺省行为（Default），Linux 为每个信号都定义了一个缺省的行为，你可以在 Linux 系统中运行 man 7 signal来查看每个信号的缺省行为。

我们再来了解一下 SIGKILL (9)，这个信号是 Linux 里两个特权信号之一。什么是特权信号呢？

特权信号就是 Linux 为 kernel 和超级用户去删除任意进程所保留的，不能被忽略也不能被捕获。那么进程一旦收到 SIGKILL，就要退出。

## 现象解释

在我们运行 kill 1 这个命令的时候，希望把 SIGTERM 这个信号发送给 1 号进程。在 Linux 实现里，kill 命令调用了 kill() 的这个系统调用（所谓系统调用就是内核的调用接口）而进入到了内核函数 sys_kill()。而内核在决定把信号发送给 1 号进程的时候，会调用 sig_task_ignored() 这个函数来做个判断，这个判断有什么用呢？它会决定内核在哪些情况下会把发送的这个信号给忽略掉。如果信号被忽略了，那么 init 进程就不能收到指令了。Linux 内核针对每个 Namespace 里的 init 进程，把只有 default handler 的信号都给忽略了。如果我们自己注册了信号的 handler（应用程序注册信号 handler 被称作"Catch the Signal"），那么这个信号 handler 就不再是 SIG_DFL 。即使是 init 进程在接收到 SIGTERM 之后也是可以退出的。

由于 SIGKILL 是一个特例，因为 SIGKILL 是不允许被注册用户 handler 的（还有一个不允许注册用户 handler 的信号是 SIGSTOP），那么它只有 SIG_DFL handler。所以 init 进程是永远不能被 SIGKILL 所杀，但是可以被 SIGTERM 杀死。

说到这里，我们该怎么证实这一点呢？我们可以做下面两件事来验证。

第一件事，你可以查看 1 号进程状态中 SigCgt Bitmap。

我们可以看到，在 Golang 程序里，很多信号都注册了自己的 handler，当然也包括了 SIGTERM(15)，也就是 bit 15。

而 C 程序里，缺省状态下，一个信号 handler 都没有注册；bash 程序里注册了两个 handler，bit 2 和 bit 17，也就是 SIGINT 和 SIGCHLD，但是没有注册 SIGTERM。

所以，C 程序和 bash 程序里 SIGTERM 的 handler 是 SIG_DFL（系统缺省行为），那么它们就不能被 SIGTERM 所杀。

第二件事，给 C 程序注册一下 SIGTERM handler，捕获 SIGTERM。

我们调用 signal() 系统调用注册 SIGTERM 的 handler，在 handler 里主动退出，再看看容器中 kill 1 的结果。这次我们就可以看到，在进程状态的 SigCgt bitmap 里，bit 15 (SIGTERM) 已经置位了。同时，运行 kill 1 也可以把这个 C 程序的 init 进程给杀死了。

具体我们可以看一下 /proc 系统的进程状态：
 
```
### golang init
# cat /proc/1/status | grep -i SigCgt
SigCgt:     fffffffe7fc1feff

### C init
# cat /proc/1/status | grep -i SigCgt
SigCgt:     0000000000000000

### bash init
# cat /proc/1/status | grep -i SigCgt
SigCgt:     0000000000010002

```

## 总结

好了，到这里我们可以确定这两点：

1. kill -9 1 在容器中是不工作的，内核阻止了 1 号进程对 SIGKILL 特权信号的响应。
2. kill 1 分两种情况，如果 1 号进程没有注册 SIGTERM 的 handler，那么对 SIGTERM 信号也不响应，如果注册了 handler，那么就可以响应 SIGTERM 信号。

