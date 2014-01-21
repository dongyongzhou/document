---
layout: master
title: android-device-driving
---

# OverView

对硬件的支持分成了两层，一层放在用户空间（User Space），一层放在内核空间（Kernel Space），其中，硬件抽象层运行在用户空间，而Linux内核驱动程序运行在内核空间。
为什么要这样安排呢？把硬件抽象层和内核驱动整合在一起放在内核空间不可行吗？从技术实现的角度来看，是可以的，然而从商业的角度来看，把对硬件的支持逻辑都放在内核空间，可能会损害厂家的利益。我们知道，Linux内核源代码版权遵循GNU License，而Android源代码版权遵循Apache License，前者在发布产品时，必须公布源代码，而后者无须发布源代码。如果把对硬件支持的所有代码都放在Linux驱动层，那就意味着发布时要公开驱动程序的源代码，而公开源代码就意味着把硬件的相关参数和实现都公开了，在手机市场竞争激烈的今天，这对厂家来说，损害是非常大的。因此，Android才会想到把对硬件的支持分成硬件抽象层和内核驱动层，内核驱动层只提供简单的访问硬件逻辑，例如读写硬件寄存器的通道，至于从硬件中读到了什么值或者写了什么值到硬件中的逻辑，都放在硬件抽象层中去了，这样就可以把商业秘密隐藏起来了。

也正是由于这个分层的原因，Android被踢出了Linux内核主线代码树中。大家想想，Android放在内核空间的驱动程序对硬件的支持是不完整的，把Linux内核移植到别的机器上去时，由于缺乏硬件抽象层的支持，硬件就完全不能用了，这也是为什么说Android是开放系统而不是开源系统的原因。

撇开这些争论，学习Android硬件抽象层，对理解整个Android整个系统，都是极其有用的，因为它从下到上涉及到了Android系统的硬件驱动层、硬件抽象层、运行时库和应用程序框架层等等，下面这个图阐述了硬件抽象层在Android系统中的位置，以及它和其它层的关系：

- Linux device driver
- Android device HAL
- Android device JNI
- Android device Framework
- Android device APP

# Linux device driver

驱动程序的功能主要是向上层提供访问设备的寄存器的值，包括读和写。
提供了三种访问设备寄存器的方法

- 一是通过proc文件系统来访问
- 二是通过传统的设备文件的方法来访问
- 三是通过devfs/sysfs文件系统来访问

# Android device HAL
Android的硬件抽象层，简单来说，就是对Linux内核驱动程序的封装，向上提供接口，屏蔽低层的实现细节。
## Android standard device HALs

- Framework/base/services/input
- hardware/libhardware/modules
- hardware/libhardware_legacy

## Add device HAL
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

# Android device JNI

Android系统的应用程序是用Java语言编写的，而硬件驱动程序是用C语言来实现的，Java提供了JNI方法调用去访问C接口，同样，在Android系统中，Java应用程序通过JNI来调用硬件抽象层接口。
## Location

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
## Compile

mm

生成什么？：：system/lib/libandroid_servers.so

## Add new device JNI
1. frameworks/base/services/jni 新建com_android_server_xxxService.cpp文件
实现JNI方法。注意文件的命令方法，com_android_server前缀表示的是包名，表示硬件服务xxxService是放在frameworks/base/services/java目录下的com/android/server目录的，即存在一个命令为com.android.server.xxxService的类。

2. 头文件、定义JNI方法,JNI方法表

3. 修改同目录下的onload.cpp文件，首先在namespace android增加register_android_server_xxxService函数声明。这样，在Android系统初始化时，就会自动加载该JNI方法调用表。

4.  修改同目录下的Android.mk文件，在LOCAL_SRC_FILES变量中增加一行：

  LOCAL_SRC_FILES:= \

      com_android_server_xxxService.cpp /

# Android device Framework
Application Frameworks层为其提供硬件服务

在Android系统中，硬件服务一般是运行在一个独立的进程中为各种应用程序提供服务。因此，调用这些硬件服务的应用程序与这些硬件服务之间的通信需要通过代理来进行。为此，我们要先定义好通信接口。

## Android framework services

1. AIDL

frameworks/base/Android.mk

## Add device framework service

1. 进入到frameworks/base/core/java/android/os目录，新增IxxxService.aidl接口定义文件：

2. 返回到frameworks/base目录，打开Android.mk文件，修改LOCAL_SRC_FILES变量的值，新增IxxxService.aidl源文件.
就会根据IHelloService.aidl生成相应的IxxxService.Stub接口。

3. 进入到frameworks/base/services/java/com/android/server目录，新增xxxService.java文件

4. 修改同目录的SystemServer.java文件，在ServerThread::run函数中增加加载xxxService的代码：
ServiceManager.addService("xxx", new xxxService());
# Reference

5. building
mmm frameworks/base/services/java; make snod

# Android APP

应用程序通过ServiceManager接口获取指定的服务，然后通过这个服务来获得硬件服务。

程序通过ServiceManager.getService("xxx")来获得xxxService，接着通过IxxxService.Stub.asInterface函数转换为IHelloService接口。其中，服务名字“xxx”是系统启动时加载xxxService时指定的，而IxxxService接口定义在android.os.IxxxService中.

# Reference

*[Android硬件抽象层（HAL）概要介绍和学习计划](http://blog.csdn.net/luoshengyang/article/details/6567257)

