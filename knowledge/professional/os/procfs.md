---
layout: master
title: Linux sys file system 
---

## Overview

Linux用户态访问内核数据方法有多种，除了基本的驱动设备节点外，还有虚拟的基于内存的文件系统procfs和sysfs。

查看和设定内核参数，更适合使用虚拟文件系统procfs和sysfs。

**procfs与sysfs比较**

一个 proc 虚拟文件可能有内部格式，并且读写格式不一样，代表不同的操作，应用程序中读到了这个文件的内容一般还需要进行字符串解析，而在写入时需要先用字符串格式化按指定的格式写入字符串进行操作；

sysfs 的设计原则是一个属性文件只做一件事情， sysfs 属性文件一般只有一个值，直接读取或写入。

新设计的内核机制应该尽量使用 sysfs 机制，而将 proc 保留给纯净的“进程文件系统”。

## procfs

procfs文件系统按约定总是被挂载在 /proc上

## Reference

