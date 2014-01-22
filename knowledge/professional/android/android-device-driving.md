---
layout: master
title: android-device-driving
---

## Overview


![](http://hi.csdn.net/attachment/201201/14/0_13265292792eh8.gif)

- Linux device driver
- Android device HAL
- Android device JNI
- Android device service
- Android device manager
- Android device APP

## Linux device driver

驱动程序的功能主要是向上层提供访问设备的寄存器的值，包括读和写。

### 接口机制

- 一是通过procfs文件系统来访问
	
	procfs读写格式不一样，代表不同的操作，应用程序中读到了这个文件的内容一般还需要进行字符串解析，而在写入时需要先用字符串 格式化按指定的格式写入字符串进行操作
- 二是通过传统的设备文件的方法来访问

	file_operations： read、write、IOCTL、llseek...
- 三是通过sysfs文件系统来访问
	
	sysfs属性文件, 设计原则是一个属性文件只做一件事情， sysfs 属性文件一般只有一个值，直接读取或写入。系统化

### Device Class

- input_dev
- timed_output_class

## Android device HAL
Android的硬件抽象层，简单来说，就是对Linux内核驱动程序的封装，向上提供接口，屏蔽低层的实现细节。

对硬件的支持分成了两层，一层放在用户空间（User Space），一层放在内核空间（Kernel Space），其中，硬件抽象层运行在用户空间，而Linux内核驱动程序运行在内核空间。

内核驱动层只提供简单的访问硬件逻辑，例如读写硬件寄存器的通道，至于从硬件中读到了什么值或者写了什么值到硬件中的逻辑，都放在硬件抽象层中去了。

Linux内核源代码版权遵循GNU License，而Android源代码版权遵循Apache License，前者在发布产品时，必须公布源代码，而后者无须发布源代码。这样就可以把商业秘密隐藏起来了。

也正是由于这个分层的原因，Android被踢出了Linux内核主线代码树中。Android放在内核空间的驱动程序对硬件的支持是不完整的，把Linux内核移植到别的机器上去时，由于缺乏硬件抽象层的支持，硬件就完全不能用了，这也是为什么说Android是开放系统而不是开源系统的原因。



Android的HAL的实现需要通过JNI(Java Native Interface)，JNI简单来说就是java程序可以调用C/C++写的动态链接库，这样的话，HAL可以使用C/C++语言编写，效率更高。

### libhardware_legacy(old)
libhardware_legacy 是将 *.so 文件当作shared library来使用，在runtime（JNI 部份）以 direct function call 使用 HAL module。通过直接函数调用的方式，来操作驱动程序。

JNI->libhardware_legacy.so->内核驱动接口

- hardware/libhardware_legacy

### Stub(New)
HAL stub 是一种代理人（proxy）的概念，stub 虽然仍是以 *.so檔的形式存在，但HAL已经将 *.so 档隐藏起来了。Stub 向 HAL提供操作函数（operations），而 runtime 则是向 HAL 取得特定模块（stub）的 operations，再callback 这些操作函数。这种以 indirect function call 的架构，让HAL stub 变成是一种包含关系，即 HAL 里包含了许许多多的 stub（代理人）。Runtime 只要说明类型，即 module ID，就可以取得操作函数。对于目前的HAL，可以认为Android定义了HAL层结构框架，通过几个接口访问硬件从而统一了调用方式。

JNI->通用硬件模块->硬件模块->内核驱动接口
JNI->libhardware.so->xxx.xxx.so->kernel，具体来说：android frameworks中JNI调用hardware.c中定义的hw_get_module函数来获取硬件模块，然后调用硬件模块中的方法，硬件模块中的方法直接调用内核接口完成相关功能

- hardware/libhardware/->libhardware.so
/hardware/libhardware/include/hardware/hardware.h

- Framework/base/services/input
	system/lib/libinput.so


### Add device HAL
1. 在hardware/libhardware/include/hardware目录，新建xxxhal.h文件

2. 新建xxxhal.h中分别定义模块ID、模块结构体以及硬件接口结构体。
在硬件接口结构体中，fd表示设备文件描述符，对应我们将要处理的设备文件，以及为该HAL对上提供的函数接口。

3. 在hardware/libhardware/modules目录，新建xxx目录，并添加xxx.c文件。
实例变量名必须为HAL_MODULE_INFO_SYM，tag也必须为HARDWARE_MODULE_TAG，这是Android硬件抽象层规范规定的。

4. 定义编译规则。
LOCAL_MODULE的定义规则，hello后面跟有default，xxx.default能够保证我们的模块总能被硬象抽象层加载到。


      
        LOCAL_PATH := $(call my-dir)
        include $(CLEAR_VARS)
        LOCAL_MODULE_TAGS := optional
        LOCAL_PRELINK_MODULE := false
        LOCAL_MODULE_PATH := $(TARGET_OUT_SHARED_LIBRARIES)/hw
        LOCAL_SHARED_LIBRARIES := liblog
        LOCAL_SRC_FILES := xxx.c
        LOCAL_MODULE := xxx.default
        include $(BUILD_SHARED_LIBRARY)

编译成功后，就可以在out/target/product/xxxxx/system/lib/hw目录下看到xxx.default.so文件了

## Android device JNI

Android系统的应用程序是用Java语言编写的，而硬件驱动程序是用C语言来实现的，Java提供了JNI方法调用去访问C接口，同样，在Android系统中，Java应用程序通过JNI来调用硬件抽象层接口。
### Location

frameworks/base/services/jni

CPP

    LOCAL_SRC_FILES:= \
    com_android_server_AlarmManagerService.cpp \
    com_android_server_BatteryService.cpp \
    com_android_server_input_InputApplicationHandle.cpp \
    com_android_server_input_InputManagerService.cpp \
    com_android_server_input_InputWindowHandle.cpp \
    com_android_server_LightsService.cpp \
    com_android_server_power_PowerManagerService.cpp \
    com_android_server_SerialService.cpp \
    com_android_server_SystemServer.cpp \
    com_android_server_UsbDeviceManager.cpp \
    com_android_server_UsbHostManager.cpp \
    com_android_server_VibratorService.cpp \
    com_android_server_location_GpsLocationProvider.cpp \
    com_android_server_connectivity_Vpn.cpp \
    onload.cpp
### Compile

mm

生成什么？：：system/lib/libandroid_servers.so

### Add new device JNI
1. frameworks/base/services/jni 新建com_android_server_xxxService.cpp文件
实现JNI方法。注意文件的命令方法，com_android_server前缀表示的是包名，表示硬件服务xxxService是放在frameworks/base/services/java目录下的com/android/server目录的，即存在一个命令为com.android.server.xxxService的类。

2. 头文件、定义JNI方法,JNI方法表

3. 修改同目录下的onload.cpp文件，首先在namespace android增加register_android_server_xxxService函数声明。这样，在Android系统初始化时，就会自动加载该JNI方法调用表。

4.  修改同目录下的Android.mk文件，在LOCAL_SRC_FILES变量中增加一行：

  LOCAL_SRC_FILES:= \

      com_android_server_xxxService.cpp /

## Android device Framework
Application Frameworks层为其提供硬件服务

在Android系统中，硬件服务一般是运行在一个独立的进程中为各种应用程序提供服务。因此，调用这些硬件服务的应用程序与这些硬件服务之间的通信需要通过代理来进行。为此，我们要先定义好通信接口。

### Android framework service

1. AIDL

frameworks/base/Android.mk

## Add device framework service

1. 进入到frameworks/base/core/java/android/os目录，新增IxxxService.aidl接口定义文件：

2. 返回到frameworks/base目录，打开Android.mk文件，修改LOCAL_SRC_FILES变量的值，新增IxxxService.aidl源文件.
就会根据IHelloService.aidl生成相应的IxxxService.Stub接口。

3. 进入到frameworks/base/services/java/com/android/server目录，新增xxxService.java文件

4. 修改同目录的SystemServer.java文件，在ServerThread::run函数中增加加载xxxService的代码：
ServiceManager.addService("xxx", new xxxService());

5. building
mmm frameworks/base/services/java; make snod

## Android Manager

在Android下访问HAL大致有以下两种方式：

- Android的app可以直接通过service调用.so格式的jni
- 经过Manager调用service


## Android APP

应用程序通过ServiceManager接口获取指定的服务，然后通过这个服务来获得硬件服务。

程序通过ServiceManager.getService("xxx")来获得xxxService，接着通过IxxxService.Stub.asInterface函数转换为IHelloService接口。其中，服务名字“xxx”是系统启动时加载xxxService时指定的，而IxxxService接口定义在android.os.IxxxService中.

## Reference

* [Android硬件抽象层（HAL）概要介绍和学习计划](http://blog.csdn.net/luoshengyang/article/details/6567257)
* [Android Hal 分析](http://www.cnblogs.com/armlinux/archive/2012/01/14/2396768.html)

