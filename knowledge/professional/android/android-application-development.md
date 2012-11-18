---
layout: master
title: android-application-development
---

## OverView

1. Create a normal Anroid Application

2. Create a Service Application

3. Create a system apk


## Create a system apk

###a AndroidManifest.xml

    package="com.xxx.autotriggersystemtask"
    android:sharedUserId="android.uid.system"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="15" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <service
            android:name=".AutoTriggerSystemService"
            android:enabled="true" >
            <intent-filter>
                <action android:name="com.xxx.AutoTriggerSystemTask" />
            </intent-filter>
        </service>
    </application>

###b AndroidManifest.xml

### Problems

1. A resource exists with a different case

The project was not built due to "A resource exists with a different case: '/AutoTriggerSystemTask/bin/classes/com/xxx/Autotriggersystemtask'.". Fix the problem, then try refreshing this project and building it since it may be inconsistent


Solution: package name 在AndroidManifest和实际文件中的大小写不一致

    package="com.xxx.autotriggersystemtask"

package com.xxx.Autotriggersystemtask;

2. INSTALL_FAILED_SHARED_USER_INCOMPATIBLE

Details

     [2012-10-20 11:10:39 - AutoTriggerSystemTask] Installing AutoTriggerSystemTask.apk...
     [2012-10-20 11:10:40 - AutoTriggerSystemTask] Installation error: INSTALL_FAILED_SHARED_USER_INCOMPATIBLE
     [2012-10-20 11:10:40 - AutoTriggerSystemTask] Please check logcat output for more details.
     [2012-10-20 11:10:40 - AutoTriggerSystemTask] Launch canceled!


[Reference](http://blog.csdn.net/happyhell/article/details/5903389)

Android.mk


     LOCAL_PATH:= $(call my-dir)
     include $(CLEAR_VARS)

     LOCAL_MODULE_TAGS := debug

     LOCAL_SRC_FILES := $(call all-java-files-under, src)

     LOCAL_PROGUARD_FLAG_FILES := proguard.flags(unneccessary )

     LOCAL_PACKAGE_NAME :=  AutoTriggerSystemTask
     LOCAL_CERTIFICATE := platform

     include $(BUILD_PACKAGE)

     include $(BUILD_MULTI_PREBUILT)(unneccessary )

     include $(call all-makefiles-under, $(LOCAL_PATH))



## Reference

[Android文件或文件夹内容改变监听器（FileObserver）](http://blog.csdn.net/mayingcai1987/article/details/6210904)

