---
layout: master
title: Android Source Code Building
---

##1 Android Source Code Build Command

###1.1 Linux Kernel 

**clean mrproper distclean**

    # Cleaning is done on three levels.
    # make clean     Delete most generated files
    #                Leave enough to build external modules
    # make mrproper  Delete the current configuration, and all generated files
    # make distclean Remove editor backup files, patch leftover files and the like

http://en.wikipedia.org/wiki/Mr._Clean

 "make mrproper" is a command in the Linux kernel build system, used to "clean up" all files from past builds and restore the build directory to its original clean state. The reason "make mrproper" is used instead of "make mrclean" is because Linus Torvalds, the father of Linux, was familiar with the name "Mr. Proper" as this is the brand widely known in Europe.

## Android Source Code Build Result

## Reference
