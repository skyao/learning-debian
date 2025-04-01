---
title: "准备工作"
linkTitle: "准备工作"
date: 2025-03-31
weight: 10
description: >
  准备工作：虚拟机，通用工具，ssh 证书
---

## 准备虚拟机

从模版 template-debian12-basic-v01 （取最新版本） 克隆一个虚拟机，命名为 template-debian12-dev-v01，VM ID 为 990201.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 32g（mini 8192，memory 32768）。

### 准备 ssh 证书

从其他开发机器复制 ssh 证书过来，或者重新生成一份：


## 准备通用工具

```bash
sudo apt install unzip zip
