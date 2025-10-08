---
title: "Shadowsocks-rust"
linkTitle: "Shadowsocks-rust"
date: 2025-04-07
weight: 70
description: >
  Shadowsocks-rust 科学上网软件服务器端和客户端
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

## 服务器端

### 安装ssserver

直接复制文件即可：

```bash
sudo mv ssserver /usr/local/bin/
```

### 配置ssserver

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

### 运行ssserver

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

## 客户端

### 安装sslocal

直接复制文件即可：

```bash
sudo mv sslocal /usr/local/bin/
```

### 配置sslocal

创建配置文件:

```bash
sudo mkdir -p /etc/shadowsocks-rust
sudo vi /etc/shadowsocks-rust/config.json
```

文件内容为：

```bash
{
  "server": "46.202.xx.xx",
  "server_port": 8388,
  "password": "xxxxxxxxxx",
  "method": "aes-128-gcm",
  "local_address": "127.0.0.1",
  "local_port": 1080
}

```

密码和端口需要和服务器端一致。local_address 设置为 127.0.0.1，local_port 设置为 1080，这样sslocal 就会在本地监听 1080 端口，然后通过这个端口进行代理。

### 运行sslocal

手工运行，执行命令：

```bash
sudo sslocal -c /etc/shadowsocks-rust/config.json
```

如果希望开机自动启动，创建 systemd 服务文件：

```bash
sudo vi /etc/systemd/system/shadowsocks-rust-client.service
```

内容为：

```properties
[Unit]
Description=Shadowsocks-Rust Client
After=network.target
```

[Service]
ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks-rust/config.json
Restart=on-failure
User=root
LimitNOFILE=51200
```

[Install]
WantedBy=multi-user.target
```

重新加载 systemd：

```bash
sudo systemctl daemon-reload
```

启动：

```bash
sudo systemctl start shadowsocks-rust-client
```

设置开机自启：

```bash
sudo systemctl enable shadowsocks-rust-client
```

查看状态：

```bash
sudo systemctl status shadowsocks-rust-client
```

### 配置代理

```bash
vi ~/.zshrc
```
  
增加以下内容：

```bash
alias proxyon-shadowsocks='export all_proxy=socks5://127.0.0.1:1080;export http_proxy=http://127.0.0.1:1080;export https_proxy=http://127.0.0.1:1080;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```

保存退出。

### 日常使用

平时不使用时，执行 proxyoff 命令关闭代理。

使用时执行 proxyon 命令开启代理。

## 附录：Shadowsocks各组件功能说明

sslocal / ssserver / ssurl / ssmanager / ssservice 是 Shadowsocks 项目中的核心二进制可执行文件，各自承担不同的角色。

Shadowsocks 采用客户端-服务器架构。简单来说，`sslocal` 是客户端，`ssserver` 是服务端，而其他工具则用于辅助管理、分享和系统集成。

### 1. `sslocal` - Shadowsocks 本地客户端

**核心功能：** 作为 Shadowsocks **客户端**运行在用户的本地设备上。

**详细解释：**

- **职责**：它负责与远端的 `ssserver` 进行加密通信。
- **工作流程**：
  1. 在本地启动一个 SOCKS5 代理服务器（默认监听 `127.0.0.1:1080`）。
  2. 当你的浏览器或其他应用配置为使用这个 SOCKS5 代理时，它们发出的网络流量会被发送到 `sslocal`。
  3. `sslocal` 会将这些流量用 Shadowsocks 协议进行加密，然后转发给配置中指定的 `ssserver`。
  4. 同时，它也接收从 `ssserver` 返回的加密数据，进行解密后，再通过 SOCKS5 代理返回给最初的应用程序。

**简单比喻**：它就像你家里的一个“加密信使”，负责把普通的信件（你的网络流量）加密后寄出去，并把收到的加密回信解密后交给你。

**常用场景**：在个人电脑、手机或路由器上运行，用于科学上网。

### 2. `ssserver` - Shadowsocks 服务器

**核心功能：** 作为 Shadowsocks **服务端**运行在防火墙之外的远程服务器上（例如 VPS）。

**详细解释：**

- **职责**：它是流量的中转站，负责解密来自客户端的流量，并将其转发到真正的目标网站，反之亦然。
- **工作流程**：
  1. 监听一个特定的端口，等待 `sslocal` 的连接。
  2. 收到来自 `sslocal` 的加密数据后，使用预设的密码和加密方法进行解密，得到原始的网络请求（例如访问 `google.com` 的请求）。
  3. 将这个原始请求发送到互联网上的目标地址。
  4. 收到目标地址的响应后，再用同样的加密方式加密，发回给 `sslocal`。

**简单比喻**：它就像你在国外的“代理邮局”，负责接收“加密信使”发来的加密信件，解密后投递给真正的收件人（如 Google），并把回信加密后再寄回给你的“加密信使”。

**常用场景**：部署在境外的云服务器上，作为翻墙的出口节点。

### 3. `ssurl` - Shadowsocks 链接工具

**核心功能：** 用于生成和解析 Shadowsocks 的分享链接。

**详细解释：**

- **生成链接**：可以将一个服务器的配置信息（服务器地址、端口、密码、加密方法等）编码成一个标准的 `ss://` 链接。这种链接易于分享，用户只需扫描二维码或点击链接即可导入配置。
- **解析链接**：也可以将一个 `ss://` 链接解码，显示出明文的服务器配置信息，方便你手动检查或配置。

**常用场景**：

- 服务器管理员生成配置链接分发给用户。
- 用户从别人那里获得一个 `ss://` 链接时，可以先用它来检查配置详情。

**示例**：

bash

```
# 生成一个链接
ssurl -e -s 1.2.3.4 -p 8388 -m aes-256-gcm -k my_password

# 解码一个链接
ssurl -d ‘ss://YWVzLTI1Ni1nY206bXlfcGFzc3dvcmQ@1.2.3.4:8388‘
```

### 4. `ssmanager` - Shadowsocks 管理器

**核心功能：** 一个用于**管理多用户/多端口**的服务端守护进程。

**详细解释：**

- **目标用户**：主要面向服务提供商或需要为多个用户（每个用户有独立的端口和密码）提供服务的场景。
- **工作模式**：它本身不处理具体的 Shadowsocks 流量加密和解密，而是作为一个管理器和流量路由器。
- **通信方式**：它通过一个 **Unix Domain Socket** 或 **TCP 接口** 接收管理命令（例如添加用户、删除用户）。
- **优势**：允许动态地添加、修改和删除端口与密码，而无需重启服务。这比直接运行 `ssserver`（配置写死在文件里，改动需要重启）要灵活得多。

**常用场景**：商业化的 Shadowsocks 服务后端，或者自己搭建需要灵活配置的多用户代理服务。

### 5. `ssservice` - Shadowsocks 系统服务

**核心功能：** 这是一个在 **Windows 系统** 上特有的工具。

**详细解释：**

- **目的**：为了将 `sslocal` 或 `ssserver` 以 **Windows 系统服务** 的形式在后台运行。
- **解决的问题**：在 Windows 中，普通的命令行程序在用户注销后就会被关闭。通过 `ssservice` 将其安装为系统服务，可以实现在开机后自动启动、在后台无窗口运行、即使用户注销也不中断。
- **本质**：它本身不是一个独立的代理核心，而是一个“服务包装器”，用来托管和运行 `sslocal.exe` 或 `ssserver.exe`。

**常用场景**：在 Windows 服务器上长期稳定地运行 `ssserver`，或者在 Windows 电脑上让 `sslocal` 在后台开机自启。

### 总结表格

| 二进制文件      | 角色                | 主要功能                          | 典型运行位置                 |
| :-------------- | :------------------ | :-------------------------------- | :--------------------------- |
| **`sslocal`**   | **客户端**          | 本地 SOCKS5 代理，加密/解密流量   | 用户自己的设备               |
| **`ssserver`**  | **服务端**          | 接收/转发流量，解密/加密流量      | 远程服务器/VPS               |
| **`ssurl`**     | **工具**            | 生成和解析 `ss://` 分享链接       | 任何需要生成或检查配置的地方 |
| **`ssmanager`** | **服务端管理器**    | 动态管理多用户、多端口配置        | 远程服务器/VPS               |
| **`ssservice`** | **Windows服务工具** | 将其他组件作为Windows系统服务运行 | Windows 系统                 |



