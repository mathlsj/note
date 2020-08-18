# iptables 的增删改查

关于 iptables 的四表五链，请看[iptables 四表五链](./iptables-base.md)

## 查看

```
iptables -nvL --line-number

-L  查看当前表的所有规则，默认查看的是 filter 表，如果要其它表，加上 -t 参数
-v  输出详细信息，包含通过该规则的数据包数量，总字节数及相应的网络接口 
-n  数值输出。IP地址和端口号将以数字格式打印。默认情况下，程序将尝试将它们显示为主机名、网络名或服务（只要适用）
--line-number 显示规则的序列号，这个参数在删除或修改规则时会用到
```

```
iptable -S INPUT

-S 查看当前表的其中一条链，可以是自定义链。默认查看的是 filter 表。如果要查看其它表加上 -t 参数
```

## 添加

添加规则有两个参数 `-I` 和 `-A`

+ -I（insert）：默认添加到规则的首部
+ -A（append）：默认添加规则的到尾部

```
## 增加单个 IP
iptables -I INPUT -s 192.168.239.200 --sport 80 -j ACCEPT

## 增加网段
iptables -A INPUT -s 192.168.239.0/24 -p TCP --dport 22 -j ACCEPT

## 多端口
iptables -A INPUT -s 127.0.0.1,192.168.239.1 -p tcp -m multiport --dport 22,23 -j ACCEPT
```

由于 iptables 的规则是从上往下匹配，如果策略不冲突的情况下，－I 和 －A 是一样的。

## 修改

```
iptables -R INPUT 2 -s 192.168.239.1 -p TCP --dport 22 -j DROP 
2是--line-nember查到的，所有参数都要写 -R 不是在原策略基础上修改而就是直接替代。
```

## 删除

```
iptables -D INPUT 1 
1是 --line-number 查到的，此句会删除第1条策略

iptables -X 是清空自定义的规则
iptables -F 是清除所有链的规则
```