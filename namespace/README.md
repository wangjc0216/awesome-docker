# Namespace理解

## Net Namespace

tun/tap分别是网络层与链路层的**虚拟网络设备**。
Tun用于收发第三层数据报文包，因此用用于点对点IP隧道，如OpenVPN，IPSec等。
Tap用于收发第二层数据报文包，Tap最常见的用途就是做虚拟机的网卡。因为它的普通的物理网卡更加相近，所以也经常用作普通机器的虚拟网卡。
