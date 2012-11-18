---
layout: master
title: about-system-rw-SDCARD-permission
---

## OverView

Processes that continue holding open fds on the sdcard a little after it is  
requested to be unmounted will be killed so that it can unmount.  

We don't want the system process to be able to access the sdcard to avoid  
these kinds of issues (and just general security cleanliness), so that it  
does not have permission to access it.  

## 设置SDCARD权限原生位置

Android1.6

/system/core/vold/volmgr_vfat.c

rc = mount(devpath, vol->mount_point, "vfat", flags,"utf8,uid=1000,gid=1000,fmask=711,dmask=700,shortname=mixed");


android2.2+

/system/core/vold/Volume.cpp 文件

doMount(devicePath, path, false, false, false,1000, 1015, 0702, true)改为

