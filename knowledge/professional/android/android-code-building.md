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


## Android Source Code Build Result

## Reference

- [http://tools.android.com/tech-docs/new-build-system](http://tools.android.com/tech-docs/new-build-system)
- [http://tools.android.com/tech-docs/new-build-system/build-system-concepts](http://tools.android.com/tech-docs/new-build-system/build-system-concepts)
- [https://github.com/android/platform_build/blob/master/core/build-system.html](https://github.com/android/platform_build/blob/master/core/build-system.html)
