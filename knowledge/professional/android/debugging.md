---
layout: master
title: Android User Space Debugging
---

## Content

- Java Exception
- Dalvik VM Crash (Tombstone Log)
- ANR

## Methods

- ADB
- Tomstone (similar to dmesg from Android framework point of view)
- Trace.txt (call stack information for all Java threads)
- Dumpstate (top, proc/interrups, ps –ef, wakelock)
- Useful binaries for Android framework debugging

    /system/bin/dumpstate #Show dumpstate
    /system/bin/bugreport #Show Tomstone, Trace.txt, dumpstate

## Java Exception

### Reference

* [Exception Handling](http://androidcookbook.com/Recipe.seam;jsessionid=5424397F3130CE7769FF47DD67742911?recipeId=75&recipeFrom=ViewTOC)
 
### Description

Java has had two categories of Exceptions (actually of its parent, Throwable) from the beginning, checked and unchecked.

Exception, and all of its subclasses other than RuntimeException or any of its subclasses, is checked. All else is unchecked.

"checked exception", meaning that the programmer would have to check for it, either by having a try-catch clause inside the file-using method or by having a throws clause on the method definition. 

RuntimeException subclasses include things like the excessively-long-named ArrayIndexOutOfBoundsException


### Reason and Result

Java has a mechanism of exception raise and catch. If the user uses a method raising an exception and does not catch it, it is processed as an unhandled exception, which could be an error.

It is from the user space Java application running on Dalvik VM.

Most of the time, it is not fatal and will not crash the target, but kills the application and restarts if its system is a privileged one.

Sometimes (but rarely) it can cause a target reset if it interferes with the process of an RPC call to another subsystem.

### Handle

Look for the origin of exception from the call list. The rule of thumb is to find the user application’s call among those on the list.

If you cannot find it, you can open up the direct call line just before the exception to understand what it is.


## Dalvik VM Crash

### What and Why

VM generates a crash report tombstone when there is a processor-defined exception (signal from the kernel – SIGILL, SIGABRT, SIGBUS, SIGFPE, SIGSEGV, SIGSTKFLT).

Once the core processor sees the problem, the kernel sends a signal to the debugger daemon socket in user space 

    system/core/debuggerd.c 
    bionic/linker/debugger.c

### How

The daemon generates a Tombstone Log (/data/tombstones/tombstone_xx) by getting a stack dump from each user process and the current CPU register value.

Use the following toolchain tools to map the failed address to the source code:

    arm-eabi-objdump
    arm-eabi-addr2line


### Result

If a crash happens in the **user space application**, e.g., market applications or system-privileged OEM application, it kills the process and restarts if it is necessary.

If a crash happens in **system_server**, it restarts Android user space.

## ANR

### What and Why


- If there is no response to the **input**, e.g., keypress, touch, within 5 sec, WindowManager triggers ANR.

InputManager kicks out the timer when it dispatches a key event to the UI chain of the foreground process. If it does not respond within time, WindowManager detects and prints out the message with the trace log and kills the process. An alert message also displays on the screen.

- If **BroadcastReceiver** does not finish executing within 10 sec, ActivityManager triggers ANR.

Every time there is a broadcast event, e.g., Wi-Fi on/off, GPS on/off, boot complete message, etc., it kicks a timeout before it broadcasts the event to all the processes that had registered a receiver. If there is a process not responding with a complete message back within a given time, ActivityManager kills that process and prints out the log.

### How

check the trace log to see the status of each thread in the process and try to find where it hangs.

    /data/anr/traces.txt


## Android Software Watchdog

### What

It is another **watchdog Java thread** running with system privilege over VM.

This thread belongs to ActivityManager.

    frameworks/base/services/java/com/android/server/watchdog.java

    static final int TIME_TO_RESTART = DB ? 15*1000 : 60*1000;
    static final int TIME_TO_WAIT = TIME_TO_RESTART / 2;

Normal threshold is 60 sec, but it is15 sec for debugging.

It detects a **system hang** by the heartbeat Monitor signal to the handler. Once it detects the lockup, it dumps the whole stack dump of each process. Then it kills the system process to resurrect the system (you can change to not kill the system process for debugging purposes).

This generates the zygote tombstone, which is the forking system_server process and it dumps all of the stack traces of all threads.

By looking at the traces log showing thread stack traces of all of the processes, you can try to find the deadlock situation if there is any.

Not only this, you can generate a tombstone of a certain process by sending SIGABRT to the process with # kill -6 <pid>.


