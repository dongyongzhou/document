---
layout: master
title: Development Environment Setup
---

##1 Overview


##2 Setting external mounting storage

##2.1 smbmount for windows Dropbox in Linux

If no smbmount. install it :

    $ sudo apt-get install smbfs

Mount dropbox

    $ mkdir dropbox
    $ sudo smbmount //<ip>/Dropbox dropbox/ -o user=<username>
    Password:<password of username>

The commands will create a mount point on the Linux machine that mirrors the Dropbox on <username> windows machine.


###2.2 NFS

#### install nfs

- apt-get install nfs-kernel-server
- apt-get install portmap nfs-common

#### /etc/exports
- <nfs path>  *(rw,sync,no_root_squash)

#### restart nfs

- sudo /etc/init.d/nfs-kernel-server restart
- sudo /etc/init.d/portmap restart

#### usage

mount

- sudo mount -t nfs -o rw <nfs server ip>:<path-to-be-mounted> <path-to-mount-to>


#### more

- /etc/mtab: 记载的是现在系统已经装载的文件系统，包括操作系统建立的虚拟文件等；
- /etc/fstab: 是系统准备装载的。直接使用mount和确定就是通过查询它而来的。

#### Mount NFS to Win7

REFERENCE:

[UNIX 到 Windows 的迁移：如何在 Windows 上安装 NFS 客户端](http://www.kutea.net/60.html)

1. Win7 has NFS services.

2. Control panel -> programs -> Turn windows feature on or off -> select Services for NFS
	
	
		Add Adminstrative tools
		Add client for NFS

After installation, 

it is in control panel -> manage tool

Control Panel\System and Security\Administrative Tools
\Services for Network File Sysem
3. show remote NFSs

	查看远程服务器上的 NFS 共享目录
	运行命令提示符
	> showmount -e HOSTNAME
	(导出列表在 HOSTNAME:
	/raid0/data/files)

4. mount

 After the installation start a dos box or power shell. And enter the following command to mount the share backup on server 192.168.1.1 and assign the drive letter k:

mount \\192.168.1.1\nfspath k:

Example: mount [options] \\nfs-server\unc-nameshare-name [drive letter]


##3 设置SecureCRT

font: consolas 16pt

Monochrome

Backgroud:

	Hue色调:85
	sat饱和度:123
	Lum亮度:205
	Red:204
	Green:232
	Blue:207

Emulation:

Terminal:Xterm

select ANSI Color
select Use color scheme

##4 setup adb on Ubuntu

### problem
	xxx@xxx-gv:~$ adb devices
	List of devices attached 
	????????????    no permissions
	
	xx@xxx-gv:~$ adb shell
	error: insufficient permissions for device
	dongyong@dongyong-gv:~$ lsusb
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
	Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
	Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
	Bus 003 Device 044: ID 05c6:9025 xxx, Inc. 

### solutions

Bus 003 Device 044: ID 05c6:9025 xxx, Inc. 
	
	$sudo vim /etc/udev/rules.d/70-android.rules	
	SUBSYSTEM=="usb", ATTRS{idVendor}=="05c6", ATTRS{idProduct}=="9025",MODE="0666"

	$sudo chmod 777 /etc/udev/rules.d/70-android.rules	
	$sudo service udev restart	
	sudo ./adb kill-server	
	./adb devices
	./adb root 	

##4 setup adb on windows

Thinking of the VID/PID has or have been changed.

Reference for adb device

USB\VID_05C6&PID_9025&REV_0228&MI_01
USB\VID_05C6&PID_9025&MI_01

for new one

USB\VID_271D&PID_9025&REV_0228&MI_01


1. find android_winusb.inf under adb driver, Add as follow:

%CompositeAdbInterface%     = USB_Install, USB\VID_271D&PID_9025&MI_01

2. 右击我的电脑->属性->高级->环境变量，添加 ANROID_SDK_HOME 环境变量。如果你有 android SDK, 就设成 SDK 的路径；如果没有，那也没关系，设为你觉得方便的任何路径。
3. 在前面设置的 ANDROID_SDK_HOME 对应的路径下，寻找 .android 目录，如果没有就创建一个；在 .android 目录下新建一个文件，叫adb_usb.ini, 记住，后缀是 "ini" 哦；添加前面获得的 VID 到 adb_usb.ini 中，如 0xAAAA。
4. install adb driver with the specific path.
5. dos cmd, adb kill-server, adb start-server, adb devices；
6. adb devices 如果没显示。那么把2.的“adb_usb.ini” 拷贝到“C:\Users\<username>\.android”

##问题及解决

- [Eclipse](eclipse-problem.html)
