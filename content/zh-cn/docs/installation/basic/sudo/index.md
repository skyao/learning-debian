---
title: "sudo设置"
linkTitle: "sudo"
date: 2024-01-18
weight: 30
description: >
  增加账号的sudo权限
---

## 安装

先 su 到 root 账号，安装 sudo：

```bash
apt install sudo
```

## 问题

安装时默认的 sky 账号是没有 sudo 权限的：

```bash
$ id
uid=1000(sky) gid=1000(sky) groups=1000(sky),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)

$ sudo ls
[sudo] password for sky: 
sky is not in the sudoers file.
```

## 解决方案

先 su 为 root，然后为 sky 账号加入 sudo：

```bash
# 如果没有修复 path 的问题，则会报错找不到 usermod
# export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
usermod -aG sudo sky
```

退出 sky 账号，再次登录就可以 sudo 了:

```bash
$ ls                                                                                      
[sudo] password for sky: 
bin  debian12  temp  work
$ id
uid=1000(sky) gid=1000(sky) groups=1000(sky),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
```

## 问题2

在某些 debian 12 上（比如腾讯云），即使使用 usermod 加入了 sudo 组，仍然不能使用 sudo：

```bash
$ sudo ls 
[sudo] password for sky: 
sky is not in the sudoers file.
$ id
uid=1003(sky) gid=1003(sky) groups=1003(sky),27(sudo),100(users)
```

解决的办法是用 root 帐号执行

```bash
visudo
```

命令，然后添加下面这样内容：

```bash
%sudo ALL=(ALL:ALL) ALL
```

保存退出，然后 sky 帐号重新登录可以使用 sudo 。