---
title: "新建帐号"
linkTitle: "帐号"
date: 2024-01-18
weight: 15
description: >
  新建帐号，避免使用 root 帐号
---

## 新建帐号

1. 安装 sudo

   虽然大多数云镜像自带 sudo，但为了保险起见，确认一下：

   ```bash
   apt update
   apt install sudo -y
   ```

2. 创建用户 sky


   ```bash
   adduser sky
   ```

3. 将 sky 加入 sudo 组

   在 Debian 中，只要将用户加入 sudo 组即可获得管理员权限，不需要修改 /etc/sudoers 文件。

   ```bash
   usermod -aG sudo sky
   ```


