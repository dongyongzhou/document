---
layout: master
title: MIT-6.828
---

## Overview

Resources

http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-828-operating-system-engineering-fall-2006/

6.828 teaches the fundamentals of engineering operating systems. The following topics are studied in detail: 

- virtual memory
- kernel and user mode
- system calls
- threads
- context switches
- interrupts
- interprocess communication
- coordination of concurrent activities
- the interface between software and hardware

Most importantly, the interactions between these concepts are examined. 
The course is divided into two blocks 

- the first block introduces an operating system, xv6, which runs on x86 SMPs and provides the basic Unix semantics of Unix v6. 
- The second block of lectures covers important operating systems concepts invented after Unix® v6, which was introduced in 1976.

## Studying OS

### What is an operating system? 

- o a piece of software that turns the hardware into something useful 
- o layered picture: hardware, OS, applications 
- o Three main functions: fault isolate applications, abstract hardware, manage hardware 

### OS Abstractions 

- o processes: fork, wait, exec, exit, kill, getpid, brk, nice, sleep, trace 
- o files: open, close, read, write, lseek, stat, sync 
- o directories: mkdir, rmdir, link, unlink, mount, umount 
- o users + security: chown, chmod, getuid, setuid 
- o interprocess communication: signals, pipe 
- o networking: socket, accept, snd, recv, connect 
- o time: gettimeofday 
- o terminal:

## Lab: designing and implementing miminal OS

### Tools/Env

http://pdos.csail.mit.edu/6.828/2014/tools.html

#### Compiler Toolchain

A "compiler toolchain" is the set of programs, including a C compiler, assemblers, and linkers, that turn code into executable binaries. 

#### QEMU Emulator

QEMU is a modern and fast PC emulator. QEMU version 0.15 is set up on Athena for x86 machines in the 6.828 locker.

Unfortunately, QEMU's debugging facilities, while powerful, are somewhat immature, so we highly recommend you use our patched version of QEMU instead of the stock version that may come with your distribution. The version installed on Athena is already patched. To build your own patched version of QEMU:

Clone the IAP 6.828 QEMU git repository git clone https://github.com/geofft/qemu.git -b 6.828-1.7.0
On Linux, you may need to install the SDL development libraries to get a graphical VGA window. On Debian/Ubuntu, this is the libsdl1.2-dev package.
Configure the source code
Linux: ./configure --disable-kvm [--prefix=PFX] [--target-list="i386-softmmu x86_64-softmmu"]
OS X: ./configure --disable-kvm --disable-sdl [--prefix=PFX] [--target-list="i386-softmmu x86_64-softmmu"] The prefix argument specifies where to install QEMU; without it QEMU will install to /usr/local by default. The target-list argument simply slims down the architectures QEMU will build support for.
Run make && make install




### Booting a PC

This lab is split into three parts. 

- The first part concentrates on getting familiarized with x86 assembly language, the Bochs x86 emulator, and the PC's power-on bootstrap procedure. 
- The second part examines the boot loader for our 6.828 kernel, which resides in the boot directory of the lab1 tree. 
- Finally, the third part delves into the initial template for our 6.828 kernel itself, named JOS, which resides in the kernel directory.


### 设置input设备 input_dev.

支持的事件类型、事件码、事件值的范围、input_id等信息


### 向子系统报告事件

	void input_event(struct input_dev *dev, unsigned int type, unsigned int code, int value)

## Reference

