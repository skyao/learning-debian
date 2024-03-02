---
title: "网络代理"
linkTitle: "网络代理"
date: 2024-01-18
weight: 90
description: >
  设置网络代理
---

## 设置网络代理

在 openwrt 上安装 openclash，支持 socks5 和 http 代理，因此设置很简单，在 `.zshrc` 中加入以下内容：

```bash
# proxy
alias proxyon='export all_proxy=socks5://192.168.20.1:7891;export http_proxy=http://192.168.20.1:7890;export https_proxy=http://192.168.20.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```

需要开启代理时，输入 proxyon 命令就设置好代理了。不使用时 proxyoff 关闭。

## 安装配置 privoxy

但在 tplink 6088 路由器上安装的 openwrt 上没有 openclash，只有 ShadowSocksR Plus+ , 只支持 socks5 代理。之前还能在 openwrt 上通过安装 privoxy 软件来将 socks5 转为 http，但最近 openwrt 上的 privoxy 软件已经没有了。因此需要自己在电脑上安装 privoxy 软件来进行转换。

https://www.privoxy.org/

```bash
sudo apt install privoxy
```

修改配置前备份一下：

```bash
 sudo cp /etc/privoxy/config /etc/privoxy/config.original
```

修改配置内容为：

```properties
# 这里修改原有的配置
listen-address  0.0.0.0:7890
listen-address  [::1]:7890

# 在最后加入这行
forward-socks5t   /               192.168.0.1:7891 .
```

重启 privoxy：

```bash
sudo systemctl restart privoxy.service
```

验证，

```bash
export http_proxy=http://127.0.0.1:7890;export https_proxy=http://127.0.0.1:7890
wget https://www.youtube.com/
```

最后，在 `.zshrc` 中加入以下内容，将 http/https 代理设置为本地 privoxy 的端口：

```bash
# proxy
alias proxyon='export all_proxy=socks5://192.168.20.1:7891;export http_proxy=http://127.0.0.1:7890;export https_proxy=http://127.0.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```

参考：

- https://reintech.io/blog/setting-up-web-proxy-privoxy-debian-12