---
title: "clash"
linkTitle: "clash"
date: 2025-04-07
weight: 70
description: >
  clash 科学上网软件
---

### 下载

原网站 https://github.com/Dreamacro/clash/ 网站已经无法访问。

> https://clashcn.com/clash-releases 这里有一些备份，待验证。

在这里找到一个备份：

https://github.com/moshaoli688/clash/releases

下载 clash-linux-amd64-v1.18.0.gz：

```bash
wget https://github.com/moshaoli688/clash/releases/download/v1.18.0/clash-linux-amd64-v1.18.0.gz
```

### 安装

解压后将 clash 文件移动到 /usr/local/bin/clash :

```bash
gunzip clash-linux-amd64-v1.18.0.gz
chmod +x clash-linux-amd64-v1.18.0

sudo mv clash-linux-amd64-v1.18.0 /usr/local/bin/clash
```

检验：

```bash
$ clash -v      

Clash v1.18.0 linux amd64 with go1.21.5 Wed Dec 27 13:02:19 UTC 2023
```

### 初始化

初始化配置文件：

```bash
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download              
FATA[0000] Initial configuration directory error: can't initial MMDB: can't download MMDB: Get "https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb": read tcp 10.1.20.3:57604->8.7.198.46:443: read: connection reset by peer 
```

mmdb 是 ip 数据库文件，需要下载，但是这个地址很容易被墙导致无法下载，可以多试几次。或者用其他方式将这个 https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb 下载下来，然后放到 `～/.config/clash` 目录下。

顺利下载完成后，clash 就能启动了：

```bash
$ clash
INFO[0000] inbound mixed://127.0.0.1:7890 create success. 
```

### 配置

我这个只是给服务器零时用一下，因此配置很简单

```yaml
vi ~/.config/clash/config.yaml
```

配置文件内容：

```yaml
mixed-port: 7890
allow-lan: false
bind-address: "0.0.0.0"
#运行模式: 规则Rule,全局Global,直连Direct
mode: rule
#log-level: silent
log-level: info
```

然后配置的其他内容，如各种服务器，需要从代理提供商那边获取，将配置文件下载下来，将里面的服务器配置信息添加到上面的配置文件中：

```yaml
# 以上内容忽略
# 只保留以下内容

dns:
  enable: true
  # listen: 0.0.0.0:53
  ipv6: false
  ......
```

```bash
vi ~/.zshrc
```

添加：

```bash
# set proxy 
alias proxyon='export all_proxy=socks5://127.0.0.1:7890;export http_proxy=http://127.0.0.1:7890;export https_proxy=http://127.0.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'

alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```

### 日常使用

平时 clash 不开启，必要时开启：

```bash
clash
```

在其他终端开启代理：

```bash
proxyon
```

使用完成后关闭 clash 即可。

### 强制走代理

有时会遇到设置 http_proxy 等之后，依然无法走代理，比如我在腾讯云国内的服务器上用 npm install （莫名其妙不知道为什么）。这时可以用 proxychains 强制 npm 走代理：

1. 安装 proxychains

    ```bash
    sudo apt install proxychains4
    ```

2. 配置 proxychains

    ```bash
    vi /etc/proxychains4.conf
    ```

    在末尾添加：

    ```properties
    socks5 127.0.0.1 7890
    ```

3. 使用 proxychains 运行 npm

    ```bash
    proxychains4 npm install postcss-cli
    ```






