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


## 文件操作



