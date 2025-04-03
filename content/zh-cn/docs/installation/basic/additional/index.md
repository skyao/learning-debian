---
title: "安装其他软件"
linkTitle: "其他软件"
date: 2024-01-18
weight: 100
description: >
  安装其他软件
---

## 系统类

### linux-headers

安装 dkms 和用于 pve 的 linux-headers：

```bash
sudo apt install -y gcc make dkms
sudo apt install -y pve-headers-$(uname -r)
sudo apt install --fix-broken
```

## 工具类

```bash
sudo apt install htop unzip zip curl
```

### 修复 locale 报错

默认安装之后经常在执行各种命令时看到如下的警告：

```bash
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

可以通过执行 `locale` 命令来重现这个警告：

解决方案，

```bash
vi ~/.zshrc
```

增加内容：

```bash
export LC_CTYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

执行

```bash
source ~/.zshrc
```

验证结果。

参考：

- https://orcacore.com/system-locale-setup-debian-12-bookworm-command-line/

## 网络类

### iperf

```bash
sudo apt install -y net-tools iperf iperf3
```

Iperf3 安装时会询问是否系统服务（自动启动），选择 yes，这样方便需要时排查网络。

### sftp server

pve 默认是关闭 sftp 的，需要手动开启：

```bash
sudo vi /etc/ssh/sshd_config
```

找到 `Subsystem sftp /usr/lib/openssh/sftp-server` 这一行，注释掉，然后新加一行内容：

```bash
# Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

重启 ssh 服务：

```bash
sudo /etc/init.d/ssh restart
```

之后用支持 sftp 的客户端连接即可

### nfs client

```bash
sudo apt install nfs-common
```

