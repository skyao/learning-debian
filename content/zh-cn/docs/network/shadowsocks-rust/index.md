---
title: "Shadowsocks-rust"
linkTitle: "Shadowsocks-rust"
date: 2025-04-07
weight: 80
description: >
  Shadowsocks-rust 科学上网软件服务器端
---

临时备用方案，遇到科学上网工具不可用时临时顶替一下。

服务器端运行在国外的 vps 机器上，速度比较慢，但聊胜于无。

选择 Shadowsocks-rust 是因为这个安装起来比较简单，凑合用一下。

## 下载

下载地址：

https://github.com/shadowsocks/shadowsocks-rust/releases

找到最新版本，下载 linux 版本的，比如 shadowsocks-v1.23.5.x86_64-unknown-linux-gnu.tar.xz。

```bash
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.23.5/shadowsocks-v1.23.5.x86_64-unknown-linux-gnu.tar.xz

tar xvf shadowsocks-v1.23.5.x86_64-unknown-linux-gnu.tar.xz
```

解压缩出来文件：

```bash
$ ls
sslocal
ssserver
ssurl
ssmanager
ssservice
```

## 安装

直接复制文件即可：

```bash
sudo mv ssserver /usr/local/bin/
```

## 配置

 创建配置文件:

```bash
sudo mkdir -p /etc/shadowsocks-rust
sudo vi /etc/shadowsocks-rust/config.json
```

文件内容为：

```bash
{
  "server": "0.0.0.0",
  "server_port": 8388,
  "password": "STRONG_PASSWORD",
  "method": "aes-128-gcm"
}
```

密码自行修改，后面连接时要用到。

## 运行

手工运行，执行命令：

```bash
sudo ssserver -c /etc/shadowsocks-rust/config.json
```

如果希望开机自动启动，创建 systemd 服务文件：

```bash
sudo vi /etc/systemd/system/shadowsocks-rust.service
```

内容为：

```properties
[Unit]
Description=Shadowsocks-Rust Server
After=network.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks-rust/config.json
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
sudo systemctl start shadowsocks-rust
```

设置开机自启：

```bash
sudo systemctl enable shadowsocks-rust
```

查看状态：

```bash
sudo systemctl status shadowsocks-rust
```









