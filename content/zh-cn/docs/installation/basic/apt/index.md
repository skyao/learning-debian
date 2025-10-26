---
title: "apt更新"
linkTitle: "apt更新"
date: 2025-10-26
weight: 110
description: >
  设置 apt 更新源并更新系统
---

安装时已经设置好了国内的镜像源。

### 移除  CD-ROM 源

但直接执行 apt update 时会报错：

```bash
$apt update

Ign:1 cdrom://[Debian GNU/Linux 13.1.0 _Trixie_ - Official amd64 DVD Binary-1 with firmware 20250906-10:24] trixie InRelease
Err:2 cdrom://[Debian GNU/Linux 13.1.0 _Trixie_ - Official amd64 DVD Binary-1 with firmware 20250906-10:24] trixie Release
  Please use apt-cdrom to make this CD-ROM recognized by APT. apt-get update cannot be used to add new CD-ROMs
Hit:3 http://mirrors.ustc.edu.cn/debian trixie InRelease
Hit:4 http://mirrors.ustc.edu.cn/debian trixie-updates InRelease
Hit:5 http://security.debian.org/debian-security trixie-security InRelease
Error: The repository 'cdrom://[Debian GNU/Linux 13.1.0 _Trixie_ - Official amd64 DVD Binary-1 with firmware 20250906-10:24] trixie Release' does not have a Release file.
Notice: Updating from such a repository can't be done securely, and is therefore disabled by default.
Notice: See apt-secure(8) manpage for repository creation and user configuration details.
```

```bash
sudo vi /etc/apt/sources.list
```

注释掉下面的这一行：

```properties
deb cdrom:[Debian GNU/Linux 13.1.0 _Trixie_ - Official amd64 DVD Binary-1 with firmware 20250906-10:24]/ trixie contrib main non-free-firmware
```