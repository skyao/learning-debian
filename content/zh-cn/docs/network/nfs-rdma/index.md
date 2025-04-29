---
title: "nfs rdma 支持"
linkTitle: "nfs rdma"
date: 2024-09-16
weight: 50
description: >
  在 debian12 下为 nfs 添加 rdma 支持
---

## 准备工作

### 前置条件

- 已经配置好 nfs 服务器和客户端：参考上一章
- 已经配置好硬盘：参考上一章

### 安装 MLNX_OFED 驱动

cx-5 100g网卡参考： https://skyao.io/learning-computer-hardware/nic/cx5-huawei-sp350/driver/debian12/

nfs 服务器和客户端都要安装网卡驱动。

### 网卡直连

为了避免 linux bridge 或者 ovs 的桥接，直接使用dac线连接100G网卡的网口，以直连的方式连接两台服务器（后续再尝试openvswitch）。

服务器端设置网卡地址：

```bash
sudo vi /etc/network/interfaces
```

增加内容：

```bash
allow-hotplug enp1s0f1np1
# iface enp1s0f1np1 inet dhcp
iface enp1s0f1np1 inet static
address 192.168.119.1
netmask 255.255.255.0
gateway 192.168.3.1
dns-nameservers 192.168.3.1
```

重启网卡：

```bash
sudo systemctl restart networking
```

客户端设置网卡地址：

```bash
sudo vi /etc/network/interfaces
```

增加内容：

```bash
allow-hotplug enp1s0np0
# iface enp1s0np0 inet dhcp
iface enp1s0np0 inet static
address 192.168.119.2
netmask 255.255.255.0
gateway 192.168.119.1
dns-nameservers 192.168.119.1
```

重启网卡：

```bash
sudo systemctl restart networking
```

验证两台机器可以互联：

```bash
ping 192.168.119.1
ping 192.168.119.2
```

验证两台机器之间的网速，服务器端自动 iperf 服务器端：

```bash
iperf -s 192.168.119.1
```

客户端 iperf 测试：

```bash
iperf -c 192.168.119.1 -i 1 -t 20 -P 4
```

测试出来的速度大概是 94.1 Gbits/sec，比较接近100G网卡的理论最大速度了：

```bash
[  4] 0.0000-20.0113 sec  39.2 GBytes  16.8 Gbits/sec
[  2] 0.0000-20.0111 sec  70.1 GBytes  30.1 Gbits/sec
[  3] 0.0000-20.0113 sec  70.8 GBytes  30.4 Gbits/sec
[  1] 0.0000-20.0111 sec  39.2 GBytes  16.8 Gbits/sec
[SUM] 0.0000-20.0007 sec   219 GBytes  94.1 Gbits/sec
```

此时 nfs 服务器端和客户端之间的网络连接已经没有问题了，可以开始配置 nfs 了。

由于启用了两块网卡，有时默认网络路由会出问题，导致无法访问外网。此时需要检查网络路由：

```bash
$ ip route

default via 192.168.3.1 dev enp1s0f1np1 onlink 
192.168.3.0/24 dev enp6s18 proto kernel scope link src 192.168.3.227 
192.168.119.0/24 dev enp1s0f1np1 proto kernel scope link src 192.168.119.1
```

可以看到默认路由是 192.168.3.1， 但使用的网卡是 enp1s0f1np1， 而不是 enp6s18。

此时需要手动设置默认路由：

```bash
sudo ip route del default
sudo ip route add default dev enp6s18
```

## 安装 nfsrdma

```bash
sudo cp ./mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb /tmp/
cd /tmp
sudo apt-get install ./mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb
```

输出为：

```bash
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'mlnx-nfsrdma-dkms' instead of './mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb'
The following NEW packages will be installed:
  mlnx-nfsrdma-dkms
0 upgraded, 1 newly installed, 0 to remove and 2 not upgraded.
Need to get 0 B/71.2 kB of archives.
After this operation, 395 kB of additional disk space will be used.
Get:1 /home/sky/temp/mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb mlnx-nfsrdma-dkms all 24.10.OFED.24.10.2.1.8.1-1 [71.2 kB]
Selecting previously unselected package mlnx-nfsrdma-dkms.
(Reading database ... 74820 files and directories currently installed.)
Preparing to unpack .../mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb ...
Unpacking mlnx-nfsrdma-dkms (24.10.OFED.24.10.2.1.8.1-1) ...
Setting up mlnx-nfsrdma-dkms (24.10.OFED.24.10.2.1.8.1-1) ...
Loading new mlnx-nfsrdma-24.10.OFED.24.10.2.1.8.1 DKMS files...
First Installation: checking all kernels...
Building only for 6.1.0-31-amd64
Building for architecture x86_64
Building initial module for 6.1.0-31-amd64
Done.
Forcing installation of mlnx-nfsrdma

rpcrdma.ko:
Running module version sanity check.
 - Original module
 - Installation
   - Installing to /lib/modules/6.1.0-31-amd64/updates/dkms/

svcrdma.ko:
Running module version sanity check.
 - Original module
 - Installation
   - Installing to /lib/modules/6.1.0-31-amd64/updates/dkms/

xprtrdma.ko:
Running module version sanity check.
 - Original module
 - Installation
   - Installing to /lib/modules/6.1.0-31-amd64/updates/dkms/
depmod...
```

如果安装时最后有如下报错：

```bash
N: Download is performed unsandboxed as root as file '/home/sky/temp/mlnx-nfsrdma-dkms_24.10.OFED.24.10.2.1.8.1-1_all.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```

最后的 Permission denied 提示是因为 _apt 用户无法直接访问下载的 .deb 文件（比如位于 /home/sky/temp/），需要先复制到 /tmp/ 目录下。

重启。

### 检查 DKMS 状态

确认 DKMS 模块已注册：

```bash
sudo dkms status | grep nfsrdma
```

输出：

```bash
mlnx-nfsrdma/24.10.OFED.24.10.2.1.8.1, 6.1.0-31-amd64, x86_64: installed
```


## 配置 nfs server（rdma）

为了确保安全，尤其是升级内核之后，先安装内核头文件并重新编译 nfsrdma 模块： 

```bash
sudo apt install linux-headers-$(uname -r)
sudo dkms autoinstall
```

### 检查 RDMA 相关模块是否已加载

运行以下命令检查模块是否已加载：

```bash
lsmod | grep -E "rpcrdma|svcrdma|xprtrdma|ib_core|mlx5_ib"
```

预计输出：

```bash
xprtrdma               16384  0
svcrdma                16384  0
rpcrdma                94208  0
rdma_cm               139264  2 rpcrdma,rdma_ucm
mlx5_ib               495616  0
ib_uverbs             188416  2 rdma_ucm,mlx5_ib
ib_core               462848  9 rdma_cm,ib_ipoib,rpcrdma,iw_cm,ib_umad,rdma_ucm,ib_uverbs,mlx5_ib,ib_cm
sunrpc                692224  18 nfsd,rpcrdma,auth_rpcgss,lockd,nfs_acl
mlx5_core            2441216  1 mlx5_ib
mlx_compat             20480  15 rdma_cm,ib_ipoib,mlxdevm,rpcrdma,mlxfw,xprtrdma,iw_cm,svcrdma,ib_umad,ib_core,rdma_ucm,ib_uverbs,mlx5_ib,ib_cm,mlx5_core
```

~~如果没有，手动加载模块~~（实际操作下来发现不手工加载也可以，系统会自动加载）：

```bash
sudo modprobe rpcrdma
sudo modprobe svcrdma
sudo modprobe xprtrdma
sudo modprobe ib_core
sudo modprobe mlx5_ib
```

确认 DKMS 模块已注册：

```bash
sudo dkms status | grep nfsrdma
```

输出：

```bash
mlnx-nfsrdma/24.10.OFED.24.10.2.1.8.1, 6.1.0-31-amd64, x86_64: installed
```

检查内核日志：

```bash   
sudo dmesg | grep rdma
[  178.512334] RPC: Registered rdma transport module.
[  178.512336] RPC: Registered rdma backchannel transport module.
[  178.515613] svcrdma: svcrdma is obsoleted, loading rpcrdma instead
[  178.552178] xprtrdma: xprtrdma is obsoleted, loading rpcrdma instead
```

### 测试 RDMA 连接

在服务器端执行：

```bash
ibstatus
```

我这里开启了一个网卡 mlx5_1：

```bash
Infiniband device 'mlx5_0' port 1 status:
	default gid:	 fe80:0000:0000:0000:1e34:daff:fe5a:1fec
	base lid:	 0x0
	sm lid:		 0x0
	state:		 1: DOWN
	phys state:	 3: Disabled
	rate:		 100 Gb/sec (4X EDR)
	link_layer:	 Ethernet

Infiniband device 'mlx5_1' port 1 status:
	default gid:	 fe80:0000:0000:0000:1e34:daff:fe5a:1fed
	base lid:	 0x0
	sm lid:		 0x0
	state:		 4: ACTIVE
	phys state:	 5: LinkUp
	rate:		 100 Gb/sec (4X EDR)
	link_layer:	 Ethernet
```

或者执行 ibdev2netdev 显示：

```bash
$ ibdev2netdev

mlx5_0 port 1 ==> enp1s0f0np0 (Down)
mlx5_1 port 1 ==> enp1s0f1np1 (Up)
```

因此在 mlx5_1 上启动 ib_write_bw 测试：

```bash
ib_write_bw -d mlx5_1 -p 18515
```

显示：

```bash
************************************
* Waiting for client to connect... *
************************************
```

在客户端执行：

```bash
ibstatus
```

显示为：

```bash
Infiniband device 'mlx5_0' port 1 status:
	default gid:	 fe80:0000:0000:0000:8e2a:8eff:fe88:a136
	base lid:	 0x0
	sm lid:		 0x0
	state:		 4: ACTIVE
	phys state:	 5: LinkUp
	rate:		 100 Gb/sec (4X EDR)
	link_layer:	 Ethernet
```

因此在 mlx5_0 上启动 ib_write_bw 测试：

```bash
ib_write_bw -d mlx5_0 -p 18515 192.168.119.1
```

客户端显示测试结果为：

```bash
---------------------------------------------------------------------------------------
                    RDMA_Write BW Test
 Dual-port       : OFF		Device         : mlx5_0
 Number of qps   : 1		Transport type : IB
 Connection type : RC		Using SRQ      : OFF
 PCIe relax order: ON		Lock-free      : OFF
 ibv_wr* API     : ON		Using DDP      : OFF
 TX depth        : 128
 CQ Moderation   : 1
 Mtu             : 1024[B]
 Link type       : Ethernet
 GID index       : 3
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0000 QPN 0x009b PSN 0x57bb8f RKey 0x1fd4bc VAddr 0x007f0e0c8aa000
 GID: 00:00:00:00:00:00:00:00:00:00:255:255:192:168:119:02
 remote address: LID 0000 QPN 0x0146 PSN 0x860b7f RKey 0x23c6bc VAddr 0x007fb25aea4000
 GID: 00:00:00:00:00:00:00:00:00:00:255:255:192:168:119:01
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[MiB/sec]    BW average[MiB/sec]   MsgRate[Mpps]
 65536      5000             11030.78            11030.45		     0.176487
---------------------------------------------------------------------------------------
```

服务器端显示测试结果为：

```bash
---------------------------------------------------------------------------------------
                    RDMA_Write BW Test
 Dual-port       : OFF		Device         : mlx5_1
 Number of qps   : 1		Transport type : IB
 Connection type : RC		Using SRQ      : OFF
 PCIe relax order: ON		Lock-free      : OFF
 ibv_wr* API     : ON		Using DDP      : OFF
 CQ Moderation   : 1
 Mtu             : 1024[B]
 Link type       : Ethernet
 GID index       : 3
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0000 QPN 0x0146 PSN 0x860b7f RKey 0x23c6bc VAddr 0x007fb25aea4000
 GID: 00:00:00:00:00:00:00:00:00:00:255:255:192:168:119:01
 remote address: LID 0000 QPN 0x009b PSN 0x57bb8f RKey 0x1fd4bc VAddr 0x007f0e0c8aa000
 GID: 00:00:00:00:00:00:00:00:00:00:255:255:192:168:119:02
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[MiB/sec]    BW average[MiB/sec]   MsgRate[Mpps]
 65536      5000             11030.78            11030.45		     0.176487
---------------------------------------------------------------------------------------
```

测试成功，说明 RDMA 网络就绪。

### 开启 nfs server 的 RDMA 支持

nfs server 需要开启 rdma 支持： 

```bash
sudo vi /etc/nfs.conf
```

修改 /etc/nfs.conf 文件的 [nfsd] 部分（备注：尽量要把 nfs 版本那几行也放开，限制为只提供4.2版本）：

```bash
[nfsd]
# debug=0
# 线程改大一点，默认8太少了
threads=128
# host=
# port=0
# grace-time=90
# lease-time=90
# udp=n
# tcp=y
# 版本这几行一定要设置，注意只保留4.2版本
vers3=n
vers4=y
vers4.0=n
vers4.1=n
vers4.2=y
rdma=y
rdma-port=20049
```

然后重启 NFS 服务：

```bash
sudo systemctl restart nfs-server
```

然后检查 nfsd 的监听端口，验证 rdma 是否生效：

```bash
sudo cat /proc/fs/nfsd/portlist                    
rdma 20049
rdma 20049
tcp 2049
tcp 2049
```

如果看到 rdma 20049 端口，说明 rdma 配置成功。反之如果没有看到 rdma 20049 端口，说明 rdma 配置失败，需要检查前面的配置。

## 配置 nfs client（rdma）

### 检查 rdma 模块

确保客户端和 rdma 相关的模块都已经加载：

```bash
lsmod | grep -E "rpcrdma|svcrdma|xprtrdma|ib_core|mlx5_ib"
```

~~如果没有，手动加载模块~~（实际操作下来发现不手工加载也可以，系统会自动加载）：

```bash
sudo modprobe rpcrdma
sudo modprobe svcrdma
sudo modprobe xprtrdma
sudo modprobe ib_core
sudo modprobe mlx5_ib
```

确认 DKMS 模块已注册：

```bash
sudo dkms status | grep nfsrdma
```

预计输出：

```bash
mlnx-nfsrdma/24.10.OFED.24.10.2.1.8.1, 6.1.0-31-amd64, x86_64: installed
```

检查内核日志：

```bash
sudo dmesg | grep rdma
```

预计输出：

```bash
[ 3273.613081] RPC: Registered rdma transport module.
[ 3273.613082] RPC: Registered rdma backchannel transport module.
[ 3695.887144] svcrdma: svcrdma is obsoleted, loading rpcrdma instead
[ 3695.923962] xprtrdma: xprtrdma is obsoleted, loading rpcrdma instead
```

### 配置 nfs 访问（rdma）

准备挂载点：

```bash
cd /mnt
sudo mkdir -p nfs-rdma
```

带 nfsrdma 方式的挂载 nfs：

```bash
sudo mount -t nfs -o rdma,port=20049,vers=4.2,nconnect=16 192.168.119.1:/exports/shared /mnt/nfs-rdma
```

挂载成功后，测试一下读写速度：

```bash
# nfs 写入100G数据，速度大概在 1.0 GB/s
sudo dd if=/dev/zero of=./test-100g.img bs=1G count=100 oflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 104.569 s, 1.0 GB/s
# nfs 读取100G数据，速度大概在 2.2 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 47.4093 s, 2.2 GB/s
```

对比三者的速度，同一块铠侠 cd6 3.84T ssd 硬盘直通到虚拟机：

| 场景               | 100g 单文件写入 | 100g 单文件读取 |
| ------------------ | --------------- | --------------- |
| 直接读写硬盘       | 1.3 GB/s        | 4.0 GB/s        |
| nfs 挂载(non-rdma) | 1.0 GB/s        | 2.5 GB/s        |
| nfs 挂载(rdma)     | 1.0 GB/s        | 2.2 GB/s        |

