---
layout: master
title: Linux Debug
---

##1 Overview

To be continued.


##2 Consepts


###2.1 Common ARM Exceptions

* Data abort: initiated when trying to **access** bad memory locations/Limited on access permission.
> 	Data Aborts (common case)

* Prefetch (instruction) abort: initiated when trying to **read the next instruction** from a bad memory location.
* Undefined instruction abort: initiated when trying to **execute** bad instruction code.
> Prefetch Aborts/Undefined instructions (Cases that usually indicate a bad device or memory errors/corruption)



###2.2 Virtual kernel memory layout:

>       vector  : 0xffff0000 - 0xffff1000   (   4 kB)
>       fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
>       vmalloc : 0xdd000000 - 0xff000000   ( 544 MB)
>       lowmem  : 0xc0000000 - 0xdcb00000   ( 459 MB)
>       pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
>       modules : 0xbf000000 - 0xbfe00000   (  14 MB)
>         .text : 0xc0008000 - 0xc0860d98   (8548 kB)
>         .init : 0xc0861000 - 0xc0898c40   ( 224 kB)
>         .data : 0xc089a000 - 0xc0957b20   ( 759 kB)
>          .bss : 0xc0b00024 - 0xc0ce1820   (1926 kB)

kernel/Documentation/arm/memory.txt

##3 Debuging Method& featues


###3.1 Linux Kernel Debug Configs


* CONFIG_DEBUG_SPINLOCK
> With this option enabled, the kernel catches operations on **uninitialized** spinlocks and various other errors (such as **unlocking a lock twice**).

* CONFIG_DEBUG_SPINLOCK_SLEEP
> This option enables a check for attempts to sleep while holding a spinlock. In fact, it complains if you call a function that could potentially sleep, even if the call in question would not sleep.

* CONFIG_DEBUG_STACKOVERFLOW



Additional Kernel Debug Features: slub debug,  spinlock etc



### Watchdog Bark

If the Linux kernel cannot schedule the watchdog kicking work queue because either the processor is **spinning around in an int-locked state** or the **processor is hanging due to a bus freeze**, watchdog bark FIQ is invoked.

Watchdog bark FIQ is handled in TrustZone (TZ) code.

The TZ FIQ handler checks to see if watchdog bark has happened and saves the register values, such as Program Counters (PC), link registers (LR), R13, etc., for all CPUs.

To debug show:

- CPU registers at the time of watchdog bark
- Call stack for all Linux processes at the time of failure
- Items in global work queue at the time of failure

For Crash dumps

- CPU Register Dump
- Process Call Stack
- Kernel dmesg
- Interrupt Status
- Kernel Global Work Queue

### Kernel – Panic

- BUG() or BUG_ON(<condition>)

General way to check the sanity of the variable status in kernel

CONFIG_BUG needs to be set

If CONFIG_DEBUG_BUGVERBOSE is set, it is printing the filename and line number showing where the BUG line is and causes a kernel panic; otherwise, it is just hanging.


**Explicit kernel BUGs** (will use undefined instruction panic handler, but will print out a “kernel BUG at” message)

- Oops message

    arch/arm/kernel/trap.c

Oops message is generated from:

    Oops – Undefined instruction
    Oops – Bad mode
    Oops – Bad syscall

- Page fault

    arch/arm/mm/fault.c

    1 Alignment exception
    2 Unhandled prefetch abort
    3 Unable to handle kernel address (kernel page fault)
    4 Unable to handle page fault (user page fault)

### Kernel – Low Memory Killer

**Description**

Low memory killer is part of the kernel that monitors the remaining available memory.

If the available memory goes under the **determined threshold**, it chooses a victim process and kills it.

It calculates **available memory** by summing up active + inactive pages.

If available memory size goes down to a threshold, e.g., 6144*4 Kbytes, it picks the process that has a 15 adj value and kills it until it gets enough memory back.

It sends SIGKILL to the process to kill it. More debug messages could be printed by changing lowmem_debug_level.

This is an important performance tuning point.

### Kernel – sys/proc fs List

### Kernel – Apps Hardware Watchdog


## Debuging datas

###1 Dmesg Buffer

**1.1 Buffer define **

/kernel/kernel/printk.c

    char __log_buf[__LOG_BUF_LEN];
    static char *log_buf = __log_buf;
    static int log_buf_len = __LOG_BUF_LEN;

    #define __LOG_BUF_LEN   (1 << CONFIG_LOG_BUF_SHIFT)

**1.2 Buffer size define **

/kernel/init/Kconfig

    config LOG_BUF_SHIFT
        int "Kernel log buffer size (16 => 64KB, 17 => 128KB)"
        range 12 21
        default 17
        help
          Select kernel log buffer size as a power of 2.
          Examples:
                     17 => 128 KB
                     16 => 64 KB
                     15 => 32 KB
                     14 => 16 KB
                     13 =>  8 KB
                     12 =>  4 KB

**1.3 Buffer size define example**

    CONFIG_LOG_BUF_SHIFT=17

128KB

###2 Locat Buffer

**2.1	驱动层**

定义LOG名称

drivers/staging/android/logger.h

    #define LOGGER_LOG_RADIO    "log_radio" 	/* radio-related messages */
    #define LOGGER_LOG_EVENTS   "log_events"    /* system/hardware events */
    #define LOGGER_LOG_SYSTEM   "log_system"    /* system/framework messages */
    #define LOGGER_LOG_MAIN     "log_main"      /* everything else */

drivers/staging/android/logger.c

    #define DEFINE_LOGGER_DEVICE(VAR, NAME, SIZE) \
    static unsigned char _buf_ ## VAR[SIZE]; \
    static struct logger_log VAR = { \
        .buffer = _buf_ ## VAR, \
        .misc = { \
            .minor = MISC_DYNAMIC_MINOR, \
            .name = NAME, \
            .fops = &logger_fops, \
            .parent = NULL, \
        }, \
        .wq = __WAIT_QUEUE_HEAD_INITIALIZER(VAR .wq), \
        .readers = LIST_HEAD_INIT(VAR .readers), \
        .mutex = __MUTEX_INITIALIZER(VAR .mutex), \
        .w_off = 0, \
        .head = 0, \
        .size = SIZE, \
    };


    DEFINE_LOGGER_DEVICE(log_main, LOGGER_LOG_MAIN, 256*1024)
    DEFINE_LOGGER_DEVICE(log_events, LOGGER_LOG_EVENTS, 256*1024)
    DEFINE_LOGGER_DEVICE(log_radio, LOGGER_LOG_RADIO, 256*1024)
    DEFINE_LOGGER_DEVICE(log_system, LOGGER_LOG_SYSTEM, 256*1024)


符号表

    _buf_log_main[256*1024];
    _buf_log_events[256*1024];
    _buf_log_radio[256*1024];
    _buf_log_system[256*1024];

注册设备

logger_init

    ret = init_log(&log_main);
    ret = init_log(&log_events);
    ret = init_log(&log_radio);
    ret = init_log(&log_system);

init_log

    ret = misc_register(&log->misc);

对应设备节点

    /dev/misc/log_main
    /dev/misc/log_events
    /dev/misc/log_radio
    /dev/misc/log_system

发送创建/dev/misc/log_XXX节点的event，设备节点的创建将由system/core/init/devices进行创建。

**2.2	中间层init**

system/core/init/devices.c

对应处理设备event

handle_device_fd

创建设备/dev/log目录

名字格式化成main、radio、events和system

    else if(!strncmp(uevent->subsystem, "misc", 4) &&
                    !strncmp(name, "log_", 4)) {
            base = "/dev/log/";
            mkdir(base, 0755);
            name += 4;
        }


然后创建设备真正的节点

    /dev/log/main
    /dev/log/radio
    /dev/log/events
    /dev/log/system

    if (!devpath_ready)
        snprintf(devpath, sizeof(devpath), "%s%s", base, name);
    if(!strcmp(uevent->action, "add")) {
        make_device(devpath, uevent->path, block, uevent->major, uevent->minor);
        if (links) {
            for (i = 0; links[i]; i++)
                make_link(devpath, links[i]);
        }
    }

**2.3	应用层logcat**

system/core/logcat/logcat.cpp

    main

    devices = new log_device_t(strdup("/dev/"LOGGER_LOG_MAIN), false, 'm');
    dev->fd = open(dev->device, mode);
    android::readLogLines(devices);


## Reference


