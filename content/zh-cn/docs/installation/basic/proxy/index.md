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
alias proxyon-shanghai='export all_proxy=socks5://192.168.3.1:7891;export http_proxy=http://192.168.3.1:7890;export https_proxy=http://192.168.3.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyon-cudy='export all_proxy=socks5://192.168.5.1:7891;export http_proxy=http://192.168.5.1:7890;export https_proxy=http://192.168.5.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
# set default proxy by this line
alias proxyon='proxyon-nansha'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
# uncomment next line to enable proxy by default when zsh is opened
# proxyon
```

### 开启网络代理

需要开启代理时，输入 proxyon 命令就设置好代理了。不使用时 proxyoff 关闭。
