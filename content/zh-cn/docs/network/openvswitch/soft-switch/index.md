---
title: "open vSwitch 简单软交换"
linkTitle: "软交换"
date: 2025-04-29
weight: 30
description: >
  在 debian 12 下简单搭建 open vswitch 软交换
---

## 背景

前面用 linux bridge 做软交换，功能没问题，性能看似还行，但其实有不少问题：

- 跑不到网卡极限
- cpu占用较高
- rdma 之类的高级特性缺失

### cx5 介绍

https://www.nvidia.com/en-us/networking/ethernet/connectx-5/

> NVIDIA® Mellanox® ConnectX®-5适配器提供先进的硬件卸载功能，可减少CPU资源消耗，并实现极高的数据包速率和吞吐量。这提高了数据中心基础设施的效率，并为Web 2.0、云、数据分析和存储平台提供了最高性能和最灵活的解决方案。

这里提到的 "先进的硬件卸载功能，可减少CPU资源消耗"，正是我需要的。

实现最高效率的主要特性：

- NVIDIA RoCE技术封装了以太网上的数据包传输，并降低了CPU负载，从而为网络和存储密集型应用提供高带宽和低延迟的网络基础设施。

- 突破性的 NVIDIA ASAP² 技术通过将 Open vSwitch 数据路径从主机CPU卸载到适配器，提供创新的 SR-IOV 和 VirtIO 加速，从而实现极高的性能和可扩展性。

- ConnectX 网卡利用 SR-IOV 来分离对虚拟化环境中物理资源和功能的访问。这减少了I/O开销，并允许网卡保持接近非虚拟化的性能。

关键特性：

- 每个端口最高可达100 Gb/s以太网
- 可靠传输上的自适应路由
- NVMe over Fabric（NVMf）目标卸载
- **增强的vSwitch / vRouter卸载**
- NVGRE和VXLAN封装流量的硬件卸载
- 端到端QoS和拥塞控制

cx5 网卡资料：

https://network.nvidia.com/files/doc-2020/pb-connectx-5-en-card.pdf

### 硬件加速的基本知识

NVIDIA DPU白皮书：SR-IOV vs. VirtIO加速性能对比

https://aijishu.com/a/1060000000228117




- https://docs.nvidia.com/doca/archive/doca-v2.2.0/switching-support/index.html

- https://www.cnblogs.com/dream397/p/14432472.html


## 搭建 open vswitch 软交换

### 安装 open vswitch


```bash
sudo apt install openvswitch-switch
```



查看Open vSwitch（OVS）的版本：

```bash
$ sudo ovs-vsctl show

cb860a93-1368-4d4f-be38-5cb9bbe5273d
    ovs_version: "3.1.0"
```

和 openvswitch 模块的信息：

```bash
$ sudo modinfo openvswitch 

filename:       /lib/modules/6.1.0-33-amd64/kernel/net/openvswitch/openvswitch.ko
alias:          net-pf-16-proto-16-family-ovs_ct_limit
alias:          net-pf-16-proto-16-family-ovs_meter
alias:          net-pf-16-proto-16-family-ovs_packet
alias:          net-pf-16-proto-16-family-ovs_flow
alias:          net-pf-16-proto-16-family-ovs_vport
alias:          net-pf-16-proto-16-family-ovs_datapath
license:        GPL
description:    Open vSwitch switching datapath
depends:        nf_conntrack,nsh,nf_nat,nf_defrag_ipv6,nf_conncount,libcrc32c
retpoline:      Y
intree:         Y
name:           openvswitch
vermagic:       6.1.0-33-amd64 SMP preempt mod_unload modversions 
sig_id:         PKCS#7
signer:         Debian Secure Boot CA
sig_key:        32:A0:28:7F:84:1A:03:6F:A3:93:C1:E0:65:C4:3A:E6:B2:42:26:43
sig_hashalgo:   sha256
signature:      7E:3C:EA:A0:18:FE:81:6D:2C:A8:08:8A:1D:BD:D5:13:F1:5D:FE:C4:
		06:2C:3B:4B:B2:4A:6D:1E:30:AD:65:CC:DB:87:73:F4:D7:D5:30:76:
		D6:FF:E2:77:28:0A:AA:17:92:C4:C5:DF:EC:E8:E5:95:88:B4:62:36:
		AF:BF:58:96:D0:C1:ED:A3:6D:23:18:DD:A0:CF:A6:2C:6E:71:B2:83:
		AD:45:F0:59:8D:FB:6D:8C:FB:9D:80:4D:0A:16:0A:9B:CE:A3:61:60:
		BC:85:9D:EE:70:4D:5A:62:6E:E3:33:C1:58:2B:C4:CE:36:27:C9:A5:
		BB:6C:7D:F3:B5:74:C8:FA:C3:5F:E5:1B:28:46:55:7E:26:0E:2A:7A:
		54:4B:DD:74:E8:EA:40:43:2B:62:F6:DC:13:A6:A3:C6:EA:BF:1B:41:
		2B:0A:92:01:2D:57:02:EA:0A:24:C9:75:EB:F3:34:41:35:D7:31:67:
		65:96:9B:3B:65:47:1B:2E:60:97:E9:C3:40:10:9F:C6:91:EB:C4:DB:
		0C:D5:5D:9C:99:ED:DF:3C:CA:B3:DB:61:44:A9:A0:C5:1D:1D:C8:CF:
		01:39:D6:F3:FE:81:2D:43:2C:DE:F7:A1:06:E5:EE:79:31:DC:41:83:
		59:BC:30:42:BB:68:C8:27:AF:AE:69:30:51:2E:02:6A
```

检查 openvswitch 模块的正确加载：

```bash
$ lsmod | grep openvswitch

openvswitch           192512  0
nsh                    16384  1 openvswitch
nf_conncount           24576  1 openvswitch
nf_nat                 57344  1 openvswitch
nf_conntrack          188416  3 nf_nat,openvswitch,nf_conncount
nf_defrag_ipv6         24576  2 nf_conntrack,openvswitch
libcrc32c              16384  3 nf_conntrack,nf_nat,openvswitch
```

查看 openvswitch-switch 服务的运行情况：

```bash
$ sudo systemctl status openvswitch-switch

● openvswitch-switch.service - Open vSwitch
     Loaded: loaded (/lib/systemd/system/openvswitch-switch.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2025-04-29 14:39:38 CST; 1min 54s ago
    Process: 1816 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 1816 (code=exited, status=0/SUCCESS)
        CPU: 340us

Apr 29 14:39:38 debian12 systemd[1]: Starting openvswitch-switch.service - Open vSwitch...
Apr 29 14:39:38 debian12 systemd[1]: Finished openvswitch-switch.service - Open vSwitch.
```

查看 ovsdb-server 服务：

```bash
$ sudo systemctl status ovsdb-server

● ovsdb-server.service - Open vSwitch Database Unit
     Loaded: loaded (/lib/systemd/system/ovsdb-server.service; static)
     Active: active (running) since Tue 2025-04-29 14:39:38 CST; 2min 28s ago
    Process: 1718 ExecStart=/usr/share/openvswitch/scripts/ovs-ctl --no-ovs-vswitchd --no-monito>
   Main PID: 1761 (ovsdb-server)
      Tasks: 1 (limit: 19089)
     Memory: 2.2M
        CPU: 38ms
     CGroup: /system.slice/ovsdb-server.service
             └─1761 ovsdb-server /etc/openvswitch/conf.db -vconsole:emer -vsyslog:err -vfile:inf>

Apr 29 14:39:38 debian12 systemd[1]: Starting ovsdb-server.service - Open vSwitch Database Unit.>
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Backing up database to /etc/openvswitch/conf.db.backup8.>
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Compacting database.
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Converting database schema.
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Starting ovsdb-server.
Apr 29 14:39:38 debian12 ovs-vsctl[1762]: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait -- >
Apr 29 14:39:38 debian12 ovs-vsctl[1767]: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait set>
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Configuring Open vSwitch system IDs.
Apr 29 14:39:38 debian12 ovs-ctl[1718]: Enabling remote OVSDB managers.
Apr 29 14:39:38 debian12 systemd[1]: Started ovsdb-server.service - Open vSwitch Database Unit.
```

查看 ovs-vswitchd 服务：

```bash
$ sudo systemctl status ovs-vswitchd

● ovs-vswitchd.service - Open vSwitch Forwarding Unit
     Loaded: loaded (/lib/systemd/system/ovs-vswitchd.service; static)
     Active: active (running) since Tue 2025-04-29 14:39:38 CST; 3min 28s ago
    Process: 1771 ExecStart=/usr/share/openvswitch/scripts/ovs-ctl --no-ovsdb-server --no-monito>
   Main PID: 1810 (ovs-vswitchd)
      Tasks: 1 (limit: 19089)
     Memory: 2.1M
        CPU: 17ms
     CGroup: /system.slice/ovs-vswitchd.service
             └─1810 ovs-vswitchd unix:/var/run/openvswitch/db.sock -vconsole:emer -vsyslog:err ->

Apr 29 14:39:38 debian12 systemd[1]: Starting ovs-vswitchd.service - Open vSwitch Forwarding Uni>
Apr 29 14:39:38 debian12 ovs-ctl[1771]: Starting ovs-vswitchd.
Apr 29 14:39:38 debian12 ovs-ctl[1771]: Enabling remote OVSDB managers.
Apr 29 14:39:38 debian12 systemd[1]: Started ovs-vswitchd.service - Open vSwitch Forwarding Unit.
```

### 创建网桥

创建网桥前，先查看当前的网络接口： 

```bash
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:1c:d5:48 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.227/24 brd 192.168.3.255 scope global dynamic enp6s18
       valid_lft 43156sec preferred_lft 43156sec
    inet6 fdfb:ddbe:c71b:0:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft forever preferred_lft forever
    inet6 240e:3a1:5055:c180:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft 6989sec preferred_lft 2523sec
    inet6 fe80::be24:11ff:fe1c:d548/64 scope link 
       valid_lft forever preferred_lft forever
3: enp1s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ec brd ff:ff:ff:ff:ff:ff
4: enp1s0f1np1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ed brd ff:ff:ff:ff:ff:ff
```

当前有 3 个网口，分别是 enp6s18, enp1s0f0np0, enp1s0f1np1。enp6s18 是 cx4 25g 网卡，接交换机，enp1s0f0np0 和 enp1s0f1np1 是 cx5 100g 双头网卡，准备接另外两台机器。

创建 ovs 网桥：

```bash
sudo ovs-vsctl add-br br0 
```

此时执行 `ip addr` 可以看到 ovs-system 和 br0：

```bash
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:1c:d5:48 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.227/24 brd 192.168.3.255 scope global dynamic enp6s18
       valid_lft 43027sec preferred_lft 43027sec
    inet6 fdfb:ddbe:c71b:0:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft forever preferred_lft forever
    inet6 240e:3a1:5055:c180:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft 6860sec preferred_lft 2394sec
    inet6 fe80::be24:11ff:fe1c:d548/64 scope link 
       valid_lft forever preferred_lft forever
3: enp1s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ec brd ff:ff:ff:ff:ff:ff
4: enp1s0f1np1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ed brd ff:ff:ff:ff:ff:ff
5: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 46:39:3e:eb:b0:0b brd ff:ff:ff:ff:ff:ff
6: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 8a:1f:49:5c:10:4d brd ff:ff:ff:ff:ff:ff
```

注意： 此时网桥的 mac 地址 8a:1f:49:5c:10:4d 和其他三个网卡都不一样。

目前网桥下还没有配置任何端口：

```bash
sudo ovs-vsctl show    

cb860a93-1368-4d4f-be38-5cb9bbe5273d
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "3.1.0"
```

查看 br0 的详细信息：

```bash
sudo ovs-vsctl list br br0 

_uuid               : a345132c-00d9-4196-bb2c-2489c3669b47
auto_attach         : []
controller          : []
datapath_id         : "00002e1345a39641"
datapath_type       : ""
datapath_version    : "<unknown>"
external_ids        : {}
fail_mode           : []
flood_vlans         : []
flow_tables         : {}
ipfix               : []
mcast_snooping_enable: false
mirrors             : []
name                : br0
netflow             : []
other_config        : {}
ports               : [1e053d35-c79f-4b5b-88cc-f5cba9519ca0]
protocols           : []
rstp_enable         : false
rstp_status         : {}
sflow               : []
status              : {}
stp_enable          : false
```

先修改 /etc/network/interfaces 文件，安全起见备份一份：

```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.bak
sudo vi /etc/network/interfaces
```

内容为：

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
#allow-hotplug enp6s18
#iface enp6s18 inet dhcp

# Open vSwitch Configuration
# active br0 interface
auto br0
allow-ovs br0

# config IP for interface br0
iface br0 inet static
address 192.168.3.227
netmask 255.255.255.0
gateway 192.168.3.1
ovs_type OVSBridge
ovs_ports enp6s18

# attach enp6s18 to bridge br0
allow-br0 enp6s18
iface enp6s18 inet manual
ovs_bridge br0
ovs_type OVSPort
```

然后给网桥中添加端口，先加 enp6s18：

```bash
sudo ovs-vsctl add-port br0 enp6s18
```

执行完成后网络就不能用了。只能在 pve 控制台继续控制。

加入 enp6s18 后，网桥的 mac 地址变为 enp6s18 的 mac 地址：

```bash
sudo ovs-vsctl show

cb860a93-1368-4d4f-be38-5cb9bbe5273d
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port enp6s18
            Interface enp6s18
    ovs_version: "3.1.0"
```

重启网络：

```bash
sudo systemctl restart networking
```

此时虚拟机的网络应该恢复了，可以重新 ssh 连接。查看此时网口信息：

```bash
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master ovs-system state UP group default qlen 1000
    link/ether bc:24:11:1c:d5:48 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::be24:11ff:fe1c:d548/64 scope link 
       valid_lft forever preferred_lft forever
3: enp1s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ec brd ff:ff:ff:ff:ff:ff
4: enp1s0f1np1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 1c:34:da:5a:1f:ed brd ff:ff:ff:ff:ff:ff
9: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 46:39:3e:eb:b0:0b brd ff:ff:ff:ff:ff:ff
10: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether bc:24:11:1c:d5:48 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.227/24 brd 192.168.3.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fdfb:ddbe:c71b:0:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft forever preferred_lft forever
    inet6 240e:3a1:5055:c180:be24:11ff:fe1c:d548/64 scope global dynamic mngtmpaddr 
       valid_lft 6477sec preferred_lft 2874sec
    inet6 fe80::be24:11ff:fe1c:d548/64 scope link 
       valid_lft forever preferred_lft forever
```

注意： 此时 br0 的 mac 地址变为 enp6s18 的 mac 地址 bc:24:11:1c:d5:48。

检查一下是否可以访问外网：

```bash
ping 192.168.3.1
ping www.baidu.com
```

如果可以访问外网，则说明网络配置成功, openvswitch 网桥建立并可以工作。

### 往网桥中加入其他网口

目前网桥中只有 enp6s18 一个网口，只能用于访问外部访问，需要继续加入其他网口。


```bash
......

# config IP for interface br0
iface br0 inet static
address 192.168.3.227
netmask 255.255.255.0
gateway 192.168.3.1
ovs_type OVSBridge
ovs_ports enp6s18 enp1s0f0np0 enp1s0f1np1

# attach enp6s18 to bridge br0
allow-br0 enp6s18
iface enp6s18 inet manual
ovs_bridge br0
ovs_type OVSPort

# attach enp1s0f0np0 to bridge br0
allow-br0 enp1s0f0np0
iface enp1s0f0np0 inet manual
ovs_bridge br0
ovs_type OVSPort

# attach enp1s0f1np1 to bridge br0
allow-br0 enp1s0f1np1
iface enp1s0f1np1 inet manual
ovs_bridge br0
ovs_type OVSPort
```

重启网络

```bash
sudo systemctl restart networking
```

之后查看网桥信息：

```bash
sudo ovs-vsctl show

cb860a93-1368-4d4f-be38-5cb9bbe5273d
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port enp1s0f0np0
            Interface enp1s0f0np0
        Port enp6s18
            Interface enp6s18
        Port enp1s0f1np1
            Interface enp1s0f1np1
    ovs_version: "3.1.0"
```

此时网桥中已经有三个网口，可以用来连接其他机器扮演交换机的角色了。


在另外一台机器的虚拟机中，直通 cx5 网卡，网线直连接到前面这台机器的 cx5 双头网卡上。注意去掉虚拟机的 vmbr 网卡，只使用 cx5 网卡。

修改网络配置文件：

```bash
sudo vi /etc/network/interfaces
```

添加如下内容（dhcp或者静态地址二选一）：

```bash
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp1s0np0
iface enp1s0np0 inet dhcp
#iface enp1s0np0 inet static
#address 192.168.3.130
#netmask 255.255.255.0
#gateway 192.168.3.1
#dns-nameservers 192.168.3.1
```

重启网络：

```bash
sudo systemctl restart networking
```

检查此时是否可以正常连接网络。

### 简单性能测试

用 iperf 简单测试一下性能：

```bash
# 在服务器端运行
iperf -s

# 在客户端运行
iperf -c 192.168.3.227 -i 1 -t 20 -P 4
```

测试结果：

```bash
[SUM] 11.0000-11.7642 sec  8.37 GBytes  94.1 Gbits/sec
[SUM] 0.0000-11.7642 sec   129 GBytes  94.1 Gbits/sec
```

可以看到，服务器端和客户端的带宽都达到了 94.1 Gbits/sec，接近 100G 的理论极限。

## 设置 MTU

目前网桥的 MTU 是 1500，可以设置为 9000：

```bash
sudo ip link set br0 mtu 9000
sudo ip link set enp1s0f0np0 mtu 9000
sudo ip link set enp1s0f1np1 mtu 9000
sudo ip link set enp6s18 mtu 9000
```

注意： 三块网卡都要设置，如果只设置 enp1s0f0np0 和 enp1s0f1np1， 没有设置 enp6s18， 速度不会有提升。

客户端也要设置 MTU：

```bash
sudo ip link set enp1s0np0 mtu 9000
```

重新测试，这次 iperf 测试出来的速度从 94.1 Gbits/sec 提升到了 98.8 Gbits/sec， 提升还是比较明显的。

```bash
[SUM] 0.0000-20.0004 sec   230 GBytes  98.8 Gbits/sec
```

当这个方式重启之后就 MTU 又回复成 1500 了。

修改网络配置文件：

```bash
sudo vi /etc/network/interfaces
```

服务器端修改，将 3 个网卡都设置为 mtu 9000 即可：

```bash
# attach enp6s18 to bridge br0
allow-br0 enp6s18
iface enp6s18 inet manual
ovs_bridge br0
ovs_type OVSPort
mtu 9000

# attach enp1s0f0np0 to bridge br0
allow-br0 enp1s0f0np0
iface enp1s0f0np0 inet manual
ovs_bridge br0
ovs_type OVSPort
mtu 9000

# attach enp1s0f1np1 to bridge br0
allow-br0 enp1s0f1np1
iface enp1s0f1np1 inet manual
ovs_bridge br0
ovs_type OVSPort
mtu 9000
```

客户端修改：

```bash
allow-hotplug enp1s0np0
iface enp1s0np0 inet static
address 192.168.3.205
netmask 255.255.255.0
gateway 192.168.3.1
dns-nameservers 192.168.3.1
mtu 9000
```

特别注意： 测试中发现，如果用 static 静态地址， 这样设置 MTU 9000 可以生效。但是如果用 dhcp 动态地址， 这样设置 MTU 9000 不会生效，会继续使用 1500。估计是 dhcp 动态地址获取的 MTU 是 1500。

重启网络：

```bash
sudo systemctl restart networking
sudo reboot
```

## 开启硬件卸载


```bash
sudo ovs-vsctl set Open_vSwitch . other_config:hw-offload=true
sudo systemctl restart networking
sudo systemctl restart openvswitch-switch
```


https://infoloup.no-ip.org/openvswitch-debian12-creation/


➜  ~ sudo ip link set br0 mtu 9000
➜  ~ sudo ip link set enp1s0f1np1 mtu 9000
➜  ~ sudo ip link set enp1s0f0np0 mtu 9000
➜  ~ sudo ip link set enp6s18 mtu 9000



https://docs.openstack.org/neutron/latest/admin/config-ovs-offload.html

https://www.openvswitch.org/support/ovscon2019/day2/0951-hw_offload_ovs_con_19-Oz-Mellanox.pdf

https://docs.nvidia.com/doca/archive/doca-v2.2.0/switching-support/index.html