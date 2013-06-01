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

##问题及解决

- [Eclipse](eclipse-problem.html)
