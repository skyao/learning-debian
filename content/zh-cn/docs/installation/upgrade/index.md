---
title: "升级"
linkTitle: "升级"
date: 2024-01-18
weight: 300
description: >
  升级 debian 大版本
---

背景：购买的某云服务器，默认安装的是 debian11，现在需要升级到 debian13。由于 debian 不支持跨大版本升级，所以需要先升级到 debian12，再从 debian12 升级到 debian13。

## debian11 升级

Debian 11 Bullseye -> Debian 12 Bookworm

### 更新 debian11

确保当前的 Debian 11 已经是最新状态，减少冲突。

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
```

完成后，重启机器：

```bash
sudo reboot
```

如果遇到报错：

```bash
Err:5 http://deb.debian.org/debian bullseye-backports Release
404  Not Found [IP: 151.101.194.132 80]
Err:6 http://security.debian.org bullseye/updates Release
404  Not Found [IP: 151.101.66.132 80]
Reading package lists... Done
E: The repository 'http://deb.debian.org/debian bullseye-backports Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'http://security.debian.org bullseye/updates Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

修改源文件：

```bash
sudo vi /etc/apt/sources.list
```

删除原来的内容，替换为：

```bash
deb http://deb.debian.org/debian bullseye main contrib non-free
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb http://security.debian.org/debian-security bullseye-security main contrib non-free
```

### 修改源

修改源文件：

```bash
sudo vi /etc/apt/sources.list
```

将原有的内容删除，替换为：

```bash
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

### 升级

```bash
sudo apt update

# 只升级那些不需要删除其他包就能升级的软件，减少冲突
sudo apt upgrade --without-new-pkgs -y

# 更新内核、核心库，并处理复杂的依赖关系
sudo apt full-upgrade -y
sudo apt autoremove -y
```

在执行 full-upgrade 的过程中，屏幕会停下来询问问题，请留意：

1. apt-listchanges（更新日志展示）：

   如果屏幕显示一大段更新日志，按 q 键退出即可继续。

2. Restart services during package upgrades?（是否在升级时重启服务）

   强烈建议选 Yes。这样系统会自动重启服务，不需要你手动干预。

3. Configuration file conflict（配置文件冲突）

   系统会提示你：某配置文件修改过，现在有新版本，问怎么办。

   选项通常有：

   - install the package maintainer's version（安装新版）
   - keep the local version currently installed（保留你现在的版本）
   - show the differences（查看差异）

   建议：通常选择 keep the local version (保留当前版本) 是最安全的。如果系统比较干净，可以选择 install the package maintainer's version（安装新版）。我这次升级选 install the package maintainer's version 顺利升级成功。

更新完成后，重启：

```bash
sudo reboot
```

再查看 debian 版本：

```bash
cat /etc/debian_version

# 显示 12.13 ，说明升级成功
```

### 清理

升级完成后，清理一下不需要的包：

```bash
sudo apt autoremove -y
```

清理不需要的内核：

```bash
$ uname -r
# 显示 6.1.xxx, 说明内核已经升级到 6.1 内核, 可以安全的删除旧的 5.10 内核了
```

查看当前安装的内核：

```bash
dpkg --list | grep linux-image
# 输出类似：
rc  linux-image-5.10.0-14-cloud-amd64 5.10.113-1                           amd64        Linux 5.10 for x86-64 cloud (signed)
rc  linux-image-5.10.0-32-cloud-amd64 5.10.223-1                           amd64        Linux 5.10 for x86-64 cloud (signed)
ii  linux-image-6.1.0-43-cloud-amd64  6.1.162-1                            amd64        Linux 6.1 for x86-64 cloud (signed)
ii  linux-image-cloud-amd64           6.1.162-1                            amd64        Linux for x86-64 cloud (meta-package)
```

清理旧内核：

```bash
sudo apt purge linux-image-5.10.0-14-cloud-amd64 linux-image-5.10.0-32-cloud-amd64 -y
# 使用通配符清理所有残留配置
sudo apt purge ~c -y
```

重启：

```bash
sudo reboot
```

至此，debian11 升级 debian12 完成。内核从默认的 5.10 升级到 6.1 内核。

## debian12 升级到 debian13

继续升级到 debian13，注意安全起见，先重启再升级。

```bash
sudo reboot
```

### 修改源

修改源文件：

```bash
sudo vi /etc/apt/sources.list
```

将原有的内容删除，替换为：

```bash
deb http://deb.debian.org/debian trixie main contrib non-free non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
```

### 升级

```bash
sudo apt update

# 只升级那些不需要删除其他包就能升级的软件，减少冲突
sudo apt upgrade --without-new-pkgs -y

# 更新内核、核心库，并处理复杂的依赖关系
sudo apt full-upgrade -y
sudo apt autoremove -y
```

交互问题提示同上，请留意。

升级完成后，重启：

```bash
sudo reboot
``
   
重启后，查看 debian 版本：

```bash
cat /etc/debian_version
13.3
# 显示 13.3 ，说明升级成功
```

### 清理

升级完成后，清理一下不需要的包：

```bash
sudo apt autoremove -y
```

清理不需要的内核：

```bash
$ uname -r

6.12.69+deb13-cloud-amd64
# 显示 6.12.xxx, 说明内核已经升级到 6.12 内核, 可以安全的删除旧的内核了
```

查看当前安装的内核：

```bash
dpkg --list | grep linux-image
ii  linux-image-6.1.0-43-cloud-amd64      6.1.162-1                            amd64        Linux 6.1 for x86-64 cloud (signed)
ii  linux-image-6.12.69+deb13-cloud-amd64 6.12.69-1                            amd64        Linux 6.12 for x86-64 cloud (signed)
ii  linux-image-cloud-amd64               6.12.69-1                            amd64        Linux for x86-64 cloud (meta-package)
```

清理旧内核：

```bash
sudo apt purge linux-image-6.1.0-43-cloud-amd64 -y
# 使用通配符清理所有残留配置
sudo apt purge ~c -y
```

刷新引导菜单后重启：

```bash
sudo update-grub
sudo reboot
```

再查看当前安装的内核：

```bash
dpkg --list | grep linux-image

ii  linux-image-6.12.69+deb13-cloud-amd64   6.12.69-1                            amd64        Linux 6.12 for x86-64 cloud (signed)
ii  linux-image-cloud-amd64                 6.12.69-1                            amd64        Linux for x86-64 cloud (meta-package)
```

至此，debian12 升级 debian13 完成。内核从默认的 6.1 升级到 6.12 内核。


