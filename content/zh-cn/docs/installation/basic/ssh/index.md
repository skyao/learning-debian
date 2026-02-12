---
title: "ssh设置"
linkTitle: "ssh"
date: 2024-01-18
weight: 25
description: >
  ssh基本设置
---

## 防止空闲超时

某些 ssh 服务器会自动断开空闲连接，导致连接中断，尤其是某些 ecs 服务器，超时时间只有1分钟，非常不方便。需要设置一下防止空闲超时。

```bash
vi /etc/ssh/sshd_config
```

加入以下内容：

```bash
ClientAliveInterval 30
ClientAliveCountMax 3
```

ClientAliveInterval 设置为 30，表示每隔30秒发送一次心跳包，防止连接超时。
ClientAliveCountMax 设置为 3，表示如果3次心跳包都没有响应，则认为连接已断开。

重启 ssh 服务：

```bash
sudo systemctl restart ssh.service
```

## 密钥登录

上传本机的 `.ssh/id_isa.pub` 文件到服务器端：

```bash
scp ~/.ssh/id_rsa.pub sky@192.168.0.10:/home/sky 
```

在服务器上运行：

```bash
mkdir -p .ssh
touch ~/.ssh/authorized_keys
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
```

以后简单输入 “ssh server.ip” 即可自动登录。

## 指定特定端口登录

ssh默认采用22端口，对于需要进行端口映射导致不能访问22端口的情况，需要在ssh时通过 -p 参数指定具体的端口。

如下面的命令，有一台服务器的22端口是通过路由器的2122端口进行端口映射，则远程ssh连接的命令为：

```bash
ssh -p 2122 sky@dev.sky.net
```

对于服务器，可能需要使用特定的用户名登录，比如：

```bash
ssh -p 2122 root@dev.sky.net
```

## 快捷登陆

修改本机的 `~/.zshrc` 文件:

```bash
vi ~/.zshrc
```

加入以下内容：

```bash
# ssh to servers
alias ssh-ecs1="ssh sky@ecs1.skyao.net"
alias ssh-ecs2="ssh sky@ecs2.skyao.net"
alias ssh-ecs3="ssh sky@ecs3.skyao.net"
```

载入：

```bash
source ~/.zshrc
```

以后就可以一个简单命令直接ssh到远程服务器了：

```bash
ssh-ecs1
ssh-ecs2
ssh-ecs3
```

