---
layout: master
title: Development Environment Setup
---

##Overview


### smbmount for windows Dropbox in Linux

If no smbmount. install it :

    $ sudo apt-get install smbfs

Mount dropbox

    $ mkdir dropbox
    $ sudo smbmount //<ip>/Dropbox dropbox/ -o user=<username>
    Password:<password of username>

The commands will create a mount point on the Linux machine that mirrors the Dropbox on <username> windows machine.


### NFS

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


##问题及解决

- [Eclipse](eclipse-problem.html)
