---
title: "nginx"
linkTitle: "nginx"
date: 2025-04-07
weight: 50
description: >
  nginx web 服务器
---

### 安装

```bash
sudo apt install nginx
```

安装之后的几个目录如下：

- /etc/nginx/nginx.conf 主配置文件
- /etc/nginx/sites-available/ 存放虚拟主机配置文件
- /etc/nginx/sites-enabled/ 存放虚拟主机配置文件
- /etc/nginx/conf.d/ 存放虚拟主机配置文件
- /etc/nginx/conf.d/ 存放虚拟主机配置文件
- /etc/nginx/conf.d/ 存放虚拟主机配置文件


### ipv6报错

nginx 默认开启 ipv6 监听,但是有时我没有开启 ipv6, 导致启动时报错:

```bash
Nov 04 10:30:22 debian13 systemd[1]: Starting nginx.service - A high performance web server
 and a reverse proxy server...
Nov 04 10:30:22 debian13 nginx[3611]: nginx: [emerg] socket() [::]:80 failed (97: Address f
amily not supported by protocol)
Nov 04 10:30:22 debian13 nginx[3611]: nginx: configuration file /etc/nginx/nginx.conf test 
failed
Nov 04 10:30:22 debian13 systemd[1]: nginx.service: Control process exited, code=exited, st
atus=1/FAILURE
Nov 04 10:30:22 debian13 systemd[1]: nginx.service: Failed with result 'exit-code'.
Nov 04 10:30:22 debian13 systemd[1]: Failed to start nginx.service - A high performance web
 server and a reverse proxy server.
```

关键错误信息是:

```bash
[::]:80 failed (97: Address f
amily not supported by protocol)
```

这个是因为 nginx 自带的默认 default site 的配置文件中设置了监听 ipv6 [::]:80 地址, 去掉就可以了:

```bash
sudo vi /etc/nginx/sites-enabled/default
```

注释掉 ipv6 这一行:

```bash
server {
        listen 80 default_server;
#       listen [::]:80 default_server;
```

再启动 nginx 即可.

```bash
sudo systemctl start nginx
```