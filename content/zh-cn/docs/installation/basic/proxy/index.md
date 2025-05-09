---
title: "网络代理"
linkTitle: "网络代理"
date: 2024-01-18
weight: 90
description: >
  设置网络代理
---

### 设置网络代理

我通常在 openwrt 上安装 openclash，支持 socks5 和 http 代理，因此设置很简单:

```bash
vi ~/.zshrc
```

加入以下内容：

```bash
# set proxy for different locations
alias proxyon-nansha='export all_proxy=socks5://192.168.0.1:7891;export http_proxy=http://192.168.0.1:7890;export https_proxy=http://192.168.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-tianhe='export all_proxy=socks5://192.168.2.1:7891;export http_proxy=http://192.168.2.1:7890;export https_proxy=http://192.168.2.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-fenhu='export all_proxy=socks5://192.168.3.1:7891;export http_proxy=http://192.168.3.1:7890;export https_proxy=http://192.168.3.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-qingpu='export all_proxy=socks5://192.168.5.1:7891;export http_proxy=http://192.168.5.1:7890;export https_proxy=http://192.168.5.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
# set default proxy by this line
alias proxyon='proxyon-nansha'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
# uncomment next line to enable proxy by default when zsh is opened
# proxyon
```

### 开启网络代理

需要开启代理时，输入 proxyon 命令就设置好代理了。不使用时 proxyoff 关闭。

### 为 sudo 开启代理

出于安全考虑，sudo 默认会重置环境变量（如 http_proxy、https_proxy），防止用户通过环境变量传递潜在的危险参数或越权操作，然后就导致前面的代理设置（本质上是设置环境变量 http_proxy、https_proxy）失效。

但有时我们又不得不使用 sudo 来执行某些需要使用代理的命令，这时就需要为 sudo 开启代理。

此时可以在 sudo 命令前加上 `-E` 参数，表示保留当前用户的环境变量，如：

```bash
sudo -E apt-get update

# 安装 docker 时有时会遇到 download.docker.com 的连接问题，此时可以设置代理
sudo -E apt-get install docker-ce
```

如果想永久生效，可以修改 sudo 的配置文件：

```bash
sudo visudo
```

在文件中找到下面的内容：

```bash
# This preserves proxy settings from user environments of root
# equivalent users (group sudo)
#Defaults:%sudo env_keep += "http_proxy https_proxy ftp_proxy all_proxy no_proxy"
```

将 `#Defaults:%sudo env_keep` 这一行的注释去掉，然后 `contrl + o` 保存退出（备注：这个文件的名字就是 /etc/sudoers.tmp ，不要以为是保存错了）。

此时再使用 sudo 时，就会自动保留当前用户的环境变量，不需要再收工加 -E 参数。


