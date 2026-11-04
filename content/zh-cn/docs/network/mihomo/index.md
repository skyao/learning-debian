---
title: "mohomo"
linkTitle: "mohomo"
date: 2025-04-07
weight: 100
description: >
  mohomo 是 clash verge 的核心，在没有图形界面时使用。
---

## 安装

下载地址：

https://github.com/MetaCubeX/mihomo/releases/

下载最新版本并安装：

```bash
wget https://github.com/MetaCubeX/mihomo/releases/download/v1.19.33/mihomo-linux-amd64-v1-v1.19.33.deb

sudo apt install ./mihomo-linux-amd64-v1-v1.19.33.deb
```

安装完成后验证：

```bash
mihomo -v       
Mihomo Meta v1.19.33 linux amd64 with go1.26.10 Wed Oct 21 16:39:55 UTC 2026
Use tags: with_gvisor

which mihomo
/usr/bin/mihomo
```

## 配置

### 配置文件

新建配置文件：

```bash
vi /etc/mihomo/config.yaml
```

内容为：

```bash
port: 7890
mixed-port: 7897
allow-lan: false
bind-address: 127.0.0.1

mode: rule
log-level: info
ipv6: false

tun:
  enable: false

proxies:
  - name: WEBSHARE
    type: http
    server: 1.1.1.1
    port: 1234
    username: 'xxxxx'
    password: 'xxxxxx'  
rules:
  - DOMAIN-SUFFIX,anthropic.com,WEBSHARE
  - DOMAIN-SUFFIX,claude.ai,WEBSHARE
  - DOMAIN-SUFFIX,claude.com,WEBSHARE
  - DOMAIN-SUFFIX,claudeusercontent.com,WEBSHARE

  # Claude Code 的可选遥测
  - DOMAIN,http-intake.logs.us5.datadoghq.com,WEBSHARE
  - DOMAIN,browser-intake-us5-datadoghq.com,WEBSHARE

  # 用于验证 Webshare 出口
  - DOMAIN,ipv4.webshare.io,WEBSHARE

  # GitHub、软件源及其他目标直连
  - MATCH,DIRECT
```

设置文件权限:

```bash
sudo chown root:root /etc/mihomo/config.yaml
sudo chmod 0600 /etc/mihomo/config.yaml
```

检查配置：

```bash
sudo /usr/bin/mihomo -t -d /etc/mihomo
```

## 启动

设置服务，开机自动启动：

```bash
sudo systemctl enable --now mihomo.service
sudo systemctl status mihomo.service --no-pager
```

验证一下，ipv4.webshare.io 是 Webshare 的测试域名。走代理时应返回代理的ip地址：

```bash
curl --noproxy "" -x http://127.0.0.1:7890 \
    -fsS --connect-timeout 10 --max-time 30 \
    https://ipv4.webshare.io/
```

普通目标应直连，返回自己的ip地址：

```bash
  curl --noproxy "" -x http://127.0.0.1:7890 \
    -fsS --connect-timeout 10 --max-time 30 \
    https://ipinfo.io/ip
```


```bash
curl --noproxy "" -x http://127.0.0.1:7890 \
    -sS --connect-timeout 10 --max-time 30 \
    -o /dev/null -w 'Claude API HTTP=%{http_code}\n' \
    https://api.anthropic.com/v1/messages 


curl --noproxy "" -x http://127.0.0.1:7890 \
   -sS --connect-timeout 10 --max-time 30 \
   -o /dev/null -w 'GitHub HTTP=%{http_code}\n' \
   https://github.com/
```

## 使用代理

以 claude code 为例，参考 [官方代理配置文档](https://code.claude.com/docs/en/network-config)：
 
```bash
# claude code
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"

export NO_PROXY=localhost,127.0.0.1,::1,192.168.0.0/16
export no_proxy="$NO_PROXY"

unset ALL_PROXY all_proxy
```

