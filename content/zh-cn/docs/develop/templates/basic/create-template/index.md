---
title: "构建 basic 模板"
linkTitle: "创建模板"
date: 2025-05-07
weight: 10
description: >
  创建debian 12 基础模板
---

## 制作过程

参考 [debian12 学习笔记](../../../installation/pve) 的 pve 安装文档， 安装 debian 12 操作系统。

然后完成基本配置，内核配置，并通过 timeshift 进行系统备份。

### 模板内容

- apt 升级到 debian 12.10
- 更新 apt 到最新内核 6.1.0-34
- 安装 timeshift, zsh/ohmyzsh, git, dkms, iperf/iperf3, unzip zip curl
- 添加 proxyon/proxyoff alias for different locations
- 修复 path 不足问题，包括用户 root 和 sky
- 安装 sftp server, nfs server 和 client, samba server



