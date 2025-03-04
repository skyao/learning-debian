---
title: "使用qbittorrent"
linkTitle: "qbittorrent"
date: 2024-01-18
weight: 10
description: >
  在debian12下利用qbittorrent实现PT下载
---

## 选择版本

让我们快速了解一下 qBittorrent 安装方法之间的一些基本区别：

- qBittorrent 桌面优势：

  - 用户友好界面： 提供直观且易于浏览的图形界面。
  - 简洁的体验： 作为开源软件，它没有广告和不必要的捆绑软件。
  - 功能丰富：包括顺序下载、带宽调度等功能。

- qBittorrent-nox 的优势：

  - 专为无头系统优化： 专为最小化资源占用而设计，是受限系统的理想选择。
  - 网络接口： 允许通过基于网络的界面进行操作。
  - 远程管理： 通过网络接口方便管理服务器和远程系统。

## 自动安装

我用的debian是服务器版本，没有UI界面，因此选择安装 qbittorrent-nox （无图形界面版本）：

```bash
sudo apt install qbittorrent-nox
```

为 qBittorrent 创建专用系统用户和组：

```bash
sudo adduser --system --group qbittorrent-nox
sudo adduser sky qbittorrent-nox
```

为 qBittorrent-nox 创建 Systemd 服务文件

```bash
sudo vi /etc/systemd/system/qbittorrent-nox.service
```

内容如下：

```bash
[Unit]
Description=qBittorrent Command Line Client
After=network.target

[Service]
Type=forking
User=qbittorrent-nox
Group=qbittorrent-nox
UMask=007
ExecStart=/usr/bin/qbittorrent-nox -d --webui-port=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

重启 daemon-reload：

```bash
sudo systemctl daemon-reload
```

### 启动

启动 qbittorrent-nox 准备必要的目录：

```bash
sudo mkdir /home/qbittorrent-nox
sudo chown qbittorrent-nox:qbittorrent-nox /home/qbittorrent-nox
sudo usermod -d /home/qbittorrent-nox qbittorrent-nox
```

启动：

```bash
sudo systemctl start qbittorrent-nox
```

查看启动状态：

```bash
sudo systemctl status qbittorrent-nox
```

看到信息如下：

```bash
qbittorrent-nox.service - qBittorrent Command Line Client
     Loaded: loaded (/etc/systemd/system/qbittorrent-nox.service; enabled; preset: enabled)
     Active: active (running) since Sun 2024-05-05 01:09:48 EDT; 3h 7min ago
    Process: 768 ExecStart=/usr/bin/qbittorrent-nox -d --webui-port=8080 (code=exited, status=0/SUCCESS)
   Main PID: 779 (qbittorrent-nox)
      Tasks: 21 (limit: 9429)
     Memory: 6.4G
        CPU: 5min 23.810s
     CGroup: /system.slice/qbittorrent-nox.service
             └─779 /usr/bin/qbittorrent-nox -d --webui-port=8080

May 05 01:09:48 skynas3 systemd[1]: Starting qbittorrent-nox.service - qBittorrent Command Line Client...
May 05 01:09:48 skynas3 systemd[1]: Started qbittorrent-nox.service - qBittorrent Command Line Client.
```

设置开机自动启动：

```bash
sudo systemctl enable qbittorrent-nox
```

### 管理

访问 http://192.168.20.2:8080/ ，默认登录用户/密码为 `admin` 和 `adminadmin`。

- 下载
  - 默认保存路径："/mnt/storage2/download"
- 连接
  - 用于传入连接的端口：在路由器上增加这个端口的端口映射

- Web-ui
  - 语言：用户语言界面选择"简体中文"
  - 修改用户密码
  - 勾选 "对本地主机上的客户端跳过身份验证"
  - 勾选 "对 IP 子网白名单中的客户端跳过身份验证":  `192.168.0.0/24,192.168.192.0/24`
- 高级
  - 勾选 "允许来自同一 IP 地址的多个连接"
  - 勾选 "总是向同级的所有 Tracker 汇报"



### 参考资料

- [How to Install qBittorrent on Debian 12, 11, or 10 - LinuxCapable](https://www.linuxcapable.com/install-qbittorrent-on-debian-linux/)



## 手工安装

### 安装依赖包

先安装依赖包：

```bash
sudo apt update
sudo apt install build-essential pkg-config automake libtool git libgeoip-dev python3 python3-dev
sudo apt install libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev libssl-dev
sudo apt install qtbase5-dev qttools5-dev-tools libqt5svg5-dev zlib1g-dev
```

### 安装 libtorrent 1.2.19

安装libtorrent 1.2.19:

```bash
wget https://github.com/arvidn/libtorrent/releases/download/v1.2.19/libtorrent-rasterbar-1.2.19.tar.gz
tar xf libtorrent-rasterbar-1.2.19.tar.gz
cd libtorrent-rasterbar-1.2.19
./configure --disable-debug --enable-encryption --with-libgeoip=system CXXFLAGS=-std=c++14
make -j$(nproc)
sudo make install
sudo ldconfig
```

如果遇到错误：

```bash
checking whether g++ supports C++17 features with -std=c++17... yes
checking whether the Boost::System library is available... no
configure: error: Boost.System library not found. Try using --with-boost-system=lib
```

需要先安装 libboost-system-dev ：

```bash
sudo apt install libboost-system-dev
```

如果遇到错误：

```bash
checking whether compiling and linking against OpenSSL works... no
configure: error: OpenSSL library not found. Try using --with-openssl=DIR or disabling encryption at all.
```

需要先安装 libssl-dev ：

```bash
sudo apt install libssl-dev
```

### 安装 qbittorrent

不敢用太新的版本，还是找个稍微久一点的，继续沿用参考文档里面用的 4.3.9 版本：

```bash
wget https://github.com/qbittorrent/qBittorrent/archive/refs/tags/release-4.3.9.tar.gz
tar xf release-4.3.9.tar.gz
cd qBittorrent-release-4.3.9
./configure --disable-gui --disable-debug
make -j$(nproc)
sudo make install
```

### 设置开机自启

```bash
sudo vi /etc/systemd/system/qbittorrent.service
```

输入以下内容：

```properties
[Unit]
Description=qBittorrent Daemon Service
After=network.target

[Service]
LimitNOFILE=512000
User=root
ExecStart=/usr/local/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
```

启用开机自启：

```bash
sudo systemctl enable qbittorrent.service
```

### 第一次启动

先手工启动第一次：

```bash
sudo qbittorrent-nox                     

*** Legal Notice ***
qBittorrent is a file sharing program. When you run a torrent, its data will be made available to others by means of upload. Any content you share is your sole responsibility.

No further notices will be issued.

Press 'y' key to accept and continue...
```

按 Y 后，Ctrl+C退出。

用 systemctl 后台启动 qbittorrent：

```bash
sudo systemctl start qbittorrent.service
```

### 参考资料

- [Debian12 编译安装qbittorrent](https://www.smzdm.win/index.php/archives/1025/)
- [Debian12编译qBittorrent安装](https://zxqme.com/archives/138/)

