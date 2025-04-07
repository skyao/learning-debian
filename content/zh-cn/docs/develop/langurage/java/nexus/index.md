---
title: "Nexus"
linkTitle: "Nexus"
date: 2025-04-03
weight: 10
description: >
  安装和配置 Nexus
---

## 下载

假定 nexus 安装路径为 `/mnt/data/nexus`，准备目录：

```bash
sudo mkdir -p /mnt/data/nexus
cd /mnt/data/nexus
```

从下面地址找到 nexus 安装包：

https://help.sonatype.com/en/download-archives---repository-manager-3.html

从官方下载 nexus 安装包 nexus-unix-x86-64-3.79.0-09.tar.gz：

```bash
sudo wget https://download.sonatype.com/nexus/oss/nexus-3.79.0-09-unix.tar.gz
```

解压安装包：

```bash
sudo tar -zxvf nexus-3.79.0-09-unix.tar.gz
```

得到如下目录：

```bash
$ ls
nexus-3.79.0-09  sonatype-work
```

重命名 nexus 目录去掉版本信息：

```bash
sudo mv nexus-3.79.0-09 nexus
```

## 安装

参考官方文档：

https://help.sonatype.com/en/install-nexus-repository.html

新建两个目录以存放 nexus 数据和日志：

```bash
sudo mkdir -p /mnt/data/nexus/data
sudo mkdir -p /mnt/data/nexus/logs
```

此时目录结构如下：

```bash
$ ls
data  logs  nexus  sonatype-work

$ pwd
/mnt/data/nexus
```

修改 nexus 配置文件以设置数据和日志目录：

```bash

```





