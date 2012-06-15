---
layout: master
title: Android Dalvik
---

## Overview

Dalvik is the process virtual machine (VM) in Google's Android operating system. It is the software that runs the apps on Android devices. Dalvik is thus an integral part of Android, which is typically used on mobile devices such as mobile phones and tablet computers. Programs are commonly written in Java and compiled to bytecode. They are then converted from Java Virtual Machine-compatible .class files to Dalvik-compatible .dex (Dalvik Executable) files before installation on a device. The compact Dalvik Executable format is designed to be suitable for systems that are constrained in terms of memory and processor speed.

- Provides Android app portability and consistency across various hardware (like a JVM)
- Licensed under Apache 2.0 open-source license
- Runs Dalvik byte-code, stored in .dex files (not Java byte code .class file). Build-tools compile Java’s .class files into a .dex file before packaging (into .apk files)
- Core libraries based on Java SE 5 (mostly from Apache Harmony), with many differences
- No support for *java.applet, java.awt, java.lang.management and javax.management (JMX),
java.rmi and javax.rmi, javax.accessibiliy, javax.activity, javax.imageio, javax.naming
(JNDI), javax.print, javax.security.auth.kerberos, javax.security.auth.spi, javax.security.spi,
javax.security.sasl, javax.sound, javax.swing, javax.transaction, javax.xml (except for
javax.xml.parsers), org.ietf, org.omg, org.w3c.dom.* (subpackages)*
- support for *Android APIs (including wrappers for OpenGL, SQLite, etc.), Apache HTTP Client (org.apache.http),
JSON parser (org.json), XML SAX parser (org.xml.sax), XML Pull Parser (org.xmlpull), etc.*

### Why not Java SE?

– Java SE is too bloated for mobile environment
– Would require too much redundancy (at the library level)
– Not well-optimized for mobile (at the bytecode and interpreter level)

###  Why not Java ME?

– It Costs  - hinders adoption
– Designed by a committee - hard to imagine iOS-like developer appeal
– Apps share a single VM - not great for security sandboxing
– Apps are second-rate citizens - don’t get access to all the hardware

### Dalvik`s optimization for embedded environment

– **Minimal-memory footprint** while providing a **secure sandboxing model**

    * Uncompressed .dex files are smaller than compressed .jar files due to more efficient bytecode
    · On average Dalvik byte code is 30% smaller than JVM byte code
    · Multiple classes in one .dex file
    · Shared constant pool (assumes 32-bit indexes)
    · Simpler class-loading
    · Because it is uncompressed, dex code can be memory-mapped and shared (i.e. mmap()-ed)

    * **Each app runs in a separate instance of Dalvik**
    · At startup, system launches zygote, a half-baked Dalvik process, which is forked any time a new VM is needed
    · Due to copy-on-write support, large sections of the heap are shared (including

– **Register-based fixed-width CPU-optimized byte-code** interpreter

    * Standard JVM bytecode executes **8-bit stack instructions** - local variables must be copied to or from the operand stack by separate instructions

    · Memory speed to CPU speed is amplified on mobile CPUs - we want to minimize access to the main memory

    * Dalvik uses **16-bit instruction set** that works directly on local variables (managed via a 4-bit virtual register field)
    – With JIT support, as of Froyo (2-5x performance improvement in CPU-bound code)

    * Trace-level granularity (more optimal than whole method-level compilations)

    * Fast context-switching (interpreted mode to native mode and back)

    * Well-balanced from performance vs. memory overhead perspective (~ 100-200KB overhead per app)

    * ~ 1:8 ratio of Dalvik to native code (mostly due to optimizations, like inlining)

## Dalvik Startup

### Overview

android系统中，每一个java应用被设计成可以运行在一个单独的Linux进程中。

每个进程都包含一个运行中的dalvik虚拟机实例，用来执行该应用中的java字节码。

很多**java基础类**（比如，java.lang.*)和一些系统级的**共享性资源**（drawable/color...)等几乎会被所有的进程使用。如果每次虚拟机启动后都去加载这些资源，必然是个很大浪费。其实每个虚拟机里加载这些资源的过程都是一模一样的。Android系统就使用一个巧妙的方式[子程序继父程序的资源]解决这个问题。那就是**Zygote**的用处。


### Android 启动流程

![](http://hiphotos.baidu.com/j_fo/pic/item/c782b11b915b0b5a8618bfbb.jpg)

![](http://hi.csdn.net/attachment/201005/14/0_1273850751070U.gif)

### Zygote

[Zygote](zygote.html)


Zygote其实是一个系统级本地服务，在init.rc里定义（如下所示），在系统启动早期被启动起来。它原始名字是app_process
源代码

    frameworks/base/cmds/app_process/app_main.cpp

    system/core/rootdir/int.rc  

    service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
        class main
        socket zygote stream 666
        onrestart write /sys/android_power/request_state wake
        onrestart write /sys/power/state on
        onrestart restart media
        onrestart restart netd

Zygote服务会在启动后加载前文提到基础类和资源，并且建立socket（名称是如上所示的zygote）。

后面其他的java应用程序启动后，AMS（ActivityManagerService，android中用来管理activity的java服务）会通过该管道请求Zygote fock()一个子进程。改子进程继承zygote的内存区，从而直接用于zygote加载的类和资源。用这种方法，运行java应用的linux进程就避免了重复加载android基础类和通用资源，从而极大的节省了时间开销。

![](http://hi.csdn.net/attachment/201203/17/0_1331996001BBLt.gif)


dalvik虚拟机实例启动与zygote直接关联的。



### Dalvik 启动代码分析

Zygote是如何启动Dalvik虚拟机实例？？

**Dalvik启动关闭函数**

    dalvik/vm/Init.h

/*
 * Standard VM initialization, usually invoked through JNI.
 */

    std::string dvmStartup(int argc, const char* const argv[],
            bool ignoreUnrecognized, JNIEnv* pEnv);
    void dvmShutdown(void);

**Dalvik启动函数调用**

dalvik/vm/Jni.cpp  

JNI_CreateJavaVM->

            dvmStartup(argc, argv.get(), args->ignoreUnrecognized, (JNIEnv*)pEnv);


**调用JNI_CreateJavaVM**

dalvik实现部署包含两个部分，**dalvikvm可执行程序** 和 **libdvm共享库**。这个两个调用正是对应了两种调用启动dalvik虚拟机的方式的入口。

- 可执行程序 dalvikvm在main.cpp中执行JNI_CreateJavaVM启动虚拟机实例，

    dalvik/dalvikvm/Main.cpp:    if (JNI_CreateJavaVM(&vm, &env, &initArgs) < 0) {

- 而其他的模块则通过 AndroidRuntime.cpp来启动虚拟机实例。正常流程应该是从Runtime启动

    frameworks/base/services/surfaceflinger/DdmConnection.cpp:    if (JNI_CreateJavaVM(&vm, &env, &args) == 0) {
    frameworks/base/core/jni/AndroidRuntime.cpp:    if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
    frameworks/base/core/jni/AndroidRuntime.cpp:        LOGE("JNI_CreateJavaVM failed\n");

/*
 * Start the Dalvik Virtual Machine.
 *
 * Various arguments, most determined by system properties, are passed in.
 * The "mOptions" vector is updated.
 *
 * Returns 0 on success.
 */

    int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv)
    {
    ....
    if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
        LOGE("JNI_CreateJavaVM failed\n");
        goto bail;
    }
    ....

/*
 * Start the Android runtime.  This involves starting the virtual machine
 * and calling the "static void main(String[] args)" method in the class
 * named by "className".
 *
 * Passes the main function two arguments, the class name and the specified
 * options string.
 */

    void AndroidRuntime::start(const char* className, const char* options)
    {
    ....
    if (startVm(&mJavaVM, &env) != 0) {
        return;
    }
    ....

**在哪里启动AndroidRuntime呢？**

    frameworks/base/cmds/app_process/app_main.cpp 

    int main(int argc, const char* const argv[])
    {
    ....
    AppRuntime runtime;


        if (zygote) {
            runtime.start("com.android.internal.os.ZygoteInit",
                    startSystemServer ? "start-system-server" : "");
        } else if (className) {
    ....

AppRuntime是AndroidRuntime的子类，并且没有它并没有覆盖start()方法，那么这里runtime.start必然是调用了AndroidRuntime的start()方法

start启动的类"com.android.internal.os.ZygoteInit"

它的源码在 

   frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

内容包括Zygote与DVM交互的实现。比如注册socket，加载基础类以及资源等。

![DVM实例启动的全过程](http://hi.csdn.net/attachment/201203/17/0_1331996781qLjc.gif)
## abbreviations

- JIT:  Just-In-Time，运行时编译执行的技术， Java语言即采用该技术
- JNI:　Java Native Interface

## Reference

* [WIKI Page: Dalvik (software)](http://en.wikipedia.org/wiki/Dalvik_(software))
* [Google I/O 2010 - A JIT Compiler for Android's Dalvik VM](http://www.youtube.com/create_account?action_creation_redirect=true&next=%2Fwatch%3Fv%3DLs0tM-c4Vfo%26noredirect%3D1)
* [Dalvik VM Internals - Presentation from Google I/O 2008, by Dan Bornstein](https://sites.google.com/site/io/dalvik-vm-internals)
* [Detailed Dalvik specifications documents](http://www.netmite.com/android/mydroid/dalvik/docs/)
* [The Java Virtual Machine Specification, Second Edition](http://docs.oracle.com/javase/specs/jvms/se5.0/html/VMSpecTOC.doc.html)
* [The Java Language Specification, Third Edition](http://docs.oracle.com/javase/specs/jls/se5.0/html/j3TOC.html)
* [Dalvik VM的启动过程解析](http://blog.csdn.net/rambo2188/article/details/7365249)

