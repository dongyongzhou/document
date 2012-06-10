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


##问题及解决

- [Eclipse](eclipse-problem.html)
