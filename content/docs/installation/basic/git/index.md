---
title: "git"
linkTitle: "git"
date: 2024-01-18
weight: 40
description: >
  安装配置 git
---

直接 apt 安装：

```bash
apt install git
```

检查版本:

```bash
$ git version
git version 2.39.2
```

配置代理

```bash
vi ~/.ssh/config
```

增加内容为：

```properties
Host github.com
HostName github.com
User git
# http proxy
#ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=3333
# socks5 proxy
ProxyCommand nc -v -x 192.168.0.1:7891 %h %p
```