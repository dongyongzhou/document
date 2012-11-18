---
layout: master
title: Android Source Code Structure
---


## Android Source Code Structure

* bionic/ - the home of the Bionic (libc) library
* bootable/ - the home of Android’s bootloader, diskinstaller, and recovery image support
* build/ - the home of the Android build system
* cts/ - Android’s Compatibility Test Suite
– See http://source.android.com/compatibility/cts-intro.html for more info.
* dalvik/ - the home of the Dalvik VM
* development/ - development tools, configuration files and sample apps
* device/ - device-specific binaries (like kernel and device drivers) and source
–Gingerbread tree supports HTC Nexus One and Samsung Nexus S
* external/ - 3rd party libraries (mostly native, but also Java), which are synced from their own repositories
* frameworks/ - Android-specific native utilities (e.g. app_process, bootanimation, etc.), daemons (e.g.
installd, servicemanager, system_server), and libraries (including JNI wrappers and HAL support), as
well as Java APIs (i.e. all of android.*) and services (all all Application Framework support)

android.util.Log: frameworks/base/core/java/android/util/Log.java

* hardware/ - hardware-abstraction-layer (HAL) definitions (libhardware and libhardware_legacy) and
some device-specific implementations (e.g. msm7k’s libaudio.so and TI OMAP3’s libstagefrighthw.so)
both in source code and binaries
* libcore/ - Apache Harmony (see libcore/luni/src/main/java/) as well as test/support libraries
* ndk/ - the home of NDK
* out/ - the location where binaries built by make go
* packages/ - the home of the built-in applications (e.g. Phone, Browser, Gallery, etc), content providers (e.g. Contact
Provider, Media Provider, etc.), wall papers (including live wall papers), input methods (e.g. LatinIME), etc.
* prebuilt/ - pre-built kernels (mostly for QEMU) as well as other binaries (mostly 3rd party development tools)
* sdk/ - the home of Android SDK tools (ddms, traceview, ninepatch, etc.)
* system/ - the home of the Android root file system, configuration files, init and init.rc, as well as some of the
native daemons


### Android Kernel Module Path

/system/lib/modules/

### Android insmod Kernel Module file 

/system/core/rootdir/etc/init.xxx.rc

    on boot
        ....
        insmod /system/lib/modules/xxx.ko
