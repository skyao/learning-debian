---
title: "sdkman"
linkTitle: "sdkman"
date: 2025-04-07
weight: 20
description: >
  软件包管理工具，可以管理多个版本的软件包，包括 java，golang，python 等
---

## 介绍

sdkman 是一个软件包管理工具，可以管理多个版本的软件包，包括 java，golang，python 等。

官方网站：

https://sdkman.io/

可以将 sdkman 理解为多个平台上可用的工具包以简化多版本的 SDK 管理。

sdkman 支持的 jdk 列表请见： https://sdkman.io/jdks

sdkman 支持的 sdk 列表请见： https://sdkman.io/sdks

## 安装

安装方法参考官方文档：

https://sdkman.io/install

```bash
curl -s "https://get.sdkman.io" | bash
```

安装完成后，关闭当前终端重新打开新的终端，或者在当前终端中执行下面的命令立即更新：

```bash
source ~/.sdkman/bin/sdkman-init.sh
```

验证一下安装的版本：

```bash
$ sdk version  

SDKMAN!
script: 5.19.0
native: 0.7.4 (linux x86_64)
```

默认情况下，sdkman 安装在 HOME 下的 `.sdkman` 子目录中：

```bash
$ ls ~/.sdkman 
bin  candidates  contrib  etc  ext  libexec  src  tmp  var
```


## 使用

参考官方文档：

https://sdkman.io/usage

常用命令：

```bash
# 列出所有支持的软件包  
sdk list
# 列出所有支持的 jdk
sdk list java 
# 安装 jdk
sdk install java 17.0.14-zulu
# 卸载 jdk
sdk uninstall java 17.0.14-zulu

# 在当前 shell 中使用特定版本的 jdk
sdk use java 17.0.14-zulu
# 设置 jdk 为默认版本
sdk default java 17.0.14-zulu
# 查看当前使用的 jdk 版本
sdk current java
```

