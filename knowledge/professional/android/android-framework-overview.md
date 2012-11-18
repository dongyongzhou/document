---
layout: master
title: Android framework overview
---

## Overview

Framework定义了客户端和服务端的组件功能以及接口。


### 客户端

Activity类等

### 服务端

两个重要的类

- WindowManagerService（WmS）: 为所有的应用程序分配窗口，并管理这些窗口。大小、叠放次序、隐藏或显示窗口。
- ActivityManagerService (AmS)： 管理所有应用程序中的Activity

两个消息处理类

- KeyQ类： WmS内部类，继承于KeyInputQueue。创建KeyQ对象后，立即启动一个线程，不断读取用户的UI操作消息，比如按键、触摸屏等，并放到消息队列QueueEvent中。
- InputDispatcherThread：也会创建一个线程，不断从消息队列QueueEvent中取出用户消息，过滤后发送给当前活动的客户端程序。

### Linux驱动

包括SurfaceFlinger和Binder，每一个窗口对应一个Surface,SF 驱动作用是把各个Surface显示到同一个屏幕上。