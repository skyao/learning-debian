---
title: "dmidecode"
linkTitle: "dmidecode"
weight: 10
description: >
  dmidecode通过读取 SMBIOS 数据表来提取有关系统硬件组件的详细信息。
---

## 介绍

`dmidecode` 是一个在 Linux 系统下非常强大的工具，它通过读取 **SMBIOS**（System Management BIOS）数据表，来提取有关系统硬件组件的详细信息。

简单来说，使用 `dmidecode` 不需要拆开机箱，就能查看到主板型号、内存插槽使用情况、CPU 详细规格、BIOS 版本、序列号等。

## 安装

大多数主流 Linux 发行版都预装了该工具。如果没有，可以通过以下命令安装：

```bash
apt update && apt install dmidecode -y
```

## 基本使用

`dmidecode` 必须以 **root** 权限运行，因为它需要访问 `/dev/mem` 或系统内核接口。

```bash
sudo dmidecode [选项]
```

直接运行 `sudo dmidecode` 会输出所有的硬件信息（非常长，不建议直接看）。

### 按类型查询

`-t` (或 `--type`) 是最常用的参数。可以通过 **数字编号** 或 **关键字** 来筛选信息。

常用类型对照表：

| 编号 | 关键字 | 说明 |
| :--- | :--- | :--- |
| 0 | bios | BIOS 信息 (版本、日期、大小) |
| 1 | system | 系统信息 (制造商、产品名、序列号、UUID) |
| 2 | baseboard | 主板信息 (型号、资产标签) |
| 3 | chassis | 机箱信息 (类型、锁状态) |
| 4 | processor | 处理器信息 (型号、主频、核心数、插槽) |
| 16 | memory | 物理内存阵列 (最大支持容量、插槽数) |
| 17 | memory | 内存设备 (单个插槽的内存大小、频率、厂家) |

示例：

* 查看主板型号：`sudo dmidecode -t baseboard`
* 查看内存详情：`sudo dmidecode -t memory`
* 查看 CPU 信息：`sudo dmidecode -t processor`

### 提取特定字符串

如果你只想在脚本中获取某个具体的值（如序列号），可以使用 `-s`。

常用关键字：

- `bios-version`
- `system-serial-number`
- `system-product-name`
- `baseboard-product-name`

示例：

* 只获取服务器序列号：`sudo dmidecode -s system-serial-number`
* 查看 BIOS 版本：`sudo dmidecode -s bios-version`


## 典型使用场景

### 查看主板型号

```bash
dmidecode -t baseboard
```

可以查看主板信息：

```bash
Getting SMBIOS data from sysfs.
SMBIOS 3.4.0 present.

Handle 0x0002, DMI type 2, 15 bytes
Base Board Information
	Manufacturer: ASUSTeK COMPUTER INC.
	Product Name: PRIME Z690-P D4
	Version: Rev 1.xx
	Serial Number: 220300372501005
	Asset Tag: Default string
	Features:
		Board is a hosting board
		Board is replaceable
	Location In Chassis: Default string
	Chassis Handle: 0x0003
	Type: Motherboard
	Contained Object Handles: 0
  ......
```

### 查看主板内存支持

想知道主板有几个内存插槽，最大支持多大内存？

```bash
dmidecode -t 16
```

在上面的华硕 z690-p 主板上显示为：

```bash
# dmidecode 3.6
Getting SMBIOS data from sysfs.
SMBIOS 3.4.0 present.

Handle 0x0046, DMI type 16, 23 bytes
Physical Memory Array
	Location: System Board Or Motherboard
	Use: System Memory
	Error Correction Type: None
	Maximum Capacity: 128 GB
	Error Information Handle: Not Provided
	Number Of Devices: 4
```

输出结果中的 `Maximum Capacity` 即最大支持容量（当前为128GB），`Number Of Devices` 即内存插槽总数，当前为4插槽。

### 查看内存容量和频率

想知道每个内存插槽插了多大的内存，内存频率是多少？

```bash
dmidecode -t 17
```

当前这块华硕 z690-p 主板上插了两条32GB的 ddr4 4000 内存：

```bash
# dmidecode 3.6
Getting SMBIOS data from sysfs.
SMBIOS 3.4.0 present.

Handle 0x0047, DMI type 17, 92 bytes
Memory Device
	Array Handle: 0x0046
	Error Information Handle: Not Provided
	Total Width: Unknown
	Data Width: Unknown
	Size: No Module Installed
	Form Factor: Unknown
	Set: None
	Locator: Controller0-ChannelA-DIMM0
	Bank Locator: BANK 0
	Type: Unknown
	Type Detail: None

Handle 0x0048, DMI type 17, 92 bytes
Memory Device
	Array Handle: 0x0046
	Error Information Handle: Not Provided
	Total Width: 64 bits
	Data Width: 64 bits
	Size: 32 GB
	Form Factor: DIMM
	Set: None
	Locator: Controller0-ChannelA-DIMM1
	Bank Locator: BANK 0
	Type: DDR4
	Type Detail: Synchronous
	Speed: 4000 MT/s
	Manufacturer: Corsair
	Serial Number: 00000000
	Asset Tag: 9876543210
	Part Number: CM4X32GC3600C18K2D  
	Rank: 2
	Configured Memory Speed: 4000 MT/s
	Minimum Voltage: 1.2 V
	Maximum Voltage: 1.35 V
	Configured Voltage: 1.2 V
	Memory Technology: DRAM
	Memory Operating Mode Capability: Volatile memory
	Firmware Version: Not Specified
	Module Manufacturer ID: Bank 3, Hex 0x9E
	Module Product ID: Unknown
	Memory Subsystem Controller Manufacturer ID: Unknown
	Memory Subsystem Controller Product ID: Unknown
	Non-Volatile Size: None
	Volatile Size: 32 GB
	Cache Size: None
	Logical Size: None

Handle 0x0049, DMI type 17, 92 bytes
Memory Device
	Array Handle: 0x0046
	Error Information Handle: Not Provided
	Total Width: Unknown
	Data Width: Unknown
	Size: No Module Installed
	Form Factor: Unknown
	Set: None
	Locator: Controller1-ChannelA-DIMM0
	Bank Locator: BANK 0
	Type: Unknown
	Type Detail: None

Handle 0x004A, DMI type 17, 92 bytes
Memory Device
	Array Handle: 0x0046
	Error Information Handle: Not Provided
	Total Width: 64 bits
	Data Width: 64 bits
	Size: 32 GB
	Form Factor: DIMM
	Set: None
	Locator: Controller1-ChannelA-DIMM1
	Bank Locator: BANK 0
	Type: DDR4
	Type Detail: Synchronous
	Speed: 4000 MT/s
	Manufacturer: Corsair
	Serial Number: 00000000
	Asset Tag: 9876543210
	Part Number: CM4X32GC3600C18K2D  
	Rank: 2
	Configured Memory Speed: 4000 MT/s
	Minimum Voltage: 1.2 V
	Maximum Voltage: 1.35 V
	Configured Voltage: 1.2 V
	Memory Technology: DRAM
	Memory Operating Mode Capability: Volatile memory
	Firmware Version: Not Specified
	Module Manufacturer ID: Bank 3, Hex 0x9E
	Module Product ID: Unknown
	Memory Subsystem Controller Manufacturer ID: Unknown
	Memory Subsystem Controller Product ID: Unknown
	Non-Volatile Size: None
	Volatile Size: 32 GB
	Cache Size: None
	Logical Size: None
```

关注 `Size` (大小), `Speed` (速率), `Manufacturer` (制造商), `Part Number` (型号)。

直接看内存运行频率：

```bash
dmidecode -t memory | grep -i speed
	Speed: 4000 MT/s
	Configured Memory Speed: 4000 MT/s
	Speed: 4000 MT/s
	Configured Memory Speed: 4000 MT/s
```

### 查看主板 BIOS 信息

查看当前主板 BIOS 版本：

```bash
dmidecode -t 0
```

输出为：

```bash
# dmidecode 3.6
Getting SMBIOS data from sysfs.
SMBIOS 3.4.0 present.

Handle 0x0000, DMI type 0, 26 bytes
BIOS Information
	Vendor: American Megatrends Inc.
	Version: 3801
	Release Date: 05/14/2025
	Address: 0xF0000
	Runtime Size: 64 kB
	ROM Size: 24 MB
	Characteristics:
		PCI is supported
		BIOS is upgradeable
		BIOS shadowing is allowed
		Boot from CD is supported
		Selectable boot is supported
		BIOS ROM is socketed
		EDD is supported
		Japanese floppy for NEC 9800 1.2 MB is supported (int 13h)
		Japanese floppy for Toshiba 1.2 MB is supported (int 13h)
		5.25"/360 kB floppy services are supported (int 13h)
		5.25"/1.2 MB floppy services are supported (int 13h)
		3.5"/720 kB floppy services are supported (int 13h)
		3.5"/2.88 MB floppy services are supported (int 13h)
		Print screen service is supported (int 5h)
		Serial services are supported (int 14h)
		Printer services are supported (int 17h)
		CGA/mono video services are supported (int 10h)
		USB legacy is supported
		BIOS boot specification is supported
		Targeted content distribution is supported
		UEFI is supported
	BIOS Revision: 38.1
```

## 高级选项

*   **`-u` (Dump):** 以原始十六进制格式显示。通常用于调试。
*   **`--dump-bin file`:** 将 DMI 数据保存为二进制文件。
*   **`--from-dump file`:** 从之前保存的二进制文件中读取信息（用于离线分析）。

## 注意事项

1.  **权限限制：** 必须使用 `sudo`，否则会报错。
2.  **虚拟化限制：** 在虚拟机（如 VMware, KVM, VirtualBox）中运行 `dmidecode`，你看到的通常是虚拟机的硬件信息（如 "Manufacturer: VMware, Inc."），而不是物理机的。
3.  **数据准确性：** `dmidecode` 读取的是 BIOS 写入的数据。如果 BIOS 厂商没有在 DMI 表中写入信息（常见于某些组装机或廉价主板），某些字段可能会显示为 `Unknown` 或 `Not Specified`。


