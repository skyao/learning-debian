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
sudo apt install -y linux-headers-$(uname -r)
sudo apt install --fix-broken
```

## 工具类

```bash
sudo apt install -y htop unzip zip curl
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

可以通过执行 `locale` 命令来重现这个警告. 另外有时编辑文件时, 如果输入中文, 会出现乱码, 一般也是因为这里的 locale 设置有问题. 解决之后就不会中文乱码了.

最简单的修改方案：

```bash
vi /etc/default/locale
```

增加内容：

```bash
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```

重启.

## 网络类

### iperf

```bash
sudo apt install -y net-tools iperf iperf3
```

Iperf3 安装时会询问是否系统服务（自动启动），选择 yes，这样方便需要时排查网络。

### nc

debian 12/13 自带的 nc 是 netcat-traditional 包提供的版本:

```bash
$ nc -h 

[v1.10-47]
connect to somewhere:	nc [-options] hostname port[s] [ports] ... 
listen for inbound:	nc -l -p port [-options] [hostname] [port]
```

而我一直在 ubuntu / linux mint 中用的 nc 是 netcat-openbsd 包提供的版本:

```bash
$ nc -h    

OpenBSD netcat (Debian patchlevel 1.226-1ubuntu2)
usage: nc [-46CDdFhklNnrStUuvZz] [-I length] [-i interval] [-M ttl]
	  [-m minttl] [-O length] [-P proxy_username] [-p source_port]
	  [-q seconds] [-s sourceaddr] [-T keyword] [-V rtable] [-W recvlimit]
	  [-w timeout] [-X proxy_protocol] [-x proxy_address[:port]]
	  [destination] [port]
```

而我在设置 git 代理时，通常会使用 `nc` 来设置，类似：

```bash
ProxyCommand nc -v -x 192.168.3.1:7891 %h %p
```

这个命令在 netcat-traditional 下会报错：

```bash
nc: invalid option -- 'x'
nc -h for help
```

因此需要将默认的 `netcat-traditional` 包替换为 `netcat-openbsd` 包：

```bash
sudo apt remove -y netcat-traditional
sudo apt install -y netcat-openbsd
```

### 安装 socat

debian12/13 下没有 socat 命令（用于 git http代理），需要安装：

```bash
sudo apt install -y socat
```

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

### smb server

安装 samba 服务：

```bash
sudo apt install samba
```

假设要共享的目录是 `/mnt/data/shared`， 则需要先创建该目录：

```bash
sudo mkdir -p /mnt/data/shared
```

设置目录权限：

```bash
sudo chmod -R 777 /mnt/data/shared
```

设置目录所有者：

```bash
sudo chown -R nobody:nogroup /mnt/data/shared
```

配置 samba 服务：

```bash
sudo vi /etc/samba/smb.conf
```

在文件末尾添加以下内容:

```properties
[shared]
   comment = Shared Folder
   path = /mnt/data/shared
   browseable = yes
   read only = no
   guest ok = yes 
   create mask = 0777
   directory mask = 0777
```

并删除这个文件中 [home] 和 [printers] 这两段内容，否则在smb 共享文件列表中会出现 nobody 和 printers 两个目录。

启动 samba 服务并设置为开机启动：

```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

之后就可以访问了，路径为：

- linux 系统：`smb://192.168.3.175`
- windows 系统：`\\192.168.3.175`



