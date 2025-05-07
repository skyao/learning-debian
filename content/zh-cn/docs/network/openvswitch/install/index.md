---
title: "open vSwitch 安装"
linkTitle: "安装"
date: 2025-04-29
weight: 10
description: >
  在 debian 12 下安装 open vswitch
---

## ~~用 apt 安装~~

```bash
sudo apt install openvswitch-switch
```

查看Open vSwitch（OVS）的版本：

```bash
$ sudo ovs-vsctl show

cb860a93-1368-4d4f-be38-5cb9bbe5273d
    ovs_version: "3.1.0"
```

## 从源码开始编译安装

参考官方文档： 

https://github.com/openvswitch/ovs/blob/main/Documentation/intro/install/debian.rst

https://docs.openvswitch.org/en/stable/intro/install/debian/

### 下载源码

```bash
mkdir -p ~/temp/
cd ~/temp/
git clone https://github.com/openvswitch/ovs.git
cd ovs

git checkout v3.5.0
```

### 准备编译

```bash
apt-get install build-essential fakeroot
```

打开 ovs 源码文件中的 `debian/control.in` 文件，找到 `Build-Depends: ` 这一段，将这里列出来的依赖都用 apt 安装一遍。

```bash
sudo apt install autoconf automake bzip2 debhelper-compat dh-exec dh-python dh-sequence-python3 dh-sequence-sphinxdoc graphviz iproute2 libcap-ng-dev 

sudo apt install libdbus-1-dev  libnuma-dev  libpcap-dev libssl-dev libtool libunbound-dev openssl pkg-config procps python3-all-dev python3-setuptools python3-sortedcontainers python3-sphinx

sudo apt install libdpdk-dev
```

准备编译：

```bash
$ ./boot.sh && ./configure --with-dpdk=shared && make debian
```

检查依赖是否都满足要求：

```bash
$ dpkg-checkbuilddeps

dpkg-checkbuilddeps: error: Unmet build dependencies: libdpdk-dev (>= 24.11)
```

但 apt 能安装的最新版本也就是 22.11.7 版本：

```bash
apt list -a libdpdk-dev                                          
Listing... Done
libdpdk-dev/stable,stable-security,now 22.11.7-1~deb12u1 amd64 [installed]
```

只好删除这个版本：

```bash
sudo apt remove libdpdk-dev
```

然后手工安装最新版本的 libdpdk-dev

### 安装 libdpdk-dev

下载页面：

https://core.dpdk.org/download/

找到 DPDK 24.11.2 (LTS) 这个版本，下载下来：

```bash
cd ~/temp/
wget https://fast.dpdk.org/rel/dpdk-24.11.2.tar.xz
tar xvf dpdk-24.11.2.tar.xz
cd dpdk-stable-24.11.2
```

参考文档： https://github.com/openvswitch/ovs/blob/main/Documentation/intro/install/debian.rst
以及参考 deepseek 的安装指导。

```bash
sudo apt install build-essential meson ninja-build python3-pyelftools libnuma-dev pkg-config
meson build
ninja -C build

cd build
sudo ninja install

# 更新动态链接库
sudo ldconfig 
```

安装完成之后，验证一下：

```bash
pkg-config --modversion libdpdk
```

输出为：

```bash
24.11.2
```

### 继续安装

准备编译：

```bash
# 进入 ovs 源码目录
cd ovs

$ ./boot.sh && ./configure --with-dpdk=shared && make debian
```

检查依赖是否都满足要求：

```bash
$ dpkg-checkbuilddeps

dpkg-checkbuilddeps: error: Unmet build dependencies: libdpdk-dev (>= 24.11)
```

这里要求的是 libdpdk-dev，但前面源码安装出来的是 libdpdk

找到编译出来的 libdpdk.pc 文件：

```bash
sudo find / -name "libdpdk.pc" 2>/dev/null
/home/sky/temp/dpdk-stable-24.11.2/build/meson-private/libdpdk.pc
/usr/local/lib/x86_64-linux-gnu/pkgconfig/libdpdk.pc
```

将 libdpdk.pc 文件复制到 /usr/share/pkgconfig/ 目录：

```bash
sudo mkdir -p /usr/share/pkgconfig/
sudo cp /usr/local/lib/x86_64-linux-gnu/pkgconfig/libdpdk.pc /usr/share/pkgconfig/
```

更新 PKG_CONFIG_PATH：

```bash
export PKG_CONFIG_PATH=/usr/share/pkgconfig:$PKG_CONFIG_PATH
```

备注：如果想永久生效，可添加到 ~/.zshrc

```bash
sudo mkdir -p /usr/include/dpdk
sudo ln -s /usr/local/include/* /usr/include/dpdk/
sudo ldconfig
```

继续编译：

```bash
make debian-deb
```

报错：

```bash
dpkg-shlibdeps: error: no dependency information found for /usr/local/lib/x86_64-linux-gnu/librte_vhost.so.25 (used by debian/openvswitch-switch-dpdk/usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk)
Hint: check if the library actually comes from a package.
dh_shlibdeps: error: dpkg-shlibdeps -Tdebian/openvswitch-switch-dpdk.substvars debian/openvswitch-switch-dpdk/usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk returned exit code 255
dh_shlibdeps: error: Aborting due to earlier error
make[1]: *** [debian/rules:8: binary] Error 255
make[1]: Leaving directory '/home/sky/temp/ovs'
make: *** [Makefile:7300: debian-deb] Error 2
```

报错的原因是 dpkg-shlibdeps 是 Debian 打包工具的一部分，用于自动检测二进制文件的动态库依赖。由于我们通过源码安装了 DPDK，dpkg 不知道 librte_vhost.so.25 属于哪个包，因此报错。

考虑到生成 deb 包不是必须，我们可以直接安装 OVS（不打包成 .deb），这样就不触发 dpkg-shlibdeps 检查。

```bash
sudo make install
```

检查安装之后的版本：

```bash
$ ovs-vswitchd --version

ovs-vswitchd (Open vSwitch) 3.5.0
DPDK 24.11.2
```

## 安装不带 dpdk

不带 dpdk 会简单很多：

https://cloudspinx.com/build-open-vswitch-from-source-on-debian-and-ubuntu/


sudo dpkg -i ./openvswitch-common_3.5.0-1_amd64.deb
sudo dpkg -i ./openvswitch-common-dbgsym_3.5.0-1_amd64.deb
sudo dpkg -i ./openvswitch-switch_3.5.0-1_amd64.deb
sudo dpkg -i ./openvswitch-switch-dbgsym_3.5.0-1_amd64.deb


sudo apt install libxdp1 libxdp-dev
sudo apt install libfdt1 libfdt-dev


ovs-vsctl show
ovs-vsctl: unix:/usr/local/var/run/openvswitch/db.sock: database connection failed (No such file or directory)

