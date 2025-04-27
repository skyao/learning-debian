---
title: "openvswitch软交换"
linkTitle: "openvswitch"
date: 2024-01-18
weight: 30
description: >
  在 debian 12 下搭建 openvswitch  软交换，并利用cx5网卡的片上卸载功能降低cpu占用
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


## open vswitch


```bash
sudo apt install openvswitch-switch
```



查看Open vSwitch（OVS）的版本：

```bash
$ sudo ovs-vsctl show

2a434d84-f9ec-4da7-af56-a762b068a2e8
    ovs_version: "3.1.0"
```

和 openvswitch 模块的信息：

```bash
$ sudo modinfo openvswitch 

filename:       /lib/modules/6.1.0-25-amd64/kernel/net/openvswitch/openvswitch.ko
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
vermagic:       6.1.0-25-amd64 SMP preempt mod_unload modversions 
sig_id:         PKCS#7
signer:         Debian Secure Boot CA
sig_key:        32:A0:28:7F:84:1A:03:6F:A3:93:C1:E0:65:C4:3A:E6:B2:42:26:43
sig_hashalgo:   sha256
signature:      52:AF:27:99:1F:1F:EF:6C:2E:7D:16:DB:D9:B9:8E:46:99:13:AA:37:
		53:5F:19:45:38:8B:C6:7C:B6:D7:93:A6:5C:B3:59:C7:F4:70:A5:D8:
		94:9C:00:4F:17:06:8E:63:BF:69:F1:01:76:85:BB:FC:C7:6B:F4:E3:
		C2:BD:01:7F:9A:98:04:76:23:C5:05:58:10:77:21:EC:95:4B:98:DD:
		51:DB:58:32:A9:49:19:7B:47:D7:60:FF:D8:BB:81:90:C7:54:14:42:
		80:D6:1B:D4:64:E8:DB:EC:A2:3F:49:FD:7E:34:05:78:50:98:A5:BF:
		33:47:59:C0:F8:76:7F:F6:26:3E:F0:6F:E5:72:7C:51:CA:31:B7:EE:
		B7:C2:03:30:77:DA:B5:13:03:F5:EF:26:CB:32:F8:DA:3F:A0:60:E5:
		8F:39:D8:8F:BF:6E:25:15:EA:AB:48:A5:02:56:15:5A:08:BB:34:73:
		52:C4:DA:E5:9F:89:BF:7B:6E:6E:B9:D8:9B:6B:46:BA:C9:29:06:0B:
		2A:97:4F:D4:92:61:88:18:5B:AF:C1:FE:AA:6D:3D:77:18:96:0C:DF:
		EB:20:DE:4F:D2:49:72:B5:F9:02:72:F2:0D:44:3D:6C:DF:C5:D8:3B:
		35:A7:8C:08:11:76:B5:F9:34:1B:F7:80:A6:3E:A6:0E
```

检查 openvswitch 模块的正确加载：

```bash
lsmod | grep openvswitch
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
sudo systemctl status openvswitch-switch
 openvswitch-switch.service - Open vSwitch
     Loaded: loaded (/lib/systemd/system/openvswitch-switch.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2024-09-10 04:27:46 EDT; 3min 59s ago
    Process: 1025 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 1025 (code=exited, status=0/SUCCESS)
        CPU: 420us

Sep 10 04:27:46 dev100 systemd[1]: Starting openvswitch-switch.service - Open vSwitch...
Sep 10 04:27:46 dev100 systemd[1]: Finished openvswitch-switch.service - Open vSwitch.
```



```bash
sudo systemctl status ovsdb-server
 ovsdb-server.service - Open vSwitch Database Unit
     Loaded: loaded (/lib/systemd/system/ovsdb-server.service; static)
     Active: active (running) since Tue 2024-09-10 04:27:46 EDT; 7min ago
    Process: 738 ExecStart=/usr/share/openvswitch/scripts/ovs-ctl --no-ovs-vswitchd --no-monitor --system-id=random --no-record-hostname start $O>
   Main PID: 816 (ovsdb-server)
      Tasks: 1 (limit: 38413)
     Memory: 15.0M
        CPU: 41ms
     CGroup: /system.slice/ovsdb-server.service
             └─816 ovsdb-server /etc/openvswitch/conf.db -vconsole:emer -vsyslog:err -vfile:info --remote=punix:/var/run/openvswitch/db.sock --pr>

Sep 10 04:27:46 dev100 systemd[1]: Starting ovsdb-server.service - Open vSwitch Database Unit...
Sep 10 04:27:46 dev100 ovs-ctl[738]: Starting ovsdb-server.
Sep 10 04:27:46 dev100 ovs-vsctl[817]: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait -- init -- set Open_vSwitch . db-version=8.3.1
Sep 10 04:27:46 dev100 ovs-vsctl[831]: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait set Open_vSwitch . ovs-version=3.1.0 "external-ids:syst>
Sep 10 04:27:46 dev100 ovs-ctl[738]: Configuring Open vSwitch system IDs.
Sep 10 04:27:46 dev100 ovs-ctl[738]: Enabling remote OVSDB managers.
Sep 10 04:27:46 dev100 systemd[1]: Started ovsdb-server.service - Open vSwitch Database Unit.
```

```bash
sudo systemctl status ovs-vswitchd
● ovs-vswitchd.service - Open vSwitch Forwarding Unit
     Loaded: loaded (/lib/systemd/system/ovs-vswitchd.service; static)
     Active: active (running) since Tue 2024-09-10 04:27:46 EDT; 8min ago
    Process: 835 ExecStart=/usr/share/openvswitch/scripts/ovs-ctl --no-ovsdb-server --no-monitor --system-id=random --no-record-hostname start $O>
   Main PID: 1012 (ovs-vswitchd)
      Tasks: 1 (limit: 38413)
     Memory: 5.6M
        CPU: 29ms
     CGroup: /system.slice/ovs-vswitchd.service
             └─1012 ovs-vswitchd unix:/var/run/openvswitch/db.sock -vconsole:emer -vsyslog:err -vfile:info --mlockall --no-chdir --log-file=/var/>

Sep 10 04:27:46 dev100 systemd[1]: Starting ovs-vswitchd.service - Open vSwitch Forwarding Unit...
Sep 10 04:27:46 dev100 ovs-ctl[883]: Inserting openvswitch module.
Sep 10 04:27:46 dev100 ovs-ctl[835]: Starting ovs-vswitchd.
Sep 10 04:27:46 dev100 ovs-ctl[835]: Enabling remote OVSDB managers.
Sep 10 04:27:46 dev100 systemd[1]: Started ovs-vswitchd.service - Open vSwitch Forwarding Unit.
```


```bash
sudo ovs-vsctl add-br br0 
sudo ovs-vsctl show    

sudo ovs-vsctl list br br0 

sudo ovs-vsctl add-port br0 enp6s18
sudo ovs-vsctl add-port br0 enp1s0f0np0
sudo ovs-vsctl add-port br0 enp1s0f1np1
sudo ovs-vsctl show


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