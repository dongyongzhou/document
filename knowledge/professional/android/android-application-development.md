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


3. How to solve Error: This attribute must be localized. (at 'text' with value 'BOTTOM_LEFT')

**Decription**

xml

    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="BOTTOM_LEFT" />

**Analysis**

Best practice for Android apps is to define all of the non-dynamic content in resource files. This lets you define different resource files for different languages, for example. Normally, this is just a recommendation and the Android SDK doesn't complain if you hard-code values in your layout xml. The Android source build system, however, requires that all strings be defined in a "values" resource. This is probably intended to protect system builders from accidentally leaving content in a system image that won't display in the user's chosen language.


**solution 1**

 use

	LOCAL_MODULE_TAGS := tests
in the Android.mk to omits the localization check.

Another way is to disable localization check in build system. Comment the line 81 in build/core/package.mk

	#LOCAL_AAPT_FLAGS := $(LOCAL_AAPT_FLAGS) -z

**solution 2**

What you need to do is move those string values out of the layout and define them in res/values/ instead. The usual place for string values is in res/values/strings.xml, but the actual file can be named anything you like as long as it's in that directory.

For example, in res/values/string.xml:

<string name="topLeftContent">TOP_LEFT</string>
And in your main.xml layout, refer to the content by name:

    android:text="@string/topLeftContent"
For more details on the how and why of this, see Google's documentation on Localization in Android.

## Reference

[Android文件或文件夹内容改变监听器（FileObserver）](http://blog.csdn.net/mayingcai1987/article/details/6210904)

