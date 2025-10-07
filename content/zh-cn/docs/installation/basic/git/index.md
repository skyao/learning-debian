---
title: "git"
linkTitle: "git"
date: 2024-09-16
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
git version 2.39.5
```

配置代理

```bash
mkdir -p ~/.ssh/
vi ~/.ssh/config
```

增加内容为：

```properties
# for  github
Host github.com
    HostName github.com
    User git
    # for HTTP proxy
    #ProxyCommand socat - PROXY:192.168.0.1:%h:%p,proxyport=7890
    #ProxyCommand socat - PROXY:192.168.2.1:%h:%p,proxyport=7890
    #ProxyCommand socat - PROXY:192.168.3.1:%h:%p,proxyport=7890
    #ProxyCommand socat - PROXY:192.168.5.1:%h:%p,proxyport=7890
    # for socks5 proxy
    #ProxyCommand nc -v -x 192.168.0.1:7891 %h %p
    #ProxyCommand nc -v -x 192.168.2.1:7891 %h %p
    #ProxyCommand nc -v -x 192.168.3.1:7891 %h %p
    #ProxyCommand nc -v -x 192.168.5.1:7891 %h %p
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
```

其他设置参考： https://skyao.net/learning-git/docs/installation/initial/