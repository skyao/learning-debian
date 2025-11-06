---
title: "物理机安装"
linkTitle: "物理机安装"
date: 2024-01-18
weight: 15
description: >
  在物理机上安装 debian12 和 debian13
---


## 准备工作

### 安装版本

更新时间: 2024-09-16, 安装的版本是 debian 12.7。

更新时间: 2024-11-05, 安装的版本是 debian 13.1。

### 制作U盘

用下载下来的 iso 文件制作启动 u 盘。

windows 下一般用 refus 软件。

## 安装

启动后进入 debian 安装界面，选择 "Graphical install"

- select language: english
- select your location: other -> Asia -> China
- configure locales: United States en_US.UTF-8
- keyboard: American English

配置网络：

 - hostname: debian12.local 
 - hostname: debian13.local 

配置用户和密码：

 - password of root: 
 - 创建新用户
   - full name： Sky Ao
   - username: sky
   - password:


硬盘分区，选择 Guided - use entire disk -> all files in one partition, 等默认分区方案出来之后， 先删除除了 esp 分区之后的分区，然后在空闲空间上选择 create a new partition，大小为空闲空间 - 50g左右（留给 timeshift 做备份）， mount 为 `/`。剩下的约 50g 空间继续分区，mount 为 `/var/timeshift`。


"Finish partitioning and write changes to disk" , 选下一步，会提示没有 swap 空间，询问要不要退回去，选择 no。

确认分区，然后开始安装 base system。

configure the package manager, 选 "china" -> "mirrors.ustc.edu.cn"。

software selection，这是选择要继续安装的内容：

- debian desktop environment: 桌面，我当服务器用就不需要了，取消勾选
- 选择 ssh server 和 standard system utilities

等待安装完成，重启。

## 注意事项

曾经试过用 debian-13.1.0-amd64-netinst.iso (文件大小为 791M) 这个网络安装版本,在没有网络的情况下, 安装到最后一步 software selection, 没有任何可选项, 连最基本的 ssh server 都没有。导致安装结束之后, 机器可以正常启动, 但是因为没有 ssh server 导致无法远程连接. 

## 小结

安装比较简单，而且过程中可以配置包管理器，这样避免了从国外拉包的麻烦，用国内的服务器速度还是非常好的，因此推荐用网络安装的方式，这样安装完成之后系统和各个包都是最新版本了。
