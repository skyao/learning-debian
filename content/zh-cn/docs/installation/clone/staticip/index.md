---
title: "静态IP地址"
linkTitle: "静态IP地址"
date: 2024-01-18
weight: 30
description: >
  克隆后设置静态IP地址
---

每个克隆后的虚拟机都应该有自己固定的ip地址，但是通过 dhcp 来修改不合适，毕竟这些经常变化。

因此需要设置静态 ip 地址，每次克隆之后虚拟机就自行设置。

### 设置静态ip

修改前先备份：

```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.bak
```

看一下目前的网络适配器情况，enp6s18 是正在使用的网卡：

```bash
$ ip addr
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:97:c4:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.228/24 brd 192.168.20.255 scope global dynamic enp6s18
```

修改网络设置：

```bash
sudo vi /etc/network/interfaces
```

将 enp6s18 原有的配置：

```bash
# The primary network interface
allow-hotplug enp6s18
iface enp6s18 inet dhcp
```

修改为:

```bash
# The primary network interface
allow-hotplug enp6s18
# iface enp6s18 inet dhcp
iface enp6s18 inet static
address 192.168.20.25
netmask 255.255.255.0
gateway 192.168.20.1
dns-nameservers 192.168.20.1
```

保存后重启网络：

```bash
sudo systemctl restart networking
```

之后就可以用新的 ip 地址访问了。

### 参考资料

- https://itslinuxfoss.com/set-up-static-ip-address-debian-12-linux/