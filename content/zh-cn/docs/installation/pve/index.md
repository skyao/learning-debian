---
title: "pve安装"
linkTitle: "pve安装"
date: 2025-10-26
weight: 20
description: >
  在 pve 下安装 debian 
---

备注：在 pve8 下安装 debian12 和在 pve9 下安装 debian13 是非常类似的。

## 准备工作

### 创建虚拟机

General：

- name： debian12 或者 debian13

OS:

- iso image： 选 netinst.iso
- type: linux
- version: 6.x - 2.6 kernel

System:

- Graphic card: default
- Machine: q35 (直通时必须)
- SCSI controller: Virtio SCSI single
- qemu agent: 勾选
- bios: OVMF(UEFI) (直通时必须)
- EFI storage: local
- Format: qemu image format(qcow2)

Disk:

- bus/device： SCSI
- SCSI Controller: 自动选择 Virtio SCSI single
- Cache： default
- IO Thread： 勾选
- Storage： local
- format： QEMU image format(qcow2)
- 高级选项中：勾选 backup
- side: 512g

cpu：

- Type：host （暂时不用担心迁移的问题）
- Socket：1
- Cores： 4

Memory:

- memory: 8192
- minimum memory: 2048

Network:

- bridage: vmbr0
- model: virtIO(paravirtualized)

确认之后，先别启动，在虚拟机属性中，找到 boot order ，去掉 net0，保留 csi0 和 ide2 光盘启动。

> 备注：一定要保留硬盘启动并且在光盘前，否则安装完成后重启又进入安装流程了。而 pve 有bug，这种情况下无法关机，只能把整机重启，非常烦人。

## 安装

### 启动并开始安装


启动后进入 debian 安装界面，选择 "Graphical install"

- select language: english
- select your location: other -> Asia -> China
- configure locales: United States en_US.UTF-8
- keyboard: American English

配置网络：

 - hostname: debian12.local 或者 debian13.local

配置用户和密码：

 - password of root: 
 - 创建新用户
   - full name： Sky Ao
   - username: sky
   - password:


硬盘分区，选择 Guided - use entire disk -> all files in one partition, 等默认分区方案出来之后， 先删除除了 esp 分区之后的分区，然后在空闲空间上选择 create a new partition，大小为空闲空间 - 50g左右（留给 timeshift 做备份）， mount 为 `/`。剩下的约 50g 空间继续分区，mount 为 `/timeshift`。


"Finish partitioning and write changes to disk" , 选下一步，会提示没有 swap 空间，询问要不要退回去，选择 no。

确认分区，然后开始安装 base system。

安装完成后，询问要不要扫描更多的媒体，选择 no。

configure the package manager, 选 "china" -> "mirrors.ustc.edu.cn"。

software selection，这是选择要继续安装的内容：

- debian desktop environment: 桌面，我当服务器用就不需要了，取消勾选
- 选择 ssh server 和 standard system utilities

等待安装完成，重启。
