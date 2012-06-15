---
layout: master
title: Android Startup
---

## Overview

Android 启动流程可划分为四大部分

- Linux内核启动
- Android init启动
- Zygote启动
- System Server启动

##2 Linux内核启动

###2.1 上电

###2.2 运行Boot ROM 

- 找到到第一阶段bootloader位置
- 加载第一阶段bootloader到internal RAM
- 跳转到第一阶段bootloader内存位置，并启动

###2.3 运行第一阶段bootloader

- **初始化外部RAM**
- 找到第二阶段bootloader
- 加载第二阶段bootloader到外部RAM
- 跳转到第二阶段bootloader内存位置，并启动

###2.4 运行第二阶段bootloader

- 设置文件系统Setup file systems (一般是Flash 设备)
- 有**选择性地驱动**显示、网络、辅助存储器以及其他设备
- 使能其他CPU功能
- 使能 low-level 内存保护
- 选择性地加载安全保护措施 (e.g. ROM validation code)
- 找到Linux内核
- 加载Linux内核到内存
- 将Linux内核对启动参数放置到内存，使能内核知道启动时应该运行什么
- 跳转到Linux内核内存地址，并启动LInux

###2.5 运行Linux 内核

- 在内存中生成一个表，描述物理内存的布局
- 初始化并驱动输入设备
- 初始化并驱动磁盘控制器（特指MTD），并映射可用的块设备到内存
- 初始化高级电源管理支持（APM）
- 初始化中断处理程序: Interrupt Descriptor Table (IDT), Global Descriptor Table (GDT), and Programmable
Interrupt Controllers (PIC)
- 重置浮点运算单元(FPU)
- 从实模式切换到保护模式(i.e. enable memory protection)
- 初始化段寄存器segmentation registers和 临时的堆栈
- 零初始化内存
- 解压内核镜像
- 初始化临时的内核页表并使能分页
- 为0进程设置内核模式栈
- 用空的中断处理程序填充IDT
- 用系统参数初始化第一个页框
- 确定CPU模型
- 用GDT和IDT地址初始化相应的寄存器
- 初始化并启动内核

   * 调度程序Scheduler
   * 内存分区Memory zones
   * 伙伴系统分配器Buddy system allocator
   * 中断描述符表IDT
   * 软中断SoftIRQs
   * 日期和时间Date and Time
   * Slab分配器Slab allocator
   * . . . .

- 创建进程1（/init）并运行


## Android init启动

To be continued...

init的源文件多放在system/core/init下

init.rc放在system/core/rootdir/init.rc下。

不同平台（device）的init.rc放在device/下。

### init分析


## Zygote启动

To be continued...

## System Server启动

To be continued...


## Reference

*[android 启动流程](http://hi.baidu.com/billycoder/blog/item/76023c1f103c5b9486d6b65c.html)

