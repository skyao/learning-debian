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

