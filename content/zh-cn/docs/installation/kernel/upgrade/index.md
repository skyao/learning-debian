---
title: "更新内核"
linkTitle: "更新内核"
weight: 20
description: >
  更新 debian 核心
---

## 自动更新内核

可以单独更新 linux 内核：

```bash
$ sudo apt update
$ sudo apt install linux-image-amd64
```

也可以使用 `apt upgrade` 更新所有内容，期间如果有内核更新则自动安装：

```bash
$ sudo apt update
$ sudo apt upgrade
```

这个方式会自动更新 linux 内核，但是是安装 debian 12 的设定，比如目前即使是 debian 12.9 版本，也只会使用 linux 6.1.x 内核，不会使用更新的内核版本。


## 手工更新内核

如果要更新为比默认的 linux 6.1.x 更新的内核版本，则需要手工更新内核。

```bash
apt list -a linux-image-amd64
```

输出为：

```bash
Listing... Done
linux-image-amd64/stable-backports 6.12.12-1~bpo12+1 amd64
linux-image-amd64/stable-security 6.1.133-1 amd64
linux-image-amd64/stable 6.1.129-1 amd64
```

可以看到有多个版本，通常自动更新时只会选择最新的 stable 或者 stable-security 版本，如果需要选择其他版本，则需要手工更新。

比如这里的 stable-backports 6.12.12-1~bpo12+1 版本，手工安装命令为：

```bash
sudo apt install -t stable-backports linux-image-amd64=6.12.12-1~bpo12+1
```

这里的 `-t stable-backports` 参数表示从 stable-backports 仓库安装软件包，`linux-image-amd64=6.12.12-1~bpo12+1` 指定了要安装的内核版本。

安装完成后，重启系统：

```bash
sudo reboot
```

重启后，查看内核版本：

```bash
uname -r
```

输出为：

```bash
6.12.12+bpo-amd64
```

可以看到已经更新为 6.12.12 版本。