---
layout: master
title: shell command
---

## Reference

* [鸟哥的Linux私房菜简体首页](http://vbird.dic.ksu.edu.tw/)


## 帐号管理

### 给已有的用户增加工作组

usermod -G groupname username

或者：gpasswd -a user group

### 有效群组的切换

newgrp groupname 


### 修改用户所属的默认组

username -g　groupname username

### 增加群组

 建立一个群组名为“testgroup"，设置"abc"为管理员

	[root@localhost fsy]# groupadd testgroup
	[root@localhost fsy]# gpasswd testgroup
	Changing the password for group testgroup
	New Password: 
	Re-enter new password: 
	[root@localhost fsy]# gpasswd -A abc testgroup

username -g　groupname username
## 文件操作



