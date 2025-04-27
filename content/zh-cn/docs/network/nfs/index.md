---
title: "nfs 网络文件系统"
linkTitle: "nfs"
date: 2024-09-16
weight: 40
description: >
  在 debian12 下利用 nfs 实现网络文件系统
---

## nfs 服务器端

### 安装 nfs server

```bash
sudo apt install nfs-kernel-server -y
```

### 准备硬盘和分区

直通一块 3.84T 的 KIOXIA CD6 pcie4 SSD 进来虚拟机。查看这块 ssd 硬盘：

```bash
lspci | grep Non-Volatile
02:00.0 Non-Volatile memory controller: KIOXIA Corporation NVMe SSD Controller Cx6 (rev 01)
```

硬盘分区：

```bash
sudo fdisk /dev/nvme0n1
```

g 转为 GPT partition table p 打印分区表 n 创建新分区，这里就只创建一个大分区。

```bash
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   512G  0 disk
├─sda1        8:1    0   512M  0 part /boot/efi
├─sda2        8:2    0 465.7G  0 part /
└─sda3        8:3    0  45.8G  0 part /timeshift
nvme0n1     259:0    0   3.5T  0 disk
└─nvme0n1p1 259:1    0   3.5T  0 part

```

将硬盘格式化为 ext4 文件系统：

```bash
sudo mkfs.ext4 /dev/nvme0n1p1
```

查看 ssd 分区的 uuid：

```bash
$ sudo lsblk -f
NAME        FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1      vfat   FAT32       BE75-FC62                             505.1M     1% /boot/efi
├─sda2      ext4   1.0         81fdaf25-6712-48ee-bb53-1c4a78c8ef9f  430.4G     1% /
└─sda3      ext4   1.0         4b922cfb-2123-48ce-b9fe-635e73fb6aa8     39G     8% /timeshift
nvme0n1
└─nvme0n1p1 ext4   1.0         1dee904a-aa51-4180-b65b-9449405b841f
```

准备挂载这块硬盘

```bash
sudo vi /etc/fstab
```

增加以下内容：

```bash
# data storage was on /dev/nvme0n1p1(3.84T)
UUID=1dee904a-aa51-4180-b65b-9449405b841f /mnt/data      ext4    defaults        0       2
```

重启之后再看：

```bash
$ sudo lsblk -f

NAME        FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1      vfat   FAT32       BE75-FC62                             505.1M     1% /boot/efi
├─sda2      ext4   1.0         81fdaf25-6712-48ee-bb53-1c4a78c8ef9f  430.4G     1% /
└─sda3      ext4   1.0         4b922cfb-2123-48ce-b9fe-635e73fb6aa8     39G     8% /timeshift
nvme0n1
└─nvme0n1p1 ext4   1.0         1dee904a-aa51-4180-b65b-9449405b841f    3.3T     0% /mnt/data
```

### 准备伪文件系统

为了方便后续的管理，采用伪文件系统:

```bash
sudo mkdir -p /mnt/data/shared

sudo chown -R nobody:nogroup /mnt/data/shared

cd /mnt/data 
```

创建 export 目录：

```bash
sudo mkdir -p /exports/shared

sudo chown -R nobody:nogroup /exports
```

修改 `/etc/fstab` 文件来 mount 伪文件系统和 exports :

```bash
sudo vi /etc/fstab
```

增加如下内容:

```bash
# nfs exports
/mnt/data/shared /exports/shared     none bind
```

### 配置 nfs export

```bash
sudo vi /etc/exports
```

增加以下内容：

```bash
/exports/shared   192.168.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

重启 nfs-kernel-server，查看 nfs-kernel-server 的状态：

```bash
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

验证：

```bash
ps -ef | grep nfs
```

输出为：

```
root         918       1  0 01:25 ?        00:00:00 /usr/sbin/nfsdcld
root        1147       2  0 01:26 ?        00:00:00 [nfsd]
root        1148       2  0 01:26 ?        00:00:00 [nfsd]
root        1149       2  0 01:26 ?        00:00:00 [nfsd]
root        1150       2  0 01:26 ?        00:00:00 [nfsd]
root        1151       2  0 01:26 ?        00:00:00 [nfsd]
root        1152       2  0 01:26 ?        00:00:00 [nfsd]
root        1153       2  0 01:26 ?        00:00:00 [nfsd]
root        1154       2  0 01:26 ?        00:00:00 [nfsd]
```

查看当前挂载情况：

```bash
$ sudo showmount -e
Export list for debian12:
/exports/shared 192.168.0.0/16
```

### 设置 nfs 版本支持

查看目前服务器端支持的 nfs 版本：

```bash
sudo cat /proc/fs/nfsd/versions
```

默认情况下，nfs server 支持的 nfs 版本为：

```bash
+3 +4 +4.1 +4.2
```

通常我们只需要保留 nfs 4.2 版本，其他版本都删除：

```bash
sudo vi /etc/nfs.conf
```

将默认的 nfsd：

```bash
[nfsd]
# debug=0
# threads=8
# host=
# port=0
# grace-time=90
# lease-time=90
# udp=n
# tcp=y
# vers3=y
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
# rdma=n
# rdma-port=20049
```

修改为:

```bash
[nfsd]
# debug=0
threads=32
# host=
# port=0
# grace-time=90
# lease-time=90
# udp=n
# tcp=y
vers3=n
vers4=y
vers4.0=n
vers4.1=n
vers4.2=y
# rdma=n
# rdma-port=20049
```

顺便把 nfs 线程数量从默认的 8 修改为 32。

重启 nfs-kernel-server：

```bash
sudo systemctl restart nfs-kernel-server
```

然后验证 nfs 版本：

```bash
sudo cat /proc/fs/nfsd/versions
```

输出为：

```bash
-3 +4 -4.0 -4.1 +4.2
```

注意： +4 是必须保留的，只有先设置 +4 版本，才能设置 4.0/4.1/4.2 版本，如果 -4 则 4.0/4.1/4.2 版本都会不支持。

也可以通过 rpcinfo 命令查看 nfs 版本：

```bash
rpcinfo -p localhost
```

输出为：

```bash
   program vers proto   port  service
    ......
    100003    4   tcp   2049  nfs
```

这里的 4 代表 nfs 4.x 版本，但是没法区分 4.0/4.1/4.2 版本。

## nfs 客户端

### 安装 nfs-common

然后安装 nfs-common 作为 nfs client：

```bash
sudo apt-get install nfs-common 
```

### 配置 nfs 访问

准备好挂载点：

```bash
cd /mnt
sudo mkdir -p nfs
```

不带 nfsrdma 方式的挂载 nfs：

```bash
sudo mount -t nfs 192.168.3.227:/exports/shared /mnt/nfs
```

挂载成功后，测试一下读写速度：

```bash
cd nfs

# nfs 写入10G数据，速度大概在 610 MB/s
sudo dd if=/dev/zero of=./test-10g.img bs=1G count=10 oflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 17.6079 s, 610 MB/s

# nfs 读取100G数据，速度大概在 1.1 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 96.5171 s, 1.1 GB/s
```

对比一下在 nfs server 端直接硬盘读写 100G 数据的速度：

```bash
# 直接硬盘写入100G数据，速度大概在 1.3 GB/s
sudo dd if=/dev/zero of=./test-100g.img bs=1G count=100 oflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 82.5747 s, 1.3 GB/s

# 直接硬盘读取100G数据，速度大概在 4.0 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 26.9138 s, 4.0 GB/s
```

写入性能差异很大（1.3 GB/s 降低到 610 MB/s），估计是用 pve vmbr 的网卡，导致写入性能下降。读取的性能有更大的差异（4.0 GB/s 降低到 1.1 GB/s）。

现在 nfs 服务器端和客户端之间的网络共享配置完成。