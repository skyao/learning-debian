---
title: "zerotier"
linkTitle: "zerotier"
date: 2025-04-07
weight: 80
description: >
  zerotier VPN
---

## 客户端

### 安装

参考: https://www.zerotier.com/download/

```bash
curl -s https://install.zerotier.com | sudo bash
```

安装下载的依赖包挺多的:

```bash
......
*** Enabling and starting ZeroTier service...
Synchronizing state of zerotier-one.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable zerotier-one

*** Waiting for identity generation...

*** Success! You are ZeroTier address [ 7ab42fe1c1 ].
```

设置开机自动启动:

```bash
sudo systemctl start zerotier-one
sudo systemctl enable zerotier-one
```

### 加入网络

```bash
sudo zerotier-cli join 41d49af6c2cf7f96
```

> 注意:一定要 sudo. 

## Moon

Moon 是用户自建的 zerotier 的 “卫星根服务器”，可以让设备在连接时优先通过 Moon 节点发现和中继。

因为我有一台有固定 IPV4 地址的服务器, 因此安装 moon 来加速我的 zerotier 设备的相互发现速度. 

### 安装 moon

安装 ZeroTier:

```bash
curl -s https://install.zerotier.com | sudo bash
sudo systemctl enable zerotier-one
sudo systemctl start zerotier-one
```

查看节点 ID:

```bash
sudo zerotier-cli info
```

输出内容为:

```bash
200 info e329f9xxxx 1.16.0 ONLINE
```

格式为 "200 info <nodeID> <version> ONLINE". 记下这里的 nodeID e329f9xxxx 备用.

注意: moon 节点不需要加入 zerotier 网络, 也就是不要需要执行 `zerotier-cli join` 命令.

### 搭建 moon

进入生成 ZeroTier 节点身份文件:

```bash
cd /var/lib/zerotier-one/
```

这个目录下有 identity.public, 可以用来生成 Moon 配置文件:

```bash
sudo zerotier-idtool initmoon identity.public | sudo tee moon.json > /dev/null
```

编辑生成的 moon.js:

```bash
vi moon.js
```

在 stableEndpoints 设置公网IP地址和端口:

```bash
"stableEndpoints": ["159.75.xxx.xxx/9993"]
```

另外记录下 id,这个后面要作为 moonID 使用. 

```json
{ 
 "id": "e329f95577", 
}
```


签名生成 .moon 文件:

```bash
sudo zerotier-idtool genmoon moon.json
```

输出为:

```bash
wrote 000000e329f95577.moon (signed world with timestamp 1761795759910)
```

把生成的 .moon 文件放到 /var/lib/zerotier-one/moons.d/ 目录下，并重启服务：

```bash
sudo mkdir -p /var/lib/zerotier-one/moons.d
sudo cp 000000*.moon ls 
sudo systemctl restart zerotier-one
```

### 使用 moon

#### linux 客户端

在 zerotier 的客户端如另外一台 linux 机器上执行命令:

```bash
sudo zerotier-cli orbit e329f95577 e329f95577
```

输出:

```bash
200 orbit OK
```

验证一下:

```bash
sudo zerotier-cli listpeers
```

输出的信息中有 moon 节点, 就说明成功了:

```bash
200 listpeers <ztaddr> <path> <latency> <version> <role>
200 listpeers 41d49af6c2 35.208.54.217/59841;5685;5475 205 1.15.3 LEAF
200 listpeers 778cde7190 - -1 - PLANET
200 listpeers 9b0823e410 202.182.98.67/3126;50736;54726 234 1.14.2 LEAF
200 listpeers a0ddbe8017 121.228.151.51/50472;135162;50063 -1 1.14.2 LEAF
200 listpeers a599bc239e 14.145.158.61/62057;8583;8583 55 1.10.6 LEAF
200 listpeers b23346cdea 116.21.253.112/35799;136192;51083 53 1.10.6 LEAF
200 listpeers cafe04eba9 84.17.53.155/9993;25705;65562 189 - PLANET
200 listpeers cafe80ed74 185.152.67.145/9993;25705;60342 288 - PLANET
200 listpeers cafefd6717 79.127.159.187/9993;680;25577 132 - PLANET
200 listpeers e329f95577 - -1 1.16.0 MOON
200 listpeers e9c4d1196f 121.228.198.7/61818;137609;52516 32 1.14.2 LEAF
```


#### openwrt 客户端

ssh 登录 openwrt 机器或者在 pve 页面上操作控制台, 创建目录:

```bash
mkdir -p /etc/zerotier/moons.d
```

scp 从 moon 节点机器上复制 moon 文件过来:

```bash
scp sky@skyao.net:/var/lib/zerotier-one/moons.d/000000e329f95577.moon /etc/zerotier/moons.d
```

再执行 `orbit` 命令:

```bash
zerotier-cli orbit e329f95577 e329f95577
```

重启 zerotier:

```bash
/etc/init.d/zerotier restart
```

检查:

```bash
zerotier-cli listpeers
```

如果看到 moon 节点说明生效了:

```bash
......
# 连接 moon 中(时间大于10秒)
200 listpeers e329f95577 - -1 - MOON

# 连接 moon 完成后
200 listpeers e329f95577 159.75.84.176/30483;497;490 6 1.16.0 MOON
```

## 附录 - zerotier基本概念

### 节点

可以通过执行 `zerotier-cli listpeers` 命令列出和当前节点相关的其他对端信息,如:

```bash
200 listpeers <ztaddr> <path> <latency> <version> <role>
200 listpeers 41d49af6c2 35.208.54.217/21040;1330;1330 -1 1.15.3 LEAF
200 listpeers 778cde7190 103.195.103.66/9993;44970;74783 197 - PLANET
200 listpeers 7ab42fe1c1 114.87.0.22/61914;1;2 28 1.16.0 LEAF
200 listpeers 9b0823e410 202.182.98.67/3126;14955;14726 229 1.14.2 LEAF
200 listpeers a0ddbe8017 121.228.151.51/64087;4947;4945 3 1.14.2 LEAF
200 listpeers a599bc239e 14.145.158.61/9993;9952;9920 32 1.10.6 LEAF
200 listpeers b23346cdea 116.21.253.112/41867;9952;6003 30 1.10.6 LEAF
200 listpeers cafe04eba9 84.17.53.155/9993;44970;74767 215 - PLANET
200 listpeers cafe80ed74 185.152.67.145/9993;44970;74828 152 - PLANET
200 listpeers cafefd6717 79.127.159.187/9993;44970;74804 178 - PLANET
200 listpeers e329f95577 159.75.84.176/27453;4947;4910 36 1.16.0 MOON
```

第一行标记了格式, 为 `200 listpeers <ztaddr> <path> <latency> <version> <role>`. 字段含义如下: 

| 字段        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| `<ztaddr>`  | 节点的 ZeroTier ID（唯一标识符，通常显示前 10 位）。         |
| `<path>`    | 当前通信路径，格式为 `IP:端口;本地端口;远端端口`，反映 NAT 映射情况。 |
| `<latency>` | 与该节点的延迟（毫秒），`-1` 表示未测到直连。                |
| `<version>` | 对端运行的 ZeroTier 版本号。                                 |
| `<role>`    | 节点角色（见下表）。                                         |

节点角色说明:

| 角色       | 含义                                                        |
| ---------- | ----------------------------------------------------------- |
| **PLANET** | 官方的根服务器（Earth 节点），用于节点发现和引导。          |
| **MOON**   | 自建的 Moon 节点（自定义根服务器），客户端 orbit 后会显示。 |
| **LEAF**   | 普通客户端节点（网络中的其他成员）。                        |

实例如下:

```properties
200 listpeers cafe04eba9 84.17.53.155/9993;4587;4372 215 - PLANET
```

- ID: `cafe04eba9`
- 路径: `84.17.53.155:9993`
- 延迟: 215ms
- 版本: 未显示（可能是根节点）
- 角色: **PLANET**（官方根服务器）

```properties
200 listpeers e329f95577 159.75.84.176/27453;4587;4550 36 1.16.0 MOON
```

- ID: `e329f95577`
- 路径: `159.75.84.176:27453`
- 延迟: 36ms
- 版本: 1.16.0
- 角色: **MOON**（自建的 Moon 节点，说明客户端已成功使用）

```properties
200 listpeers a0ddbe8017 121.228.151.51/9993;14589;24587 3 1.14.2 LEAF
```

- ID: `a0ddbe8017`
- 路径: `121.228.151.51:9993`
- 延迟: 3ms（非常快，说明直连成功）
- 版本: 1.14.2
- 角色: **LEAF**（普通客户端）

网络速度分析, 官方的 PLANET 节点,延时都在 150 - 200 ms 期间:

```bash
200 listpeers 778cde7190 103.195.103.66/9993;44970;74783 197 - PLANET
200 listpeers cafe04eba9 84.17.53.155/9993;44970;74767 215 - PLANET
200 listpeers cafe80ed74 185.152.67.145/9993;44970;74828 152 - PLANET
200 listpeers cafefd6717 79.127.159.187/9993;44970;74804 178 - PLANET
```

我自己建立的 moon 服务器, 延迟是 36 毫秒,符合日常 ping 值:

```properties
200 listpeers e329f95577 159.75.84.176/27453;4587;4550 36 1.16.0 MOON
```

