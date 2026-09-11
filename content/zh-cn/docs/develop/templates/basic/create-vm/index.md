---
title: "创建虚拟机"
linkTitle: "创建虚拟机"
date: 2025-10-26
weight: 20
description: >
  创建使用 debian 13 基础模板的虚拟机
---

## 创建虚拟机

### 临时使用

为了加速创建，可以先将模板克隆到本地，然后从本地的模板中采用 Linked Clone 方式克隆。

这样从模板 clone 虚拟机， 速度会非常快，而且空间占用也非常小。

使用完成后删除即可。因为不需要长期使用和维护，因此不存在多版本问题。每次 Linked Clone 方式克隆最快最不占用磁盘空间。

### 长期使用

长期使用时，还是采用 full clone 的方式比较好，后续可以单独为这台虚拟机进行 apt upgrade。

Linked Clone 方式克隆最大的麻烦是无法跨 template 升级，因此如果 template 有更新，就会出现版本割裂。

## 修改虚拟机配置

主要是修改虚拟机的 hostname，以及必要时从使用 dhcp 自动获取 ip 地址修改为使用静态 ip 地址。





