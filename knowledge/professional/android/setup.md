---
layout: master
title: Setting up the building environment
---


# Setting up the building environment

## Requirement

* Linux (specifically Ubuntu 10.04 or later) and Mac OS X are officially supported
* Windows is explicitly not supported
– Approximately 12GB of disk space (2.6GB for sources and 10GB for a complete build)
– A case-sensitive file-system
* Most Unix/Linux-based file systems are already case-sensitive
* Mac OS Extended (HFS+) file system is case-insensitive by default - we need to create "case sensitive, journaled"
volume (e.g. via "Disk Utility")
– Python 2.4—2.7
– Java Development Kit (official Sun/Oracle release is recommended)
* JDK 5 for( Froyo
* JDK 6 for >= Gingerbread
– Git 1.5.4 or newer
– Build tools and libraries: flex, bison, gcc, make, zlib, libc-dev, ncurses, 32-bit dev libs, etc.
– See http://source.android.com/source/initializing.html for the easy copy+paste installation instructions for the required
binaries

## Setup

**Install necessary building tools**

`sudo apt-get install samba apt-file smbfs curl subversion git-core binutils binutils-dev kernel-package build-essential gcc make ant autoconf automake autogen flex bison lemon sed gawk expect patch doxygen perl padre python eric ruby linux-image-generic gnupg gperf libsdl-dev libesd0-dev libwxgtk2.6-dev libncurses5-dev zlib1g-dev libc6-dev texinfo zlib1g-dev zip x11proto-core-dev libx11-dev libgl1-mesa-dev g++-multilib mingw32 tofrodos python-software-properties`

**Install Repo**

    $ mkdir ~/bin
    $ export PATH=~/bin:$PATH
    $ curl http://android.git.kernel.org/repo > ~/bin/repo
    $ chmod a+x ~/bin/repo
## Problems

### /usr/bin/ld: cannot find -lz

> /usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/gcc/x86_64-linux-gnu/4.5.2/../../../libz.so when searching for -lz
/usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/gcc/x86_64-linux-gnu/4.5.2/../../../libz.a when searching for -lz
/usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/libz.so when searching for -lz
/usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/libz.a when searching for -lz
/usr/bin/ld: cannot find -lz
collect2: ld returned 1 exit status

Solution:
sudo apt-get install lib32z1-dev

### /usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory
compilation terminated.

Solution:
sudo apt-get install libc6-dev-i386

### /usr/bin/ld: cannot find -lncurses

>/usr/bin/ld: skipping incompatible /usr/lib/libncurses.so when searching for -lncurses
/usr/bin/ld: skipping incompatible /usr/lib/libncurses.a when searching for -lncurses
/usr/bin/ld: cannot find -lncurses
collect2: ld returned 1 exit status

Solution:

sudo apt-get install libncurses5-dev

or

sudo apt-get install lib32ncurses5-dev


### Error occurred during initialization of VM

Buld Error 7:
target Dex: core-hostdex
Error occurred during initialization of VM
Could not reserve enough space for object heap
Could not create the Java virtual machine.
make: *** [out/host/common/obj/JAVA_LIBRARIES/core-
hostdex_intermediates/classes.dex] Error 1
make: *** Waiting for unfinished jobs....

solution:

build/core/definitions.mk:1528: $(if $(findstring windows,$(HOST_OS)),,-JXms16M -JXmx**2048**M) \   2048->1536

