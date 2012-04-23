---
layout: master
title: TRACE32-ICD
---

# TRACE32-ICD 

## 目的

* 熟悉在线调试器的主要功能
* 掌握在线调试器TRACE32的调试功能
* 熟悉编写启动过程需要的批处理文件
* 熟练使用ICD调试系统
* 熟悉Simulator

## In-Circuit Debugger

Most CPUs provide an onchip debug system implemented in the CPU.

Most CPUs provide an onchip debug system implemented in the CPU. Typical examples
are the BDM interface from Motorola, the JTAG interface for the ARM7 or the JTAG
interface for the PowerPC family. The debug interface usually requires a few CPU pins that
are used for the communication between the onchip debug system and a third party
development tool. The onchip debug system provides the following basic features:

- Read/write memory
- Read/write CPU register
- Single step and real time execution
- Hardware breakpoints and trigger features (not supported by all CPUs)

## TRACE32-ICD功能

The In-Circuit Debugger TRACE32-ICD uses these basic features of the onchip debug
system to provide a powerful debug tool that offers:

- Easy high-level and assembler debugging
- Display of internal and external peripherals on a logical level
- Onchip break and trigger support
- RTOS awareness
- Flash programming
- Powerful script language
- Multiprocessor debugging

## TRACE32-ICD使用

### TRACE32-ICD安装

### TRACE32-ICD启动

要使In-Circuit调试器正常工作，必须要有一个能正常运转的目标机系统。

注意电源打开/关闭时的正确顺序：

- 打开：先调试器，再目标机。
- 关闭：先目标机，再调试器。

### TRACE32-ICD设置

### 

## TRACE32-ICD调试

## 参考资料

TRACE 32 In-Circuit Debugger Quick Installation and Tutorial