---
title: "Java"
linkTitle: "Java"
date: 2024-01-18
weight: 10
description: >
  搭建 Java 编程开发环境
---

## 安装 jdk

安装 jdk17

```bash
sdk install java 17.0.10-zulu
```

验证：

```bash
java --version
openjdk 17.0.10 2024-01-16 LTS
OpenJDK Runtime Environment Zulu17.48+15-CA (build 17.0.10+7-LTS)
OpenJDK 64-Bit Server VM Zulu17.48+15-CA (build 17.0.10+7-LTS, mixed mode, sharing)
```

## 安装 maven

```bash
sdk install maven 3.9.6
```

验证：

```bash
$ mvn --version

Apache Maven 3.9.6 (bc0240f3c744dd6b6ec2920b3cd08dcc295161ae)
Maven home: /home/sky/.sdkman/candidates/maven/current
Java version: 17.0.10, vendor: Azul Systems, Inc., runtime: /home/sky/.sdkman/candidates/java/17.0.10-zulu
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "6.1.0-17-amd64", arch: "amd64", family: "unix"
```