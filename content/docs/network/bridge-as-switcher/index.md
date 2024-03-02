---
title: "用bridge实现软交换"
linkTitle: "软交换"
date: 2024-01-18
weight: 1000
description: >
  在 debian 12.4 下利用 linux bridge 实现软交换
---

## 新建 linux bridge

安装 bridge 工具包：

```bash
sudo apt install bridge-utils -y
```

查看当前网卡情况：

```bash
$ ip addr

2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:97:c4:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.25/24 brd 192.168.20.255 scope global enp6s18
3: enp1s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:ff:7c brd ff:ff:ff:ff:ff:ff
4: enp1s0f1np1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:ff:7d brd ff:ff:ff:ff:ff:ff
```

enp6s18 是对外的网卡（wan），enp1s0f0np0 和 enp1s0f1np1 准备用来连接其他机器（lan）。

修改 network interfaces，删除所有和 enp6s18 / 是对外的网卡（wan），enp1s0f0np0 / enp1s0f1np1 有关的内容：

```bash
sudo vi /etc/network/interfaces
```

最后保留的内容应该非常少，类似于:

```bash
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
```

然后新建一个 br0 文件来创建网桥：

```bash
sudo vi /etc/network/interfaces.d/br0
```

内容为：

```bash
auto br0
iface br0 inet static
	address 192.168.20.2
	broadcast 192.168.20.255
	netmask 255.255.255.0
	gateway 192.168.20.1
	bridge_ports enp6s18 enp1s0f0np0 enp1s0f1np1
	bridge_stp off  
  bridge_waitport 0  
  bridge_fd 0  
```

注意这里要把所有需要加入网桥的网卡都列举在 bridge_ports 中（包括 wan 和 lan, 会自动识别） 保存后重启网络:

```bash
sudo systemctl restart networking
```

或者直接重启机器。（如果之前有配置wan口网卡的ip地址，并且新网桥的ip地址和wan的ip地址一致，就必须重启机器而不是重启网路。）

查看改动之后的网络:

```bash
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br0 state UP group default qlen 1000
    link/ether bc:24:11:97:c4:00 brd ff:ff:ff:ff:ff:ff
3: enp1s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:ff:7c brd ff:ff:ff:ff:ff:ff
4: enp1s0f1np1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:ff:7d brd ff:ff:ff:ff:ff:ff
6: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 6e:38:a0:77:09:dc brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.2/24 brd 192.168.20.255 scope global br0
       valid_lft forever preferred_lft forever
```

检查网络是否可以对外/对内访问。如果没有问题，说明第一步成功，网桥创建好了。

```bash
$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.6e38a07709dc	no		enp1s0f0np0
							enp1s0f1np1
							enp6s18

$ bridge link
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 state forwarding priority 32 cost 100 
3: enp1s0f0np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 state forwarding priority 32 cost 1 
3: enp1s0f0np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 hwmode VEB 
4: enp1s0f1np1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 state forwarding priority 32 cost 1 
4: enp1s0f1np1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 hwmode VEB 
```

网卡另一头接上其他电脑，通过 dhcp 自动获取了IP地址，而且内外可以互通，测度测试也正常。25g网卡连接速度25g，用 iperf2 测试出来的带宽有23.5g，接近25g的理论值。

软交换就这么搭建好了。

### 小结

上面这个方案，比之前在 ubuntu server 用的方案要简单很多，当时用了 bridage，需要建立子网，安装 dnsmaxq 做 dhcp 服务器端，配置内核转发，还要设置静态路由规则。

这个方案什么都不用做，就需要建立一个 bridge 就可以了。

## 开启内核转发

> 备注： 发现不做这个设置，也可以正常转发。不过还是加上吧。

```bash
sudo vi /etc/sysctl.conf 
```

把这两行注释取消，打开配置：

```bash
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

让改动生效：

```bash
sudo sysctl --load /etc/sysctl.conf
```

## 参考资料

- [Network Bridge Configuration on Debian 12 Linux](https://orcacore.com/network-bridge-configuration-debian-12-linux/)
- [Debian Linux: Configure Network Interfaces As A Bridge / Network Switch](https://www.cyberciti.biz/faq/debian-network-interfaces-bridge-eth0-eth1-eth2/)
- [Bridging Network Connections](https://wiki.debian.org/BridgeNetworkConnections)
- [Create Linux Bridge on VLAN Interface in Debian 12/11/10](https://techviewleo.com/create-linux-bridge-on-vlan-interface-in-debian-ubuntu/)
