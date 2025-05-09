---
title: "创建模板"
linkTitle: "创建模板"
date: 2025-05-08
weight: 10
description: >
  创建基于debian12 的 dev 开发模板
---

## 制作过程

### 准备虚拟机

从模版 template-debian12-basic-v03/v04 （取最新版本） 克隆一个虚拟机，命名为 template-debian12-dev-v01，VM ID 为 990201.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 32g（mini 8192，memory 32768）。

### 准备 ssh 证书

重新生成一份 ssh 证书，这个是要提交给 github 的，单独用一份。


### 搭建开发环境

#### 安装 docker

- docker/docker-compose: https://skyao.io/learning-docker/docs/installation/debian12/
- kubectl

#### 安装开发工具

参考本读书笔记中的 [开发工具](../../../tools/) 一节, 安装开发工具:

- sdkman

#### 语言 sdk 和 nexus 私库

参考本读书笔记中的 [编程语言](../../../langurage/) 一节, 安装语言 sdk 和对应的 nexus 私库:

- Java： 包括 maven
- golang
- rust
- python
- nodejs



