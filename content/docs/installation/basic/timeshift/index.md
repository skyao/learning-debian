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

https://skyao.io/learning-ubuntu-server/docs/installation/timeshift.html

## 安装

```bash
apt install timeshift
```

## 配置

找到 timeshift 分区的 UUID：

```bash
lsblk -f
```

d5aaad4b-8382-4153-adfc-f7c797e74ee5

然后设置 backup_device_uuid：

```bash
vi /etc/timeshift/default.json
```

验证一下：

```bash
timeshift --list
```

## 创建快照

```bash
timeshift --create --comments "first backup after install"
```

第一次备份大概要消耗2.3g的存储空间。