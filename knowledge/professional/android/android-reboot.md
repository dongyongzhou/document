---
layout: master
title: Android Reboot/Reset
---

## Ways to reset or reboot the system

- Adb shell Reboot
- Kernel Panic
- Watchdog Timeout

## Adb shell Reboot
           
From windows commands prompt run

     C:\>adb shell reboot

### Analysis

- “adb shell reboot” is a shell command
- This invokes a **system call** from userspace, that will eventually call the *kernel_restart* in the kernel.
- *Kernel_restart* is the function that handles restart in the kernel. 
- This in turn will call *arch_reset*()  This will do the target specific functionality of rebooting the target.
- *arch_reset* will save the *reset_reason* and then invokes the Watchdog Timer for resetting the target.


## Kernel Panic

### What is Kernal panic

A kernel panic is an action taken by an operating system upon detecting an internal fatal error from which it cannot safely recover. 

### What will it do?

The kernel routines that 

- handle panics are outputs an error message to the console
- dump an image of kernel memory to disk for post-mortem debugging 
- then either wait for the system to be manually rebooted, or initiate an automatic reboot.(kernel/panic.c)

### How?

The panic timeout sets how long (in seconds) the target will wait before resetting automatically when a Linux kernel panic occurs. Currently on the **Android mainline this is set to 5**, 

- so if a panic occurs, the panic message and backtrace are sent to the dmesg log, the target waits 5 seconds, then flushes caches and resets. 
- If you set it to 0, the target will never reset as a result of a panic. The reset on panic functionality can be disabled by:
    - 1) echo 0 to /proc/sys/kernel/panic (on the target at runtime) 
    - 2) adding panic=0 to the kernel command line
    - 3) From “make kernelconfig” ,general setup submenu, and setting default panic timeout to 0


## Watchdog Timeout


### Why

Watchdog Timeout happens if the processor is struck in some thread infinitely. 

Apps watchdog timer will get fired if its not refreshed in 4 seconds. (watchdog driver handles all)

### How

Hardware watchdog is driven by the apps processor. 

The watchdog driver currently schedules itself to pet the watchdog every 3 seconds.

- If the hardware watchdog goes without being pet for 4 seconds, an interrupt is asserted which will panic the Linux kernel and print a back trace (triggering the panic timeout and reset if it is configured,). This is known as the "**dog bark**." 
- If the watchdog goes for 5 seconds without being pet, the target will reset. This is the "**dog bite**." 
     
Note that the **dog bark** will only be received if the target is not stuck with interrupts disabled and the hardware is still in a functional state. 

If the dog bark cannot be processed, and the system remains wedged for whatever reason, the dog bite will occur an reset the target.


