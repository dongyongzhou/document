---
layout: master
title: samba
---

## Knownledge



## 设置Samba服务器中新建文件/目录的权限 

/etc/samba/smb.conf

    create mode = 0644
    force create mode = 0644
    directory mode = 0755
    force directory mode = 0755 



- create mode – 这个配置定义新创建文件的属性。Samba在新建文件时，会把dos文件的权限映射成对应的unix权限，在映射后所得的权限，会与这个参数所定义的值进行与操作。然后再和下面的force create mode进行或操作，这样就得到最终linux下的文件权限。
- force create mode – 见上面的描述。相当于此参数所设置的权限位一定会出现在文件属性中。
- directory mode – 这个配置与create mode参数类似，只是它是应用在新创建的目录上。Samba在新建目录时，会把dos–>linux映射后的文件属性，与此参数所定义的值相与，再和force directory mode相或，然后按这个值去设置目录属性。
- force directory mode – 见上面的描述。相当于此参数中所设置的权限位一定会出现在目录的属性中。

说明一点，上面的create mode和create mask参数是同义词，用哪个都可以；而directory mode和directory mask参数是相同的。



### 问题:文件权限属性对，但文件夹不然。？