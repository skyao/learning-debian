---
title: "创建模板"
linkTitle: "创建模板"
date: 2025-10-26
weight: 10
description: >
  创建基于 debian13 的 docker 模板
---

## 制作过程

### 准备虚拟机

从模版 template-debian13-basic-vxx （取最新版本） full clone 克隆一个虚拟机，命名为 template-debian13-docker-vxx，VM ID 为 9903xx.

### 搭建开发环境

#### 安装 docker

- docker/docker-compose: https://skyao.net/learning-docker/docs/installation/debian13/
- kubectl

### 地域迁移

当迁移到其他地域时，需要修改的设置除了有 basic 的改动之外，还有 devserver9x 相关的配置：

1. docker 使用的 habor 代理 ： https://skyao.net/learning-docker/docs/repository/harbor/transparent-proxy/
