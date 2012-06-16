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

### Source code

- init的源文件多放在system/core/init下
- init.rc放在system/core/rootdir/init.rc下
- 不同平台（device）的init.rc放在device/下

### Overview

- 所有其他进程的祖先都是/init(PID=1)
- Custom initialization script (with its own language)， Different than more traditional /etc/inittab and SysV-init-levels initialization options
- The init program never exists (if it were, the kernel would panic), so it continues to monitor started services

### init分析

When started by the kernel, init parses and executes commands from two files:

- /init.rc - provides generic initialization instructions (see system/core/rootdir/init.rc)
- /init.board-name.rc - provides machine-specific initialization instructions - sometimes overriding /init.rc

    /init.goldfish.rc for the emulator
    /init.trout.rc for HTC’s ADP1
    /init.herring.rc for Samsung’s Nexus S (see device/samsung/crespo/init.herring.rc)

### init’s language

The init’s language consists of four broad classes of statements (see system/core/init/readme.txt)

#### **Actions** - named sequences of commands queued to be executed on a unique trigger

    `on <trigger>
     <command>
     <command>
     <command>`

#### **Triggers** - named events that trigger actions

* `early-init, init, early-fs, fs, post-fs, early-boot, boot` - built-in stages of init
* `<name>=<value>` - fires when a system property is set to `<name>=<value>`
* `device-added-<path>` - fires when a device node at `<path>` is added
* `device-removed-<path>` - fires when a device node at `<path>` is removed
* `service-exited-<name>` - fires when a service by `<name>` exists

#### **Commands** - a command to be queued and run

* chdir `<directory>` - change working directory
* chmod `<octal-mode> <path>` - change file access permissions
* chown `<owner> <group> <path>` - change file user and group ownership
* chroot `<directory>` - change process root directory
* class_start `<serviceclass>` - start all non-running services of the specified class
* class_stop `<serviceclass>` - stop all running services of the specified class
* domainname `<name>` - set the domain name.
* exec `<path> [ <argument> ]*` - fork and execute `<path> <argument> ... (blocks init until
exec returns)`
* export `<name> <value>` - set globally visible environment variable `<name> =<value>`
* `hostname <name>` - set the host name
* `ifup <interface>` - bring the network interface <interface> online
* `import <filename>` - parse and process <filename> init configuration file (extends the current script)
* `insmod <path> `- install the kernel module at `<path>`
* `loglevel <level>` - initialize the logger to `<level>`
* `mkdir <path> [mode] [owner] [group]` - create a directory at `<path> `and optionally change its default permissions `(755) and user (root) / group (root) `ownership
* `mount <type> <device> <dir> [ <mountoption> ]*` - attempt to mount named <device> at
`<dir> `with the optional `<mountoption>`s
* `setprop <name> <value> - set system property <name>=<value>`
* `setrlimit <resource> <cur> <max> - set the rlimit for a <resource>`
* `start <service> - start named <service> if it is not already running`
* `stop <service> - stop name <service> if it is currently running`
* `symlink <target> <path> - symbolically link <path> to <target>`
* `sysclktz <mins_west_of_gmt> - set the system clock base (0 for GMT)`
* `trigger <event> - trigger an event - i.e. call one action from another`
* `write <path> <string> [ <string> ]* - write arbitrary <string>`s to file by specified
`<path>`

#### Services - persistent daemon programs to be launched (optionally) restarted if they exit

    `service <name> <pathname> [ <argument> ]*
    <option>
    <option>
    ...`

#### Options - modifiers to services, modifying how/when they are run re/launched

* critical - a device-critical service - devices goes into recovery if the service exits more than 4 times in 4 minutes
* disabled - not started by default - i.e. it has to be started explicitly by name
* `setenv <name> <value> - set the environment variable <name>=<value> in the service`
* `socket <name> <dgram|stream|seqpacket> <perm> [ <user> [ <group> ] ] - create a unix
domain socket named /dev/socket/<name> and pass its fd to the service`
* `user <username> - change service’s effective user ID to <username>`
* `group <groupname> [ <groupname> ]* - change service’s effective group ID to <groupname> (additional groups are supplemented via setgroups())`
* oneshot - ignore service exist (default is to auto-restart)
* `class <name> - set the class name of the service (defaults to default) so that all services of a particular class can be started/stopped together`
* onrestart - execute a command on restart


### init启动分析


### The default init.rc script

1. Starts ueventd
2. Initializes the system clock and logger
3. Sets up global environment
4. Sets up the file system (mount points and symbolic links)
5. Configures kernel timeouts and scheduler
6. Configures process groups
7. Mounts the file systems
8. Creates a basic directory structure on /data and applies permissions
9. Applies permissions on /cache
10. Applies permissions on certain /proc points
11. Initializes local network (i.e. localhost)
12. Configures the parameters for the low memory killer
13. Applies permissions for systemserver and daemons
14. Defines TCP buffer sizes for various networks
15. Configures and (optionally) loads various daemons (i.e. services): ueventd, console, adbd, servicemanager, vold, netd, debuggerd, rild, **zygote (which in turn starts system_server)**, mediaserver, bootanimation(one time), and various Bluetooth daemons (like dbus-daemon, bluetoothd, etc.), installd, racoon, mtpd, keystore

### Nexus S’ init.herring.rc additionally

1. Sets up product info
2. Initializes device-driver-specific info, file system structures, and permissions: battery, wifi, phone, uart_switch,
GPS, radio, bluetooth, NFC, lights
3. Initializes and (re)mounts file systems
4. Loads additional device-specific daemons


## Zygote启动

To be continued...

## System Server启动

To be continued...


## Reference

*[android 启动流程](http://hi.baidu.com/billycoder/blog/item/76023c1f103c5b9486d6b65c.html)

