---
title: "准备工作"
linkTitle: "准备工作"
date: 2024-01-18
weight: 10
description: >
  搭建编程开发环境的准备工作
---

## 准备虚拟机

从模版 template-debian12.4-basic 克隆一个虚拟机，命名为 dev101.

开发需要的 cpu 和内存稍大，修改虚拟机参数，cpu 修改为 16核，内存 32g。

### 基本配置修改

修改 hostname 和 使用静态 IP 地址。

### 准备 ssh 证书

从其他开发机器复制 ssh 证书过来，或者重新生成一份：

```bash
scp id_rsa.pub sky@192.168.0.101:/home/sky/.ssh/
scp id_rsa sky@192.168.0.101:/home/sky/.ssh/
```

## 准备通用工具

### 安装 sdkman

参考： 

- https://skyao.io/learning-ubuntu-server/docs/development/common/sdkman.html
- https://skyao.io/learning-macos/docs/programing/common/sdkman.html

```bash
sudo apt install unzip zip
curl -s "https://get.sdkman.io" | bash

source "/home/sky/.sdkman/bin/sdkman-init.sh"
sdk version
```


 sdk install java 17.0.10-zulu