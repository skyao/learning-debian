---
title: "创建虚拟机"
linkTitle: "创建虚拟机"
date: 2025-05-13
weight: 20
description: >
  创建使用 k8s 模板的虚拟机
---

## 创建虚拟机

从模板 template-debian12-k8s-v01 克隆一个虚拟机，命名为 k8sxxx。

## 配置虚拟机

参考： [克隆后的配置](../../../installation/clone/) 一节

修改hostname 为 k8sxxx，设置 ip 地址为固定ip。

重启。

## 搭建k8s

```bash
wd k8s

sudo ./install_k8s_prewarm.zsh 192.168.3.112
```

安装完成之后记得 timeshift 备份：

```bash
sudo timeshift --create --comments "setup k8s to be host k8s112"
```



