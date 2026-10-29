---
title: "mysql 数据库"
linkTitle: "mysql"
date: 2025-04-03
weight: 10
description: >
  安装和配置 mysql 数据库
---

## 下载

https://dev.mysql.com/downloads/mysql/

选择 MySQL Community Server 的版本：

- 26.7.0
- 9.7.2 LTS
- 8.4.11 LTS
- 8.0.46

操作系统选择 ubuntu / debian 等。

- linux mint 22.3： 选择 ubuntu linux ，版本选 24.04

然后下载 DEB Bundle，大概 500 MB，如 mysql-server_8.0.46-1ubuntu24.04_amd64.deb-bundle.tar。

## 安装

### 8.0 版本

以在 linux mint 22.3 上安装 mysql-server_8.0.46 为例。解压缩 mysql-server_8.0.46-1ubuntu24.04_amd64.deb-bundle.tar 文件，得到：

```bash
-rw-r--r-- 1 sky sky 1.5M Apr  8  2026 libmysqlclient21_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  28M Apr  8  2026 libmysqlclient-dev_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  57K Apr  8  2026 mysql-client_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  59K Apr  8  2026 mysql-common_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky 2.0M Apr  8  2026 mysql-community-client_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky 2.1M Apr  8  2026 mysql-community-client-core_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky 1.4M Apr  8  2026 mysql-community-client-plugins_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  69K Apr  8  2026 mysql-community-server_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  27M Apr  8  2026 mysql-community-server-core_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  34M Apr  8  2026 mysql-community-server-debug_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky 386M Apr  8  2026 mysql-community-test_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky 8.5M Apr  8  2026 mysql-community-test-debug_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  57K Apr  8  2026 mysql-server_8.0.46-1ubuntu24.04_amd64.deb
-rw-r--r-- 1 sky sky  57K Apr  8  2026 mysql-testsuite_8.0.46-1ubuntu24.04_amd64.deb
```

先安装两个依赖包：

```bash
sudo apt install -y mecab-ipadic-utf8 libaio-dev
```

再执行安装：

```bash
sudo apt install "*.deb"
```

安装过程中要设置 root 密码，选择密码加密方式（默认即可）


## 简单验证

安装完成后：

```bash
$ which mysql                        
/usr/bin/mysql

$ mysql --version 
mysql  Ver 8.0.46 for Linux on x86_64 (MySQL Community Server - GPL)

$ sudo systemctl status mysql.service
     
● mysql.service - MySQL Community Server
     Loaded: loaded (/usr/lib/systemd/system/mysql.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-16 10:47:14 CST; 17min ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
   Main PID: 16634 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 16589)
     Memory: 353.5M (peak: 377.1M)
        CPU: 8.510s
     CGroup: /system.slice/mysql.service
             └─16634 /usr/sbin/mysqld

Oct 16 10:47:14 skyt14s systemd[1]: Starting mysql.service - MySQL Community Server...
Oct 16 10:47:14 skyt14s systemd[1]: Started mysql.service - MySQL Community Server.
```

简单连接上去看一眼：

```bash
mysql -u root -p
```

登录进去之后执行：

```bash
show databases;
```