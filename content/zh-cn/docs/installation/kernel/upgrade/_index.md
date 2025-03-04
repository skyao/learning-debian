---
title: "更新内核"
linkTitle: "更新内核"
weight: 20
description: >
  更新 debian 核心
---

可以单独更新 linux 内核：

```bash
$ sudo apt update
$ sudo apt install linux-image-amd64
```

也可以使用 `apt upgrade` 更新所有内容：

```bash
$ sudo apt update
$ sudo apt upgrade
```


