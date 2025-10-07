---
title: "创建模板"
linkTitle: "创建模板"
date: 2025-05-13
weight: 10
description: >
  创建 debian 12 k8s 模板
---

## 准备工作

### 准备虚拟机

从模版 template-debian12-basic-v03 （取最新版本） 克隆一个虚拟机，命名为 template-debian12-k8s-v01，VM ID 为 990401.

k8s 需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 16g（mini 8192，memory 16384）。

## 搭建k8s

目前采用的是预热安装的方式，详细参考：

https://skyao.net/learning-kubernetes/docs/installation/kubeadm/debian12/prewarm/

