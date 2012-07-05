---
layout: master
title: Android Debug Logs
---

## Overview

- Main log
- Event log
- Radio log
- Kernel log
- Tombsontes 
- ANR trace

## Android Logs

### Main log

CASES:

- Application Not responding ->/data/anr/traces.txt
    - Caused when an application doesn’t respond to key press within 5 seconds
    - The stack trace of the application is save in /data/anr/traces.txt
    - The main thread (“main”) is of interest and needs further investigation.

- Java crash (exceptions)
    - Main log would have the exception details including the stack trace.


- Native crash ->/data/tombstones
- Android framework reboot (eof)
    - Watchdog killing system_server
    - Fatal exceptions in system_server
    - Excessive JNI references

### Event log

### Radio log

### Kernel Logs

- Helps in debugging target crash
- Kernel logs are usually associated with a log level, 0, 1, 2, 3 etc
- Any log with log level of 0,1,2 needs to be looked at.

### tombstone

- Helps in debugging native user space crash
- **User space segmentation fault** will have **SIGSEGV** in the tombstone  and the stack trace of the faulting thread.
- The stack trace is a set of addresses which is parsed by the stack tool 

#### How to debug Stack Trace

script is in the ics source tree: <android_build>/development/scripts/stack

Run: $ source build/envsetup.sh

Run: $ choosecombo #select all relevant flags

Run the stack utility as follows

 $ANDROID_BUILD_TOP/development/scripts/stack --symbols-dir=out/target/product/<board>/symbols

To see the stack for all threads with tombstones

- Pull the tombstone file corresponding to the crash from /data/tombstones on the phone

- Run the script as follows

    $ANDROID_BUILD_TOP/development/scripts/stack --symbols-dir=out/target/product/<board>/symbols <tombstone_file>

## TRACE debug

T32 very useful tool for

- Loading builds.
- Collecting ramdump.
- Debugging live crash.

### When to debug? 

- Target crashed but not in Download Mode.
- Target Up but adb not working.
- Putting breakpoints.
- Logs don’t provide enough information about crash


## Source Code List


