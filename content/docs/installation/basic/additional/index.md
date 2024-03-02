---
title: "安装其他软件"
linkTitle: "其他软件"
date: 2024-01-18
weight: 100
description: >
  安装其他软件
---

### 安装操作系统相关的软件

```bash
sudo apt install dkms
```

这个过程中会自动安装 linux-headers 。

### 常用工具软件

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
