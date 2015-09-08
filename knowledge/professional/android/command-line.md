---
layout: master
title: command-line
---

## Overview

## Getprop

adb device, and the device is ro.boot.serialno

	cat /sys/class/android_usb/android0/iSerial
	or getprop ro.boot.serialno


## Commands

### Command to control App

adb shell am start -n package/package.activity 

	  **package**="com.xxx.yyy">
	  <application
	    android:name=".CrashLogApp"
	    android:icon="@drawable/xxxx"
	    android:label="@string/xxxxxx">
	    <**activity**
	      android:name=".yyy"
	      android:label="@string/app_name">


## Reference


