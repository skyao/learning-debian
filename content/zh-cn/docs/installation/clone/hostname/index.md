---
title: "修改hostname"
linkTitle: "修改hostname"
date: 2025-04-02
weight: 20
description: >
  克隆后修改hostname
---

可以通过 hostnamectl 来修改 hostname，避免 hostname 重复：

```bash
sudo hostnamectl set-hostname newNameHere
```

完成后再额外修改一下 hosts 文件中的 hostname：

```bash
sudo vi /etc/hosts
```

完成后重启即可：

```bash
sudo reboot
```
