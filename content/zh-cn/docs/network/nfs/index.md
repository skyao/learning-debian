---
title: "nfs 网络文件系统"
linkTitle: "nfs"
date: 2024-09-16
weight: 40
description: >
  在 debian 下利用 nfs 实现网络文件系统
---

debian12 和 debian 13 下操作基本相同, 但debian 13 下需要处理一些额外的问题.

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

### debian13的特殊处理

debian12 下工作正常,但是在 debian 13 下会出现 "nfsdctl: lockd configuration failure" 的报错: 

```bash
sudo systemctl status nfs-kernel-server

Nov 07 09:58:15 devserver193 systemd[1]: Starting nfs-server.service - NFS server and services...
Nov 07 09:58:15 devserver193 sh[892]: nfsdctl: lockd configuration failure
Nov 07 09:58:15 devserver193 systemd[1]: Finished nfs-server.service - NFS server and services.
```

这是因为在 Debian 13 中，nfs-lock.service 已经不存在了:

```bash
sudo systemctl status nfs-lock 

Unit nfs-lock.service could not be found.
```

在 Debian 13 中, 文件锁功能由 nfs-utils 包中的 statd 和内核的 lockd 模块提供，不再单独以 nfs-lock.service 形式出现。这就是上面报错 “Unit nfs-lock.service could not be found” 的原因。

Debian 12 和 13 的不同在于:

- Debian 12 (Bookworm) 还保留了 nfs-lock.service，它主要负责启动 rpc.statd，用于 NFS 文件锁。

- Debian 13 (Trixie/unstable) 中，NFS 服务架构调整，nfs-lock.service 被移除，相关功能直接由 rpc.statd 和 rpcbind 管理。

解决方式:

- 检查 rpcbind 和 statd

   ```bash
   $ systemctl status rpcbind

    ● rpcbind.service - RPC bind portmap service
        Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; preset: enabled)
        Active: active (running) since Fri 2025-11-07 10:06:15 CST; 10min ago

   $ ps aux | grep statd

   statd        790  0.0  0.0   4556  2040 ?        Ss   10:06   0:00 /usr/sbin/rpc.statd
   ```

   这两个在 debian13 上都存在.

- 检查内核模块 lockd

   ```bash
   $ lsmod | grep lockd

    lockd                 163840  1 nfsd
    grace                  12288  2 nfsd,lockd
    sunrpc                872448  12 nfsd,auth_rpcgss,lockd,nfs_acl
   ```

- 配置 nfs.conf

   在 Debian 13 中，锁相关配置放在 /etc/nfs.conf 的 [lockd] 部分

   ```bash
   sudo vi /etc/nfs.conf
   ```

   默认内容:

   ```properties
   [lockd]
   # port=0
   # udp-port=0
   ```

   修改为:

   ```properties
   [lockd]
   port=32769
   udp-port=32803
   ```

- 配置文件

   ```bash
   sudo vi /etc/modprobe.d/lockd.conf
   ```

   内容为:

  ```bash
   options lockd nlm_tcpport=32769 nlm_udpport=32803
  ```

   更新 initramfs:

  ```bash
   sudo update-initramfs -u
  ```

- 重启nfs server, 最好是重启机器:

   ```bash
   sudo systemctl restart nfs-server
   ```

但这个错误并没有消息, 只是后来我发现 lock 的功能只在 nfs3 中使用, 我目前是关闭 nfs3, 只开启 nfs 4.2 (包括 nfs 4.0 和 4.1 也关闭了), 所以应该不会有影响. 暂时先忽略这个错误.

AI 给的解释: 在只启用 NFSv4.2 的情况下，报错 “lockd configuration failure” 可以完全忽略，因为 lockd 仅用于 NFSv2/v3 的文件锁。NFSv4 自带内建的锁机制，不依赖 lockd 模块。

参考:

- https://www.reddit.com/r/archlinux/comments/1l7jwh3/nfsdctl_lockd_configuration_failure_i_cant_find/

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
sudo mount -t nfs 192.168.3.193:/exports/shared /mnt/nfs
```

现在 nfs 服务器端和客户端之间的网络共享配置完成。
