---
layout: master
title: Linux Crash
---

## Overview

Linux内核崩溃内存转储有多个开源项目工具:

- LKCD

LKCD stands for Linux Kernel Crash Dump. This tool allows the Linux system to write
the contents of its memory when a crash occurs, so that they can be later analyzed for
the root cause of the crash.

LKCD(Linux Kernel Crash Dump) 是 Linux 下第一个内核崩溃内存转储项目，它最初由 SGI 的工程师开发和维护。它提供了一种可靠的方法来发现、保存和检查系统的崩溃。LKCD 作为 Linux 内核的一个补丁，它一直以来都没有被接收进入内核的主线。目前该项目已经完全停止开发。

- Kdump 

Kdump is a much more flexible tool, with extended network-aware capabilities. It aims
to replace LKCD, while providing better scalability. Indeed, Kdump supports network
dumping to a range of devices, including local disks, but also NFS areas, CIFS shares
or FTP and SSH servers. This makes if far more attractive for deployment in large
environments, without restricting operations to a single server per subnet.

Kdump 是一种基于 kexec 的Linux内核崩溃转储机制，目前它已经被内核主线接收，成为了内核的一部分，它也由此获得了绝大多数 Linux 发行版的支持。与传统的内存转储机制不同不同，基于 Kdump 的系统工作的时候需要两个内核，一个称为系统内核，即系统正常工作时运行的内核；另外一个称为捕获内核，即正常内核崩溃时，用来进行内存转储的内核。 这个机制的原理是在内存中保留一块区域，这块区域用来存放capture kernel，当前的内核发生crash后，通过kexec把保留区域的capture kernel运行起来，由capture kernel负责把crash kernel的完整信息--包括CPU寄存器、堆栈数据等--转储到文件中，文件的存放位置可以是本地磁盘，也可以是网络。

## Kdump

### Kernels

- Standard (production) kernel - kernel we normally work with
- Crash (capture) kernel - kernel specially used for collecting crash dumps5

### components

**Kexec**

Kexec is a fastboot mechanism that allows booting a Linux kernel from the context of an
already running kernel without going through BIOS. BIOS can be very time consuming,
especially on big servers with numerous peripherals. This can save a lot of time for
developers who end up booting a machine numerous times.

**Kdump**

Kdump is a new kernel crash dumping mechanism and is very reliable. The crash dump
is captured from the context of a freshly booted kernel and not from the context of the
crashed kernel. Kdump uses Kexec to boot into a second kernel whenever the system
crashes. This second kernel, often called a crash or a capture kernel, boots with very
little memory and captures the dump image.

The first kernel reserves a section of memory that the second kernel uses to boot. Kexec
enables booting the capture kernel without going through BIOS hence the contents of
the first kernel’s memory are preserved, which is essentially the kernel crash dump.

## Reference

[Linux Kernel Crash Book](http://www.dedoimedo.com/computers/crash-book.html#purchase)
