---
layout: master
title: Android Source Code Building
---

##1 Android Source Code Build Command

###1.1 Linux Kernel 

**Where is Android Kernel config included**

/device/vendor/board/AndroidBoard.mk

    ifeq ($(KERNEL_DEFCONFIG),)
        KERNEL_DEFCONFIG := board_defconfig
    endif
    include kernel/AndroidKernel.mk

 
**clean mrproper distclean**

    # Cleaning is done on three levels.
    # make clean     Delete most generated files
    #                Leave enough to build external modules
    # make mrproper  Delete the current configuration, and all generated files
    # make distclean Remove editor backup files, patch leftover files and the like

http://en.wikipedia.org/wiki/Mr._Clean

 "make mrproper" is a command in the Linux kernel build system, used to "clean up" all files from past builds and restore the build directory to its original clean state. The reason "make mrproper" is used instead of "make mrclean" is because Linus Torvalds, the father of Linux, was familiar with the name "Mr. Proper" as this is the brand widely known in Europe.

###1.2 Android

build/core/main.mk

     # -------------------------------------------------------------------
     # This is used to to get the ordering right, you can also use these,
     # but they're considered undocumented, so don't complain if their
     # behavior changes.

     .PHONY: prebuilt
     prebuilt: $(ALL_PREBUILT)

     # An internal target that depends on all copied headers
     # (see copy_headers.make).  Other targets that need the
     # headers to be copied first can depend on this target.

     .PHONY: all_copied_headers
     all_copied_headers: ;

     # All the droid stuff, in directories
     .PHONY: files
     files: prebuilt \
             $(modules_to_install) \
             $(modules_to_check) \
             $(INSTALLED_ANDROID_INFO_TXT_TARGET)
     
     # -------------------------------------------------------------------

     .PHONY: checkbuild
     checkbuild: $(modules_to_check)

     .PHONY: ramdisk
     ramdisk: $(INSTALLED_RAMDISK_TARGET)

     .PHONY: systemtarball
     systemtarball: $(INSTALLED_SYSTEMTARBALL_TARGET)

     .PHONY: boottarball
     boottarball: $(INSTALLED_BOOTTARBALL_TARGET)

     .PHONY: userdataimage
     userdataimage: $(INSTALLED_USERDATAIMAGE_TARGET)

     ifneq (,$(filter userdataimage, $(MAKECMDGOALS)))
     $(call dist-for-goals, userdataimage, $(BUILT_USERDATAIMAGE_TARGET))
     endif

     .PHONY: userdatatarball
     userdatatarball: $(INSTALLED_USERDATATARBALL_TARGET)

     .PHONY: persistimage
     persistimage: $(INSTALLED_PERSISTIMAGE_TARGET)

     .PHONY: persisttarball
     persisttarball: $(INSTALLED_PERSISTTARBALL_TARGET)

     .PHONY: bootimage
     bootimage: $(INSTALLED_BOOTIMAGE_TARGET)

     .PHONY: cacheimage
     cacheimage: $(INSTALLED_CACHEIMAGE_TARGET)

     .PHONY: tombstonesimage
     tombstonesimage: $(INSTALLED_TOMBSTONESIMAGE_TARGET)

     .PHONY: aboot
     aboot: $(INSTALLED_BOOTLOADER_TARGET)

### 1.3 Building command 

Source ./build/envsetup.sh

      - croot: Changes directory to the top of the tree.

      - m: Makes from the top of the tree.
      - mm: Builds all of the modules in the current directory.
      - mmm: Builds all of the modules in the supplied directories.
      - cgrep: Greps on all local C/C++ files.
      - jgrep: Greps on all local Java files.
      - resgrep: Greps on all local res/*.xml files.
      - godir: Go to the directory containing a file.


- make systemimage: system.img
- make userdataimage: userdata.img
- make ramdisk: ramdisk.img
- make snod : package ../system into system.img (with this command, it will build a new system.img very quickly. well, you cannot use “make snod” for all the situations. it would not check the dependences. if you change some code in the framework which will effect other applications)

## Android Source Code Build Result

- out/target/product/generic/system/app: Android系统自带的App
- out/target/product/generic/system/bin: Android系统的一些可执行文件，例如C编译的可执行文件
- out/target/product/generic/system/lib: 动态链接库文件
- out/target/product/generic/system/lib/hw: 硬件抽象层（HAL）接口文件

## Reference

- [http://tools.android.com/tech-docs/new-build-system](http://tools.android.com/tech-docs/new-build-system)
- [http://tools.android.com/tech-docs/new-build-system/build-system-concepts](http://tools.android.com/tech-docs/new-build-system/build-system-concepts)
- [https://github.com/android/platform_build/blob/master/core/build-system.html](https://github.com/android/platform_build/blob/master/core/build-system.html)
