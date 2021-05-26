# Namespace理解




## Linux tun/tap 

### tun/tap基础概念
tun/tap分别是网络层与链路层的**虚拟网络设备**。

Tun用于收发第三层数据报文包，如IP包。因此用用于点对点IP隧道，如OpenVPN，IPSec等。
Tap用于收发第二层数据报文包，如以太网帧。Tap最常见的用途就是做虚拟机的网卡。因为它的普通的物理网卡更加相近，所以也经常用作普通机器的虚拟网卡。


tun/tap可以通过网络接口/字符设备两种方式进行操作。

当应用程序使用标准网络接口socket API操作tun/tap设备时，和操作一个真实网卡无异。

当应用程序打开字符设备时，系统会自动创建对应的虚拟设备接口，一般以tunX和tapX方式命名，虚拟设备接口创建成功后，可以为其配置IP、MAC地址、路由等。
当一切配置完毕，应用程序通过此字符文件设备写入IP封包或以太网数据帧，tun/tap的驱动程序会将数据报文直接发送到内核空间，
内核空间收到数据后再交给系统的网络协议栈进行处理，最后网络协议栈选择合适的物理网卡将其发出，到此发送流程完成。
而物理网卡收到数据报文时会交给网络协议栈进行处理，网络协议栈匹配判断之后通过tun/tap的驱动程序将数据报文原封不动的写入到字符设备上，
应用程序从字符设备上读取到IP封包或以太网数据帧，最后进行相应的处理，收取流程完成。

当应用程序关闭字符设备时，系统也会自动删除对应的虚拟设备接口，并且会删除创建的路由等信息。

### 基本使用
tap/tun增加：
```shell
//add tun/tap
ip tuntap add dev tun0 mode tun 
ip tuntap add dev tap0 mode tap 

//del tun/tap
ip tuntap del dev tun0 mode tun
ip tuntap del dev tap0 mode tap
ip link del tun0
ip link del tap0
```

参考文章：[Linux tun/tap 详解](https://mp.weixin.qq.com/s/YW3_22bkxZcjjzOxRXlUBQ)


## Linux Net Namespace

ip命令用来操纵linux主机的路由、网络设备、策略路由和隧道，**是linux比较新且功能强大的网络配置工具。**


```shell
# 添加并启动虚拟网卡tap设备
ip tuntap add dev tap0 mode tap 
ip tuntap add dev tap1 mode tap 
ip link set tap0 up
ip link set tap1 up
# 配置IP
ip addr add 11.0.0.1/24 dev tap0
ip addr add 11.0.0.2/24 dev tap1
# 查看tap,addr可以查看ip,但是link查看不到ip
ip addr show tap0
ip addr show tap1
ip link show tap0
ip link show tap1

# 添加netns
ip netns add ns0
ip netns add ns1
# 将虚拟网卡tap0，tap1分别移动到ns0和ns1中
ip link set tap0 netns ns0
ip link set tap1 netns ns1
```
这个时候使用如下命令：
```shell
# 以下命令发现都是
ping 11.0.0.1
ping 11.0.0.2
ip netns exec ns0 ping 11.0.0.2
ip netns exec ns0 ping 11.0.0.2

# 进入ns0的net namespace
ip netns exec ns0 bash
ip link show / ip addr show 
```
可以看到不仅本地环回lo和tap设备的状态都是DOWN，甚至就连tap设备的IP信息也没有了，这是因为在不同的网络命名空间中移动虚拟网络接口时会重置虚拟网络接口的状态。

```shell
ip netns exec ns0 ip link set lo up
ip netns exec ns0 ip link set tap0 up
ip netns exec ns0 ip addr add 10.0.0.1/24 dev tap0

ip netns exec ns1 ip link set lo up
ip netns exec ns1 ip link set tap1 up
ip netns exec ns1 ip addr add 10.0.0.2/24 dev tap1

```
再执行如下：
```shell
[root@fabric-test-4 centos]# ip netns exec ns0 ping 11.0.0.1
PING 11.0.0.1 (11.0.0.1) 56(84) bytes of data.
64 bytes from 11.0.0.1: icmp_seq=1 ttl=64 time=0.073 ms
64 bytes from 11.0.0.1: icmp_seq=2 ttl=64 time=0.034 ms
^C
--- 11.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.034/0.053/0.073/0.020 ms
[root@fabric-test-4 centos]# ip netns exec ns0 ping 11.0.0.2
PING 11.0.0.2 (11.0.0.2) 56(84) bytes of data.

From 11.0.0.1 icmp_seq=1 Destination Host Unreachable
From 11.0.0.1 icmp_seq=2 Destination Host Unreachable
From 11.0.0.1 icmp_seq=3 Destination Host Unreachable
From 11.0.0.1 icmp_seq=4 Destination Host Unreachable
^C
--- 11.0.0.2 ping statistics ---
5 packets transmitted, 0 received, +4 errors, 100% packet loss, time 4000ms
```
可以看出没有任何ICMP回复包，netns确实把在同一台主机上的两张虚拟网卡隔离起来了。在这里我们只是简单的使用ping命令来测试网络的连通性，实际上可以做到更多，
例如修改某一个netns的路由表或者防火墙规则，完全不会影响到其他的netns，当然也不会影响到宿主机器，在这里由于篇幅原因就不再展开实验了，
感兴趣的同学可以实验一下。下一节我们将学习另一个网络设备veth pair，使用它来把两个netns连接起来，让两个隔离的netns之间可以互相通信。


ip addr| link | route | netns  


