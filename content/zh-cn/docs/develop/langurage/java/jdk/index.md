---
title: "安装JDK"
linkTitle: "JDK"
date: 2025-04-03
weight: 10
description: >
  使用 sdkman 安装JDK
---

强烈建议使用 sdkman 之类的多版本管理方案来安装 jdk。

## 安装 JDK

列出当前系统中所有可用的 jdk 版本：

```bash
sdk list java
```

我偏好使用 zulu 的 openjdk 版本，所以这里以 zulu 为例。

### jdk24（non-LTS）

追踪一下最新版本的 jdk：

```bash
sdk install java 24.0.1-zulu
```

### jdk21（LTS）

虚拟线程正式发布，分代 ZGC、序列集合等新特性。

```bash
sdk install java 21.0.7-zulu
```

### jdk17（LTS）

包含密封类（Sealed Classes）、模式匹配等新特性。

```bash
sdk install java 17.0.15-zulu
```

### jdk11（LTS）

移除 Java EE 模块，引入 HTTP Client API、局部变量类型推断（var）等

```bash
sdk install java 11.0.27-zulu
```

### jdk8（LTS）

jdk8 是最广泛使用的版本，支持 Lambda 表达式、Stream API 等。

```bash
sdk install java 8.0.452-zulu
```

## 使用 jdk

列出目前已经安装的 jdk 版本：

```bash
ls ~/.sdkman/candidates/java/
```

输出如下：

```bash
11.0.26-zulu  17.0.14-zulu  21.0.6-zulu  8.0.442-zulu  current
```

设置默认的 jdk 版本：

```bash
sdk default java 17.0.14-zulu
```

在当前 shell 中使用指定版本的 jdk，可覆盖默认设置：

```bash
sdk use java 11.0.26-zulu
```

查看当前使用的 jdk 版本：

```bash
sdk current java
```

查看 jdk 版本信息：

```bash 
$ java --version
openjdk 17.0.14 2025-01-21 LTS
OpenJDK Runtime Environment Zulu17.56+15-CA (build 17.0.14+7-LTS)
OpenJDK 64-Bit Server VM Zulu17.56+15-CA (build 17.0.14+7-LTS, mixed mode, sharing)
```




