---
title: "创建模板"
linkTitle: "创建模板"
date: 2025-10-26
weight: 10
description: >
  创建基于debian12/13 的 dev 开发模板
---

## 制作过程-v1

以 debian 13 为例。

### 准备虚拟机

从模版 template-debian13-basic-v01 （取最新版本） full clone 克隆一个虚拟机，命名为 template-debian13-dev-v01，VM ID 为 990101.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 32g（mini 8192，memory 32768）。

### 准备 ssh 证书

重新生成一份 ssh 证书，这个是要提交给 github 的，单独用一份。

### 搭建开发环境

#### 安装 docker

- docker/docker-compose: https://skyao.net/learning-docker/docs/installation/debian12/
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

## 制作过程-v2

v2要放到广州南沙的开发环境， 网段是 192.168.0.0/24， devserver92 的 ip 是 192.168.0.92，因此所有相关的 ip 信息都要修改。

将 dev-v1 的模板传送到广州南沙，然后在这个基础上，按照上面制作 dev-v1 的流程，重头走一边制作流程，注意需要修改 ip 地址的地方，就可以完成 dev-v2 的制作。

