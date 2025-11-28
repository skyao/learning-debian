---
title: "tmpfs"
linkTitle: "tmpfs"
date: 2025-11-28
weight: 10
description: >
  用 tmpfs 实现 ramdisk
---

## 手工挂载

手工操作, 实验一下. 先创建挂载点

```bash
sudo mkdir -p /mnt/ramdisk
```

手动挂载一个 8G 的内存盘：

```bash
sudo mount -t tmpfs -o size=8G tmpfs /mnt/ramdisk
```

测试写入速度和读取:

```bash
sudo dd if=/dev/zero of=/mnt/ramdisk/test5g.img bs=1G count=5 oflag=dsync

sudo dd if=/mnt/ramdisk/test5g.img of=/dev/null bs=8M
```

不同机器的cpu和内存性能不一样, 测试出来的 ramdisk 读写性能也不一样, 以下是在家用平台的测试: 

|              机器配置              | 写入性能 | 读取性能  |
| :--------------------------------: | :------: | :-------: |
| x99 + e5 2650v4 2.2GHz + ddr4 2133 | 1.4 GB/s | 5.5 GB/s  |
|      e3 1265l v3 + ddr3 1600       | 2.7 GB/s | 8.8 GB/s  |
|   intel 13900hk 5.4g + ddr4 4000   | 3.6 GB/s | 14.7 GB/s |
|   intel 13700k 5.4G + ddr4 4000    | 4.3 GB/s | 17.0 GB/s |

测试完成手工卸载并删除目录: 

```bash
sudo umount  /mnt/ramdisk
sudo rm -rf /mnt/ramdisk
```

## 自动挂载

创建 systemd 服务 ramdisk:

```bash
sudo vi /etc/systemd/system/ramdisk.service
```

内容为:

```bash
[Unit]
Description=Mount tmpfs and copy data to RAM disk
DefaultDependencies=no
After=local-fs.target
Before=umount.target

[Service]
Type=oneshot
# 确保挂载点目录存在
ExecStartPre=/bin/mkdir -p /mnt/ramdisk
# 挂载 tmpfs
ExecStart=/bin/mount -t tmpfs -o size=8G tmpfs /mnt/ramdisk
# 复制数据到内存盘
ExecStart=/bin/bash -c 'if [ -d /opt/data-for-ramdisk ] && [ "$(ls -A /opt/data-for-ramdisk)" ]; then cp -a /opt/data-for-ramdisk/* /mnt/ramdisk/; else echo "Source dir missing or empty, skip copy"; fi'
# 挂载为只读
ExecStartPost=/bin/mount -o remount,ro /mnt/ramdisk

# 在关机时自动卸载
ExecStop=/bin/umount /mnt/ramdisk

RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

启动, 并设置开机自动启动:

```bash
sudo systemctl daemon-reload
sudo systemctl enable ramdisk.service
sudo systemctl start ramdisk.service
```

