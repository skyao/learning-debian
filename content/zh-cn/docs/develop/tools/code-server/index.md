---
title: "Code Server"
linkTitle: "Code Server"
date: 2025-04-07
weight: 60
description: >
  浏览器中的 VS Code
---

## 安装

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

输出:

```bash
bian GNU/Linux 13 (trixie)
Installing v4.138.0 of the amd64 deb package from GitHub.

+ mkdir -p ~/.cache/code-server
+ curl -#fL -o ~/.cache/code-server/code-server_4.138.0_amd64.deb.incomplete -C - https://github.com/coder/code-server/releases/download/v4.138.0/code-server_4.138.0_amd64.deb
######################################################################## 100.0%
+ mv ~/.cache/code-server/code-server_4.138.0_amd64.deb.incomplete ~/.cache/code-server/code-server_4.138.0_amd64.deb
+ sudo dpkg -i ~/.cache/code-server/code-server_4.138.0_amd64.deb
[sudo] password for sky: 
Selecting previously unselected package code-server.
(Reading database ... 60228 files and directories currently installed.)
Preparing to unpack .../code-server_4.138.0_amd64.deb ...
Unpacking code-server (4.138.0) ...
Setting up code-server (4.138.0) ...

deb package has been installed.

To have systemd start code-server now and restart on boot:
  sudo systemctl enable --now code-server@$USER
Or, if you don't want/need a background service you can run:
  code-server

Deploy code-server for your team with Coder: https://github.com/coder/coder
```

也可以先下载下来，再安装：

```bash
# curl -fOL https://github.com/coder/code-server/releases/download/v$VERSION/code-server_${VERSION}_amd64.deb
curl -fOL https://github.com/coder/code-server/releases/download/v4.138.0/code-server_4.138.0_amd64.deb
sudo dpkg -i code-server_${VERSION}_amd64.deb
```

启动并设置为开机启动启动：

```bash
sudo systemctl enable --now code-server@$USER
```

如果不想开机启动，可以通过命令手工启动：

```bash
code-server
```

## 配置

### 基本配置

```bash
vi ~/.config/code-server/config.yaml
```

默认内容：

```bash
bind-addr: 127.0.0.1:8080
auth: password
password: ae89d22101b5b74bc2f5cbf5
cert: false
```

修改 build-addr 地址为 0.0.0.0:8080，修改密码，开启 https：

```bash
bind-addr: 0.0.0.0:8080
auth: password
password: simple
cert: true
```

重启：

```bash
sudo systemctl restart code-server@sky.service
```

访问 https://192.168.3.111:8080 输入密码就可以使用了。

### 安装扩展

安装编程语言的扩展：

| 编程语言             | 扩展名                                   | publisher           |
| -------------------- | ---------------------------------------- | ------------------- |
| Golang               | Go                                       | golang              |
| Java                 | Language Support for Java(TM) by Red Hat | Red Hat             |
| python               | Red Hatpython                            | publisher ms-python |
| Rust                 | rust-analyzer                            | rust-lang           |
| Javasript/Typescript | 内置                                     | 内置                |
|                      |                                          |                     |

