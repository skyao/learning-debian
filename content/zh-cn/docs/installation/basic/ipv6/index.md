---
title: "禁用 ipv6"
linkTitle: "禁用 ipv6"
date: 2025-10-26
weight: 120
description: >
  禁止使用 ipv6 网络
---

暂时不使用 ipv6 网络，先禁用 ipv6 网络。

禁用前，查看网络情况时能看到 ipv6 地址：

```bash
$ ip addr

2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:08:b8:99 brd ff:ff:ff:ff:ff:ff
    altname enp6s18
    altname enxbc241107b899
    inet 192.168.3.232/24 brd 192.168.3.255 scope global dynamic noprefixroute ens18
       valid_lft 43161sec preferred_lft 37761sec
    inet6 fe80::bb04:7452:5f65:df82/64 scope link
       valid_lft forever preferred_lft forever
```

修改 grub 配置文件：

```bash
sudo vi /etc/default/grub
```

在 `GRUB_CMDLINE_LINUX_DEFAULT` 中加入 `ipv6.disable=1`，如：

```properties
GRUB_CMDLINE_LINUX_DEFAULT="quiet ipv6.disable=1"
```

更新 grub：

```bash
sudo update-grub
```

重启即可。

禁用后，查看网络情况时看不到 ipv6 地址：

```bash
$ ip addr

2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:08:b8:99 brd ff:ff:ff:ff:ff:ff
    altname enp6s18
    altname enxbc241107b899
    inet 192.168.3.232/24 brd 192.168.3.255 scope global dynamic noprefixroute ens18
       valid_lft 42938sec preferred_lft 37538sec
```

