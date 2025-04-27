---
title: "查看内核版本"
linkTitle: "查看内核"
weight: 10
description: >
  查看 debian 核心版本信息
---


查看当前 debian 内核的版本：

```bash
$ uname -r
6.1.0-31-amd64
$ uname -v
#1 SMP PREEMPT_DYNAMIC Debian 6.1.128-1 (2025-02-07)
$ uname -a
Linux debian12 6.1.0-31-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.128-1 (2025-02-07) x86_64 GNU/Linux
```

这说明安装时的内核版本为 6.1.0-31-amd64，当前已经更新到 6.1.0-31-amd64 版本（debian 12.9，更新时间 2025-02-07）。

其他方式：

```bash
$ cat /proc/version
Linux version 6.1.0-31-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.128-1 (2025-02-07)
```

dmesg 命令：

```bash
$ sudo dmesg | grep Linux
[    0.000000] Linux version 6.1.0-31-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.128-1 (2025-02-07)
......
```

dpkg 命令：
```bash
$ dpkg --list | grep linux-image
ii  linux-image-6.1.0-31-amd64      6.1.128-1                       amd64        Linux 6.1 for 64-bit PCs (signed)
ii  linux-image-amd64               6.1.128-1                       amd64        Linux for 64-bit PCs (meta-package)

$ dpkg --list | grep -Ei 'linux-image|linux-headers|linux-modules'
ii  linux-image-6.1.0-31-amd64      6.1.128-1                       amd64        Linux 6.1 for 64-bit PCs (signed)
ii  linux-image-amd64               6.1.128-1                       amd64        Linux for 64-bit PCs (meta-package)
```
