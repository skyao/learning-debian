---
title: "stress-ng"
linkTitle: "stress-ng"
weight: 10
description: >
  stress-ng支持对 CPU、内存、磁盘等多种组件进行压力测试
---

## 介绍

stress-ng 是 stress 工具的增强版，支持对 CPU、内存、磁盘等多种组件进行压力测试，适合长时间稳定性测试。

## 安装

```bash
apt install -y stress-ng
```

## cpu压力测试

测试方式：

- 对所有 cpu 核心进行满载测试

   ```bash
   stress-ng --cpu $(nproc) --cpu-method fft --cpu-load 100 --timeout 1800s
   ```

   - `--cpu $(nproc)`：使用所有 CPU 核心（nproc 会自动获取核心数）。
   - `--cpu-load 100`：每个核心负载 100%。
   - `--timeout 1800s`：测试持续 30 分钟（可调整时间）。
   - `--cpu-method fft` 指定测试方法
      - `matrix`：矩阵运算（中等负载，适合长时间测试）
      - `fft`：快速傅里叶变换（高负载，侧重浮点运算）
      - `sha256`：SHA256 哈希计算（高负载，侧重整数运算）

## cpu内存测试

同时测试 CPU 和内存

此命令会让所有 CPU 核心满载，并同时分配 32GB 内存进行读写，持续 1 小时。

```bash
stress-ng --cpu $(nproc) --cpu-load 100 --cpu-method fft --vm 4 --vm-bytes 8G --timeout 3600s
```
