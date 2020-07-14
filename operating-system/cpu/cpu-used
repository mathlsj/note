# CPU 使用率

cpu 使用率通过测量一段时间内 CPU 实例忙于执行工作的时间比例获得，以百分比显示。CPU 花在执行用户态应用程序代码的时间称为用户时间，而执行内核态代码的时间称为内核时间。内核时间包括系统调用，内核线程和中断时间。

Linux 作为一个多任务操作系统，将每个 CPU 的时间划分为很短的时间片，再通过调度器轮流分配给各个任务使用。

##　如何查看

查看 CPU 使用率，第一反应是用 top 和 pidstat。

+ top： 显示了系统总体的 CPU 和内存使用情况，以及各个进程的资源使用情况。
+ pidstat: 显示每个进程的资源使用情况。

如下： top 的输出格式

```
## 默认 3 秒刷新，可通过 -d 更改刷新时间。
[root@master ~]# top
top - 22:59:17 up  2:05,  2 users,  load average: 1.18, 0.69, 0.55
Tasks: 147 total,   1 running, 146 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  6.5 sy,  0.0 ni, 90.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2895444 total,   862984 free,   947616 used,  1084844 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1656988 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                          
 10880 root      20   0  636904 487664  43424 S  12.5 16.8  16:02.11 kube-apiserver                                                   
 47536 root      20   0  162012   2164   1552 R  12.5  0.1   0:00.02 top                                                              
  6565 root      20   0  836020  80684  24168 S   6.2  2.8   2:57.92 dockerd
```

在这个输出结果，第三行 %Cpu(s) 就是系统 CPU 的使用率。每一列的涵义我们来解读一下。

+ us（user 的缩写），代表用户态 CPU 时间。
+ sy（system 的缩写），代表内核态 CPU 时间。
+ ni（nice 的缩写），代表低优先级用户态 CPU 时间，也就是进程的 nice 值被调整为 1-19 之间时的 CPU 时间。这里注意，nice 可取值范围是 -20 到 19，数值越大，优先级反而越低。
+ id（idle 的缩写），代表空闲时间。它不包括等待 I/O 的时间（iowait）。
+ wa（iowait 的缩写），代表等待 I/O 的 CPU 时间。
+ hi（irq 的缩写），代表处理硬中断的 CPU 时间。
+ si（softirq 的缩写），代表处理软中断的 CPU 时间。
+ st（steal 的缩写），代表当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间。

top 默认显示所有的 CPU 的平均值，按数字 1，就可以切换到每个 CPU 的使用率。

往下看，在空行之后是进程的实时使用信息，%CPU 列就是每个进程的 CPU 使用率。这个使用率是用户态和内核态 CPU 使用率的总和，包括进程用户空间使用的 CPU、通过系统调用执行的内核空间 CPU 、以及在就绪队列等待运行的 CPU。在虚拟化环境中，它还包括了运行虚拟机占用的 CPU。

从这里可以发现，top 没有分进程的用户态和内核态。要查看每个进程的详细情况，就要用到 pidstat。

如下： pidstat 的输出格式

```
## 间隔 1 秒输出，输出 2 组
[root@master ~]# pidstat 1 2
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	07/07/2020 	_x86_64_	(2 CPU)

11:21:03 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
11:21:04 PM     0      6565    0.98    0.98    0.00    1.96     1  dockerd
11:21:04 PM     0      6566    2.94    1.96    0.00    4.90     0  kubelet


11:21:04 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
11:21:05 PM     0      6565    4.00    0.00    0.00    4.00     1  dockerd
11:21:05 PM     0      6566    5.00    3.00    0.00    8.00     0  kubelet


Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0      6565    2.48    0.50    0.00    2.97     -  dockerd
Average:        0      6566    3.96    2.48    0.00    6.44     -  kubelet

```

每个列的涵义如下：

+ PID 进程 ID。
+ %usr 用户态 CPU 使用率。
+ %system 内核态 CPU 使用率。
+ %guest 运行虚拟机 CPU 使用率。
+ %CPU 总的 CPU 使用率。

最后的 Average 部分，计算 2 组数据的平均组。

如何查看机子的 CPU 核数。 linux 的 CPU 信息都放在 `/proc/cpuinfo`

```
## 查看物理 CPU 个数
[root@master ~]# cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
1

## 查看每个物理 CPU 中 core 的个数
[root@master ~]# cat /proc/cpuinfo | grep "cpu cores" | uniq
cpu cores	: 2

## 查看逻辑 CPU 个数 
[root@master ~]# cat /proc/cpuinfo| grep processor | wc -l
2
```

## 如何计算

Linux 通过 /proc 虚拟文件系统，向用户空间提供了系统内部状态的信息，而 /proc/stat 提供的就是系统的 CPU 和任务统计信息。只关注 CPU 的话，可以用如下命令：

```
[root@master ~]# cat /proc/stat
cpu  132224 2 107505 1575279 3067 0 13496 0 0 0
cpu0 66279 0 54436 787795 1075 0 6618 0 0 0
cpu1 65944 2 53068 787484 1991 0 6877 0 0 0
```

+ 第一列： 表示的 CPU 的编号。如 CPU0、CPU1，第一行没有编号的 CPU 表示所有 CPU 的累加。
+ 第二列： user（通常缩写为 us），代表用户态 CPU 时间。注意，它不包括下面的 nice 时间，但包括了 guest 时间。
+ 第三列： nice（通常缩写为 ni），代表低优先级用户态 CPU 时间。
+ 第四列： system（通常缩写为 sys），代表内核态 CPU 时间。
+ 第五列： idle（通常缩写为 id），代表空闲时间。
+ 第六列： iowait（通常缩写为 wa），代表等待 I/O 的 CPU 时间。
+ 第七列： irq（通常缩写为 hi），代表处理硬中断的 CPU 时间。
+ 第八列： softirq（通常缩写为 si），代表处理软中断的 CPU 时间。
+ 第九列： steal（通常缩写为 st），代表当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间。
+ 第十列： guest（通常缩写为 guest），代表通过虚拟化运行其他操作系统的时间，也就是运行虚拟机的 CPU 时间。
+ 第十一列： guest_nice（通常缩写为 gnice），代表以低优先级运行虚拟机的时间。

而我们通常所说的 CPU 使用率，就是除了空闲时间外的其他时间占总 CPU 时间的百分比。`/proc/stat` 用的是开机以来 CPU 使用的累加，直接计算算出的是从开机到现在的 CPU 使用率，没有参考价值。为了计算 CPU 使用率，一般会取间隔一段时间（比如 top 的 3 秒）的两次差值。用公式来表示：

```
usertime = usertime - guest;
nicetime = nicetime - guestnice;
idlealltime = idletime + ioWait;
systemalltime = systemtime + irq + softIrq;
virtalltime = guest + guestnice;
totaltime = usertime + nicetime + systemalltime + idlealltime + steal + virtalltime;

CPU 使用率 ＝ 1 － （idlealltime － prev_idlealltime）/ （totaltime － prev_totaltime）
```