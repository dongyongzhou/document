---
layout: master
title: Android Signal Mechanism
---

## Overview

A signal is a small message that notifies a process that an event of some type has occurred in the system.

- Kernel abstraction for exceptions and interrupts.每种信号类型都对应某个类型的系统事件
- Sent from the kernel (sometimes at the request of another process) to a process.
- Different signals are identified by small integer ID’s (1-30)
- The only information in a signal is its ID and the fact that it arrived.

本文主要是描述Android基于UNIX signal信号机制的变化以及应用。

- Android信号机制的变化
- Android信号如何产生
- Android信号如何处理
- android信号机制如何应用

## Android信号机制的变化

**传统UNIX信号处理模型有两点缺陷**

1. 扩展性差。UNIX信号处理模型中大多使用32位整型位码来表示某一信号。而只提供 SIGUSR1 和 SIGUSR2供用户使用。
 
    Kernel maintains pending and blocked bit vectors in the context of each process.
    Kernel computes pnb = pending & ~blocked. 
    Choose least nonzero bit k in pnb and force process p to receive signal k.

2. 相同的信号连续到达后，大多只作为一个信号处理。也就是说你只能知道该信号是否到达，而不能确定到达了一个还是是个。

    There can be at most one pending signal of any particular type. 
    Important: Signals are not queued
    If a process has a pending signal of type k, then subsequent signals of type k that are sent to that process are discarded.

**Android信息处理的变化**

1. 重用SIGUSR1 和 SIGUSR2两个信号
2. 需要连续产生相同的信号而又要处理时，在期间加入延迟


## Android信号如何产生

To be continued...

### 如何发送

1. 在kernel里 使用 kill_proc_info(）
2. 在native应用中 使用 kill() 或者raise()
3. java 应用中使用 Procees.sendSignal()等
4. adb shell kill -num pid

## Android信号如何处理

信号处理的行为是以进程级的。就是说不同的进程可以分别设置不同的信号处理方式而互不干扰。同一进程中的不同线程虽然可以设置不同的信号屏蔽字，但是却共享相同的信号处理方式 （也就是说 在一个线程里改变信号处理方式，将作用于该进程中的所有线程）。

Android也是Linux系统。所以其信号处理方式不会有本质的改变。但是为了开发和调试的需要，android对一些信号的处理定义了额外的行为。

### SIGQUIT （ 整型值为 3）

没有遵循传统UNIX信号模型的默认行为 （终止 + core ）。

Android Dalvik应用收到该信号后，**会打印改应用中所有线程的当前状态，并且并不是强制退出**。这些状态通常保存在一个特定的叫做trace的文件中。

路径是/data/anr/trace.txt

问题是如果删除这个文件，会出错

    E/dalvikvm(17942): Unable to open stack trace file '/data/anr/traces.txt': Permission denied

### SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT

对于很多其他的异常信号 （SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT ), Android进程在退出前，会生成 tombstone文件。记录该进程退出前的轨迹。

- SIGILL          4       Illegal instruction (ANSI).非法硬件指令        终止+core
- SIGABRT         6       Abort (ANSI). 异常终止(abort)     			   终止+core
- SIGBUS          7       BUS error (4.2 BSD).总线错误                  终止+core
- SIGFPE          8       Floating-point exception (ANSI).算术异常      终止+core
- SIGSEGV         11      Segmentation violation (ANSI).内存段访问异常   终止+core
- SIGSTKFLT       16      Stack fault.协处理器故障，早期Linux定义信号     终止

### 测试

1. 对于终端发送 SIGQUIT，可以直接得到预期的结果 （生成相应的trace文件）。 


2. 对于终端发送SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT等信号，我们常常看到 “不确定” 的行为：有时候能够看到 process 终止，有时候却不能。core dump 也不是总能产生。

    F/libc    (18709): Fatal signal 6 (SIGABRT) at 0x00004709 (code=0)
    I/DEBUG   (17669): timed out waiting for pid=18709 tid=18709 uid=10001 to die
    I/DEBUG   (17669): debuggerd committing suicide to free the zombie!
    I/DEBUG   (19051): debuggerd: Jun  4 2012 02:04:50


要点是：

要产生core dump并终止某进程，我们需要 连续发送两次改信号，并且中间间隔在0.2秒到3秒之间。
如果间隔过小， Android可能无法接收第一个signal。如果时间过久，android将简单的终止进程，而没有 core dump产生。

    F/libc    (19226): Fatal signal 6 (SIGABRT) at 0x00004709 (code=0)
    I/DEBUG   (19218): *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
    I/DEBUG   (19218): Build fingerprint: 'xxxx'
    I/DEBUG   (19218): pid: 19226, tid: 19226  >>> com.android.development <<<
    I/DEBUG   (19218): signal 6 (SIGABRT), code 0 (?), fault addr --------
    .....
    I/BootReceiver(  370): Copying /data/tombstones/tombstoneNoCrash_00 to DropBox (SYSTEM_TOMBSTONE)
## android信号机制如何应用

To be continued...

## android信号定义

prebuilt/linux-x86/toolchain/i686-linux-glibc2.7-4.4.3/sysroot/usr/include/bits/signum.h

## Reference

* [unix-signal](../os/unix-signal.html)
