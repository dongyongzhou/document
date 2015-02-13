---
layout: master
title: Linux Power management
---

## Overview

## regulator机制
KERNEL/Documentation/Power/Regulator


### regulator 硬件

### regulator结构体

Regulator与模块之间是树状关系。父regulator可以给设备供电，也可以给子regulator供电：

### regulator 注册过程
### 设备使用regulator过程

## APM

APM=Advanced Power Management

分两部分：

- 电源管理部分
- 设备驱动部分

### 设备驱动部分

驱动模块的suspend和resume

### 电源管理部分

Core级别

主要结构体： pm_ops


## 低功耗管理

- CPU级: 
10ms任务调度 idle唤醒。 设100ms牺牲响应速度

主频动态管理

- 驱动设备级: sys控制

用户态管理设备驱动

- 系统平台级别： apm
 电源状态改变

## CPU变频

系统根据系统负载情况动态调节CPU

### CPUFREQ内核子系统

内核变频管理策略：
userspace, conservative, ondemand, powersave, performance


## Reference

1. http://www.ibm.com/developerworks/cn/linux/l-power/
2. 