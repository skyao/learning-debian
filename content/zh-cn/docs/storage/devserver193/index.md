---
title: "用虚拟机实现的SSD NAS存储"
linkTitle: "devserver193（虚拟机）"
date: 2025-11-03
weight: 10
description: >
  利用 debian 13 在 pve 虚拟机中实现的 SSD NAS 方案
---

## 构想

devserver192 开发服务器上，需要搭建一个 nfs 服务器，给 pve 和其他方式使用。

## 准备工作

### 准备虚拟机

基于 template-debian13-dev 的虚拟机上搭建。

### 硬盘直通

在虚拟机硬件中，增加两块 hard disk，大小为 1024g，scsi 类型，virtIO scsi 控制器。注意把 backup 选项勾选去掉。

能看到这块 ssd 硬盘：

```bash
$ lspci | grep storage

09:01.0 SCSI storage controller: Red Hat, Inc. Virtio SCSI
09:02.0 SCSI storage controller: Red Hat, Inc. Virtio SCSI
09:03.0 SCSI storage controller: Red Hat, Inc. Virtio SCSI
```

因为硬盘没有分区，所以看起来是这样：

```bash
$ lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   512G  0 disk 
|-sda1   8:1    0   976M  0 part /boot/efi
|-sda2   8:2    0 476.8G  0 part /
`-sda3   8:3    0  34.2G  0 part /timeshift
sdb      8:16   0     1T  0 disk 
sdc      8:32   0     1T  0 disk
```

sdb 和 sdc 是虚拟机的磁盘，分配了 1T 容量。

### 硬盘分区

```bash
sudo fdisk /dev/sdb
```

g 转为 GPT partition table
p 打印分区表
n 创建新分区，这里就只创建一个900g的大分区给 nas 用。

分区完成后执行

```bash
sudo fdisk -l
```

查看硬盘的情况：

```bash
Disk /dev/sda: 512 GiB, 549755813888 bytes, 1073741824 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 04D3A8CA-AB44-4DEC-8A1B-98AD4FEF1F44

Device          Start        End   Sectors   Size Type
/dev/sda1        2048    2000895   1998848   976M EFI System
/dev/sda2     2000896 1002000383 999999488 476.8G Linux filesystem
/dev/sda3  1002000384 1073739775  71739392  34.2G Linux filesystem


Disk /dev/sdc: 1 TiB, 1099511627776 bytes, 2147483648 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3d9a3f59

Device     Boot Start        End    Sectors  Size Id Type
/dev/sdc1        2048 2147483647 2147481600 1024G 83 Linux


Disk /dev/sdb: 1 TiB, 1099511627776 bytes, 2147483648 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0268fcf0

Device     Boot Start        End    Sectors  Size Id Type
/dev/sdb1        2048 2147483647 2147481600 1024G 83 Linux
```

### 硬盘格式化

将硬盘格式化为 ext4 文件系统：

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdc1
```

### 挂载分区

查看 sdb1 和 sdc1 两块分区的 uuid：

```bash
$ sudo lsblk -f
              
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                           
|-sda1 vfat   FAT32       43A0-39FF                             965.3M     1% /boot/efi
|-sda2 ext4   1.0         79a94e00-07b1-4621-992b-979ab27184f4  439.3G     1% /
`-sda3 ext4   1.0         b34600b0-581d-4d80-b1bf-d484d9ebc212   28.6G     9% /timeshift
sdb                                                                           
`-sdb1 ext4   1.0         179f9dd8-139b-4648-bcbd-953eab84527b                
sdc                                                                           
`-sdc1 ext4   1.0         be5ac1c0-11b9-4dcf-bdcd-cdd9234e7fa5
```

执行

```bash
sudo vi /etc/fstab
```

查看目前现有的系统盘的三个分区的挂载情况：

```bash
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during installation
UUID=79a94e00-07b1-4621-992b-979ab27184f4 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda1 during installation
UUID=43A0-39FF  /boot/efi       vfat    umask=0077      0       1
# /timeshift was on /dev/sda3 during installation
UUID=b34600b0-581d-4d80-b1bf-d484d9ebc212 /timeshift      ext4    defaults        0       2
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
```

增加两块 ssd 硬盘的挂载，挂载到 "/mnt/data" 和 "/mnt/app"：

```bash
# data storage was on /dev/sdb1(1t)
UUID=179f9dd8-139b-4648-bcbd-953eab84527b /mnt/data      ext4    defaults        0       2

# app storage was on /dev/sdc1(1t)
UUID=be5ac1c0-11b9-4dcf-bdcd-cdd9234e7fa5 /mnt/app      ext4    defaults        0       2
```

重启机器。再看一下分区挂载情况：

```bash
$ sudo lsblk -f

[sudo] password for sky: 
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                           
`-sda1 ext4   1.0         be5ac1c0-11b9-4dcf-bdcd-cdd9234e7fa5  955.6G     0% /mnt/app
sdb                                                                           
|-sdb1 vfat   FAT32       43A0-39FF                             965.3M     1% /boot/efi
|-sdb2 ext4   1.0         79a94e00-07b1-4621-992b-979ab27184f4  439.3G     1% /
`-sdb3 ext4   1.0         b34600b0-581d-4d80-b1bf-d484d9ebc212   28.6G     9% /timeshift
sdc                                                                           
`-sdc1 ext4   1.0         179f9dd8-139b-4648-bcbd-953eab84527b  955.6G     0% /mnt/data
```

比较奇怪的是,重启之后, 三块硬盘的顺序变化了, 系统所在的硬盘原来是 sda, 现在变成 sdb 了. 好在不影响功能, 凑合这么用吧.

### 准备共享目录

为了方便后续的管理，采用伪文件系统:

```bash
cd /mnt/data 

sudo mkdir shared
sudo mkdir pve-shared

sudo chown -R nobody:nogroup /mnt/data/shared
sudo chown -R nobody:nogroup /mnt/data/pve-shared
```

创建 export 目录：

```bash
sudo mkdir -p /exports/{shared,pve-shared}

sudo chown -R nobody:nogroup /exports
```

修改 `/etc/fstab` 文件来 mount 伪文件系统和 exports

```bash
sudo vi /etc/fstab
```

增加如下内容:

```bash
# nfs exports
/mnt/data/shared /exports/shared     none bind
/mnt/data/pve-shared /exports/pve-shared    none bind
```

重启。

## 搭建 nfs 服务器端

### 安装 nfs server

```bash
# 安装
sudo apt install nfs-kernel-server -y

# 开机自启
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server

# 验证
sudo systemctl status nfs-kernel-server

Nov 07 09:58:15 devserver193 systemd[1]: Starting nfs-server.service - NFS server and services...
Nov 07 09:58:15 devserver193 sh[892]: nfsdctl: lockd configuration failure
Nov 07 09:58:15 devserver193 systemd[1]: Finished nfs-server.service - NFS server and services.
```

备注: "nfsdctl: lockd configuration failure" 的报错可以忽略.

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

### 配置 nfs export

```bash
sudo vi /etc/exports
```

修改 nfs exports 的内容，这里我们 export shared/pve-shared 目录：

```bash
/exports/shared   192.168.0.0/16(rw,sync,no_subtree_check,no_root_squash)
/exports/pve-shared   192.168.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

重启 nfs-kernel-server，查看 nfs-kernel-server 的状态：

```bash
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

输出为：

```bash
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: enabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Fri 2025-11-07 11:04:59 CST; 4h 51min ago
 Invocation: 98ff0eaeb5d640ecbf09b338cc217903
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 1347 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 1350 ExecStart=/bin/sh -c /usr/sbin/nfsdctl autostart || /usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
   Main PID: 1350 (code=exited, status=0/SUCCESS)
   Mem peak: 2M
        CPU: 8ms

Nov 07 11:04:59 devserver193 systemd[1]: Starting nfs-server.service - NFS server and services...
Nov 07 11:04:59 devserver193 sh[1352]: nfsdctl: lockd configuration failure
Nov 07 11:04:59 devserver193 systemd[1]: Finished nfs-server.service - NFS server and services.
```

验证：

```bash
ps -ef | grep nfs
```

输出为：

```
ps -ef | grep nfs
root         714       1  0 23:04 ?        00:00:00 /usr/sbin/nfsdcld
root         866       2  0 23:09 ?        00:00:00 [nfsd]
root         867       2  0 23:09 ?        00:00:00 [nfsd]
root         868       2  0 23:09 ?        00:00:00 [nfsd]
root         869       2  0 23:09 ?        00:00:00 [nfsd]
root         870       2  0 23:09 ?        00:00:00 [nfsd]
root         871       2  0 23:09 ?        00:00:00 [nfsd]
root         872       2  0 23:09 ?        00:00:00 [nfsd]
root         873       2  0 23:09 ?        00:00:00 [nfsd]
```

查看当前挂载情况：

```bash
$ sudo showmount -e
Export list for devserver91:
/exports/pve-shared 192.168.0.0/16
/exports/shared     192.168.0.0/16
```

## 设置 nfs 客户端

### debian client

安装 nfs client：

```bash
sudo apt install nfs-common
```

准备目录:

```bash
sudo mkdir /mnt/shared193
```

挂载 nfs 到本地目录:

```bash
sudo mount -t nfs4 192.168.3.193:/exports/shared /mnt/shared193
```

这样就能手工将远程 `/exports/shared` 目录挂载到本地目录。

暂时不采用开机自动挂载的方式，避免减缓开机的速度，增加故障概率，需要 mount 时手工挂载就是。

方便起见，准备一个 mount_shared193 脚本：

```bash
sudo vi /mnt/mount_shared193.zsh
```

内容为：

```zsh
#!/usr/bin/env zsh

# 定义变量
NFS_SERVER="192.168.3.193"
NFS_EXPORT="/exports/shared"
MOUNT_POINT="/mnt/shared193"

# 检查挂载点是否存在，不存在则创建
if [[ ! -d "$MOUNT_POINT" ]]; then
    echo "创建挂载点目录: $MOUNT_POINT"
    sudo mkdir -p "$MOUNT_POINT"
    sudo chown "$USER:$USER" "$MOUNT_POINT"  # 可选：设置当前用户为所有者
fi

# 检查是否已挂载
if mount | grep -q "$MOUNT_POINT"; then
    echo "⚠️  $MOUNT_POINT 已经挂载"
else
    echo "正在挂载 NFS: $NFS_SERVER:$NFS_EXPORT → $MOUNT_POINT"
    sudo mount -t nfs4 "$NFS_SERVER:$NFS_EXPORT" "$MOUNT_POINT"
    
    # 检查挂载是否成功
    if [[ $? -eq 0 ]]; then
        echo "✅ 挂载成功"
        df -h | grep "$MOUNT_POINT"  # 显示磁盘使用情况
    else
        echo "❌ 挂载失败，请检查:"
        echo "1. NFS 服务器是否在线？"
        echo "2. 客户端是否安装 nfs-common？ (sudo apt install nfs-common)"
        echo "3. 防火墙是否放行 NFS 端口？"
    fi
fi
```

再准备一个 unmount_shared193 脚本：

```bash
sudo vi /mnt/unmount_shared193.zsh
```

内容为：

```zsh
#!/usr/bin/env zsh

# 定义变量
NFS_SERVER="192.168.3.193"
NFS_EXPORT="/exports/shared"
MOUNT_POINT="/mnt/shared193"

if mount | grep -q "$MOUNT_POINT"; then
     sudo umount -l "$MOUNT_POINT"
     echo "✅ 已卸载 $MOUNT_POINT"
else
     echo "⚠️  $MOUNT_POINT 未挂载"
fi
```

增加可执行权限:

```bash
sudo chmod +x /mnt/mount_shared193.zsh
sudo chmod +x /mnt/unmount_shared193.zsh
```

之后只要执行相应的命令就可以手工挂载和卸载 nfs shared 目录。

### pve storage

在 pve 下，点击 “datacenter” -> “storage” -> “Add”

![](images/add-nfs.png)

完成这个设置之后，该集群内的任何一台机器上，都会出现一个 `/mnt/pve/nfs193` 目录，mount 到 上面的 nfs exports。之后就可以通过这个目录像访问本地文件夹一样访问nfs。


## 测速和对比

挂载成功后，测试一下读写速度.

### 走 vmbr 虚拟网络

nfs client 运行在同一个物理机上的另一个虚拟机, 没有走物理网络, 而是走的 vmbr 网络, 测试速度为:

```bash
cd nfs

# vmbr 网络写入10G数据，速度大概在 2.1 GB/s
sudo dd if=/dev/zero of=./test-10g.img bs=1G count=10 oflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 5.17145 s, 2.1 GB/s

# vmbr 网络写入100G数据，速度大概在 2.1 GB/s(很稳定)
sudo dd if=/dev/zero of=./test-100g.img bs=1G count=100 oflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 50.9795 s, 2.1 GB/s

# vmbr 网络读取10G数据，速度大概在 3.5 GB/s
sudo dd if=./test-10g.img of=/dev/null bs=1G count=10 iflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 3.06611 s, 3.5 GB/s

# vmbr 网络读取100G数据，速度大概在 3.6 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 30.1485 s, 3.6 GB/s
```

### 走 100g 物理网络

nfs 客户端在另外一台机器上的虚拟机, 走了物理网络, 两台机器之间 100g 网卡直连, 但都通过了 vmbr , 测试速度为: 

```bash
cd nfs

# nfs 写入10G数据，速度大概在 1.8 GB/s
sudo dd if=/dev/zero of=./test-10g.img bs=1G count=10 oflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 5.80745 s, 1.8 GB/s

# nfs 写入100G数据，速度大概在 2.2 GB/s(甚至比走本机 vmbr 网络的 2.1 GB还快?)
sudo dd if=/dev/zero of=./test-100g.img bs=1G count=100 oflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 49.8112 s, 2.2 GB/s

# nfs 读取10G数据，速度大概在 3.2 GB/s
sudo dd if=./test-10g.img of=/dev/null bs=1G count=10 iflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 3.33347 s, 3.2 GB/s

# nfs 读取100G数据，速度大概在 3 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 35.2614 s, 3.0 GB/s
```

### 本机直接访问硬盘

对比一下在 nfs server 端直接硬盘读写 100G 数据的速度：

```bash
# 本地硬盘写入10G数据，速度大概在 3.1 GB/s
sudo dd if=/dev/zero of=./test-10g.img bs=1G count=10 oflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 3.49949 s, 3.1 GB/s

# 本地硬盘写入100G数据，速度大概在 2.9 GB/s
sudo dd if=/dev/zero of=./test-100g.img bs=1G count=100 oflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 37.1302 s, 2.9 GB/s

# 本地硬盘读取10G数据，速度大概在 5.4 GB/s
sudo dd if=./test-10g.img of=/dev/null bs=1G count=10 iflag=dsync
10737418240 bytes (11 GB, 10 GiB) copied, 1.97764 s, 5.4 GB/s

# 本地硬盘读取100G数据，速度大概在 5.6 GB/s
sudo dd if=./test-100g.img of=/dev/null bs=1G count=100 iflag=dsync
107374182400 bytes (107 GB, 100 GiB) copied, 19.311 s, 5.6 GB/s
```

### 速度对比

物理硬盘为凯侠 CD6 3.84T nvme SSD, u2 接口, pcie 4.0 接口.

|               | 持续写入 10 GB | 持续写入 100 GB | 持续读取 10 GB | 持续读取 100 GB |
| ------------- | -------------- | --------------- | -------------- | --------------- |
| vmbr 网络     | 2.1 GB/s       | 2.1 GB/s        | 3.5 GB/s       | 3.6 GB/s        |
| nfs 100g 网络 | 1.8 GB/s       | 2.2 GB/s        | 3.2 GB/s       | 3.0 GB/s        |
| 本地硬盘      | 3.1 GB/s       | 2.9 GB/s        | 5.4 GB/s       | 5.6 GB/s        |
|               |                |                 |                |                 |

