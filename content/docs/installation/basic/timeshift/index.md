---
title: "timeshift"
linkTitle: "timeshift"
date: 2024-01-18
weight: 10
description: >
  安装配置 timeshift 进行备份
---

备注：安装完成之后，第一时间安装 timeshift 进行备份，后续操作中出现任何失误都可以通过 timeshift 来选择性的恢复到中间状态，避免出错后需要重新安装。

具体操作参考：

https://skyao.io/learning-ubuntu-server/docs/installation/timeshift/

## 安装

先 su 到 root，再进行安装：

```bash
su root
apt install timeshift
```

## 配置

找到 timeshift 分区的 UUID：

```bash
lsblk -f
```

例如这个机器有三块硬盘， `/var/timeshift` 所在的分区 UUDI 为 `3c5ad47e-4318-4797-8342-7e602ac524d2` ：

```bash
$ lsblk -f
NAME        FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
nvme1n1                                                                            
`-nvme1n1p1 ext4   1.0         ccae6bf3-d4cd-422d-b67a-b5b8dfe83fc6  782.2G     0% /var/data2
nvme0n1                                                                            
`-nvme0n1p1 ext4   1.0         eff3a45e-be5e-4f5e-a969-0998a8a10e33  782.2G     0% /var/data
nvme2n1                                                                            
|-nvme2n1p1 vfat   FAT32       C4B5-2470                             505.1M     1% /boot/efi
|-nvme2n1p2 ext4   1.0         41949492-a351-4770-80f6-d4c7dc5b23bc  171.2G     1% /
`-nvme2n1p3 ext4   1.0         3c5ad47e-4318-4797-8342-7e602ac524d2     48G     0% /var/timeshift
```

然后设置 backup_device_uuid ，注意 timeshift 在第一次使用时会读取 default.json 文件：

```bash
vi /etc/timeshift/default.json
```

验证一下：

```bash
timeshift --list
```

这个时候还没有进行备份，没有 snapshot：

```bash
timeshift --list
First run mode (config file not found)
Selected default snapshot type: RSYNC
Mounted '/dev/nvme2n1p3' at '/run/timeshift/3424/backup'
Device : /dev/nvme2n1p3
UUID   : 3c5ad47e-4318-4797-8342-7e602ac524d2
Path   : /run/timeshift/3424/backup
Mode   : RSYNC
Status : No snapshots on this device
First snapshot requires: 0 B

No snapshots found
```

此时会自动创建配置文件 `/etc/timeshift/timeshift.json`，后续修改配置就要修改这个文件。

### 配置 excludes

除了基本的 backup_device_uuid 外，还需要配置 excludes 以排除不需要 timeshift 进行备份的内容。

```json
{
  "backup_device_uuid" : "3c5ad47e-4318-4797-8342-7e602ac524d2",
  ......
  "exclude" : [
    "/root/**",
    "/home/**",
    "/timeshift/**",
    "/var/data/**",
    "/var/data2/**",
    "/var/data3/**"
  ],
  ......
}
```

需要排除的内容通常包括用户目录（`/root/`和 `/home/`），以及资料存储如我这里的 `"/var/data/` 等几块用来存储的硬盘，以及 timeshift 自身所在目录 `/timeshift/` （取决于安装debian时选择的timeshift分区的挂载路径）。

### 配置自动备份

设置每天/每周/每月的自动备份：

```json
{
  ......
  "schedule_monthly" : "true",
  "schedule_weekly" : "true",
  "schedule_daily" : "true",
  ......
  "count_monthly" : "2",
  "count_weekly" : "3",
  "count_daily" : "5",
  ......
}
```

## 创建快照

先不做任何操作，在操作系统安装完成之后，第一时间进行备份：

```bash
timeshift --create --comments "first backup after install"
```

第一次备份大概要消耗2.3g的存储空间。