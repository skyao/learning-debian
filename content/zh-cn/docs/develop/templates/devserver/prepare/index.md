---
title: "准备工作"
linkTitle: "准备"
date: 2025-04-10
weight: 10
description: >
  准备工作：虚拟机，磁盘系统
---

## 准备虚拟机

从模版 template-debian12-basic-v01 （取最新版本） 克隆一个虚拟机，命名为 template-debian12-devserver-v01，VM ID 为 990301.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 16g（mini 8192，memory 16384）。

## 准备磁盘

devserver 预计会有两台实例，用于两个异地的开发环境。

我为每台实例都准备了 2 块三星 pm983a 900G 的 ssd 磁盘，一块用于应用（如数据库，redis，queue等），一块用于数据（如pve需要的nfs，nexus 代理仓库等）。

在 pve 中，将两块 ssd 磁盘直通给到虚拟机，并挂载到 /data 目录。

![](images/pass-through-ssd.png)

在虚拟机中可以看到这两块磁盘：

```bash
lspci | grep Non-Volatile
01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
02:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
```

## 准备网络共享

然后配置网络共享： 

- nfs server ： 参考 [用虚拟机实现的SSD NAS存储]({{< relref "../../../../storage/devserver91/" >}})
- sftp server： 参考 [安装 SFTP 服务器]({{< relref "../../../../installation/basic/additional/#sftp-server" >}})

