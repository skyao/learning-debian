---
title: "path修改"
linkTitle: "path修改"
date: 2024-01-18
weight: 20
description: >
  修改默认path路径避免找不到命令的错误
---

### 问题描述

debian 12 默认的 PATH 路径有点少：

```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/games
```

很多常见的命令都会因为不包含在 PATH 中而无法使用，报错 "command not found":

```bash
$ usermod -aG sudo sky
bash: usermod: command not found
```

打开 `/etc/profile` 文件看到：

```bash
if [ "$(id -u)" -eq 0 ]; then
  PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
else
  PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games"
fi
export PATH
```

看上去像是没有自动 source  `/etc/profile`，比如我用 su 到 root 账号后执行 source，那就正常的得到上面的 PATH:

```bash
$ su root
Password: 
$ id
uid=0(root) gid=0(root) groups=0(root)
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/games
$ source /etc/profile
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### 普通账号的修复

对于普通账号，比如我安装时建立的 sky 账号，只要简单在 .zshrc 中加入:

```
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH
export PATH=$HOME/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

重新登录 sky 账号，验证即可:

```bash
$ echo $PATH

/home/sky/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

> 这个需要先安装 zsh 和 oh-my-zsh，bash 应该类似，只是我目前基本只用 zsh。

### root 账号的修复

貌似没有找到解决不自动 source `/etc/profile` 的办法，所以只能手工执行 

```bash
source /etc/profile
```

来解决。或者，类似 sky 账号那样，安装 zsh 和 oh-my-zsh，但发现 root 账号不能改成默认使用 zsh，只能在 su 到 root 账号之后，手工执行 `zsh` 来从 bash 换到 zsh。
