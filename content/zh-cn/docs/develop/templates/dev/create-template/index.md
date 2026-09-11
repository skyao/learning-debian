---
title: "创建模板"
linkTitle: "创建模板"
date: 2025-10-26
weight: 10
description: >
  创建基于debian12/13 的 dev 开发模板
---

## 制作过程

以 debian 13 为例。

### 准备虚拟机

从模版 template-debian13-basic-vxx （取最新版本） full clone 克隆一个虚拟机，命名为 template-debian13-dev-v01，VM ID 为 990101.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 8 核，内存 32g（mini 8192，memory 32768）。

### 准备 ssh 证书

重新生成一份 ssh 证书，这个是要提交给 github 的，单独用一份。

### 搭建开发环境

#### 安装 docker

- docker/docker-compose: https://skyao.net/learning-docker/docs/installation/debian13/
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

### 地域迁移

当迁移到其他地域时，需要修改的设置除了有 basic 的改动之外，还有 devserver9x 相关的配置：

1. docker 使用的 habor 代理 ： https://skyao.net/learning-docker/docs/repository/harbor/transparent-proxy/

2. java 的 maven 配置： https://skyao.net/learning-debian/docs/develop/langurage/java/maven/

3. golang 的 goproxy 配置： https://skyao.net/learning-golang/docs/develop/installation/settings/

4. rust 的 cargo 代理源配置： ttps://skyao.net/learning-rust/docs/installation/cargo/

5. python 的 pip 代理配置： https://skyao.net/learning-python/docs/installation/pip/

6. nodejs 的 代理配置： https://skyao.net/learning-debian/docs/develop/langurage/nodejs/

