---
title: "clash"
linkTitle: "clash"
date: 2025-04-07
weight: 80
description: >
  clash 科学上网软件客户端
---

### 下载clash

原网站 https://github.com/Dreamacro/clash/ 网站已经无法访问。

> https://clashcn.com/clash-releases 这里有一些备份，待验证。

在这里找到一个备份：

https://github.com/moshaoli688/clash/releases

下载 clash-linux-amd64-v1.18.0.gz：

```bash
wget https://github.com/moshaoli688/clash/releases/download/v1.18.0/clash-linux-amd64-v1.18.0.gz
```

### 安装clash

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

### 配置clash

创建 clash 的配置目录：

```bash
sudo mkdir -p /etc/clash
cd /etc/clash
```

手工下载IP数据库文件：

```bash
sudo wget https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb
```

否则启动时会自动下载，然后一般会失败：

```bash
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download              
FATA[0000] Initial configuration directory error: can't initial MMDB: can't download MMDB: Get "https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb": read tcp 10.1.20.3:57604->8.7.198.46:443: read: connection reset by peer 
```

mmdb 是 ip 数据库文件，需要下载，但是这个地址很容易被墙导致无法下载，可以多试几次。或者用其他方式将这个 https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb 下载下来，然后放到 `/etc/clash` 目录下。

再创建配置文件：

```bash
sudo vi /etc/clash/config.yaml
```

配置文件内容：

```yaml
port: 7890
socks-port: 7891
redir-port: 7892
mixed-port: 7893
allow-lan: false
# 监听地址,安全起见用127.0.0.1,如果容许给局域网内的其他机器使用,可以设置为 0.0.0.0
bind-address: "127.0.0.1"
#运行模式: 规则Rule,全局Global,直连Direct
mode: rule
#log-level: silent
log-level: info
```

然后配置的其他内容，如各种服务器，需要从代理提供商那边获取，将配置文件下载下来，将里面的服务器配置信息添加到上面的配置文件中：

```yaml
# dns以上的内容忽略，使用上面的配置内容
# 只保留dns以下内容

dns:
  enable: true
  # listen: 0.0.0.0:53
  ipv6: false
  ......
```

配置完成后，clash 就能手工启动了：

```bash
$ sudo clash -d /etc/clash/
INFO[0000] Start initial compatible provider 自动选择       
INFO[0000] Start initial compatible provider 节点选择       
INFO[0000] inbound http://127.0.0.1:7890 create success. 
INFO[0000] inbound socks://127.0.0.1:7891 create success. 
INFO[0000] inbound redir://127.0.0.1:7892 create success. 
INFO[0000] inbound mixed://127.0.0.1:7893 create success. 
INFO[0000] RESTful API listening at: [::]:9090          
INFO[0000] DNS server listening at: 127.0.0.1:8853
```

### 开机自动启动

创建 systemd 服务文件：

```bash
sudo vi /etc/systemd/system/clash.service
```

内容为：

```properties
[Unit]
Description=Clash Client
After=network.target

[Service]
ExecStart=/usr/local/bin/clash -d /etc/clash/
Restart=on-failure
User=root
LimitNOFILE=51200

[Install]
WantedBy=multi-user.target
```

重新加载 systemd：

```bash
sudo systemctl daemon-reload
```

启动：

```bash
sudo systemctl start clash
```

设置开机自启：

```bash
sudo systemctl enable clash
```

查看状态：

```bash
sudo systemctl status clash
```

### 配置代理

```bash
vi ~/.zshrc
```

添加：

```bash
# set proxy 
alias proxyon-clash='export all_proxy=socks5://127.0.0.1:7891;export http_proxy=http://127.0.0.1:7890;export https_proxy=http://127.0.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'

alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```


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

### 指定规则

有时需要为某些特殊域名指定代理规则,比如必须直连,或者必须走代理.

典型如:

- copilot.microsoft.com: 不开放给国内访问,必须要走代理

设置方式, 以 "copilot.microsoft.com"为例, clash 设置代理规则为 Rule(默认就是), 然后修改配置文件:

```bash
sudo vi /etc/clash/config.yaml
```

找到 microsoft.com 设置的这一行, 默认是 DIRECT 直接访问, 在这行上面增加一行, 设置 copilot.microsoft.com 为 "节点选择":

```properties
 - DOMAIN-SUFFIX,copilot.microsoft.com,节点选择
 - DOMAIN-SUFFIX,microsoft.com,DIRECT
```

> 备注: 发现 copilot.microsoft.com 这行放在最后面,无法生效, 必须放在 microsoft.com 之前.

重启后生效:

```bash
sudo systemctl restart clash
```



